---
title: "Analog SDR: FM Radio"
number: 1
---

## Overview

Welcome to your first lab in the Software Defined Radio (SDR) class! In this lab, you are going to setup your desktop to interface with SDRs and create an FM radio receiver (the "Hello World" of SDR). Once that is done, you will take what you have learned form the tutorial and create your own transmitter in the unlicensed band and attempt to receive it.

## Objectives

Become familiar with basic GNU Radio blocks and processes as well as how to use different SDRs.

## Requirements

### GNU Radio

For this and all other labs, you are going to learn about and use GNU Radio. This is a very well-known and recognized software development toolkit that is used for interfacing desktops with software defined radios. There are multiple ways you can install it on your system (even as simple as `sudo apt install gnuradio`), but for this course, we recommend installing [radioconda](https://github.com/ryanvolz/radioconda). You can find installation instructions on the GitHub readme page.

Once you have it installed, make sure that you activate that environment in your terminal (`source <radioconda_location>/bin/activate`) and then run `gnuradio-companion`. You should see GRC popup.

### HackRF Tutorial

In most labs in this course, you can use any SDR available. *For this lab, however,* you must use the HackRF SDR—it is the only one strong enough at the desired frequency to receive FM radio signals.

Once you have gotten your hands on a HackRF and plugged it into your computer, go ahead and pull up [this tutorial](https://greatscottgadgets.com/sdr/1/). You will follow along with it for the first part of your lab.

> **A few important differences!**
>
> There are a few differences between the block diagram provided by the tutorial and the one you will create for this lab. The video timestamps and the changes are listed below:
>
| **Timestamp** | **Video uses**         | **Change to make**                                                                 |
|---------------|-------------------------|------------------------------------------------------------------------------------|
| **8:53**      | *osmocom Source*       | Use the "Soapy HackRF Source" (you can use Ctrl+F to find this and any other blocks mentioned).   |
| **10:05**     | *WX GUI FFT Sink*      | QT GUI Frequency Sink                                                              |
| **12:08**         | *Center frequency to 97.9e6*                    | You can find this property under the "RF Options" tag, labeled as "Center Freq (Hz)"                                                                                 |
| **15:50**         | *`Baseband Freq`*                    | In the QT GUI Frequency Sink, this is "Center Frequency (Hz)"                                                                                  |
| **19:52**         | *`center_freq` and `channel_freq`*                    | Within the engineering building, there are only a few FM radio stations that can you can hear with the SDRs. For this lab, set the `center_freq` variable to 102.8e6 and the `channel_freq` to 103.1e6.                                                                                  |
| **30:15**         | *WX GUI Slider*                    | QT GUI Range block                                                                                  |

You can stop at the 35-minute mark, don't worry about the homework for this lab (though, if you have time, it's pretty fun to dive even deeper).

**Congratulations!** You have successfully created an FM receiver on an SDR!

### FM Transmitting

Now that you can successfully receive an FM signal, you should be ready to generate one with another SDR and receive it with your current Rx setup. 

Create a new file for your transmitter and show that you can speak into a microphone into your transmitting diagram and hear it on a speaker in your receiving diagram. Here's a few things to keep in mind:

- You will need to keep your transmission in the unlicensed spectrum (for this lab, keep your channel between 902 and 928 MHz).
- You should be able to do this with with just 4 blocks—you already have some version of them on your receiver setup (you will need a different SDR block)
- A common quadrature rate for FM transmission is 192k
- A good sample rate for your transmitter is 1 million (rather than the 10 million for your receiver)
- You will need a block to rationalize between these two rates

## Testing

Once you have finished this lab, pass it off with Phil (or a TA, if there is one this semester). Show that you can receive the FM channel 103.1, and then show that you can transmit on one SDR and receive it on your same receiver diagram. Submit these two files.

## Resources

- [`radioconda`](https://github.com/ryanvolz/radioconda) comes with support for GNU Radio and several different SDRs. If you don't use it, be sure you have gone through the necessary installation for these SDRs:
    - [HackRF](https://greatscottgadgets.com/hackrf/one/)
    - [Pluto SDR](https://wiki.analog.com/university/tools/pluto/users/quick_start)
    - [UHD USRP](https://files.ettus.com/manual/page_install.html)
