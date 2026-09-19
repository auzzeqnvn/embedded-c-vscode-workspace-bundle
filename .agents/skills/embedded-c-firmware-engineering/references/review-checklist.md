# Embedded C Review Checklist

Use this checklist for code review and PR review.

## A. Critical correctness

- [ ] No plausible out-of-bounds read/write
- [ ] No invalid pointer/lifetime use
- [ ] No uninitialized read
- [ ] No undefined shift or arithmetic assumption
- [ ] No unbounded external-data copy
- [ ] No critical infinite wait
- [ ] No ISR behavior that can block system progress

## B. MISRA-oriented C safety

- [ ] Signed/unsigned interactions reviewed
- [ ] Integer promotions/intermediate widths reviewed
- [ ] Narrowing casts justified/range-checked
- [ ] Pointer casts/qualifiers/alignment justified
- [ ] Side effects/evaluation order obvious
- [ ] Function prototypes complete and consistent
- [ ] Initialization intentional
- [ ] Switch/control flow explicit
- [ ] Recursion/dynamic allocation policy respected
- [ ] Preprocessor usage bounded and understandable

## C. Naming and readability

- [ ] Names express domain meaning
- [ ] Units encoded where needed
- [ ] Boolean names express truth
- [ ] No unexplained magic values
- [ ] Functions have one main responsibility
- [ ] Comments explain why/constraints

## D. Module architecture

- [ ] Public API is narrow
- [ ] Private symbols are `static`
- [ ] No writable global exposed without need
- [ ] Dependency direction is correct
- [ ] No circular dependency
- [ ] Driver/BSP does not contain business policy
- [ ] Application does not directly manipulate registers without justification

## E. ISR / DMA / RTOS

- [ ] ISR is short and bounded
- [ ] No blocking/printf/heavy parsing in ISR
- [ ] Shared data ownership documented
- [ ] `volatile` used only for valid observability needs
- [ ] Atomicity/synchronization handled separately
- [ ] Buffer overrun behavior defined
- [ ] DMA buffer lifetime/ownership correct
- [ ] RTOS ISR-safe APIs used where required

## F. State machine / timing

- [ ] States use enum
- [ ] State/event distinction clear
- [ ] Waiting states have timeout or explicit infinite-wait rationale
- [ ] Transitions are deterministic
- [ ] Invalid state has recovery behavior
- [ ] Tick wrap handled safely
- [ ] No long blocking delays in cooperative flow

## G. Error handling

- [ ] Fallible APIs return status
- [ ] Important return values checked
- [ ] Errors handled/propagated/logged intentionally
- [ ] Retry bounded or justified
- [ ] Backoff where repeated retries are harmful
- [ ] Recovery/safe state defined
- [ ] Watchdog reflects health, not mere execution

## H. Static analysis

- [ ] Compiler warning level appropriate
- [ ] No unexplained new warnings
- [ ] Static-analysis findings triaged
- [ ] MISRA tool used when formal requirement exists
- [ ] Suppressions narrow and documented
- [ ] Deviations recorded
- [ ] Analysis rerun after fixes

## Severity

**Critical**: credible memory corruption, UB with runtime impact, race/state corruption, dangerous ISR behavior, invalid lifetime/pointer, critical deadlock/hang, or safety/integrity failure.

**Major**: conversion/type hazard, wrong synchronization/volatile use, missing timeout, ignored error, uncontrolled retry, defective state machine, invalid dependency/ownership, or credible analyzer finding.

**Minor**: naming, magic number, excessive size/nesting, duplication, comment/style issue, unnecessarily exposed symbol.
