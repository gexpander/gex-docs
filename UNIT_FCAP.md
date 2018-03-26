# Frequency / Pulse measurement

- Frequency measurement
  - Direct method (counting pulses in a time period)
  - Indirect method (measuring a single pulse)
- PWM measurement - Indirect method also reports duty cycle
- Single pulse measurement
- Free-running pulse counter

### Direct vs Indirect method

The direct method works well for a wide range of frequencies, from Hz to tens of MHz. For
very high frequencies, a prescaller can be enabled. This method has a resolution of 1 Hz. It could be increased by extending the measurement time, usually 1 second.

The indirect method has a sub-Hz resolution and is much faster than the direct method for
low frequencies. It can be used for signals up to tens of kHz, while the direct method
can handle 20 MHz or more without issues.

*NOTE:* The accuracy of all measurements is dependent on the system clock speed.
To measure the system clock using an external counter, you can output it on a pin using
the MCO function configured in `SYSTEM.INI`.

### Burst vs Continuous

Both the direct and indirect methods can operate in a free-running or burst mode.

- Burst mode performs one or several measurements (with averaging if using the indirect method).
- Free-running (continuous) mode performs periodic measurements without averaging.
  The advantage is that the last measurement is always immediately available for read-out, without having to wait for the emasurement.

## Commands

### STOP (0)

Stops all ongoing measurements.

### INDIRECT_CONT_START (1)

Start a free-running frequency / duty cycle measuring using the indirect method.

### INDIRECT_BURST_START (2)

Start a burst of indirect measurements.
Waits for the next rising edge before starting the measurements.

*Request:*
- u16 - number of pulses to average

*Response:*
- u16 - core speed in MHz
- u16 - number of pulses
- u64 - sum of all periods (in core ticks)
- u16 - sum of all on-times (positive interval of the pulse)

GEX avoids using float here to reduce code size and avoid floating
point errors. To get the period, use `period_sec = periods / (mhz * 10e6 * count)`

Frequency is `freq = 1 / period`.

The average duty cycle is `duty = ontimes / periods`

### DIRECT_CONT_START (3)

Start repeated measurement using the direct method.

*Request:*
- u16 - measurement time (ms)
  - 0 = no change
  - default 1000, configurable in the unit settings
- u8 - prescaller (1,2,4,8)
  - 0 = no change
  - default 1, configurable in the unit settings

### DIRECT_BURST_START (4)

Start a single direct measurement (pulse counting) over
a given time period. Longer periods can be used for higher precision
using averaging.

*Request:*
- u16 - measurement time (ms), often 1000 (=1s)
- u8 - prescaller (1,2,4,8)
  - 0 = no change
  - default 1, configurable in the unit settings

*Response:*
- u8 - prescaller (1,2,4,8)
- u16 - measurement time (ms)
- u32 - pulse count

Calculate the frequency by `freq = (1000 * count * presc) / time_ms`.

### FREECOUNT_START (5)

Clear and start the free-running pulse counter.

*Request:*
- u8 - prescaller (1,2,4,8)
  - 0 = no change
  - default 1, configurable in the unit settings

### MEASURE_SINGLE_PULSE (6)

Measure a single pulse on the input. Waits for a rising edge.

*NOTE:* The result may be inaccurate if the input is High when starting the measurement.

*Response:*
- u16 - core speed in MHz
- u32 - pulse length (core ticks)

### FREECOUNT_CLEAR (7)

Clear the free-running counter while retrieving the current value.
Normally no pulses should be missed, but please be aware that this
operation is not atomic and glitches can occur at high frequencies.

*Response:*
- u32 - last counter value

### INDIRECT_CONT_READ (10)

Read the latest result from the continuous indirect method measurement
(measurement of individual pulses).

*Response:*
- u16 - core speed in MHz
- u32 - period (core ticks)
- u32 - on-time (positive interval of the pulse)

Duty cycle is `duty = ontime / period`.
Period `period_sec = period / (mhz * 10e6)`.
Frequency `freq = 1 / period_sec`.

### DIRECT_CONT_READ (11)

Read the latest result from the continuous direct method measurement (pulse counting).

*Response:*
- u8 - prescaller (1,2,4,8)
- u16 - measurement time (ms)
- u32 - pulse count

Frequency is `freq = (1000 * presc * count) / time_ms`.

### FREECOUNT_READ (12)

Read the current value of the free-running pulse counter.

*Response:*
- u32 - counter value

### SET_POLARITY (20)

Set the measured pulse polarity.

*NOTE:* This is normally configured in the unit settings.
Changes done through this method are temporary, not persisted to flash. The same
applies to the other SET_* commands.

*Request:*
- u8 - active level (0,1)

### SET_DIR_PRESC (21)

Set prescaller for the direct mode measurement.

*NOTE:* This does not take effect until the direct measurement is restarted. (TODO: fix?)

*Request:*
- u8 - prescaller (1,2,4,8)

### SET_INPUT_FILTER (22)

Set input filtering (a hardware feature designed to ignore short glitches)

*Request:*
- u8 - digital filtering factor (0-15, 0=off)

### SET_DIR_MSEC (23)

Set direct measurement period.

*NOTE:* This does not take effect until the direct measurement is restarted. (TODO: fix?)

*Request:*
- u16 - period in milliseconds

### RESTORE_DEFAULTS (30)

Stop any measurement and restore all the run-time configurable settings to their default
values (as configured in the unit settings).
