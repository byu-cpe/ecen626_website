---
title: AM and SSB Modulation
number: 2
---

<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>

## Overview
In the last lab you learned one of the most popular modulation techniques with the frequency modulating radio. In this lab you will learn the second most popular audio modulation technique, amplitude modulation, as well as learn the techniques and concepts fundamental to all radio signals.

## Objectives
After completing this lab you should be able to explain
* How to make an amplitude modulating radio
* What an analytical signal is
* What makes a signal real-valued vs imaginary
* What the Hilbert transform does
* The benefits of single-sideband modulation

## Requirements
### AM Radio
Making an AM radio in Gnuradio may be even easier than the FM radio from the previous lab.
1. Download any medium-length song and convert it to a WAV file at a sample rate of 32 kHz
1. Make the following flow:

![AM radio flow]({% link assets/lab_02/am-radio-flow.png %})

* You may have to run the flow at a higher sample rate than the WAV file. Add a resampler after
    the WAV source and decimate at the end with the BPF. Interpolation will interpolate 31 samples
    between each source sample in this case.
* Add a multiply const that multiplies the input sound by `gain`
* Add a constant 1 to add in the pilot signal. This will be filtered by the band-pass filter at the end.
* Multiply by the carrier signal centered at `cf`
* Your channel model should use `1/10**(SNRdB/20)` to calculate the noise voltage. Make `SNRdB` a
  GUI slider so you can mess with this value.
* The frequency translating filter will bring your signal to baseband as well as apply a FIR filter.
  You can use `firdes.low_pass` to design the filter taps. I found a cut-off frequency of 6000Hz and a
  transition of 2000Hz worked well for my input.
* The band pass filter cleans up the audio and removes the pilot signal

Test that your signal can be heard and play with the effects of noise on your signal. You should be
able to hear your original signal well at 50dB SNR (albeit with a bit of compression). Notice how the
signal looks as it passes through the different stages of filtering with the waterfall plots.

### Analytical Signals and the Hilbert Transform
Without going into too much math, the Hilbert transform induces a +90° phase shift to negative
frequencies and a -90° phase shift to the positive frequencies. This is
represented as:
$${\displaystyle \sigma _{\operatorname {H} }(\omega )={\begin{cases}~~i=e^{+i\pi /2}&{\text{if }}\omega <0\\~~0&{\text{if }}\omega =0\\-i=e^{-i\pi /2}&{\text{if }}\omega >0\end{cases}}}$$

In our case we instead want to focus
on

$$j*\operatorname{H}(u)(t) = {\begin{cases}~~-u(t)&{\text{if }}\omega <0\\~~u(t)&{\text{if }}\omega
>0\end{cases}}$$

Notice how if we were to take our original signal x(t) and add it to j * the Hilbert transform of our
signal

$$x(t) + j\hat{x}(t)$$

we would effectively zero out all negative frequencies and be left with only the positive
frequencies. This is called the analytical signal. If we then take the real value of this signal it
would give us the original signal.

$$\operatorname{Re}\{x(t) + j\hat{x}(t)\} = x(t)$$

Remember that real-valued signals have conjugate symmetry, that is

$$X(−\omega)=X^∗(\omega)$$

### Single-Sideband Modulation
We can exploit this fact to make our AM radio more bandwidth efficient.

![SSB derivation]({% link assets/lab_02/ssb-derivation.png %})

*Image source: [Wikipedia - Single-sideband modulation](https://en.wikipedia.org/wiki/Single-sideband_modulation)*

1. Measure the bandwidth of your AM radio
2. Modify your AM radio to only transmit one of the sidebands
    * After resampling, convert your float into a complex number
    * The Hilbert transform block does NOT give you the Hilbert transform, it's the analytical
      signal. So, you will only need this block to give you the upper sideband on the transmit chain.
    * Modify the filters to better accommodate a single-sideband modulation and take the real value of the signal to
      mirror the positive frequencies to the negative frequencies.

Notice how the bandwidth is halved by only sending half of the modulated signal. Do you notice a
drop in audio quality?

## Resources
* The Wikipedia articles on the [Hilbert transform](https://en.wikipedia.org/wiki/Hilbert_transform)
  and [SSB Modulation](https://en.wikipedia.org/wiki/Single-sideband_modulation)
* Gnuradio [Frequency Xlating FIR
  Filter](https://wiki.gnuradio.org/index.php/Frequency_Xlating_FIR_Filter)
* Gnuradio [SSB example](https://wiki.gnuradio.org/index.php/Simulation_example:_Single_Sideband_transceiver)
