# TM753 Codesys-MCP

A Codex skill for operating INVT TM753 PLC projects through Invtmatic Studio,
CODESYS, and Codesys-MCP.

This repository is meant for humans who want to understand, install, or adapt
the skill. The actual agent instructions live in [SKILL.md](SKILL.md).

## What This Skill Does

The `tm753-codesys-mcp` skill teaches Codex how to work with an INVT TM753 PLC
project in a cautious, repeatable way:

- inspect an existing Invtmatic/CODESYS project before changing it
- find or set up Codesys-MCP on a new machine
- read and edit POUs, GVLs, DUTs, device mappings, and project configuration
- compile the project and inspect diagnostics
- connect to a TM753 runtime over local Ethernet
- download, start, stop, and monitor applications when explicitly requested
- troubleshoot common Codesys-MCP, Invtmatic, firmware, and physical I/O issues

It is not a PLC program template. The runlight/chaser program used during the
initial bring-up was only a smoke test and is intentionally not part of the main
workflow.

## Requirements

On the machine where Codex will use this skill, you should already have:

- INVT Invtmatic Studio installed
- a compatible TM753 project or workspace
- local Ethernet access to the PLC
- the correct TM753 firmware, device descriptions, and libraries for the project
- Node.js 18+ and Git if Codesys-MCP needs to be cloned or built

The skill assumes Invtmatic Studio is already installed. It does not install
Invtmatic, upgrade firmware, or reconfigure the PLC IP by itself. It can,
however, help locate an existing Codesys-MCP checkout or clone and build
Codesys-MCP in a machine-appropriate workspace.

## Install

Clone or copy this repository into your Codex skills directory:

```bash
mkdir -p ~/.codex/skills
git clone https://github.com/JasonHe/TM753-Codesys-MCP.git ~/.codex/skills/tm753-codesys-mcp
```

Restart Codex after installing the skill so it can discover the new metadata.

The skill name remains lowercase because Codex skill names must use lowercase
letters, digits, and hyphens:

```text
tm753-codesys-mcp
```

## Example Prompts

Use the skill by asking Codex for TM753/CODESYS work, for example:

```text
Use $tm753-codesys-mcp to inspect my TM753 project and tell me what is currently running.
```

```text
Use $tm753-codesys-mcp to read PLC_PRG, explain the current logic, and suggest a minimal change.
```

```text
Use $tm753-codesys-mcp to compile the project and summarize any diagnostics.
```

```text
Use $tm753-codesys-mcp to download this tested change to the PLC and verify the application is running.
```

## Safety Model

PLC actions can affect real hardware. The skill is written to bias toward
inspection first and mutation later:

- read project state before editing
- back up `.project` files before material changes
- avoid changing PLC IP unless the user explicitly asks
- avoid overwriting user logic unless the user explicitly asks
- compile before download
- treat download, start, and stop as live hardware operations
- verify physical I/O behavior separately from internal variable changes

If the user request is ambiguous, Codex should clarify before downloading,
starting, stopping, or otherwise changing the live PLC.

## TM753 Notes

TM753 onboard outputs Y0-Y7 require the correct onboard I/O mapping. In
CODESYS-style process image code, `%QX0.0` through `%QX0.7` only drive physical
outputs when the project has the correct TM753 device configuration.

In particular, the skill records that the `TM_HSIO` device node is needed for
TM753 onboard output mappings. If variables change online but the PLC output
indicators do not move, inspect the device tree and firmware/library versions
before assuming the Structured Text logic is wrong.

## Repository Layout

```text
.
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    └── troubleshooting.md
```

- `SKILL.md` contains the core Codex instructions.
- `agents/openai.yaml` contains UI metadata for the skill.
- `references/troubleshooting.md` contains deeper troubleshooting notes loaded
  only when needed.

## Notes

This skill was created from a real TM753 bring-up session using local Ethernet,
Invtmatic Studio V3.0.1.4, and Codesys-MCP. Local paths and addresses in
`SKILL.md` are examples to verify on each machine, not universal constants.
Codesys-MCP paths are intentionally discovered or created per machine rather
than hard-coded into the skill.
