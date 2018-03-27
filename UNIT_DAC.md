# DAC - analog output

The DAC unit has two analog outputs and can either output a DC level, or one of the pre-defined waveforms. Those waveforms are generated using a DDS principle, the numerical equivalent of a PLL.

- DC
- SINE
- TRIANGLE
- SAWTOOTH UP
- SAWTOOTH DOWN
- RECTANGLE

The DDS output speed is limited by the processor speed; properly shaped waveforms can
be generated up to tens of kHz, obviously a better resolution is achieved with slower waveforms.

DDS provides a very precise frequency adjustment at the expense of jitter. Faster frequencies tend to became more "pixelated" as there are fewer counter steps per period.

The ADC unit further supports dithering using a noise generator and a sawtooth wave,
provided by the STM32's hardware. Those are used e.g. to increase ADC resolution by introducing artificial noise through a large resistor.

## Commands

### WAVE_DC (0)
Set a DC level, disable DDS for the channel

*Request:*
- u8 - channels (1-first, 2-second, 3-both)
- u16 - level 0-4095

### WAVE_SINE (1)
Start a DDS sine waveform

*Request:*
- u8 - channels (1-first, 2-second, 3-both)

### WAVE_TRIANGLE (2)
Start a symmetrical triangle waveform

*Request:*
- u8 - channels (1-first, 2-second, 3-both)

### WAVE_SAWTOOTH_UP (3)
Start a rising sawtooth

*Request:*
- u8 - channels (1-first, 2-second, 3-both)

### WAVE_SAWTOOTH_DOWN (4)
Start a falling sawtooth

*Request:*
- u8 - channels (1-first, 2-second, 3-both)

### WAVE_RECTANGLE (5)
Start a rectangle waveform (pulse train)

*Request:*
- u8 - channels (1-first, 2-second, 3-both)
- u16 - ontime in DDS table length (presently 0-8191)
- u16 - high level 0-4095
- u16 - low level 0-4095

### SYNC (10)
Synchronize the two channels (set phase = 0)

### SET_FREQUENCY (20)
Set the frequency

*Request:*
- u8 - channels (1-first, 2-second, 3-both)
- float32 - frequency (Hz)

### SET_PHASE (21)
Set a channel's phase, relative to the initial synchronized phase.
It's recommended to set phase only of one channel, leaving the other at 0°.

*Request:*
- u8 - channels (1-first, 2-second, 3-both)
- u16 - phase in DDS table length (presently 0-8191)

### SET_DITHER (22)
Set dithering mode and strength

*Request:*
- u8 - channels (1-first, 2-second, 3-both)
- u8 - noise type (0-none, 1-white, 2-triangle)
- u8 - number of noise bits

Please note that setting high number of noise bits can lead to overflow/underflow
(probably a STM32 bug).

## Events

*No events defined for this unit type.*
