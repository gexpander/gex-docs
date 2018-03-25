# GEX's internal structure

GEX is modular, composed of functional blocks called *units*. A unit can be e.g. SPI or a digital output.

There can be more than one instance of a given unit, based on the microcontroller's hardware resources.
Some units use hardware peripherals, others are implemented in software. Those "soft" units (typically
bit-banged protocols) are limited only by the available GPIO pins.

## Resources

A system of resource claiming is implemented to avoid units conflicting with each other and interfering with
peripherals used internally by the system (such as the LED pin). Each unit claims the resources it uses during
initialization. If the resource needed is not available, it may try to claim an alternate resource or fail to
initialize.

A init failure is reported in the config file as an error message under the unit's header.

An example of defined resources:

- all GPIO pins
- timers/counters
- SPI, I2C, USART peripherals
- DMA channels

## Unit identification

Each unit has a *name* and a *callsign* (1-255) used for message addressing. Both are defined in the config file.

There can be up to 255 units, in practice fewer due to storage limitations.

## Unit commands and events

Units can receive command messages (`UNIT_REQUEST`) and report asynchronous events using `UNIT_REPORT` frames.

A command is identified by a number 0-127. The highest bit (0x80) of the command number is reserved to request
confirmation (a `SUCCESS` frame) if the command doesn't normally generate a response. This is used by the client
library to check that an operation was completed correctly (as opposed to e.g. the firmware crashing and rebooting
through the watchdog).

Unit events include the unit's callsign, a number 0-255 identifying the event type, and a 64-bit microsecond
timestamp.

See [FRAME_FORMAT.md](FRAME_FORMAT.md) for the payloads structure.

