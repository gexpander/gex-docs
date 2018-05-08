# SIPO - Multi-cast Shift Register Driver

*SIPO stands for Serial Input Parallel Output*

- Used for building user interfaces (LED displays etc.)
- Designed for driving 74xx595 or 74xx4094
- Can drive multiple data lines in parallel
- Offers direct pin access without sending data (no clock pulse)
- Supports data latch on Store pulse (see the WRITE command for details)
- Pin polarities are configurable in the unit settings.

The `DIRECT_*` commands are mostly meant for debugging purposes when trying to communicate with an unfamiliar SIPO-based board.

## Commands

### WRITE (0)

Sends data to the shift registers.

After each byte, the *Shift* pin is pulsed. After the last byte and the *Shift* pulse, the *Latch* data is output followed by the *Store* pin pulse.

The *Latch* data is normally 0 and doesn't matter; however, it's possible to design
external circuitry that latches the data pins on the *Store* pulse, e.g. for brightness
control or extra LEDs.

*Request:*
- u16 - latch data to output before the Store pulse - all outputs, packed
- (u8 array) x num_outputs
  - must be a multiple of the outputs count (if 1, simply the output data)

### DIRECT_DATA (1)

Direct write to the data pins, without any pulse.

*Request:*
- u16 - data to output, packed

### DIRECT_CLEAR (2)

Pulse the *Clear* output.

### DIRECT_SHIFT (3)

Pulse the *Shift* output.

### DIRECT_STORE (4)

Pulse the *Store* output.

## Events

*No events defined for this unit type.*
