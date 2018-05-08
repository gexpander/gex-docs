# USART

This unit provides access to the hardware USART peripheral. It is capable of
driving RS485 transceivers.

The unit uses asynchronous reception and transmission using DMA to support
low baud rates without lagging the whole platform. Reception is double-buffered
and sent in buffer-sized chunks. The remainder is sent when a timeout from
the last received byte is reached.


## Commands

### WRITE (0)

Add data to the Tx buffer. Sending is asynchronous, but the command may wait
for free space in the DMA buffer.

*Request:*
- u8[] - bytes to write

### WRITE_SYNC (1)

Add data to the Tx buffer and wait for the transmission to complete.

*Request:*
- u8[] - bytes to write

## Events

### DATA_RECEIVED (0)

Data was received on the serial port.

*Payload:*
- u8[] - received bytes
