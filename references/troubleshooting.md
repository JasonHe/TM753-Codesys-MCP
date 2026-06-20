# TM753 Codesys-MCP Troubleshooting

## Profile Dialog

Invtmatic may start at a `Select Profile` window. Choose `Invtmatic Studio V3.0` and click `Continue`. If automation is needed, find the `Continue` child button of the `Select Profile` window and send a `BM_CLICK` Win32 message.

## Project Lock

CODESYS locks open `.project` files. If MCP says the selected project is currently used by the same Windows user/machine, close extra Invtmatic instances and clear stale persistent MCP temp sessions:

```powershell
Stop-Process -Name "Invtmatic Studio" -Force -ErrorAction SilentlyContinue
Get-ChildItem "$env:TEMP\codesys-mcp-persistent" -Directory -ErrorAction SilentlyContinue |
  Remove-Item -Recurse -Force -ErrorAction SilentlyContinue
```

Then let a single MCP-backed Invtmatic instance open the project.

## MCP Timeouts

Use long server and tool timeouts for Invtmatic operations. Split long workflows into smaller calls:

1. start/list tools
2. handle profile dialog
3. open/compile
4. connect
5. download
6. start/verify

This makes it clear where the UI or runtime is blocked.

## Compile Diagnostics

`compile_project` can return only a short success line. Read the debug trace for details:

```powershell
Get-Content "$env:TEMP\codesys-mcp-compile-debug.txt" -Tail 120
```

Mojibake in Node/MCP output can still be understandable from the debug trace or from the Invtmatic UI.

## Login Failure

When login fails:

- Verify local Ethernet IP and `ping` to the PLC.
- Confirm the project device address resolves to the scanned TM753 node, often `TM753-C` with node address like `000A`.
- Set credentials for the session, commonly `Administrator` / `Administrator` if unchanged.
- Restart Invtmatic and clear persistent MCP temp sessions if online state is stuck.
- If failure started after firmware/device-library mismatch, update PLC firmware or match the Invtmatic/device package before changing the program.

## Physical Outputs Do Not Move

If online variables change but PLC output indicators do not:

- Inspect the `TM753` device tree and confirm `TM_HSIO` exists.
- Confirm the code targets a mapped output, such as `%QX0.0` through `%QX0.7`, only when `TM_HSIO` is present.
- Do not assume RUN state proves physical I/O. Ask the user to confirm Y0-Y7 indicators or terminal behavior.
- If `TM_HSIO` was removed to avoid compile errors, restore a project backup with `TM_HSIO` and resolve firmware/library issues instead.

## Library and Symbol Issues

`TON` unknown means the project lacks the library/type providing it. For a minimal first test, use scan-count logic instead of timer FBs, or add the correct library explicitly.

Unresolved symbols such as HSIO/pulse/motion functions can indicate that the `TM_HSIO` device node requires INVT HSIO libraries that are missing or version-incompatible. Check firmware and installed Invtmatic/device package versions before deleting the device node.

## Safe Backup Pattern

Before edits or downloads:

```powershell
$ts = Get-Date -Format yyyyMMdd-HHmmss
Copy-Item "C:\Tools\INVT\projects\TM753_MCP_Test.project" `
  "C:\Tools\INVT\projects\TM753_MCP_Test.backup-$ts.project" -Force
```

Keep known-good backups with hardware nodes intact, especially before modifying `TM_HSIO`, `ExtCard`, or task configuration.
