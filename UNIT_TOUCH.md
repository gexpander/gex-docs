# TOUCH

Access to the hardware touch sensing controller. 
Can be used to create capacitive touch interfaces or for rough capacitance measurement (e.g. water level in a bottle, or proximity sensing).

Button mode is implemented for simple threshold checking with hystheresis.


## Commands

### READ (0x00)

Read the raw touch pad values (lower indicates higher capacitance).
Values are ordered by group and channel.

*Response:*
- u16[] - values

### SET_BIN_THR (0x01)

Set button mode thresholds. Value 0 = button mode disabled fro the pad.

*Request:*
- u16[] - thresholds

### DISABLE_ALL_REPORTS (0x02)

Set thresholds to 0, disabling the button mode for all pads.


### SET_DEBOUNCE_TIME (0x03)

Set debounce time for the button mode (replaces the value from unit settings)

*Request:*
- u16 - debounce time milliseconds


### SET_HYSTERESIS (0x04)

Set hysteresis (replaces the default value from settings)

Hysteresis is added to the threshold value for the switch-off level
(switch-off happens when the measured value is exceeded - capacitance of the pad
drops)

*Request:*
- u16 - hystheresis


### GET_CH_COUNT (0x0A)

Get the number of enabled channels

*Response:*
- u8 - channel count



## Events

### BUTTON_CHANGE (0x00)

The binary state of some of the capacitive pads
with button mode enabled changed.

*Payload:*
- u32 - binary state of all channels (packed)
- u32 - changed / trigger-generating channels (packed)


