<h1 align="center" style="font-size: 2.5rem; font-weight: 800;">📦 KZ Duonic DSP-Compensated AutoEQ Package</h1>

<p align="center">
  <img src="./KZ%20Duonic%20Optimized%20For%20Type%20C%20Dsp/KZ_Duonic_TypeC_Toning_Graph.png" alt="KZ Duonic Type-C Toning Graph" width="100%">
</p>

---

## 🎯 Purpose
These profiles are **correction EQs** intended to be applied *after* the KZ Duonic's factory DSP preset for the selected color.  
They are not the original AutoEQ targets by themselves.

---

## 🧮 Method
The factory color EQ values were extracted directly from decoded KZ Duonic hardware registers:  
- **Green**
- **Red**
- **Blue**
- **Purple** *(treated as read-only reference)*

The frequency-response calculation uses standard Robert Bristow-Johnson (RBJ) biquad mathematics at **48 kHz**.  
*(Note: 48 kHz is a modelling assumption; KZ’s internal hardware sample rate has not been independently verified).*

---

## 🌊 Wavelet
The GraphicEQ profiles are direct sampled representations of the correction curve at standard template frequencies.

---

## 🔊 Poweramp
10-band parametric approximation:
- **1 Low Shelf**
- **8 Peaking filters**
- **1 High Shelf**

Filter center frequencies are locked to a standard 10-band layout, with gain and Q values optimized precisely against the target correction curve.

---

## 🎚️ PEQ
5-band parametric approximation:
- **1 Low Shelf**
- **3 Peaking filters**
- **1 High Shelf**

Filter centers are fixed to a standard 5-band layout, with gain and Q optimized against the target correction curve.

---

## ⚡ Preamp
The original target AutoEQ preamp values are retained to preserve dynamic range and prevent digital clipping. No additional gain offsets are applied.

---

## ❗ Important
* **Do not stack** these profiles on top of one another.
* Match your profile selection to your **active physical KZ LED color** and your desired target tuning.

---

## 🎨 Factory Color EQ Reference

| Color | Filter Settings |
| :--- | :--- |
| **Green** | `15 Hz -2 dB Q1` \| `100 Hz +1 dB Q2` \| `200 Hz +1 dB Q2` \| `1000 Hz +1 dB Q3` \| `2500 Hz +1 dB Q2` |
| **Red** | `15 Hz -3 dB Q1` \| `150 Hz +2 dB Q2` \| `1000 Hz +2 dB Q2` \| `2500 Hz +2 dB Q2` \| `3000 Hz +1 dB Q2` |
| **Blue** | `15 Hz -1 dB Q2` \| `50 Hz +2 dB Q2` \| `150 Hz +2 dB Q2` \| `1500 Hz -1 dB Q2` \| `5000 Hz 0 dB Q0.707` |
| **Purple** | `200 Hz +1 dB Q2` \| `3000 Hz +1 dB Q2` \| `5000 Hz +1 dB Q3` \| `1500 Hz 0 dB Q0.707` \| `2500 Hz 0 dB Q0.707` |

---

## 📂 Package Contents
This package contains **24 total combinations**:  
**4 Hardware Colors × 6 Target Tunings**
