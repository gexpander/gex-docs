# Digital Output

- Direct output access to one or more pins of a port.
- All selected pins are written simultaneously.
- Supports generating output pulses of precise length.

Pins are ordered in a descending order (DOWNTO) and accessed in a packed format,
e.g. if pins 1 and 4 are selected (0b10010 on the port), the control word has
two bits (4)(1) and e.g. 0b10 means pin 4, 0b11 both. This makes accessing the
block of pins easier, e.g. when using them to drive a parallel bus.

## Commands

### WRITE
Write a value to all defined pins.

*Payload:*
- u16 - new value (packed)

### SET
Set pins high

*Payload:*
- u16 - pins to set high (packed)

### CLEAR
Set pins low

*Payload:*
- u16 - pins to set low (packed)

### TOGGLE
Toggle selected pins (high - low)

*Payload:*
- u16 - pins to toggle (packed)

### PULSE
Send a pulse. The start will be aligned to 1 us or 1 ms (based on pulse length) of the internal timebase to ensure the highest length precision. (This alignment reduces time jitter.)

*Payload:*
- u16 - pins to generate the pulse on (packed)
- u8 - pulse active level (0, 1)
- u8 - range (0 - milliseconds, 1 - microseconds)
- u16 - duration in the selected range

*NOTE:* The microsecond range supports durations only up to 999 (1 ms), higher
numbers will be divided by 1000 and use the millisecond range instead.

## Events

*No events defined for this unit type.*
