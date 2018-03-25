# Digital Output

- Direct output access to one or more pins of a port.
- All selected pins are written simultaneously.
- Supports generating output pulses of a precise length.

Pins are ordered in a descending order (DOWNTO) and described in a packed format,
e.g. if pins 1 and 4 are selected (`0b10010` on the port), the control word has
two bits `(4)(1)` and e.g. `0b10` means pin 4, `0b11` both. This makes accessing
the block of pins easier, e.g. when using them to drive a parallel bus. For single-pin units, the control word is always `1`.

## Commands

### WRITE (0x00)
Write a value to all defined pins.

*Request:*
- u16 - new value (packed)

### SET (0x01)
Set pins high

*Request:*
- u16 - pins to set high (packed)

### CLEAR (0x02)
Set pins low

*Request:*
- u16 - pins to set low (packed)

### TOGGLE (0x03)
Toggle selected pins (high - low)

*Request:*
- u16 - pins to toggle (packed)

### PULSE (0x04)
Send a pulse.

The start will be aligned to 1 us or 1 ms (based on pulse length) of the internal timebase to ensure the highest length precision. This alignment reduces jitter in the pulse duration. A jitter of the pulse start time is less significant, as there's already some unpredictable delay caused by the USB connection and the PC OS scheduler.

*Request:*
- u16 - pins to generate the pulse on (packed)
- u8 - pulse active level (0, 1)
- u8 - range (0 - milliseconds, 1 - microseconds)
- u16 - duration in the selected range

*NOTE:* The microsecond range supports durations only up to 999 us, higher
numbers will be divided by 1000 and use the millisecond range instead.

An ongoing pulse is stopped by any command affecting the pin.

## Events

*No events defined for this unit type.*
