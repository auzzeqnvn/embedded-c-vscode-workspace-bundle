---
name: embedded-c-firmware-engineering
description: Enforce a production-grade Embedded C firmware engineering standard for STM32, GD32, bare-metal, HAL/SPL/CMSIS, and RTOS code. Use when writing, reviewing, refactoring, debugging, or designing .c/.h firmware, drivers, BSPs, protocol parsers, ISRs, DMA/shared-state code, state machines, watchdog/recovery logic, or when interpreting compiler/static-analysis findings. Apply Clean Code, MISRA C-oriented safety, naming conventions, module architecture, interrupt/volatile/concurrency rules, error handling, timeout/retry discipline, and static-analysis workflow. Do not claim formal MISRA compliance without licensed rule-set/tool evidence and documented deviations.
---

# Embedded C Firmware Engineering Standard

Use this skill as a production firmware engineering standard, not only as a style guide.

## Core standard

Evaluate firmware through these seven layers unless the user explicitly narrows the scope:

1. Clean Code and maintainability
2. MISRA C-oriented language safety
3. Naming convention and units
4. Module architecture and dependency direction
5. Interrupt, DMA, `volatile`, concurrency, and shared-state safety
6. Error handling, timeout, retry, recovery, and watchdog behavior
7. Compiler diagnostics and static analysis

Load references only as needed:

- Clean Code, naming, files, and module architecture: [references/coding-standard.md](references/coding-standard.md)
- MISRA-oriented C safety and compliance boundaries: [references/misra-c.md](references/misra-c.md)
- ISR, DMA, `volatile`, atomicity, and concurrency: [references/interrupt-volatile.md](references/interrupt-volatile.md)
- Error handling, state machines, timeout, retry, and watchdog: [references/error-handling.md](references/error-handling.md)
- Static-analysis workflow: [references/static-analysis.md](references/static-analysis.md)
- Review checklist and severity model: [references/review-checklist.md](references/review-checklist.md)
- BAD/GOOD patterns: [references/examples.md](references/examples.md)

## Operating modes

Infer the mode from the task.

### Write mode

When generating new firmware:

- inspect neighboring project code before inventing conventions
- preserve the project's existing HAL/SPL/CMSIS and RTOS choices
- keep APIs narrow and module ownership explicit
- write deterministic and bounded code
- include timeout, range, buffer, and error behavior where relevant
- prefer static allocation unless dynamic allocation is explicitly justified
- keep application policy out of drivers and BSP code
- keep direct register access out of application code
- make units explicit in names
- keep ISR work short and bounded
- use explicit types and conversions when C promotions could be ambiguous

### Review mode

Prioritize findings in this order:

1. memory corruption, UB, invalid lifetime, out-of-bounds, overflow with runtime impact
2. ISR/concurrency/shared-state defects
3. type conversion, arithmetic, pointer, shift, and side-effect hazards
4. missing timeout, uncontrolled retry, ignored errors, broken recovery
5. state-machine and watchdog defects
6. module ownership and dependency violations
7. readability, naming, duplication, comments, and style

For meaningful findings, use:

```text
[Severity] Category / rule family
Location
Problem
Risk
Recommended fix
Verification
```

Severity:

- **Critical**: credible memory corruption, UB, invalid lifetime/pointer, race/state corruption, dangerous ISR design, critical deadlock/hang, or safety/integrity failure.
- **Major**: type/conversion hazard, bad synchronization/volatile use, missing timeout, ignored error, uncontrolled retry, defective state machine, broken ownership/dependency, or credible analyzer defect.
- **Minor**: naming, magic values, excessive size/nesting, duplication, weak comments, style inconsistency, or unnecessary symbol exposure.

Do not flood the user with low-value style findings before correctness findings.

### Refactor mode

Refactor incrementally:

1. fix correctness, UB, bounds, lifetime, and concurrency
2. fix types, conversions, arithmetic, shifts, and pointer contracts
3. add timeouts, retry bounds, and error propagation
4. clarify state ownership and shared data
5. repair ISR and state-machine structure
6. repair layering and dependency direction
7. improve names, units, function responsibility, duplication, and comments
8. build and re-run available analysis/tests

Preserve observable behavior unless a behavior change is requested or needed to fix a defect.

## Architecture default

Prefer one-way dependency flow:

```text
Application
    -> Service
        -> Protocol / Device Driver
            -> BSP / HAL / SPL / CMSIS
                -> MCU / registers
```

Rules:

- lower layers must not depend on application policy
- drivers must not implement business rules such as overspeed or trip policy
- application code should not manipulate registers directly unless explicitly justified
- substantial protocol encoding/decoding should be separate from transport/peripheral access
- avoid circular dependencies

## Naming default

Unless the project already defines a consistent alternative:

- variables/functions: `snake_case`
- public APIs: module prefix, e.g. `gps_get_position()`
- typedefs: `_t`
- macros: `UPPER_CASE`
- booleans: names expressing truth, e.g. `is_`, `has_`, `can_`, `should_`
- time/physical values: encode units, e.g. `_ms`, `_us`, `_mv`, `_ma`, `_hz`, `_bytes`, `_kmh`
- private functions/objects: `static`

Avoid vague names such as `data`, `tmp`, `flag`, `status`, or `value` unless the local scope makes the meaning obvious.

## Module and header default

For each module:

1. expose only the required public API in `.h`
2. keep private implementation/state in `.c` with `static`
3. do not expose writable globals by default
4. do not include `.c` files
5. avoid circular dependencies
6. keep one primary responsibility per module
7. keep business policy out of BSP/driver layers
8. keep direct register access out of application code

Recommended `.c` order:

1. own header
2. standard/library headers
3. project dependencies
4. private macros/constants
5. private types
6. private state
7. private function declarations
8. public functions
9. private functions

## Interrupt, DMA, and `volatile`

In an ISR, normally only:

- acknowledge/read the hardware condition
- capture minimal data
- update a bounded buffer/counter or set an event
- exit

Do not normally perform blocking waits, long parsing, `printf`, flash erase/write, networking, or application policy in ISR context.

For ISR/main, DMA/CPU, task/task, or task/ISR shared data, identify:

- owner
- writers
- readers
- atomicity requirement
- synchronization/critical-section mechanism
- overflow/overrun behavior
- lifetime

Treat `volatile` only as an observability/compiler-access qualifier. It does not provide atomicity, locking, ordering between tasks, or race freedom.

## State machines and time

Use enums for persistent states and explicit events for transient occurrences.

Waiting states should normally define:

- entry action
- success condition
- timeout
- retry/error transition
- recovery or safe state

Use wrap-safe unsigned elapsed-time checks:

```c
if ((uint32_t)(now_ms - start_ms) >= timeout_ms)
{
    /* timeout */
}
```

Avoid long blocking delays in cooperative/main-loop state machines.

## Error handling

Every fallible API must make failure observable through `bool`, a result enum, status enum, or typed result structure.

Every meaningful error must be deliberately:

- handled locally
- propagated
- recorded for diagnostics
- converted to a recovery state
- or escalated to a safe reset/fail-safe path

Retry must be bounded or explicitly justified as persistent. Add delay/backoff when rapid retries can overload a peripheral, bus, modem, network, flash device, server, or power budget.

Watchdog refresh must reflect system health; do not refresh unconditionally from a timer ISR or loop that can continue while critical work is deadlocked.

## MISRA boundary

Use MISRA C as a safety-oriented review lens, with modern MISRA C:2023 concepts when no project edition is specified. Respect a project's declared edition when supplied.

Do not reproduce copyrighted rule text. Paraphrase intent and identify rule families/IDs only when reliable.

Never claim solely from AI inspection:

- "MISRA compliant"
- "100% MISRA compliant"
- "certified MISRA"
- "no MISRA violations"

AI review may report:

- `MISRA-oriented finding`
- `potential MISRA issue`
- `requires MISRA-tool verification`

Formal compliance requires the selected MISRA edition, licensed/appropriate analysis, project applicability decisions, deviation records, and compliance evidence.

## Static analysis

When tool execution is available, prefer:

1. build with a strong compiler warning level
2. fix actionable compiler diagnostics
3. run generic static analysis if available
4. run a MISRA-capable analyzer when formal MISRA work is required
5. review findings manually in MCU/toolchain context
6. record justified deviations instead of blanket suppressions
7. re-run analysis after fixes

Never equate a warning-free build with MISRA compliance.

For GCC/Clang-based embedded builds, consider project-appropriate warnings such as:

```text
-Wall
-Wextra
-Wshadow
-Wundef
-Wformat=2
-Wconversion
-Wsign-conversion
-Wswitch-enum
-Wcast-qual
-Wstrict-prototypes
-Wmissing-prototypes
```

Do not blindly add `-Werror` to a legacy codebase. Establish a baseline and prevent new regressions.

## Tool and project discipline

When operating as a coding agent:

- inspect build files and existing conventions before editing
- prefer the smallest safe patch
- do not replace the project's HAL/SPL/RTOS/library stack without explicit request
- do not change public APIs casually
- do not silently suppress compiler or analyzer warnings
- do not modify generated/vendor files unless the project explicitly treats them as editable
- identify assumptions when MCU family, compiler, RTOS, ISR priority, word width, or memory model affects correctness
- run the narrowest relevant build/test/static-analysis command after changes when available
- report what was verified and what remains unverified

## Final self-check

Before finishing a firmware task, check:

- bounds and pointer lifetime
- signedness, promotions, narrowing, and intermediate width
- shift range and arithmetic overflow assumptions
- state/data initialization
- ISR execution time and shared-state ownership
- `volatile` versus atomicity/synchronization
- timeout and retry behavior
- state recovery and invalid-state handling
- watchdog health logic
- module dependency direction
- error propagation
- compiler/static-analysis regressions
