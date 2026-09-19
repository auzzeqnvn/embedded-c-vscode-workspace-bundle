# Interrupt, DMA, Volatile, and Shared-State Rules

## ISR design

ISR execution should be short, bounded, and predictable.

Normally perform only:

1. identify/acknowledge interrupt source
2. read/capture minimum hardware data
3. place data into a bounded buffer or update a minimal counter/state
4. signal deferred processing
5. exit

Avoid in ISR unless specifically justified:

- delays or blocking waits
- protocol parsing loops
- `printf`/heavy logging
- flash erase/program
- network/modem sessions
- filesystem operations
- application/business policy
- memory allocation

## Volatile

Use `volatile` for objects whose value can change outside the current execution flow, such as:

- memory-mapped hardware registers
- ISR/main communication where direct observation is required
- DMA-updated memory where compiler visibility matters

Do not assume `volatile` provides:

- atomic read-modify-write
- mutual exclusion
- inter-task synchronization
- race freedom
- cache coherence on every architecture
- memory barriers/order guarantees

## Atomicity

Treat operations such as `counter++` as read-modify-write unless architecture/toolchain guarantees otherwise.

When shared access can race, use an appropriate mechanism:

- brief critical section
- architecture-supported atomic operation
- RTOS primitive
- lock-free single-producer/single-consumer design with proven assumptions

Do not disable interrupts longer than necessary.

## Ownership model

For each shared object document:

```text
Owner:
Writer(s):
Reader(s):
Context(s): ISR / main / DMA / RTOS task
Atomicity requirement:
Synchronization:
Overflow/overrun behavior:
Lifetime:
```

## Buffers

Ring buffers and DMA buffers must define:

- capacity
- read/write indexes
- index type and wrap behavior
- producer/consumer model
- full/empty distinction
- overflow policy
- cache maintenance if relevant to the MCU

Never allow an ISR to write beyond a buffer because the consumer is slow.

## ISR-to-main pattern

Prefer event/data transfer:

```c
static volatile bool uart_rx_event;

void USARTx_IRQHandler(void)
{
    uint8_t byte = bsp_uart_read_byte();

    if (ring_buffer_push_isr(&uart_rx_buffer, byte))
    {
        uart_rx_event = true;
    }
    else
    {
        uart_rx_overflow_count++;
    }
}
```

Normal context performs parsing.

## DMA

For DMA review, check:

- buffer lifetime remains valid for full transfer
- transfer length is bounded
- CPU does not mutate TX buffer prematurely
- CPU does not consume RX buffer before completion/ownership transfer
- callback/interrupt only signals completion or bounded work
- cache maintenance is correct on MCUs with D-cache
- `volatile` is not used as a substitute for ownership protocol

## RTOS

With an RTOS, prefer queues, notifications, event groups, semaphores, or mutexes as appropriate instead of ad hoc volatile flags between tasks.

Keep ISR APIs and task APIs distinct when the RTOS requires it.
