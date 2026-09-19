# BAD / GOOD Embedded C Examples

## 1. Signed/unsigned conversion

BAD:
```c
uint16_t speed_kmh;
int16_t correction_kmh;
uint16_t corrected_kmh = speed_kmh + correction_kmh;
```

GOOD:
```c
int32_t corrected_kmh;
corrected_kmh = (int32_t)speed_kmh + (int32_t)correction_kmh;
```

Range-check before converting back to an unsigned/narrower type.

## 2. Intermediate overflow

BAD:
```c
uint16_t speed = (pulse_count * 3600U) / 1000U;
```

GOOD:
```c
uint32_t speed_calc;
speed_calc = ((uint32_t)pulse_count * 3600UL) / 1000UL;
```

Then validate before narrowing.

## 3. ISR does too much

BAD:
```c
void USART1_IRQHandler(void)
{
    uint8_t byte = uart_read_byte();
    gps_parse_byte(byte);
    modem_send_packet();
    printf("RX=%u\n", byte);
}
```

GOOD:
```c
void USART1_IRQHandler(void)
{
    uint8_t byte = uart_read_byte();

    if (!ring_buffer_push_isr(&gps_rx_buffer, byte))
    {
        gps_rx_overflow_count++;
    }
}
```

Parse in normal context.

## 4. Volatile is not atomic

BAD:
```c
static volatile uint32_t pulse_count;

void EXTI_IRQHandler(void)
{
    pulse_count++;
}
```

If another context also performs read-modify-write, `volatile` does not prevent races.

GOOD: define one writer, or use a critical section/atomic primitive appropriate to the MCU/RTOS.

## 5. Blocking state machine

BAD:
```c
case MODEM_STATE_INIT:
    modem_send_init();
    delay_ms(5000U);
    modem_check_response();
    break;
```

GOOD:
```c
case MODEM_STATE_INIT:
    if (!modem_ctx.command_sent)
    {
        modem_send_init();
        modem_ctx.command_sent = true;
        modem_ctx.state_enter_ms = system_get_tick_ms();
    }

    if (modem_init_ok())
    {
        modem_set_state(MODEM_STATE_REGISTERING);
    }
    else if ((uint32_t)(system_get_tick_ms() - modem_ctx.state_enter_ms) >= MODEM_INIT_TIMEOUT_MS)
    {
        modem_set_state(MODEM_STATE_RECOVERY);
    }
    break;
```

## 6. Buffer bounds

BAD:
```c
rx_buffer[rx_index++] = byte;
```

GOOD:
```c
if (rx_index < UART_RX_BUFFER_SIZE)
{
    rx_buffer[rx_index] = byte;
    rx_index++;
}
else
{
    uart_rx_overflow_count++;
}
```

Choose reset/drop/overwrite policy deliberately.

## 7. Exposed module state

BAD header:
```c
extern modem_state_t modem_state;
```

GOOD header:
```c
modem_state_t modem_get_state(void);
```

GOOD source:
```c
static modem_state_t modem_state;
```

## 8. Error ignored

BAD:
```c
flash_write(address, data, length);
```

GOOD:
```c
flash_result_t result = flash_write(address, data, length);

if (result != FLASH_RESULT_OK)
{
    storage_handle_write_error(result);
}
```

## 9. Multiple side effects

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

## 10. Application accesses registers

BAD:
```c
void app_vehicle_process(void)
{
    GPIOA->BSRR = GPIO_PIN_5;
}
```

GOOD:
```c
void app_vehicle_process(void)
{
    ignition_output_set(true);
}
```

Keep the register operation in BSP/HAL code.

## 11. Retry forever

BAD:
```c
while (modem_connect() != MODEM_RESULT_OK)
{
}
```

GOOD: use a non-blocking retry policy with retry count/backoff/recovery state.

## 12. Static-analysis suppression

BAD:
```text
Disable all conversion warnings because there are too many.
```

GOOD:
```text
Baseline existing findings, fix high-risk conversions, suppress a specific proven false positive at the smallest scope, document rationale, and block new findings in CI.
```
