# NeoPixel

Implements the NeoPixel protocol for driving addressable LED strips.
The strip length is configured in the unit settings.

Five color data encodings are available for user convenience.

## Commands

### CLEAR (0)
Set all pixels to black. This is also automatically executed on start-up to clear the strip.

### LOAD (1)
Load packed RGB data to the strip.

*Payload:*

- a byte array `(R,G,B)` x length

### LOAD_U32_ZRGB (4)
Load 32-bit `0x00RRGGBB` words encoded in big-endian as `(0,R,G,B)`.

*Payload:*

- a byte array `(0,R,G,B)` x length

### LOAD_U32_ZBGR (5)
Load 32-bit `0x00BBGGRR` words encoded in big-endian as `(0,B,G,R)`.

*Payload:*

- a byte array `(0,B,G,R)` x length

### LOAD_U32_RGBZ (6)
Load 32-bit `0x00BBGGRR` words encoded in little-endian as `(R,G,B,0)`.

*Payload:*

- a byte array `(R,G,B,0)` x length

### LOAD_U32_BGRZ (7)
Load 32-bit `0x00RRGGBB` words encoded in little-endian as `(B,G,R,0)`.

*Payload:*

- a byte array `(B,G,R,0)` x length

### GET_LEN (10)
Read the neopixel strip length as configured in the settings.

*Response:*

- u16 - number of pixels

## Events

*No events defined for this unit type.*
