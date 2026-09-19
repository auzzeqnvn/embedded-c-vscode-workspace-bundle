# Install: Embedded C Firmware Engineering skill

This workspace bundle is designed to work with **Codex**, **Google Antigravity**, and **OpenCode** from the same project.

## Quick install

Extract this bundle into the **root of your VS Code project / Git repository** so the resulting path is:

```text
<project-root>/.agents/skills/embedded-c-firmware-engineering/SKILL.md
```

Do not place the skill inside `src/`, `Core/`, or another source subdirectory unless you intentionally want narrower scope.

## Codex

Codex discovers repo-local skills from `.agents/skills`.

Typical prompts:

```text
$embedded-c-firmware-engineering Review Core/Src/app.c and list Critical/Major findings first.
```

```text
$embedded-c-firmware-engineering Refactor this UART DMA module without changing its public API.
```

If a newly copied skill does not appear, restart the Codex session/extension.

## Antigravity

Antigravity discovers workspace skills from `.agents/skills`.

Use the Skills UI or ask the agent explicitly:

```text
Use the embedded-c-firmware-engineering skill to review this STM32 module.
```

For persistent project-wide constraints in addition to the skill, Antigravity also supports `.agents/rules/`, but this package intentionally keeps the engineering standard in one reusable skill so it stays consistent across all three agents.

## OpenCode

OpenCode also discovers `.agents/skills/<name>/SKILL.md`.

Typical prompt:

```text
Use the embedded-c-firmware-engineering skill and review the current changes for ISR, volatile, MISRA-oriented type safety, and error handling.
```

## Recommended workflow

For review:

```text
Use embedded-c-firmware-engineering.
Review only; do not edit yet.
Prioritize Critical and Major issues.
For each issue show file/line, risk, minimal fix, and verification method.
```

For refactor:

```text
Use embedded-c-firmware-engineering.
Refactor incrementally, preserve public APIs and runtime behavior, then build and report remaining warnings.
```

For new module:

```text
Use embedded-c-firmware-engineering.
Implement <module> for <MCU/framework>. Follow the existing project architecture and naming conventions. Avoid blocking design and expose only a narrow public API.
```

## Global installation (optional)

If you want the skill available across many repositories, each tool supports global skill locations, but paths differ by tool. Project-local `.agents/skills` is the simplest shared location and is recommended when the standard should travel with the repository.
