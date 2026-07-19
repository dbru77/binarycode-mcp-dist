# BinaryCode MCP

A structural AL / Business Central code-search MCP server for AI coding agents —
works with **VS Code + GitHub Copilot** and **Claude Code**.

It gives your AI agent exact, structural answers about your Business Central
codebase — references, fields, labels, events, table relations — instead of
relying on fuzzy text search over your AL source.

## Download

**Download the latest `binarycode-mcp-windows.zip` from the [Releases page](../../releases/latest) →**

No Python, no Git, nothing else to install first — the zip contains
everything you need.

## Install (Windows)

1. **Unzip** `binarycode-mcp-windows.zip` anywhere (e.g. your Downloads or Desktop).
2. **Run the installer:**
   ```powershell
   powershell -ExecutionPolicy Bypass -File install-binarycode-mcp.ps1
   ```
   (or right-click `install-binarycode-mcp.ps1` → **Run with PowerShell**)
3. **Restart VS Code** (and open a new terminal) so the changes take effect.

That's it. The installer copies `binarycode-mcp.exe` to
`%USERPROFILE%\.binarycode\bin`, adds that folder to your PATH, and registers
the MCP server with VS Code/GitHub Copilot and Claude Code automatically.

### Wiring up an AL project (optional)

To also connect a specific Business Central AL project so it can be indexed
and queried, pass `-Workspace` with the path to that project:

```powershell
powershell -ExecutionPolicy Bypass -File install-binarycode-mcp.ps1 -Workspace "C:\path\to\your\AL\project"
```

You can run the installer again at any time with `-Workspace` to add another
project.

Full step-by-step instructions are in [INSTALL.md](INSTALL.md).

## What it does

BinaryCode MCP parses your AL/C-AL source with tree-sitter and builds a
structural SQLite index, then exposes exact-lookup tools to your AI agent —
no guessing from raw text search. It currently exposes 12 MCP tools,
including:

- `ast_find_refs` — find all AL objects that reference a target (table, codeunit, procedure, etc.)
- `ast_get_object` — get the full structure of an AL object (fields, procedures, labels, references)
- `ast_get_procedures` — list all procedures on an AL object
- `ast_get_fields` — list all fields on an AL object
- `ast_find_events` — find AL event publishers (IntegrationEvent, BusinessEvent, InternalEvent)
- `ast_find_subscribers` — find AL event subscribers
- `ast_find_table_rels` — find fields with a TableRelation to a specific table
- `ast_find_field_refs` — find where AL table fields are read, written, validated, or bound
- `ast_find_labels` — find labels, captions, and tooltips containing specific text
- `index_al` / `ast_stats` / `clear_index` — manage the structural index

## Troubleshooting

- **Windows SmartScreen warning** — click **More info → Run anyway**. The
  executable isn't code-signed, which triggers this standard warning for
  unsigned binaries downloaded from the internet.
- **"Running scripts is disabled on this system"** — this is why the install
  command above includes `-ExecutionPolicy Bypass`; it only affects this one
  run, not your system-wide PowerShell policy.
- **VS Code / Claude Code still doesn't see the server** — make sure you
  fully restarted the application (not just reloaded a window), and that you
  opened a brand-new terminal window after installing.

---

This software is proprietary — © 2026 Jan-Hendrik Klaffke. All rights
reserved. Source code is not distributed. Third-party components bundled in
the binary are listed, with their licenses, in
[THIRD-PARTY-NOTICES.txt](THIRD-PARTY-NOTICES.txt).
