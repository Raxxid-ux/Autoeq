# KZ Duonic Type-C DSP — Reverse Engineering & Hardware EQ Control

A reverse-engineering log of the KZ Duonic Type-C DSP cable.

The goal of this project was to understand what is actually inside the KZ Duonic Type-C DSP cable, how its physical EQ/preset button works, and whether its internal DSP parameters can be accessed and modified directly from a computer.

This investigation was performed on Windows 11 using Python, `hidapi`, USB/HID inspection, register reads/writes, and comparison of the DSP state across the cable's physical EQ modes.

> **Status:** The live DSP EQ has been successfully read and modified. Persistent custom presets have **not** yet been achieved.

---

## 1. Hardware

### KZ Duonic Type-C DSP cable

The cable enumerates as:

```text
Manufacturer: KTMicro
Product:      KZ Acoustics M2
VID:          0x31B2
PID:          0x0311
Serial:       2020-02-20-0000-0000-0000
```

The USB device is composite:

```text
USB Audio interface
+
HID interface
```

The audio interface is handled by the Windows USB Audio driver.

The HID interface appears as a consumer-control device, but the DSP preset button does **not** behave like a normal HID button.

---

# 2. HID interface

The KZ HID interface uses:

```text
Usage Page: 0x0C
Usage:      0x01
Interface:  3
```

The HID report descriptor contains:

```text
Report ID 0x01
```

for ordinary media controls.

It also contains vendor-defined reports:

```text
Report ID 0x4B
Report ID 0x54
Usage Page 0xFF01
```

The important discovery was that the `0x4B` report is not merely a normal media-control report.

It is used as a vendor-specific command channel for the DSP.

---

# 3. DSP register protocol

The KTMicro reverse-engineering work provided the structure of the protocol used by this family of devices.

The basic packet is 11 bytes.

### Register read

```text
4B [address:LE32] 52 00 00 00 00 00
```

Response:

```text
4B [address:LE32] 52 00 [value:LE32]
```

### Register write

```text
4B [address:LE32] 57 00 [value:LE32]
```

Response:

```text
4B [address:LE32] 57 00 [status:LE32]
```

A successful write returns:

```text
0x03
```

in the status field.

The same protocol is documented by the `gxcreator/ktmicro-tools` project for KTMicro KT02H20 run-mode devices.

The KZ device uses a different PID from the documented `31B2:0111` device, so compatibility was initially only a hypothesis.

It was then experimentally verified because the KZ `31B2:0311` accepted the same register protocol and returned coherent register data.

---

# 4. Identifying the DSP family

The device exposes a register structure matching the KTMicro KT02H20 family.

Important registers include:

```text
0x01  Protocol / model flags
0x18  ADC EQ enable
0x1A  ADC EQ bands
0x24  DAC EQ enable
0x26  DAC EQ bands
0x3A  ADC analog PGA
0x3B  DAC analog PGA
0x65  ADC digital gain
0x66  DAC digital gain
0x71  Noise Gate
0x78  Limiter
0xE1  Magic value
```

The device reports:

```text
0x01 = 0x00000200
```

which corresponds to the single-DAC model flag described by the KTMicro register documentation.

The device also reports:

```text
0xE1 = 0x12345678
```

which matches the documented KTMicro magic register.

So the strongest conclusion from the available evidence is:

> **The KZ Duonic Type-C DSP cable is built around the KTMicro KT02H20 DSP platform/family.**

This is based on protocol/register compatibility and behavior; the physical chip marking itself has not yet been inspected.

---

# 5. The five-band hardware PEQ

The most important discovery was the DAC EQ block.

The DAC EQ is enabled through:

```text
0x24
```

The five bands occupy:

```text
Band 1 → 0x26 / 0x27
Band 2 → 0x28 / 0x29
Band 3 → 0x2A / 0x2B
Band 4 → 0x2C / 0x2D
Band 5 → 0x2E / 0x2F
```

Each band consists of two 32-bit registers.

### Register A

```text
bits 31-16 = frequency in Hz
bits 15-0  = signed gain × 10
```

Example:

```text
0x0064000A
```

decodes as:

```text
100 Hz
+1.0 dB
```

### Register B

```text
bits 18-16 = filter type
bits 15-0  = Q × 1000
```

For example:

```text
0x000007D0
```

means:

```text
Q = 2.000
```

The filter type used by the factory presets is:

```text
0 = Peaking
1 = LPF
2 = HPF
3 = Low Shelf
4 = High Shelf
```

---

# 6. Factory EQ presets

The cable has four physical sound modes:

```text
Purple → Green → Red → Blue → Purple
```

By reading the registers in each mode, we discovered that the physical button changes actual DSP coefficients.

### Green

```text
Band 1:  15 Hz    -2.0 dB   Q 1.000
Band 2: 100 Hz    +1.0 dB   Q 2.000
Band 3: 200 Hz    +1.0 dB   Q 2.000
Band 4: 1000 Hz   +1.0 dB   Q 3.000
Band 5: 2500 Hz   +1.0 dB   Q 2.000
```

### Red

```text
Band 1:  15 Hz    -3.0 dB   Q 1.000
Band 2: 150 Hz    +2.0 dB   Q 2.000
Band 3: 1000 Hz   +2.0 dB   Q 2.000
Band 4: 2500 Hz   +2.0 dB   Q 2.000
Band 5: 3000 Hz   +1.0 dB   Q 2.000
```

### Blue

```text
Band 1:  15 Hz    -1.0 dB   Q 2.000
Band 2:  50 Hz    +2.0 dB   Q 2.000
Band 3: 150 Hz    +2.0 dB   Q 2.000
Band 4: 1500 Hz   -1.0 dB   Q 2.000
Band 5: 5000 Hz    0.0 dB   Q 0.707
```

### Purple

```text
Band 1: 200 Hz    +1.0 dB   Q 2.000
Band 2: 3000 Hz   +1.0 dB   Q 2.000
Band 3: 5000 Hz   +1.0 dB   Q 3.000
Band 4: 1500 Hz    0.0 dB   Q 0.707
Band 5: 2500 Hz    0.0 dB   Q 0.707
```

The factory modes therefore aren't merely different LED colors or software labels.

They correspond to different DSP coefficient sets.

---

# 7. The biggest discovery: the physical button reloads the EQ

We wrote custom values directly into the DAC EQ registers.

For example, a custom five-band profile was written to:

```text
0x26–0x2F
```

Every write returned the expected acknowledgement and read-back matched the written value.

The custom EQ was audibly active, although some profiles produced only subtle differences.

However:

```text
Green
  ↓
Red
  ↓
Blue
  ↓
Green
```

caused the factory Green coefficients to reappear.

This means:

> `0x26–0x2F` are live DSP configuration registers, while the physical preset system has another source from which it reloads the factory coefficients.

This is one of the most important findings of the investigation.

---

# 8. Directly modifying the EQ

Because the register encoding is understood, arbitrary five-band PEQ settings can be written directly.

For example:

```text
100 Hz +4 dB
300 Hz -4 dB
1 kHz +4 dB
3 kHz -4 dB
6 kHz +4 dB
```

was encoded and written directly to the DSP.

The resulting registers were:

```text
0x26 = 0x00640028
0x27 = 0x000002BC

0x28 = 0x012CFFD8
0x29 = 0x000003E8

0x2A = 0x03E80028
0x2B = 0x000003E8

0x2C = 0x0BB8FFD8
0x2D = 0x000003E8

0x2E = 0x17700028
0x2F = 0x000002BC
```

The registers read back exactly.

There were audible differences between custom profiles, although the effect was smaller than initially expected.

---

# 9. AutoEQ experiments

A larger AutoEQ tuning was also converted into the cable's five-band PEQ limitation.

The original AutoEQ profile contained more filters than the hardware provides, so it had to be approximated using five bands.

## Natural

Approximation:

```text
139 Hz   +1.9 dB   Q 0.950
172 Hz   -0.3 dB   Q 1.100
977 Hz   +2.9 dB   Q 3.780
1806 Hz  +2.5 dB   Q 0.515
5178 Hz  -0.2 dB   Q 1.860
```

Encoded:

```text
0x26 = 0x008B0013
0x27 = 0x000003B6

0x28 = 0x00ACFFFD
0x29 = 0x0000044C

0x2A = 0x03D1001D
0x2B = 0x00000EC4

0x2C = 0x070E0019
0x2D = 0x00000203

0x2E = 0x143AFFFE
0x2F = 0x00000744
```

The device accepted the values and they survived until a physical preset change overwrote them.

---

# 10. Premium AutoEQ

A five-band approximation of the Premium tuning was also written directly to the DSP.

```text
0x26 = 0x00570009
0x27 = 0x000002EE

0x28 = 0x00CEFFF0
0x29 = 0x00000292

0x2A = 0x04D10010
0x2B = 0x000001F4

0x2C = 0x08900019
0x2D = 0x000002BC

0x2E = 0x36DC000F
0x2F = 0x00000334
```

The device accepted these values and read-back confirmed them.

This was the first time I had a custom AutoEQ-derived tuning actually running **inside the DAC's own DSP**, rather than applying EQ in Windows, Android, Poweramp, Wavelet, etc.

---

# 11. Digital gain

Another interesting register is:

```text
0x66
```

For the single-DAC model, the documented encoding uses the digital DAC gain.

Factory modes mostly contained:

```text
0x66 = 0x000002F4
```

which corresponds to the device's digital gain representation.

Purple was different:

```text
0x66 = 0x00000200
```

This was intentionally left untouched during experimentation.

---

# 12. Noise Gate and Limiter

The device also exposes dynamic-processing registers.

### Noise Gate

```text
0x71
0x72
0x73
```

### Limiter

```text
0x78
0x79
```

The KTMicro protocol documentation indicates that these are live DRC controls.

The device currently reports:

```text
0x71 = 0x3CE2DAD3
0x72 = 0x0014000A
0x73 = 0x01F4000A

0x78 = 0x000000FD
0x79 = 0x012C000A
```

These have not been modified during the EQ experiments.

---

# 13. HID button investigation

The physical DSP button was monitored directly through the HID interface.

Normal media buttons generated reports such as:

```text
01 04 00 00 00 00 00 00 00 00 00
01 00 00 00 00 00 00 00 00 00 00
```

But pressing the physical DSP/EQ button produced no corresponding normal HID input report.

This supports the idea that the preset button is handled internally by the device firmware rather than being exposed to Windows as a normal programmable HID button.

---

# 14. USB capture

USBPcap/Wireshark captures were also examined.

The capture was dominated by continuous USB Audio isochronous OUT traffic.

No obvious separate HID/control transfer corresponding to the physical DSP button was found.

This is consistent with the button being processed internally by the device rather than sending a normal host-visible event.

---

# 15. What has NOT been solved yet

The biggest remaining mystery is persistence.

We can currently do:

```text
Computer
   ↓
HID
   ↓
KTMicro register protocol
   ↓
DSP RAM
   ↓
custom EQ
```

But:

```text
Physical preset button
   ↓
firmware
   ↓
factory preset
   ↓
DSP RAM
```

overwrites the custom EQ.

The public KT02H20 ecosystem contains other devices with a separate persistent-save mechanism, but that protocol is different from the `4B/52/57` protocol used by this KZ. Therefore its save command has **not** been blindly applied to this device.

The KZ-specific persistent preset storage location and/or save mechanism remain under investigation.

---

# 16. Extended address space

The public protocol also describes an extended-read command (`0x08`) for addresses above `0x100`.

An initial experiment against the KZ `31B2:0311` caused an audible device beep/reconfiguration response, so the experiment was stopped.

No further extended-space probing has been performed.

This is deliberately left unresolved rather than repeatedly sending undocumented commands to the device.

---

# 17. Current state of the reverse engineering

### Confirmed

* KZ Duonic Type-C DSP is a KTMicro-based device.
* Device identifies as `KZ Acoustics M2`.
* VID/PID: `31B2:0311`.
* HID interface 3 is used for vendor-specific control.
* `4B/52/57` register protocol works on this KZ.
* DSP registers `0x00–0xFF` can be read.
* DAC EQ is enabled at `0x24`.
* Five DAC PEQ bands occupy `0x26–0x2F`.
* Frequency, gain, Q and filter type encoding is understood.
* Factory Green/Red/Blue/Purple presets have been decoded.
* Physical preset switching changes the actual DSP coefficients.
* Custom EQ coefficients can be written directly.
* Written coefficients read back correctly.
* Custom EQ produces audible changes.
* Physical preset switching overwrites custom EQ values.
* The physical DSP button does not appear as a normal HID input event.

### Not yet confirmed

* Exact physical chip marking.
* Location of factory preset tables.
* Persistent custom-preset storage mechanism.
* Safe persistence/save command for this exact KZ firmware.
* Whether firmware itself can be dumped safely from the KZ run-mode device.

---

# 18. Important warning

This project involves writing directly to audio-DSP registers.

Some experiments caused audible crackling and one test temporarily caused one side of the IEM output to stop until the cable was disconnected.

The device recovered after reconnecting.

Therefore:

**Do not blindly copy register values or undocumented commands from another KT02H20 device.**

KT02H20-family devices can use different USB protocols, firmware, register behavior and bootloader configurations.

Always back up the complete register state before experimenting.

---

# References

* `gxcreator/ktmicro-tools` — KTMicro KT02H20 register protocol and register map.
* `ParkWardRR/fiio-ja11-jcally-jm12-moondropkt02-kt02h20-dac-amp-control` — independent reverse engineering of another KT02H20-family runtime protocol.
* `Ircama/ja11-config` — KT02H20 PEQ and persistent flash work on compatible devices.

This document describes observations made specifically on a KZ Duonic Type-C DSP cable and should not be interpreted as proof that every KTMicro KT02H20 device behaves identically.
