# GEX's low-level communication protocol

## Framing layer

GEX uses [TinyFrame](https://github.com/MightyPork/TinyFrame) to form and parse data frames.

- Frames are normally sent and received using two USB bulk endpoints (VCOM or raw access) or through UART.
- Transmitted frame length is virtually unlimited
- Received frame length is limited by the Rx buffer size. This is overcome using a Bulk Write function.

### Frame structure

See the TinyFrame documentation for detailed description of the frame structure and protocol functions.

Here's the configuration used by GEX:

```none
,------+----------+---------+------+------------+- - - - -+------------,
| SOF  | frame_id | pld_len | type | head_cksum | payload | pld_cksum  |
| 1    | 2        | 2       | 1    | 1          | ...     | 1          | <- size (bytes)
'------+----------+---------+------+------------+- - - - -+------------'
```

- SOF byte is `0x01`
- Checksum = inverted XOR of all bytes

USB bulk transfers have CRC error checking built-in. The checksum field is mainly used to catch
implementation mistakes, such as a CR-LF transformation.

*Frame ID* is incremented with each transaction (message, request/response, or longer).
The highest ID bit identifies the peer who started the transaction ("master", "slave").

*Type* defines the meaning of the frame within a higher level protocol. A list of all types is attached below.

## Message types

### General messages
- `0x00` ... `SUCCESS` - Generic success response; used by default in all responses;
  - *Payload:*
    - optional, differs based on the request
- `0x01` ... `PING` - Ping request, used to test connection
  - *Response:*
    - `SUCCESS` frame with an ASCII string containing the GEX version and the hardware platform name
- `0x02` ... `ERROR` - Generic failure response, used when a request fails to execute
  - *Payload:*
    - ASCII string with the error message

### Bulk transfer (large multi-frame transfer)
Bulk transfer is used for reading / writing large files that exceed the TinyFrame buffer sizes.

- `0x03` ... `BULK_READ_OFFER` - Offer of data to read.
  - *Payload:*
    - u32 - total len
- `0x04` ... `BULK_READ_POLL` - Request to read a previously announced chunk (`BULK_READ_OFFER`)
  - *Payload:*
    - u32 - max chunk size to read
  - Response: a `BULK_DATA` frame (see below)
- `0x05` ... `BULK_WRITE_OFFER` - Offer to receive data in a write transaction.
  - This is a report of readiness after some other frame introduced a request to write bulk data
    (e.g. writing a config file)
  - *Payload:*
    - u32 - max total size
    - u32 - max chunk size
- `0x06` ... `BULK_DATA` - Writing a chunk, or sending a chunk to master.
  - *Payload:*
    - a block of data
- `0x07` ... `BULK_END` - Bulk transfer is done, no more data to read or write. Recipient shall check total length and discard the received data on mismatch.
- `0x08` ... `BULK_ABORT` - Abort and discard the ongoing transfer (sent by either peer)

### Unit messages
Units are instances of a particular unit driver. A unit driver is e.g. Digital Output or SPI.
Each unit has a TYPE (which driver to use), NAME (for user convenience) and CALLSIGN (for messages)

In the INI file, all three are shown in the unit section: `[type:name@callsign]`

Units can accept commands and generate events. Both are identified by 1-byte codes.

- `0x10` ... `UNIT_REQUEST` - Command addressed to a particular unit
  - *Payload:*
    - u8 callsign
    - u8 command
    - rest - data for a unit driver
- `0x11` ... `UNIT_REPORT` - Spontaneous report from a unit
  - *Payload:*
    - u8 callsign
    - u8 report type
    - u64 usec timestamp
    - rest - data from unit driver

### System messages
Settings management etc.

- `0x20` ... `LIST_UNITS` - Get all active unit call-signs, types and names
  - *Response:*
    - u8 number of units
    - repeat for each unit:
      - u8 callsign
      - cstring type
      - cstring name
- `0x21` ... `INI_READ` - Read the ini file via bulk
  - starts a bulk read transfer
  - *Response:*
    - a `BULK_READ_OFFER` frame (see above)
- `0x22` ... `INI_WRITE` - Write the ini file via bulk
  - starts a bulk write transfer
  - *Response:*
    - a `BULK_WRITE_OFFER` frame (see above)
- `0x23` ... `PERSIST_CFG` - Write current settings to Flash
  - This is the equivalent of replacing the lock jumper / pushing the button when the virtual filesystem is enabled.
  - If the virtual filesystem is enabled, this also closes it.

