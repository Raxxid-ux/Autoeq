KZ Duonic DSP-compensated AutoEQ package
===========================================

Purpose
-------
These profiles are CORRECTION EQs intended to be applied AFTER the KZ Duonic's
factory DSP preset for the selected color. They are not the original AutoEQ
targets by themselves.

Method
------
For each color:
    correction(f) = user AutoEQ target(f) - measured/modelled factory DSP(f)

The factory color EQ was taken from the decoded KZ Duonic registers:
Green, Red, Blue, Purple. Purple was treated as read-only/reference.

The frequency-response calculation uses standard RBJ biquad mathematics at
48 kHz. This 48 kHz sample rate is a modelling assumption; the exact KZ
internal sample rate has not been independently verified.

Wavelet
-------
The GraphicEQ is a direct sampled representation of the correction curve at
the supplied template frequencies.

Poweramp
--------
10-band approximation using:
1 Low Shelf + 8 Peaking + 1 High Shelf.
The filter centers are fixed to a practical 10-band template and gain/Q are
optimized against the correction curve.

PEQ
---
5-band approximation using:
1 Low Shelf + 3 Peaking + 1 High Shelf.
The filter centers are fixed to a practical 5-band template and gain/Q are
optimized against the correction curve.

Preamp
-------
The original target AutoEQ preamp is retained. The correction itself is not
added to the preamp because the objective is for:
    factory DSP + correction EQ = user's target EQ response.

Important
---------
Do not stack these correction profiles on top of one another.
Choose the folder matching the active physical KZ color and the desired
tuning.

Factory color EQ reference
--------------------------
Green : 15 Hz -2 dB Q1; 100 Hz +1 dB Q2; 200 Hz +1 dB Q2;
        1000 Hz +1 dB Q3; 2500 Hz +1 dB Q2
Red   : 15 Hz -3 dB Q1; 150 Hz +2 dB Q2; 1000 Hz +2 dB Q2;
        2500 Hz +2 dB Q2; 3000 Hz +1 dB Q2
Blue  : 15 Hz -1 dB Q2; 50 Hz +2 dB Q2; 150 Hz +2 dB Q2;
        1500 Hz -1 dB Q2; 5000 Hz 0 dB Q0.707
Purple: 200 Hz +1 dB Q2; 3000 Hz +1 dB Q2; 5000 Hz +1 dB Q3;
        1500 Hz 0 dB Q0.707; 2500 Hz 0 dB Q0.707

This package contains 24 combinations: 4 colors x 6 user tunings.
