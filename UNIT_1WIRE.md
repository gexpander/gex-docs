# 1-Wire

1-Wire is a Dallas (now Maxim) proprietary protocol used by the
ubiquitous DS18x20 (DS1820, DS18S20, DS18B20), iButton (contact access
chips) and other devices.

The bus uses two wires, Data and GND. A third, Vdd wire, is optional and can be left out for parasitic power from the Data line. See the DS18x20 datasheet for more details.

1-Wire requires precise timing and is implemented by bit-banging in GEX.
GEX also implements the Search algorithm for finding all devices
connected to the bus. Each device has a unique address, which are found
using a form of binary search with hardware support in the individual devices.

The search algorithm is available in a separate repository for easier re-use in other projects: [MightyPork/1wire-search](https://github.com/MightyPork/1wire-search).

## Commands

### CHECK_PRESENCE (0)

Test if there are any devices attached to the bus.

*Response:*
- u8 - presence detected (0,1)

### SEARCH_ADDR (1)

Start the search algorithm, looking for all devices.

Up to 32 devices can be found in one call. To find more, use the `SEARCH_CONTINUE`
command.

*Response:*
- u8 - have more? (0,1) - indicates whether `SEARCH_CONTINUE` should be used.
- u64[] - array of 8-byte addresses (litte-endian u64, or arrays of 8 bytes each)

### SEARCH_ALARM (2)

Identical to `SEARCH_ADDR` (see above), except only devices with
an active Alarm status are detected. This is used with thermometers
which can detect the temperature exceeding pre-selected limits.

### SEARCH_CONTINUE (3)

Continue a search (address or alarm). The report format is identical
to `SEARCH_ADDR`, but continues after the last reported address.

### READ_ADDR (4)

Read the address of the single device connected to the bus.
If more than one device is connected, this will likely fail due to a
checksum mismatch.

*Response:*
- u64 - a 8-byte addresses (litte-endian u64, or an arrays of 8 bytes)

### WRITE (10)

Write bytes to the bus.

*Request:*
- u64 - a 8-byte device address
  - 0 = all devices, uses the ROM_SKIP bus command
- u8[] - bytes to write

### READ (11)

Send a query and read bytes from the bus.

*Request:*
- u64 - a 8-byte device address
  - 0 = use the ROM_SKIP bus command, use with a single device connected only
- u16 - read length (bytes)
- u8 - verify the CRC chekcsum (0,1)
- u8[] - request bytes (written before reading)

*Response:*
- u8[] - read bytes

### POLL_FOR_1 (20)

Wait for a READY status (used in DS18x20 measurements).

*NOTE:* Not available if parasitic power is used, then it just waits the maximal measurement time of DS1820.

A response is issued after all devices are done measuring,
or a timeout error is returned.

## Events

*No events defined for this unit type.*
