# Installing BinaryCode MCP (Windows)

No Python, no Git, nothing else to install first — this zip contains everything you need.

## Steps

1. **Unzip** `binarycode-mcp-windows.zip` anywhere (e.g. your Downloads or Desktop).
2. **Right-click `install-binarycode-mcp.ps1` → Run with PowerShell.**

   If that option isn't available, open PowerShell in the unzipped folder and run:
   ```powershell
   powershell -ExecutionPolicy Bypass -File install-binarycode-mcp.ps1
   ```
3. **Restart VS Code** (and open a new terminal) so the changes take effect.

That's it. The installer:

- Copies `binarycode-mcp.exe` to `%USERPROFILE%\.binarycode\bin`.
- Adds that folder to your PATH.
- Verifies the AL parser works.
- Registers the MCP server with **VS Code / GitHub Copilot** and **Claude Code** automatically.

## Setting up an AL project (optional)

To also wire up a specific Business Central AL project so it can be indexed and queried, pass
`-Workspace` with the path to that project:

```powershell
powershell -ExecutionPolicy Bypass -File install-binarycode-mcp.ps1 -Workspace "C:\path\to\your\AL\project"
```

You can run the installer again at any time with `-Workspace` to add another project.

## Troubleshooting

- **"Running scripts is disabled on this system"** — this is why the command above includes
  `-ExecutionPolicy Bypass`; it only affects this one run, not your system-wide policy.
- **VS Code / Claude Code still doesn't see the server** — make sure you fully restarted the app
  (not just reloaded a window), and that you opened a brand-new terminal window after installing.
- For advanced usage, environment variables, and the full CLI/MCP tool reference, see
  [USAGE.md](USAGE.md) in the main repository.
