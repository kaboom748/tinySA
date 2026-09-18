

<img width="559" height="337" alt="image" src="https://github.com/user-attachments/assets/b8eda741-8ef2-4bf5-94a5-1f75351f1c2b" />

# tinySA Ultra Generator WebSerial

A single-file WebSerial control panel for the **tinySA Ultra ZS405**, designed and verified against:

- **Device:** tinySA ULTRA ZS405
- **Firmware:** `tinySA4_v1.4-199-gde12ba2`
- **Target commit/build:** `de12ba2`
- **Serial link:** 115200 baud, 8 data bits, 1 stop bit, no parity, no flow control

The application runs entirely in the browser and provides a graphical interface for RF generator control, AM/FM modulation, native sweeps, slow FSK/CFO experiments, diagnostics, and a raw serial console.

> The current UI is written in French, but the controls map directly to the tinySA shell commands documented below.

---

## Features

### RF generator control

The **Generator** tab provides direct control of the tinySA Ultra output path:

- Enter generator mode with `mode output`
- RF ON/OFF control with `output on` / `output off`
- Select the RF output path:
  - `output normal` — cleanest signal, up to approximately 4.4 GHz
  - `output mixer` — highest frequency accuracy, up to approximately 5.4 GHz
- CW frequency control using:
  - `sweep cw <Hz>`
- Quick frequency offset buttons from ±1 kHz to ±100 kHz
- RF output level control
- External gain/reference correction
- Live application-side RF and modulation status

At 2.4 GHz, the application defaults to the **mixer / Highest accuracy** path.

---

## Safe initialization

The **Safe Initialization** button intentionally starts with RF disabled.

It sends the following sequence:

```text
output off
mode output
output off
output <normal|mixer>
modulation off
levelchange 0
sweep cw <frequency_hz>
level <dbm>
```

RF remains **OFF** after initialization and must be explicitly enabled by the user.

This is useful when connecting test equipment or changing RF paths without accidentally transmitting during setup.

---

## CW frequency control

The main CW control uses:

```text
sweep cw <frequency_hz>
```

The UI displays frequency in MHz and converts it to integer Hz before sending it to the tinySA.

A separate diagnostic control exposes:

```text
freq <frequency_hz>
```

The direct `freq` command is kept in the diagnostic section because it pauses the current sweep and behaves differently from the normal CW workflow.

---

## RF level control

The application uses:

```text
level <dBm>
```

The UI clamps the requested generator level to approximately:

```text
-115 dBm ... -18.5 dBm
```

This range follows the TinySA4 generator limits used by the targeted firmware build rather than the older shell help text.

Preset buttons are included for common levels such as:

- -110 dBm
- -90 dBm
- -70 dBm
- -50 dBm
- -30 dBm
- -20 dBm

---

## External gain correction

Display/reference correction is available through:

```text
ext_gain <dB>
```

Supported UI range:

```text
-100 dB ... +100 dB
```

This is a correction value only; it is **not** an additional RF attenuator.

---

## AM and FM modulation

The **Modulation** tab supports:

```text
modulation off
modulation am
modulation fm
```

Before enabling AM or FM, the application reproduces the behavior of the official UI by forcing:

```text
sweep span 0
levelchange 0
```

### Modulation frequency

```text
modulation freq <Hz>
```

The UI exposes:

```text
1 Hz ... 3500 Hz
```

### AM depth

```text
modulation depth <0..100>
```

### FM deviation

```text
modulation deviation <Hz>
```

The UI accepts deviation in kHz and converts it to Hz before sending the command.

Supported UI range:

```text
1 kHz ... 300 kHz
```

For FM operation above approximately 1.13 GHz, the application can automatically select:

```text
output mixer
```

---

## Native sweep control

The **Sweep** tab exposes the tinySA sweep engine directly.

### Frequency sweep

```text
sweep <START_HZ> <STOP_HZ> <POINTS>
```

The UI offers up to 450 points.

### Center / span

```text
sweep center <Hz>
sweep span <Hz>
```

### Sweep time

```text
sweeptime <seconds>
```

The targeted shell reports a range of approximately:

```text
0.003 ... 60 seconds
```

### Sweep accuracy modes

```text
sweep normal
sweep precise
sweep fast
sweep noise
```

### Sweep execution

```text
sweep go
sweep abort
```

The UI can also query the current sweep configuration and instrument status.

---

## FSK / CFO lab

The application includes a small experimental **FSK / CFO Lab** for RF receiver experiments.

### Manual two-tone control

Two frequencies can be configured as `F0` and `F1`.

Pressing either button sends:

```text
sweep cw <F0_hz>
```

or:

```text
sweep cw <F1_hz>
```

A center-frequency button is also provided.

### Slow binary pattern generator

A user-defined binary pattern such as:

```text
01010101
```

can be replayed by alternating between F0 and F1.

This mode is intentionally slow. The minimum dwell implemented by the page is:

```text
120 ms per symbol
```

### Important limitation

**This WebSerial page is not a precision or high-symbol-rate FSK generator.**

Browser scheduling, USB serial latency, the application's serialized command queue, and an intentional delay after each command make it suitable for:

- slow CFO experiments
- static F0/F1 comparisons
- receiver characterization
- manual frequency hopping
- laboratory debugging

It is **not** suitable for generating accurately timed FSK at rates such as several ksymbols/s.

For real high-speed FSK generation, modulation must be generated inside the RF hardware/firmware rather than by repeatedly sending WebSerial commands.

---

## CFO sweep sequence

The CFO laboratory section can step through a list of frequency offsets around a center frequency.

Example:

```text
Center: 2412.000000 MHz
Offsets: -100,-50,-20,0,20,50,100 kHz
```

Each step is converted into a new CW frequency and sent with:

```text
sweep cw <frequency_hz>
```

This is useful for manually characterizing receiver CFO behavior.

Like the binary pattern mode, it is designed for **slow laboratory measurements**, not precise modulation.

---

## Diagnostics

The **Diagnostic** tab documents and exposes the generator-related commands used by this application.

Supported commands include:

```text
mode output
output on
output off
output normal
output mixer
sweep cw <Hz>
freq <Hz>
level <dBm>
ext_gain <dB>
modulation off
modulation am
modulation fm
modulation freq <Hz>
modulation depth <value>
modulation deviation <Hz>
sweep <START> <STOP> <POINTS>
sweep normal
sweep precise
sweep fast
sweep noise
sweep go
sweep abort
sweeptime <seconds>
levelchange <dB>
caloutput <value>
actual_freq
freq_corr
status
```

A non-destructive probe button requests command usage/state information without intentionally changing normal RF settings.

---

## Calibration-related commands

The page exposes read-only controls for:

```text
actual_freq
freq_corr
```

These are calibration-related values.

They are **not** RF frequency measurements.

The application deliberately does not provide controls for writing frequency calibration values.

The calibration output connector can be controlled with:

```text
caloutput off
caloutput 30
caloutput 15
caloutput 10
caloutput 4
caloutput 3
caloutput 2
caloutput 1
```

This controls the dedicated calibration/reference output, not the main RF generator output.

---

## Raw serial console

A built-in console allows arbitrary tinySA shell commands to be sent manually.

Features include:

- TX/RX logging
- timestamps
- Enter-to-send
- log clearing
- log export to `.txt`
- automatic RF status tracking for manual `output on` / `output off` commands

Commands are terminated with a carriage return:

```text
\r
```

---

## Browser requirements

The application uses the **Web Serial API**.

Use a Chromium-based browser with Web Serial support, such as:

- Google Chrome
- Microsoft Edge
- Chromium

The page must normally be opened from a secure context:

- `https://...`
- or `http://localhost/...`

Opening the file directly with `file://` may not provide Web Serial access depending on the browser/security configuration.

---

## Running locally

No build system or external JavaScript dependencies are required.

The project is a single HTML file.

A simple local web server can be started with Python:

```bash
python -m http.server 8000
```

Then open:

```text
http://localhost:8000/
```

Select the HTML file and click **Connect**.

The browser will ask which serial device should be authorized.

---

## Connection behavior

On connection, the application opens the serial port at:

```text
115200 baud
8 data bits
1 stop bit
no parity
no flow control
```

It then performs a conservative startup sequence:

```text
output off
info
version
```

The application serializes outgoing commands and intentionally inserts a short delay after each transmission.

This makes interactive control more reliable, but it is another reason the browser-based FSK sequencer must not be treated as a precision waveform generator.

---

## RF safety

Always verify the configured RF level and frequency before enabling the output.

The application provides:

- explicit RF ON/OFF controls
- safe initialization with RF disabled
- an immediate **RF OFF** button

When testing receivers directly with a cable, use appropriate attenuation and stay within the input limits of the connected equipment.

---

## Known limitations

- Designed specifically around the command behavior observed on the targeted tinySA Ultra firmware build.
- Other firmware versions may expose different ranges or command behavior.
- The UI itself is currently in French.
- Browser/USB timing is nondeterministic.
- The slow FSK/CFO sequencer is intended for laboratory experiments only.
- WebSerial cannot provide precise high-rate symbol timing.
- `actual_freq` and `freq_corr` are calibration values, not RF measurements.
- The application does not modify frequency calibration values.
- Application-side RF state is based on commands sent by the page and may not reflect every possible external/manual instrument state change.

---

## File structure

The application is intentionally self-contained:

```text
tinysa_ultra_generator_webserial_verified_v3.html
```

It contains:

- HTML UI
- CSS styling
- WebSerial transport
- tinySA command generation
- RF state handling
- modulation controls
- sweep controls
- slow FSK/CFO laboratory tools
- diagnostics
- serial console

No framework, package manager, build step, or backend is required.

---

## Intended use

This tool is primarily intended for:

- tinySA Ultra generator control
- 2.4 GHz receiver experiments
- ESP8266 RF/PHY research
- CW and modulation testing
- slow CFO characterization
- receiver sensitivity experiments
- RF filter-response experiments
- laboratory automation and debugging

It is particularly convenient when repeatedly switching between CW frequencies, RF levels, modulation settings, and diagnostic commands during reverse-engineering work.

---

## License

No license is embedded in this HTML file.

If you publish it in a public repository, add an appropriate `LICENSE` file and update this section to match the license you choose.

---

## Disclaimer

This is an experimental laboratory utility, not an official tinySA application.

Verify commands, RF levels, frequency limits, and connected-equipment limits before transmitting.

Use RF equipment in accordance with applicable local regulations.

