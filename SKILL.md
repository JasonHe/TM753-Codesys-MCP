---
name: tm753-codesys-mcp
description: Operate INVT TM753 PLC projects through Invtmatic Studio/CODESYS and Codesys-MCP. Use when Codex needs to inspect, read, edit, compile, download, start/stop, monitor variables, or troubleshoot TM753 PLC programs on a machine where Invtmatic Studio is already installed, especially over local Ethernet with Codesys-MCP.
---

# TM753 Codesys-MCP

Use this skill to work on INVT TM753 PLC projects through Invtmatic Studio and Codesys-MCP. Assume Invtmatic Studio is already installed. Do not install Invtmatic, change the PLC IP, overwrite user logic, or download to the PLC unless the user asks for that action.

## Core Rules

- Treat PLC download/start/stop as live hardware actions. Confirm the user intent before doing them if the request is ambiguous.
- Prefer reading the existing project first: project tree, active application, target POU/GVL/DUT, libraries, device tree, and online state.
- Back up the `.project` before editing code, restoring devices, or downloading a materially changed application.
- Keep changes narrowly scoped to the user-requested POU, GVL, device mapping, or configuration.
- For physical I/O tasks, verify the hardware mapping, not just internal variables. TM753 onboard outputs require the `TM_HSIO` device node for `%QX` mappings to drive Y0-Y7.
- Never change the PLC IP unless explicitly asked. It is acceptable to configure the local Ethernet adapter when the user asks.

## Known Local Defaults

Use these only as starting points; verify them on the target machine:

```text
Invtmatic executable: C:\INVT\Invtmatic Studio V3.0.1.4\CODESYS\Common\Invtmatic Studio.exe
Invtmatic profile:    Invtmatic Studio V3.0
Project workspace:    C:\Tools\INVT\projects
Common PLC IP:        192.168.1.10
Common local IP:      192.168.1.100/24
Common credentials:   Administrator / Administrator
Codesys-MCP repo:     C:\Users\T-Power\Documents\对话\_ext\Codesys-MCP
```

## Workflow

1. Confirm network and process state.

   Check the local Ethernet IP and `ping` the PLC. Check whether Invtmatic is already running. Avoid multiple Invtmatic instances opening the same project, because CODESYS locks the `.project`.

2. Start Codesys-MCP with Invtmatic.

   Use the built `dist/bin.js` from Codesys-MCP and pass `--codesys-path`, `--codesys-profile`, `--workspace`, `--mode persistent`, `--keep-alive`, and a long timeout. If `Select Profile` appears, click `Continue` for `Invtmatic Studio V3.0`.

3. Open and inspect the project.

   Use `open_project` with `filePath` when needed. Then read `codesys://project/status`, inspect the `TM753` device node, and inspect the target application/POU. Use read-only resources or tools before edits.

4. Read code before writing.

   Read the target `PLC_PRG` or user-specified POU/GVL/DUT. Understand current declarations, implementation, and library dependencies. Do not replace a whole program unless the user asked for a rewrite.

5. Back up, edit, and save.

   Copy the project to a timestamped backup before edits. Use `set_pou_code` or the narrowest available MCP tool to update code. Save the project after a successful edit.

6. Compile and inspect diagnostics.

   Run `compile_project`. If the MCP response is too terse or mojibake appears, read `%TEMP%\codesys-mcp-compile-debug.txt`. Do not download when there are compile errors.

7. Connect online.

   Set credentials if the runtime requires them. Connect with the configured device address or let the project resolve the scanned `TM753-C` node. Confirm:

   ```text
   Logged In: True
   ```

8. Download and run only when requested.

   Prefer `online_change` when appropriate and safe; use `full` when the project/device mapping changed, the runtime rejects online change, or the user requests a clean download. Then start the application and confirm:

   ```text
   Application: Application
   State: run
   Logged In: True
   ```

9. Verify behavior.

   Read relevant live variables and, for physical behavior, ask the user to confirm the machine or PLC indicators. Internal variables changing is not enough proof for physical I/O.

## TM753 I/O Notes

- TM753 onboard digital outputs are Y0-Y7. In CODESYS process image code they can be represented as `%QX0.0` through `%QX0.7` only when the project has the correct TM753 onboard I/O device mapping.
- The `TM_HSIO` node represents local high-speed/onboard I/O. Its device description exposes `DO0` parameter ID `38100`, with `Q0..Q7`.
- If `TM_HSIO` is missing, code using `%QX0.x` may compile with warnings or internal variables may change while the physical output indicators do nothing.
- If device nodes such as `TM_HSIO` or `ExtCard` cause unresolved symbols, suspect firmware/library/version mismatch before deleting the nodes. Upgrading PLC firmware or matching the Invtmatic package may be required.

## Common Failures

Read [references/troubleshooting.md](references/troubleshooting.md) when any of these occur:

- `Select Profile` blocks automation
- project is locked by another Invtmatic instance
- MCP request times out
- Chinese errors render as mojibake
- login fails after firmware or project changes
- `%QX` variables change but TM753 output indicators do not
- `TON` or other IEC/library symbols are unknown
- `TM_HSIO`/`ExtCard` unresolved references appear

## Optional Smoke Test

Use a small test program, such as a scan-counting output chaser, only when the user asks for a first hardware proof or smoke test. Keep it out of the normal read/edit/download workflow.
