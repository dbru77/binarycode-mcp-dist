# BinaryCode MCP

BinaryCode MCP is a structural AL / Business Central code-search MCP server
for AI coding agents. It works with VS Code + GitHub Copilot and with Claude
Code, exposing a Model Context Protocol server that agents can query directly
from an editor session.

Instead of fuzzy text search over AL source, BinaryCode gives the agent exact
structural answers: who references a table or codeunit, which fields a table
or page has, where a field is read/written/validated, which labels contain a
given text, what events an object publishes or subscribes to, and how tables
relate to each other. The answers come from a real AST parse of your AL code,
not from grepping text.

## Download

Get `bc-mcp-windows-amd64.zip` from [the latest release](../../releases/latest).

Windows x64 only. No Python, Git, Rust, or internet connection is needed
after you download the zip.

## Install

1. Extract the zip somewhere (e.g. your Downloads folder). Keep `install.ps1`
   and `bc-mcp.exe` in the same folder — the installer looks for the exe next
   to itself and fails if they are separated.
2. Open PowerShell in the extracted folder.
3. Run:

   ```powershell
   powershell -ExecutionPolicy Bypass -File install.ps1
   ```

   Add `-RegisterGlobal` to also wire up VS Code and Claude Code for all
   workspaces in one step:

   ```powershell
   powershell -ExecutionPolicy Bypass -File install.ps1 -RegisterGlobal
   ```

4. Close and reopen your terminal **and** VS Code so the PATH change takes
   effect.
5. Verify with:

   ```powershell
   bc-mcp selftest
   ```

   It should print `selftest: OK`.

What the installer actually does: it copies `bc-mcp.exe` to
`%USERPROFILE%\.binarycode\bin`, clears the Windows Mark-of-the-Web
(`Unblock-File`) so SmartScreen does not block the binary, adds that folder
to your user PATH, and runs `bc-mcp selftest`. It only registers the MCP
server with an editor if you pass `-RegisterGlobal` — otherwise nothing is
registered automatically, and you set up the server per workspace (next
section).

Two more switches: `-Workspace <path>` also scaffolds that AL workspace in
the same run, and `-NoPathSetup` skips the PATH change. See
[INSTALL.md](INSTALL.md) for the full reference.

## Use it in a Business Central workspace

If you did not use `-RegisterGlobal`, register the server per workspace:

1. Open your BC/AL workspace folder in VS Code.
2. In a terminal at the workspace root, run:

   ```powershell
   bc-mcp init-workspace .
   ```

   This scaffolds `.vscode/mcp.json`, `.vscode/tasks.json`, and the index
   config for the workspace.
3. Build the structural index:

   ```powershell
   bc-mcp index_al . --index .binarycode-index/structural.db
   ```

   or run the VS Code task that `init-workspace` scaffolded for you.
4. Reload VS Code (Command Palette -> "Developer: Reload Window"). The
   `bc-mcp` MCP server is now available to your AI agent.

## What it does

BinaryCode exposes these MCP tools:

| Tool | Description |
| --- | --- |
| `ast_find_refs` | Find all AL objects that reference a target (table, codeunit, procedure, etc.) |
| `ast_get_object` | Get the full structure of an AL object including fields, procedures, labels, and references |
| `ast_get_procedures` | List all procedures from an AL object |
| `ast_get_fields` | List or search a table or page object's fields (field number, name, data type, table relation) |
| `ast_find_events` | Find AL event publishers (IntegrationEvent, BusinessEvent, InternalEvent) |
| `ast_find_subscribers` | Find AL event subscribers |
| `ast_find_table_rels` | Find fields with TableRelation to a specific table |
| `ast_find_field_refs` | Find where AL table fields are read, written, validated, or bound |
| `ast_find_labels` | Find labels, captions, and tooltips containing specific text |
| `index_al` | Index a Business Central project with structural parsing for exact lookups |
| `ast_stats` | View statistics about the structural index |
| `clear_index` | Remove all indexed data |

## Troubleshooting

**Windows SmartScreen blocks `bc-mcp.exe`.** The binary is not code-signed.
Choose "More info" -> "Run anyway", or run `Unblock-File .\bc-mcp.exe` first.

**Windows SmartScreen / Mark-of-the-Web blocks `install.ps1`.** Run
`Unblock-File .\install.ps1` before executing it, or choose "More info" ->
"Run anyway" when prompted.

**"Running scripts is disabled on this system."** This is why the install
command includes `-ExecutionPolicy Bypass` — it only affects that one
invocation of `install.ps1` and does not change your machine's execution
policy permanently.

**`bc-mcp` is not recognized after install.** PATH is updated at the user
scope, but an already-open terminal does not pick it up. Open a **new**
terminal window.

**Installer fails with "could not write bc-mcp.exe".** Something else has
the file open — usually VS Code with the MCP server running, or a `bc-mcp
serve` process. Close VS Code and any running `bc-mcp serve`, then run
`install.ps1` again.

**The agent still doesn't see the server.** Fully restart VS Code (or Claude
Code) after install — a window reload is not always enough. Then check
that either `.vscode/mcp.json` exists in your workspace (created by
`bc-mcp init-workspace`) or that you installed with `-RegisterGlobal`.

---

BinaryCode MCP is proprietary software. Copyright (c) 2026 Jan-Hendrik
Klaffke. Source code is not distributed. See [LICENSE](LICENSE) and
[THIRD-PARTY-NOTICES.txt](THIRD-PARTY-NOTICES.txt) for third-party
components bundled inside the binary.
