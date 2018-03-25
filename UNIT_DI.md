# Digital Input

- Direct digital input on selected pins of a port
- Pin change events

Pins are described using the packed format (see [UNIT_DO.md](UNIT_DO.md))

## Commands

### READ (0x00)
Read all pins.

*Response:*
- u16 - pin states (packed)

### ARM_SINGLE (0x01)
Arm a pin or pins for a change detection.
The active edge is defined in the unit settings.
The pins are dis-armed again after a detected event.

*Request:*
- u16 - pins to arm (packed)

### ARM_AUTO (0x02)
Arm a pin or pins for a change detection with automatic re-arm.
The active edge is defined in the unit settings.

*Request:*
- u16 - pins to arm (packed)

### DISARM (0x03)
Disable change detection on the selected pins.

*Request:*
- u16 - pins to dis-arm (packed)

## Events

### PIN_CHANGE (0x00)

External interrupt, pin change(s) detected.
Reports which pins caused the event (can be multiple), and the entire unit's input captured right after the event.

*Payload:*
- u16 - pins that caused the event (packed)
- u16 - port snapshot at the time of the event (packed)
