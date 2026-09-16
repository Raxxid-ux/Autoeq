<div align="center">

# 📦 KZ Duonic DSP-Compensated AutoEQ Package

![KZ Duonic Type-C Toning Graph](https://raw.githubusercontent.com/Raxxid-ux/Autoeq/main/KZ%20Duonic%20Optimized%20For%20Type%20C%20Dsp/KZ_Duonic_TypeC_Toning_Graph.png)

</div>

---

## 🎯 Purpose
These profiles are **correction EQs** intended to be applied *after* the KZ Duonic's factory DSP preset for the selected color.  
They are not standalone AutoEQ target curves.

---

## 🧮 Method
The factory color EQ profiles were extracted directly from the decoded KZ Duonic hardware registers:
- **Green**
- **Red**
- **Blue**
- **Purple** *(treated as read-only reference)*

The frequency-response calculations use standard Robert Bristow-Johnson (RBJ) biquad mathematics at **48 kHz**.  
*(Note: 48 kHz is a modelling assumption; KZ's internal sample rate has not been independently verified).*

---

## 🌊 Wavelet
The GraphicEQ profiles are direct sampled representations of the correction curve at standard template frequencies.

---

## 🔊 Poweramp
10-band parametric approximation:
- 1 Low Shelf
- 8 Peaking
- 1 High Shelf

Filter center frequencies are locked to a practical 10-band template, with gain and Q optimized against the target correction curve.

---

## 🎚️ PEQ
5-band parametric approximation:
- 1 Low Shelf
- 3 Peaking
- 1 High Shelf

Filter center frequencies are fixed to a practical 5-band template, with gain and Q optimized against the target correction curve.

---

## ⚡ Preamp
The original target AutoEQ preamp values are retained to preserve digital headroom and prevent clipping. No additional compensation gain is added.

---

## ❗ Important
- **Do not stack** these correction profiles on top of one another.
- Choose the folder matching the **active physical KZ LED color** and your desired tuning target.

---

## 🎨 Factory Color EQ Reference

| Color | Filter Specifications |
| :--- | :--- |
| **Green** | `15 Hz -2 dB Q1` \| `100 Hz +1 dB Q2` \| `200 Hz +1 dB Q2` \| `1000 Hz +1 dB Q3` \| `2500 Hz +1 dB Q2` |
| **Red** | `15 Hz -3 dB Q1` \| `150 Hz +2 dB Q2` \| `1000 Hz +2 dB Q2` \| `2500 Hz +2 dB Q2` \| `3000 Hz +1 dB Q2` |
| **Blue** | `15 Hz -1 dB Q2` \| `50 Hz +2 dB Q2` \| `150 Hz +2 dB Q2` \| `1500 Hz -1 dB Q2` \| `5000 Hz 0 dB Q0.707` |
| **Purple** | `200 Hz +1 dB Q2` \| `3000 Hz +1 dB Q2` \| `5000 Hz +1 dB Q3` \| `1500 Hz 0 dB Q0.707` \| `2500 Hz 0 dB Q0.707` |

---

## 📂 Package Contents
This package contains **24 combinations**:  
**4 Colors × 6 User Tunings**
