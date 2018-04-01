# ADC - analog capture

The ADC unit supports instantaneous readout, exponential averaging and isochronous
sampling of up to 16 channels (+ two channels dedicated to a bandgap reference and the
internal temperature sensor). The voltage reference may be used to compensate for supply
voltage variations and thus achieve higher accuracy.

The ADC unit has been tested to work at up to 75 kSps with 1 channel on the 48 MHz
STM32F072, higher frequencies made the system unstable. This should be possible to fix
in a later firmware update. When sampling multiple channels, the maximal frequency
drops as more data needs to be sent through the communication port.

This unit has several opmodes. All opmodes use periodic sampling; the frequency
can be pre-comnfigured or set using a command.

- *direct measurement*
  - immediate read-out of the latest captured sample
  - averaging available for lower frequencies, f < 20 kHz
- *isochronous sampling*
  - a unlimited stream of samples (use e.g. for visualisation)
  - block capture of a fixed length
  - level-triggered capture with a pre-trigger buffer (similar to an oscilloscope)

Please note that the opmodes are mutually exclusive, e.g. it's not possible
to read a averaged sample while streaming - the command will return a BUSY error.

When a stream / block / trigger capture finishes, the unit returns to a periodic sampling
mode and a direct read-out is available. This is also available when armed for a trigger;
triggering is implemented by checking the value of those direct samples.

Channels must be configured in the unit settings, this is to claim the pins. They can be enabled/disabled through a command later.

## Commands

### READ_RAW (0)
Get the last raw sample of enabled channels.

*Response:*
- u16[] - values for enabled channels, ascending order

### READ_SMOOTHED (1)
Read the averaged values of enabled channels.
This function is not available for high sample rates on the STM32F072 due to the soft
float overhead.

*Response:*
- float32[] - values for enabled channels, ascending order

### READ_CAL_CONSTANTS (2)
Read calibration constants for the internal voltage reference and the temperature sensor.
See the reference manual for conversion formulas and other details (RM0091, chapter ADC).

Those constants are stored in ROM after factory testing.

*NOTE:* The given tolerances are valid for STM32F072

*Response:*
- u16 - VREFINT_CAL - ADC raw value for VREFINT, 30°C ambient
- u16 - VREFINT_CAL_VADCREF - Analog reference voltage during VREFINT calibration (mV), ±10mV - *always 3300*

- u16 - TSENSE_CAL1 - ADC raw value in point 1
- u16 - TSENSE_CAL2 - ADC raw value in point 2
- u8 - TSENSE_CAL1_TEMP - Temperature for point 1 (Celsius), ±5°C - *always 30*
- u8 - TSENSE_CAL2_TEMP - Temperature for point 2 (Celsius), ±5°C - *always 110*
- u16 - TSENSE_CAL_VADCREF - Analog reference voltage during TSENSE calibration (mV), ±10mV - *always 3300*

### GET_ENABLED_CHANNELS (10)
Get numbers of all enabled channels.

*Response:*
- u8[] - numbers of all enabled channels, ascending order

### GET_SAMPLE_RATE (11)
Get the current sample rate in Hz

*Response:*
- u32 - sample rate

### SETUP_TRIGGER (20)
Configure the trigger level and other parameters.

*Request:*
- u8 - source channel number, 0-based. Must be enabled.
- u16 - triggering level (0-4095)
- u8 - triggering edge (1-falling, 2-rising, 3-any)
- u32 - pre-trigger capture length (samples)
- u32 - post-trigger capture length (samples)
- u16 - hold-off time (milliseconds)
- u8 - auto re-arm (0,1) after hold-off

The trigger is not armed, only configured. Use the `ARM` command to arm.
This is so the user can manually re-arm without sending the configuration again.

### ARM (21)
Arm the configured trigger. If already armed, do nothing.

*Request:*
- u8 - auto re-arm (0,1) after hold-off.
  - 0xFF (255) = no change

### DISARM (22)
Dis-arm the trigger.

### ABORT (23)
Abort any ongoing capture and dis-arm the trigger.

### FORCE_TRIGGER (24)
Manually set off the triggering condition. This is useful for testing the pre-trigger
configuration.

### BLOCK_CAPTURE (25)
Start a manual block capture. This is similar to `FORCE_TRIGGER`, but includes no pre-trigger and the length is defined in the command.

*Request:*
- u32 - number of samples to capture

### STREAM_START (26)
Start a capture stream using the current sample rate and other settings.

### STREAM_STOP (27)
Stop a stream. This is effectively equivalent to `ABORT`, but only allowed when
a stream is running.

### SET_SMOOTHING_FACTOR (28)
Set the exponential averaging smoothing factor.
See the unit settings for the formula.

*Request:*
- u16 - factor 0-1000, equivalent to 0.0-1.0

### SET_SAMPLE_RATE (29)
Set the sampling frequency.

*Request:*
- u32 - frequency in Hz

### ENABLE_CHANNELS (30)
Select chanbnels to enable.

*Request:*
- u32 - bit-map of the enabled channels (bits numbered from LSB=0)

### SET_SAMPLE_TIME (31)
Set sampling time. This is the time the ADC waits for the internal sampling capacitor
to charge before starting the SAR algorithm. Shorter times should in theory allow faster
sample rates, but since the main bottleneck here is the communicaton port, this doesn't
have much consequence and setting it too short only degrades performance with no
benefits.

*Request:*
- u8 - sample time 0-7 (see the STM32F072 reference manual for details)

## Events

Data events use a *serial number* field, which is incremented with each message.
When the PC client receives an event, it checks this field and in case of discontinuity, detects a broken stream and acts accordingly (typically by seinding `ABORT`).

Samples are always interleaved (A0,B0,C0, A1,B1,C1, ...) in the array, ordered from the lowest channel number.

### TRIGGERED (50)

When a triggering condition occurs (or the trigger is forced manually),
this event is generated. It includes the pre-trigger buffer, if enabled.
This event is followed by multiple `CAPTURE_*` events with more data.

*Payload:*
- u32 - pretrigger length
- u8 - triggering edge (1-falling, 2-rising, 3-forced)
- u8 - stream serial number
- u16[] - the pre-trigger buffer

### CAPTURE_MORE (51)

A chunk of a stream that's expected to be followed by one or more chunks.
This event is used for all capture modes. All messages in a stream use the same
Frame ID, either generated for the first message (trigger, forced trigger), or taken
from the request (in the case of a stream or a block capture).

*Payload:*
- u8 - stream serial number
- u16[] - the samples buffer

### CAPTURE_DONE (52)

Indicates the stream has ended.

If a payload is present (the last chunk), it has the same format as the
`CAPTURE_MORE` payload.
