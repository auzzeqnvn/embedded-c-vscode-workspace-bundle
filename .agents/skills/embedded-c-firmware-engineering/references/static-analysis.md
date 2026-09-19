# Static Analysis Workflow for Embedded C

## Goal

Use static analysis as evidence and defect discovery, not as a substitute for engineering review.

A clean compile does not prove MISRA compliance, race freedom, memory safety, or correct hardware behavior.

## Stage 1: Compiler diagnostics

Enable the highest practical warning level for the selected compiler and fix real defects before adding suppressions.

For GCC/Clang-based builds, consider a project-appropriate subset such as:

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

Do not blindly enable every warning as `-Werror` on a legacy codebase. Establish a baseline, fix defects, then prevent regressions.

For Keil/Arm Compiler, IAR, or other vendor compilers, use the strongest practical diagnostic level and document disabled diagnostics.

## Stage 2: Generic static analysis

Use available analyzers for defect classes such as:

- null dereference
- out-of-bounds access
- uninitialized data
- dead code
- resource leaks
- suspicious casts/conversions
- unreachable branches
- invalid shifts
- concurrency issues where supported

Open-source tools such as Cppcheck or Clang-based analysis can supplement compiler diagnostics, but do not treat generic/open-source checking alone as proof of formal MISRA compliance.

## Stage 3: MISRA-capable analysis

When contractual/formal MISRA compliance is required, use a tool/license/process appropriate to the selected MISRA edition and project compliance plan.

The AI should distinguish:

```text
AI finding                 -> engineering suspicion/recommendation
Compiler warning           -> compiler diagnostic evidence
Static analyzer finding    -> analyzer evidence
MISRA analyzer finding     -> rule-oriented tool evidence
Approved deviation         -> project compliance record
```

Do not merge these categories into one claim.

## Stage 4: Triage

For each finding record:

- tool
- rule/check ID if available
- file/line
- severity
- defect rationale
- false positive? why?
- fix or deviation
- reviewer

Suppress at the narrowest possible scope.

## Stage 5: Deviation

If a rule/check is intentionally violated, record:

- exact scope
- reason
- risk
- compensating control
- verification evidence
- approval

Avoid global suppressions such as "disable all conversion warnings".

## Stage 6: CI gate

A mature project can gate on:

- zero new compiler warnings
- zero new Critical/Major static-analysis findings
- no unresolved required MISRA findings
- deviations reviewed and tracked
- analyzer configuration version-controlled

## AI review behavior

If the user supplies analyzer output:

1. explain the diagnostic in Embedded C terms
2. identify likely root cause
3. distinguish true defect from possible false positive
4. propose the smallest safe fix
5. mention any MCU/toolchain assumptions
6. never advise blanket suppression merely to make reports green
