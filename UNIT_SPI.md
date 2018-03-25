# SPI master

- Can drive shift registers or communicate with any SPI based devices.
- Configurable speed, polarity and phase
- Up to 16 slave select signals
- Supports multicast (write to multiple slaves at once)

*NOTE:* To use Multicast with bi-directionally connected devices, the MISO pins should be
connected through protection resistors to prevent a short circuit on signal collision.

## Commands

### QUERY (0x00)

Write and read some bytes.

The write and read sections can overlap if needed; some devices use this
to report a status word while the command is being written. 0x00 is output on MOSI while collecting a response.

If the overlap is not desired (first write, then read), set the number of discarded
bytes equal to the number of written bytes.

*Request:*
- u8 - slave number 0-16
- u16 - number of discarded MISO bytes before collecting the response
- u16 - response length (bytes)
- u8[] - bytes to write

*Response:*
- u8[] - received bytes

### MULTICAST (0x01)

*Request:*
- u16 - slaves (packed)
- u8[] - bytes to write

## Events

*No events defined for this unit type.*
