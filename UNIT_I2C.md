# I2C master

## Commands

### WRITE (0)

*Request:*
- u16 - slave address
- u8[] - bytes to write

### READ (1)

*Request:*
- u16 - slave address
- u16 - number of bytes to read

*Response:*
- u8[] - received bytes

### WRITE_REG (2)

Write a register; first writes the register number, then (in the same transaction)
the data. If the device supports it, can write multiple registers at once.

*Request:*
- u16 - address
- u8 - register number
- u8[] - bytes to write

### READ_REG (3)

Read a register value. First writes the register number, then reads a number of bytes.
For devices implementing auto-increment, the register width field can be used to read
multiple registers at once.

*Request:*
- u16 - address
- u8 - register number
- u16 - register width (number of bytes to read)

*Response:*
- u8[] - received bytes


## Events

*No events defined for this unit type.*
