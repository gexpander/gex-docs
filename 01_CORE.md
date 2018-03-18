# GEX core protocol

## Framing layer

GEX uses [TinyFrame](https://github.com/MightyPork/TinyFrame) to form and parse data frames.

- Frames are sent and received using two USB bulk endpoints (VCOM or raw access) or through UART.
- Transmitted frame length is virtually unlimited
- Received frame length is limited by the Rx buffer size.

### Frame structure

```none
,------+----------+---------+------+------------+- - - - -+------------,                
| SOF  | frame_id | pld_len | type | head_cksum | payload | pld_cksum  |                
| 1    | 2        | 2       | 1    | 1          | ...     | 1          | <- size (bytes)
'------+----------+---------+------+------------+- - - - -+------------'      
```

- SOF byte is `0x01`
- Checksum = inverted XOR of all bytes

The checksum is used to catch implementation mistakes (such as mistaken CR-LF transformations).
USB bulk transfers have CRC error checking built-in.

*Frame ID* is incremented with each transaction (message, request-response or longer). 
The highest ID bit identifies the peer who started the transaction.

*Type* defines the meaning of the frame. A list is attached below.

## Message types

### General messages
- `0x00` ... `SUCCESS` - Generic success response; used by default in all responses; payload is transaction-specific
- `0x01` ... `PING` - Ping request, used to test connection
  - Response: ASCII string with the version and platform name
- `0x02` ... `ERROR` - Generic failure response (when a request fails to execute)
  - Response: ASCII string with the error message

### Bulk transfer (large multi-frame transfer)
Bulk transfer is used for reading / writing large files that exceed the TinyFrame buffer sizes.

- `0x03` ... `BULK_READ_OFFER` - Offer of data to read. 
  - Payload: u32 total len
- `0x04` ... `BULK_READ_POLL` - Request to read a previously announced chunk (`BULK_READ_OFFER`)
  - Payload: u32 max chunk size to read
  - Response: a `BULK_DATA` frame
- `0x05` ... `BULK_WRITE_OFFER` - Offer to receive data in a write transaction.
  - This is an affirmation to some other frame that introduced a request to write bulk data.
  - Payload: u32 max total size, u32 max chunk size
- `0x06` ... `BULK_DATA` - Writing a chunk, or sending a chunk to master.
- `0x07` ... `BULK_END` - Bulk transfer is done, no more data to read or write. Recipient shall check total len and discard the received data on mismatch. 
- `0x08` ... `BULK_ABORT` - Discard the ongoing transfer (sent by either peer)

### Unit messages
Units are instances of a particular unit driver. A unit driver is e.g. Digital Output or SPI.
Each unit has a TYPE (which driver to use), NAME (for user convenience) and CALLSIGN (for messages)

In the INI file, all three are shown in the unit section: `[type:name@callsign]`

Units can accept commands and generate events. Both are identified by 1-byte codes.

- `0x10` ... `UNIT_REQUEST` - Command addressed to a particular unit
  - Payload: u8 callsign, u8 command, rest - data for a unit driver
- `0x11` ... `UNIT_REPORT` - Spontaneous report from a unit
  - Payload: u8 callsign, u8 type, u64 usec timestamp, rest: data from unit driver

### System messages
Settings management etc.

- `0x20` ... `LIST_UNITS` - Get all active unit call-signs, types and names
  - Response: u8 count, { u8 callsign, cstring type, cstring name }
- `0x21` ... `INI_READ` - Read the ini file via bulk
  - starts a bulk read transfer
  - Response: a `BULK_READ_OFFER` frame
- `0x22` ... `INI_WRITE` - Write the ini file via bulk
  - starts a bulk write transfer
  - Response: a `BULK_WRITE_OFFER` frame
- `0x23` ... `PERSIST_CFG` - Write current settings to Flash (the equivalent of replacing the lock jumper / pushing the button if VFS is active)

