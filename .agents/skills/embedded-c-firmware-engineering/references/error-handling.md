# Error Handling, State Machines, Timeout, Retry, and Watchdog

## Error contracts

Every fallible API must make failure observable.

Use one of:

- `bool` for simple success/failure
- module-specific result enum for multiple failure modes
- typed result/status structure when data and diagnostics must travel together

Avoid generic `-1` when callers need to distinguish causes.

Example:
```c
typedef enum
{
    MODEM_RESULT_OK = 0,
    MODEM_RESULT_TIMEOUT,
    MODEM_RESULT_NOT_READY,
    MODEM_RESULT_NO_NETWORK,
    MODEM_RESULT_INVALID_ARG,
    MODEM_RESULT_IO_ERROR
} modem_result_t;
```

## Parameter validation

Validate at trust boundaries and public APIs where invalid inputs are possible:

- null pointers
- lengths
- indexes
- enum/state values
- ranges
- packet fields
- CRC/checksum
- protocol framing

Avoid redundant checks inside tightly controlled private code when invariants are proven; keep contracts clear.

## Error ownership

Every error must be one of:

- handled locally
- propagated upward
- recorded for diagnostics
- converted to a defined recovery state
- escalated to safe reset/fail-safe behavior

Never ignore a returned error without an explicit reason.

## State machines

Represent persistent states with enums and transient occurrences with events.

Recommended pattern:

```c
typedef enum
{
    MODEM_STATE_OFF = 0,
    MODEM_STATE_POWERING,
    MODEM_STATE_INIT,
    MODEM_STATE_REGISTERING,
    MODEM_STATE_CONNECTED,
    MODEM_STATE_RECOVERY
} modem_state_t;
```

Centralize transitions when practical:

```c
static void modem_set_state(modem_state_t new_state)
{
    modem_ctx.state = new_state;
    modem_ctx.state_enter_ms = system_get_tick_ms();
}
```

## Non-blocking behavior

Do not use long delays to wait for hardware/network state.

BAD:
```c
modem_send_init();
delay_ms(5000U);
modem_check_response();
```

GOOD: send once, return to scheduler/main loop, process response or timeout on future calls.

## Timeout

Every wait on an external component normally needs a timeout unless an infinite wait is a deliberate system requirement.

Use wrap-safe unsigned subtraction:

```c
if ((uint32_t)(now_ms - start_ms) >= timeout_ms)
{
    return RESULT_TIMEOUT;
}
```

## Retry

Retry policy must define:

- maximum retry count or persistent-retry justification
- delay/backoff
- reset/reinitialization strategy
- escalation after exhaustion
- counters/diagnostics

Do not write:
```c
while (!modem_connect())
{
}
```

## Recoverable vs fatal

Recoverable examples:

- temporary GNSS fix loss
- server disconnect
- transient I2C/SPI/UART error
- network registration failure

Potentially fatal/integrity examples:

- corrupted configuration with no valid fallback
- repeated flash verification failure
- impossible internal state
- stack/heap integrity failure

Choose recovery proportional to impact.

## Watchdog

The watchdog should detect loss of system health, not merely main-loop execution.

Prefer health aggregation:

```c
if (critical_health_is_ok())
{
    watchdog_refresh();
}
```

Critical tasks may need heartbeat/deadline monitoring. Do not refresh the watchdog unconditionally from a high-priority timer ISR because this can mask a deadlocked application.
