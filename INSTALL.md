# Installing BinaryCode MCP

This is the long-form install guide. For a quick start, see
[README.md](README.md).

## Requirements

- Windows 10 or 11, x64.
- PowerShell 5.1 or later (built into Windows 10/11; PowerShell 7+ also
  works).
- Nothing else. No Python, no Git, no Rust, no internet access is required
  once you have downloaded the zip.

## 1. Download

Download `bc-mcp-windows-amd64.zip` from
[the latest release](../../releases/latest). It contains:

- `bc-mcp.exe` — the standalone binary (server + CLI).
- `install.ps1` — the installer script.
- `README.txt` — a plain-text copy of the quick-start instructions.
- `THIRD-PARTY-NOTICES.txt` — third-party license attributions.

## 2. Extract

Extract the zip to any folder, for example `Downloads\bc-mcp`. **Keep
`install.ps1` and `bc-mcp.exe` in the same folder.** The installer looks for
`bc-mcp.exe` next to itself and throws an error if it isn't there.

## 3. Run the installer

You have two options:

**Option A — PowerShell terminal.** Open PowerShell in the extracted folder
and run:

```powershell
powershell -ExecutionPolicy Bypass -File install.ps1
```

**Option B — right-click.** In File Explorer, right-click `install.ps1` and
choose "Run with PowerShell". If Windows blocks it with a SmartScreen
warning, choose "More info" -> "Run anyway", or first run
`Unblock-File .\install.ps1` in a PowerShell terminal and try again.

Add switches after `install.ps1` as needed — see the reference table below.

## 4. Restart your terminal and VS Code

The installer adds its install folder to your user PATH. Existing terminal
windows and running editors do not pick up PATH changes automatically —
close and reopen your terminal, and fully restart VS Code (and Claude Code,
if you use it), before continuing.

## 5. Verify

```powershell
bc-mcp selftest
```

A working install prints `selftest: OK`.

## Switch reference

All switches are optional and can be combined.

| Switch | Effect |
| --- | --- |
| `-NoPathSetup` | Skip adding the install folder to your user PATH. Use this if you manage PATH yourself, or are scripting the install into an environment with its own PATH handling. |
| `-Yes` | Run non-interactively, without pausing for confirmation. Safe to use in automated or unattended installs. |
| `-RegisterGlobal` | After installing, run `bc-mcp register --global --clients vscode,claude` so the `bc-mcp` MCP server is available to VS Code and Claude Code in every workspace, without per-workspace setup. |
| `-Workspace <path>` | After installing, run `bc-mcp init-workspace <path>`, scaffolding `.vscode/mcp.json`, `.vscode/tasks.json`, and the index config in that folder so the workspace is ready to index immediately. |

Example: install, register globally, and prep a workspace in one command:

```powershell
powershell -ExecutionPolicy Bypass -File install.ps1 -RegisterGlobal -Workspace C:\src\my-bc-project
```

## What the installer does, in order

1. Locates `bc-mcp.exe` next to `install.ps1` (fails if it's missing).
2. Creates `%USERPROFILE%\.binarycode\bin` if it doesn't exist.
3. Reads the version of `bc-mcp.exe` already installed there, if any, and
   the version being installed, then reports whether this is a fresh
   install, a reinstall, an upgrade, or a downgrade (by comparing the two
   `bc-mcp version` outputs).
4. Copies the new `bc-mcp.exe` into place, overwriting any existing copy.
5. Clears the Mark-of-the-Web on the copied exe (`Unblock-File`) so
   SmartScreen does not block it on first run.
6. Unless `-NoPathSetup` was passed, adds
   `%USERPROFILE%\.binarycode\bin` to your user PATH (skipped if it's
   already there).
7. Runs `bc-mcp selftest`.
8. If `-RegisterGlobal` was passed, runs
   `bc-mcp register --global --clients vscode,claude --force`.
9. If `-Workspace <path>` was passed, runs `bc-mcp init-workspace <path>`.

## Upgrade / reinstall

Running `install.ps1` again with a newer or older `bc-mcp.exe` next to it
upgrades or downgrades your existing install in place — there is no
separate uninstall-then-install step. The script detects and reports which
case applies (fresh / reinstall / upgrade / downgrade) before it copies the
file. Existing index databases and workspace configuration are untouched;
only the binary in `%USERPROFILE%\.binarycode\bin` is replaced.

## Workspace setup (if you didn't use `-RegisterGlobal` or `-Workspace`)

Run this once per Business Central / AL workspace:

```powershell
bc-mcp init-workspace .
bc-mcp index_al . --index .binarycode-index/structural.db
```

`init-workspace` scaffolds `.vscode/mcp.json` (the MCP server
registration for that workspace), `.vscode/tasks.json` (a VS Code task
you can use instead of the `index_al` command above), and the index
config. After indexing, reload VS Code (Command Palette -> "Developer:
Reload Window") so it picks up `.vscode/mcp.json`.

## Global registration (alternative to per-workspace setup)

Instead of registering `bc-mcp` per workspace, register it once for every
workspace on the machine:

```powershell
bc-mcp register --global --clients vscode,claude
bc-mcp index_al <your-al-folder> --index %USERPROFILE%\.binarycode\index\structural.db
```

`install.ps1 -RegisterGlobal` runs the `register` step for you; you still
need to index each project you want to search.

MCP registration is never automatic — it only happens when you pass
`-RegisterGlobal` to the installer, or run `bc-mcp register` yourself.

## Uninstall

There is no uninstaller script. To remove BinaryCode MCP:

1. Delete `%USERPROFILE%\.binarycode\bin\bc-mcp.exe` (and the `bin` folder
   if empty).
2. Remove `%USERPROFILE%\.binarycode\bin` from your user PATH
   (Settings -> "Edit environment variables for your account", or
   `[Environment]::SetEnvironmentVariable('Path', ..., 'User')` in
   PowerShell).
3. Delete the `.binarycode-index` folder in any workspace where you built
   an index (it holds only the local SQLite index, safe to delete).
4. Remove the `bc-mcp` entry from any workspace's `.vscode/mcp.json`, if
   present.
5. If you used `-RegisterGlobal` or ran `bc-mcp register --global`, remove
   the `bc-mcp` entry from your global VS Code and Claude Code MCP
   configuration files.

## Troubleshooting

**Windows SmartScreen blocks `bc-mcp.exe`.** The binary is not code-signed.
Choose "More info" -> "Run anyway", or run `Unblock-File .\bc-mcp.exe`
first.

**Windows SmartScreen / Mark-of-the-Web blocks `install.ps1`.** Run
`Unblock-File .\install.ps1` before executing it, or choose "More info" ->
"Run anyway" when prompted.

**"Running scripts is disabled on this system."** That's your PowerShell
execution policy. `-ExecutionPolicy Bypass` in the run command sidesteps
it for that single invocation only; it does not change your machine's
policy permanently, and you do not need admin rights to use it.

**`bc-mcp` is not recognized after install.** PATH is set at the user
scope by the installer, but any terminal that was already open keeps its
old PATH. Open a brand new terminal window.

**Installer throws "bc-mcp.exe was not found next to install.ps1".** You
extracted or moved the files apart. Re-extract the zip so `install.ps1`
and `bc-mcp.exe` sit in the same folder, then run it again.

**Installer fails with "Could not write ... bc-mcp.exe".** Something has
the existing exe open — usually VS Code with the MCP server running, or a
`bc-mcp serve` process started from a terminal. Close VS Code and stop any
running `bc-mcp serve`, then rerun `install.ps1`.

**`selftest` fails.** Rerun `bc-mcp selftest` after a fresh terminal
restart in case PATH just hadn't propagated. If it still fails, re-run
`install.ps1` to reinstall the binary; if the exe was blocked by
SmartScreen, run `Unblock-File %USERPROFILE%\.binarycode\bin\bc-mcp.exe`.

**The agent still doesn't see the `bc-mcp` MCP server.** Fully restart VS
Code (or Claude Code) — a window reload is not always sufficient after
registration changes. Then confirm either `.vscode/mcp.json` exists in
your workspace, or that you installed with `-RegisterGlobal` (or ran
`bc-mcp register --global` yourself).

---

BinaryCode MCP is proprietary software. Copyright (c) 2026 Jan-Hendrik
Klaffke. See [LICENSE](LICENSE) and
[THIRD-PARTY-NOTICES.txt](THIRD-PARTY-NOTICES.txt).
