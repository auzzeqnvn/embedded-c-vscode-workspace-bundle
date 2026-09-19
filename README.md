# Embedded C VS Code Workspace Skill

Reusable Embedded C firmware engineering skill for **Codex**, **Google Antigravity**, and **OpenCode** in VS Code.

It applies a shared engineering standard covering:

- Clean Code and maintainability
- MISRA C-oriented safety review
- naming conventions and units
- .c/.h structure and module architecture
- ISR, DMA, volatile, atomicity, and concurrency
- state machines, timeout, retry, recovery, and watchdogs
- error handling
- compiler diagnostics and static analysis

## Install

Copy the included `.agents/` directory into the root of your firmware repository so this file exists:

```text
<project-root>/.agents/skills/embedded-c-firmware-engineering/SKILL.md
```

See [INSTALL.md](INSTALL.md) for usage with Codex, Antigravity, and OpenCode.

## Scope

Designed for STM32, GD32, bare-metal, HAL/SPL/CMSIS, and RTOS firmware.

> The skill performs MISRA C-oriented review. It must not be treated as proof of formal MISRA compliance without the licensed rule set, configured analysis tool, documented deviations, and project compliance process.
