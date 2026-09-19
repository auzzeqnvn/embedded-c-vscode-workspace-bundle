# Coding Standard: Clean Code, Naming, and Module Architecture

## 1. Naming

Use names that communicate domain meaning and units.

Preferred defaults:

- `snake_case`: variables/functions
- `_t`: typedefs
- `UPPER_CASE`: macros
- module prefix on public APIs
- truth-oriented boolean names: `is_`, `has_`, `can_`, `should_`
- physical/time units in identifiers when ambiguous

BAD:
```c
uint16_t v;
uint32_t timeout;
bool flag;
```

GOOD:
```c
uint16_t battery_voltage_mv;
uint32_t modem_timeout_ms;
bool is_gnss_fix_valid;
```

Avoid encoding type in names unless required by an existing project convention.

## 2. Constants and magic values

Name thresholds, sizes, addresses, timeouts, limits, protocol IDs, and state-dependent constants.

Prefer typed constants where practical. Use macros when compile-time substitution or preprocessor use is required.

BAD:
```c
if (battery_mv < 10500U)
```

GOOD:
```c
#define MODEM_MIN_SUPPLY_MV (10500U)

if (battery_mv < MODEM_MIN_SUPPLY_MV)
```

## 3. Functions

A function should have one clear responsibility and a narrow contract.

Prefer:

- early return for invalid/precondition cases
- limited nesting
- explicit input/output contracts
- `const` pointer parameters for read-only data
- pure calculation functions separated from hardware access when possible

Avoid:

- surprising side effects
- long hidden blocking
- excessive parameter lists
- unrelated responsibilities
- modifying global state from utility functions

## 4. Public vs private API

Expose only what callers need.

- mark private functions/objects `static`
- avoid public writable globals
- prefer getters/setters or operation APIs over exposing internal state
- hide private structs when callers do not need layout

## 5. File structure

Header (`.h`):

- include guard or equivalent
- required public types/constants/API only
- minimal includes
- no private mutable storage definitions
- no implementation code except justified `static inline` helpers

Source (`.c`) recommended order:

1. own header
2. standard headers
3. dependency headers
4. private macros/constants
5. private types
6. private state
7. private prototypes
8. public API
9. private implementation

## 6. Layering

Preferred dependency direction:

```text
Application -> Service -> Protocol/Driver -> BSP/HAL -> MCU
```

Examples:

- `app_tracking.c` decides when to upload position.
- `network_service.c` manages connectivity and retry policy.
- `ec200.c` implements modem commands/sockets.
- `bsp_uart.c` operates UART/DMA.

Do not put an overspeed rule inside a UART or modem driver.

## 7. Global state

Minimize global mutable objects. Give every mutable object a clear owner.

Prefer module-private context:

```c
typedef struct
{
    modem_state_t state;
    uint32_t state_enter_ms;
    uint8_t retry_count;
} modem_context_t;

static modem_context_t modem_ctx;
```

Expose operations rather than the object itself.

## 8. Comments

Comment WHY, constraints, hardware errata, protocol quirks, safety rationale, or non-obvious timing assumptions.

Do not comment syntax that code already states.

BAD:
```c
/* Increment retry count. */
retry_count++;
```

GOOD:
```c
/* Limit rapid reconnects because repeated TX bursts can brown out this board revision. */
retry_count++;
```

## 9. Dead code and history

Delete commented-out code and obsolete branches. Use version control for history.

## 10. Testability

Separate acquisition from calculation and policy where practical.

Prefer:
```c
uint8_t fuel_calculate_percent(uint16_t adc_value);
```

over a function that internally reads ADC, updates globals, stores flash, and sends telemetry.
