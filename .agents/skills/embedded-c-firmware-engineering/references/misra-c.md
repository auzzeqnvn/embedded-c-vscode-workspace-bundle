# MISRA C-Oriented Guidance

## Scope and legal boundary

Use this file to guide MISRA-oriented analysis without reproducing copyrighted MISRA rule text.

Conceptual baseline: MISRA C:2023. Many deployed automotive/industrial projects still specify MISRA C:2012 plus amendments; follow the user's contractual edition when stated.

AI review is not formal compliance evidence. Formal compliance requires the selected MISRA edition, an applicability/compliance plan, analysis evidence, documented deviations, and appropriate review/tooling.

## High-priority MISRA rule families

### Language/undefined behavior (1.x)

Flag constructs that can invoke undefined, unspecified, or implementation-sensitive behavior. Give highest priority to plausible runtime impact.

Examples:

- invalid shifts
- signed overflow assumptions
- out-of-bounds pointer/array access
- invalid object lifetime
- unsequenced side effects
- invalid pointer conversions/dereferences

### Unused/dead/unreachable code (2.x)

Identify code that cannot execute, values written but never meaningfully used, obsolete interfaces, and disabled logic that masks defects.

### Identifier uniqueness and visibility (5.x)

Prefer unambiguous identifiers across relevant scopes and translation units. Avoid confusing reuse, shadowing, and near-collisions in safety-sensitive code.

### Declarations, definitions, linkage, and prototypes (8.x)

Require complete function prototypes and consistent declarations/definitions. Keep internal objects/functions at internal linkage. Avoid accidental external linkage.

### Initialization (9.x)

Ensure objects are initialized before use. Make aggregate initialization intentional and complete enough to avoid latent state assumptions.

### Essential types and conversions (10.x)

Treat this as a major review area.

Check:

- signed/unsigned mixing
- integer promotions
- narrowing conversions
- enum/integer mixing
- Boolean vs numeric expressions
- floating/integer mixing
- casts that hide range loss
- intermediate-expression width

BAD:
```c
uint16_t speed;
int16_t offset;
uint16_t corrected = speed + offset;
```

GOOD:
```c
int32_t corrected;
corrected = (int32_t)speed + (int32_t)offset;
```

Before casting a wider value to a narrower type, demonstrate or check the range.

### Pointer conversions and qualification (11.x)

Scrutinize pointer casts, integer/pointer conversions, incompatible pointed-to types, alignment, `const` removal, and hardware-address access.

Prefer vendor CMSIS/device headers for peripheral registers instead of ad hoc integer-to-pointer casts.

### Expressions and evaluation (12.x/13.x)

Keep evaluation order obvious. Avoid multiple side effects in one expression.

BAD:
```c
buffer[index++] = read_byte(index++);
```

GOOD:
```c
uint8_t value = read_byte(index);
buffer[index] = value;
index++;
```

Be especially cautious with volatile accesses, `++/--`, function calls inside complex Boolean expressions, and assignment embedded in conditions.

### Control flow (14.x/15.x/16.x)

Use structured, explicit control flow.

- make loops bounded or externally justified
- make `if` conditions Boolean in intent
- use braces consistently in generated production code
- make `switch` cases explicit
- avoid accidental fall-through
- define behavior for unexpected states

### Functions (17.x)

Use complete prototypes, check return values when meaningful, validate recursion policy, and keep call contracts explicit.

For constrained embedded systems, avoid recursion unless worst-case stack depth is proven and accepted.

### Arrays, pointers, object lifetime (18.x)

Check bounds, pointer arithmetic, lifetime, stack usage, VLAs, and buffer length contracts. Treat externally controlled lengths as untrusted.

### Preprocessor (20.x)

Minimize function-like macros and complex conditional compilation. Parenthesize macro parameters/expressions appropriately when macros are necessary. Prefer typed functions/`static inline` helpers when practical.

### Standard library/resources (21.x/22.x)

Review standard-library use for determinism, dynamic allocation, I/O, environment dependencies, errno/resource state, and portability.

Dynamic allocation in long-running firmware requires an explicit fragmentation/failure/lifetime strategy; otherwise prefer static allocation or fixed pools.

## Embedded-specific interpretation

MISRA-oriented review must account for legitimate embedded idioms:

- memory-mapped I/O
- volatile peripheral registers
- vendor CMSIS/HAL/SPL headers
- ISR callbacks
- DMA buffers
- linker sections
- packed protocol structures
- compiler extensions

Do not label a necessary vendor/compiler extension as wrong solely because it is non-portable. Instead:

1. identify the deviation or portability concern
2. explain why the platform needs it
3. isolate it in BSP/portability code
4. require toolchain-specific verification and documentation

## Deviation handling

When a rule must be deviated from, require at least:

- rule/family or project rule identifier
- exact location/scope
- technical reason
- risk analysis
- compensating control
- reviewer/approval mechanism in the user's process
- evidence that the deviation is bounded and intentional

Never hide a deviation using blanket warning suppression.
