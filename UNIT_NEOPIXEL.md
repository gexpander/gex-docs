# NeoPixel

Implements the NeoPixel protocol for driving addressable LED strips.
The strip length is configured in the unit settings.

## Commands

### CLEAR (0x00)
Set all pixels to black. This is automatically executed on start-up to clear the strip.

### LOAD (0x01)
Load packed RGB data to the strip.

*Payload:*

- a byte array `(R,G,B)` x length

### LOAD_U32_LE (0x02)
Load 32-bit `0x00BBGGRR` words encoded in little-endian as `(R,G,B,0)`.
This is more convenient for the PC script, but wastes 1/4 of the payload by padding
null bytes.

*Payload:*

- a byte array `(R,G,B,0)` x length

### LOAD_U32_BE (0x03)
Load 32-bit `0x00BBGGRR` words encoded in big-endian as `(0,B,G,R)`.

*Payload:*

- a byte array `(0,B,G,R)` x length


### GET_LEN (0x04)
Read the neopixel strip length as configured in the settings.

*Response:*

- u16 - number of pixels

## Events

*No events defined for this unit type.*
