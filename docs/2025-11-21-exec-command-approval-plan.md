# Plan: Add modal approval for `execute_command` in `vscode-as-mcp-server`

## Goal
Introduce the cheapest safety gate: a blocking VS Code modal dialog that prompts the user before any MCP `execute_command` is run.

## Context / Why
- Upstream extension currently executes commands without a built-in approval UI; only roadmap mentions WebView approvals.
- Host/client policies may allow commands silently; we want a guaranteed per-command prompt inside the extension.
- Keep change minimal (no WebView, no settings UI) to ship fast; can evolve later.

## Plan
1) **Locate handler**: Open `src/tools/executeCommand.ts` (or equivalent) to see how args map to terminal execution (background, timeout, cwd, modifySomething flags). Confirm any existing confirmation hooks.  
2) **Add confirm helper**: Implement a small `confirmExec` using `vscode.window.showInformationMessage({ modal: true }, 'Run', 'Cancel')` that shows command, cwd, background/timeout summary (trim long commands).  
3) **Wrap execution**: Before spawning the terminal, call `confirmExec`; on Cancel, throw an error so MCP client gets a clear denial. Keep normal path unchanged otherwise.  
4) **Wire exports**: Ensure the tool registration uses the wrapped handler; keep typings intact.  
5) **Sanity checks**: Build the extension and spot-check runtime output to ensure no lints/types regress. Optionally add a tiny unit/integration test stub if test harness exists.

## Validation
- `pnpm -C tmp/vscode-as-mcp-server/packages/extension compile` (type-check + build).  
- Optional: `pnpm -C tmp/vscode-as-mcp-server/packages/extension test` (vscode-test) if time permits.

## Risks / Mitigations
- **Double prompts from client + extension**: Acceptable for now; document in README section.  
- **Automation breakage**: All commands now require a click; note as deliberate trade-off.  
- **Long commands overflow UI**: Trim to ~120 chars in preview.  
- **Background commands**: Modal still blocks; OK for baseline.

## Deliverables
- Code change in `tmp/vscode-as-mcp-server/packages/extension/src/tools/executeCommand.ts` (or its handler file).  
- Short README note in extension package describing the new modal prompt (optional if time).  
- This plan file as execution log.
