# PWMDIM

This unit provides an up to 4-channel PWM output with shared frequency and independent duty cycles. Only one instance can be created due to using a
hardware timer.


## Commands

### SET_FREQUENCY (0)

Set the PWM frequency

*Request:*
- u32 - frequency in Hz

### SET_DUTY (1)

Set the duty cycle of one or more channels

*Request:*
- repeat 1-4 times:
  - u8 - channel number 0-3
  - u16 - duty cycle 0-1000

### STOP (2)

Stop the hardware timer. Outputs enter low level. Has no effect if stopped.

### START (3)

Start the timer. Has no effect if running.

## Events

*This unit generates no events.*
