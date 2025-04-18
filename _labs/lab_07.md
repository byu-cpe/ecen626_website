---
title: LoRa Modulation
number: 7
---

## Overview
By now, you’ve explored several common modulation schemes used in wireless communication. In this lab, we turn our attention to **LoRa**—a widely adopted modulation scheme used in many Internet of Things (IoT) applications. Along the way, we’ll explore the benefits of spread spectrum techniques in general.

LoRa, short for **"Long Range"**, was developed in 2014 by the French company Cycleo (later acquired by U.S.-based Semtech). LoRa employs a spread spectrum modulation technique known as **Chirp Spread Spectrum (CSS)**. This lab focuses exclusively on the **LoRa physical layer (LoRaPHY)**—even though the full LoRa protocol also includes a MAC layer called **LoRaWAN**.

---

### Why Spread Spectrum?

Spread spectrum modulation techniques allow signals to be transmitted successfully even in very low signal-to-noise ratio (SNR) environments—well below the noise floor. This is achieved by spreading the signal energy over the entire symbol duration. While individual samples may not have a clearly detectable signal, the **accumulated energy over time** enables the signal to be recovered reliably.

---

### Understanding Chirp Spread Spectrum (CSS)

At the core of the LoRa PHY is **chirp spread spectrum modulation**. A *chirp* is a signal whose frequency increases or decreases over time in a known, deterministic way. Chirps have been extensively studied in radar and sonar applications, and LoRa benefits from this body of knowledge.

LoRa uses **linear chirps** exclusively, which can either be:
- **Up chirps**: frequency increases over time
- **Down chirps**: frequency decreases over time

<figure class="image mx-auto" style="max-width: 750px">
  <img src="{% link /assets/lab_07/up_and_down_chirps.png %}" style="display: block; margin: auto;">
  <figcaption style="text-align: center;"><strong></strong> Up and Down Chirps in Time and Spectrogram Domains</figcaption>
</figure>

Instead of visualizing chirps in the frequency domain, we often use a spectrogram, which plots time on the x-axis and frequency on the y-axis as chirp change in frequnecy as a function of time. We define $f_0$ as the starting frequency and $f_1$ as the ending frequency of the chirp. One powerful feature of linear chirps is the ability to apply a **modulo operation** in the time domain. Instead of starting a chirp at $f_0$, we can start it at an arbitrary frequency $f_i$, increase linearly until reaching $f_1$, wrap around to $f_0$, and continue until ending at $f_i$. This results in a **cyclically shifted chirp**.

<figure class="image mx-auto" style="max-width: 750px; height: auto;">
    <img src="{% link assets/lab_07/modulo_chirp.png %}" style="display: block; margin: auto; height: 400px;">
    <figcaption style="text-align: center;"><strong></strong> Modulo Operation Performed on a Chirp</figcaption>
</figure>

In this lab, we simulate this behavior digitally using NumPy’s `np.roll` function to shift arrays. In a physical LoRa radio, this shift is accomplished via **intentional aliasing** in the analog domain.

This cyclic shifting property plays a critical role in the **demodulation process**, which we will explore in this lab.

---

### Spreading Factor (SF)

A key parameter in LoRa is the **Spreading Factor (SF)**. It controls:
- The duration of each chirp (symbol time)
- The number of possible symbols encoded in a single chirp

A chirp can encode $2^{\text{SF}}$ different symbols.

---

### Demodulation Overview

Here’s how demodulation works in LoRa:

1. A symbol is encoded by **cyclically shifting** an up chirp in time. The amount of shift corresponds to the symbol value $i$ out of $2^{\text{SF}}$ possible values.
2. This modulated chirp is **up-converted** to the carrier frequency (915 MHz in the US) and transmitted.
3. At the receiver, the signal is **down-converted** back to baseband.
4. The received signal is then **multiplied by a standard down chirp**.
   - When two signals are multiplied, their frequencies are summed.
   - The result of multiplying a modulated up chirp with a down chirp is a signal with a **constant frequency** corresponding to the symbol value. (This constant frequency value switches to a negative one after the modulo shift so the total resultant symbol can be found by combining these energies.)
   - At the point of the cyclic shift, the frequency jumps—resulting in clear spikes in the frequency domain.
5. Taking the **FFT** and analyzing the magnitude reveals the symbol index.

If you've chosen the correct number of samples $n$, the index(s) of the spectral peak corresponds to the transmitted symbol.

<video controls style="display: block; margin: auto; max-width: 750px;">
    <source src="{% link /assets/lab_07/LoRaChirpScene.mp4 %}" type="video/mp4">
   
</video>
<figcaption style="text-align: center;">Chirp Modulation and Demodulation Animation</figcaption>

### LoRa Packet Structure

<figure class="image mx-auto" style="max-width: 750px; height: auto;">
    <img src="{% link assets/lab_07/LoRa_Packet.png %}" style="display: block; margin: auto; height: 400px;">
    <figcaption style="text-align: center;"><strong>LoRa Packet Structure</strong></figcaption>
</figure>

LoRa is optimized for long-range, low-power applications—common in IoT networks. To achieve this, LoRaPHY packets are structured as follows:

1. **Preamble:**
   A series of repeated up chirps to aid in signal detection and synchronization.
2. **Sync Down Chirps:**
   A few down chirps are appended to fine-tune synchronization and correct fractional symbol offsets.
3. **Payload:**
   The actual message data.
4. **CRC:**
   Used to verify data integrity.




<figure class="image mx-auto" style="max-width: 750px; height: auto;">
    <img src="{% link assets/lab_07/Preamble.png %}" style="display: block; margin: auto;">
    <figcaption style="text-align: center;"><strong>LoRa Preamble</strong></figcaption>
</figure>

The packet begins with repeated up chirps as a preamble (see figure above). This is used for both signal acquisition and synchronization. A module is constantly demodulating incoming signals, listening for possible chirp symbols. As you can see, no matter the timing, the receiver will always see a repeated symbol. This wakes up the rest of the radio.

Because the timing is not likely to be synchronized, the repeated symbol read in will not be 0, instead it will be some value, which corresponds to how many samples the receiver needs to correct timing by to synchronize. The preamble ends with a few down chirps to synchronize to a point between two symbol indexes (i.e. a fractional offset). After this, the packet contains the payload followed by a CRC for the payload.

---

### Gray Coding in LoRaPHY

The LoRaPHY also makes use of Gray codes, which is a remapping of binary values such that an integer increase of 1 corresponds to only 1 bit flip. Because LoRa is unlike other modulation schemes in how it encodes symbols, the most common misinterpretation of a symbol is by 1 integer due to clock misalignment. Gray coding allows for these errors to only impact the received message by 1 bit which can be easily managed by forward error correction codes.

| Decimal | Binary | Gray Code |
| ------- | ------ | --------- |
| 0       | 000    | 000       |
| 1       | 001    | 001       |
| 2       | 010    | 011       |
| 3       | 011    | 010       |
| 4       | 100    | 110       |
| 5       | 101    | 111       |
| 6       | 110    | 101       |
| 7       | 111    | 100       |

---

## Objectives

In this lab, you will explore the fundamentals of LoRa modulation by implementing and analyzing the physical layer behavior of chirp spread spectrum (CSS). You will develop Python functions to generate and demodulate LoRa chirps, examine how data is encoded via cyclic shifts, and assess the modulation scheme's resilience to noise.

By the end of the lab, you will be able to:

- Generate up and down chirps using configurable parameters such as sample rate, bandwidth, spreading factor, and symbol index.
- Apply cyclic time-domain shifts (modulo operations) to simulate symbol encoding in LoRa.
- Implement a basic demodulator that identifies encoded symbols using down chirp multiplication and frequency-domain analysis.
- Decode a hidden message from a sequence of received chirps using your demodulator.
- Analyze how additive white Gaussian noise (AWGN) affects LoRa symbol detection and determine the system’s performance at low SNR levels.
- Understand the LoRaPHY packet structure, including the use of preambles and synchronization techniques.
- Use GNU Radio Companion to construct a working LoRa transmitter and receiver flowgraph using built-in LoRa blocks.

This lab bridges digital signal processing, wireless communication theory, and practical implementation to help you gain a deeper understanding of how LoRa enables low-power, long-range IoT communication.


## Requirements
---

### Step 1: Chirp Generator Function

Write a Python function that generates a NumPy array of complex samples representing a chirp. The function should take a configuration object with the following parameters:
- `Fs`: Sample rate (Hz)
- `BW`: Bandwidth (Hz)
- `F0`: Minimum frequency (Hz)
- `Up`: Boolean flag (True for up chirp, False for down chirp)
- `SF`: Spreading factor

The full function definition should be `generate_symbol(i,config)`.

Use this equation to determine the number of samples $n$ in each chirp:
$n = \left\lfloor \frac{2^{\text{SF}} \cdot F_s}{B_W} \right\rfloor$

Use the `numpy.roll` function to cyclically shift the chirp by the correct number of samples.

Test your chirp generator with provided test code.

---
### Step 2: Chirp Demodulator Function

Create a function `dechirp(signal, config)` that takes a single modulated chirp and returns the symbol index (as an integer) that was encoded.

Test your demodulator function with provided test chirps.

---

### Step 3: Secret Message Challenge

Download and import the provided NumPy array. It contains a sequence of modulated chirps using $\text{SF} = 8$ and a bandwidth of 125 kHz (from $-62.5$ kHz to $+62.5$ kHz) sampled at 1 MHz.

1. Demodulate the chirps into integer symbols. Create a demodulator function `demodulate(signal, config)` that takes a signal of multiple chirp length and returns a numpy array of the resulting symbols
2. Convert the symbols into ASCII characters.
3. Reveal the secret message!
4. Test your demodulator with the provided test code.

[Download secret_message.npy]({% link assets/lab_07/secret_message.npy %})


---

### Step 4: Noise Resilience Test

Using the provided function, add additive white Gaussian noise (AWGN) to your signal. Gradually reduce the SNR until the message becomes undecipherable.


```python
import numpy as np

def add_awgn(signal, snr_dB):
   """
   Add Additive White Gaussian Noise (AWGN) to a signal.

   Parameters:
   - signal: NumPy array of the input signal
   - snr_dB: Signal-to-Noise Ratio in decibels (dB)

   Returns:
   - noisy_signal: NumPy array of the signal with added noise
   """
   signal_power = np.sum(signal ** 2) / len(signal)
   noise_power = signal_power / (10 ** (snr_dB / 10))
   noise = np.sqrt(noise_power) * np.random.randn(len(signal))
   noisy_signal = signal + noise
   
   return noisy_signal
```



**Question:** What is the lowest SNR (in dB) at which the message is still intelligible? You can also experiment and find that as the spreading factor increases, LoRa becomes more resilient to noise!

---

### Step 5: Stream LoRa with GNU Radio Companion

LoRa transmit (TX) and receive (RX) blocks are included in the Radioconda distribution. These blocks handle the full LoRaPHY stack—including packet formatting, synchronization, and chirp modulation/demodulation.

Build a GNU Radio flowgraph to stream LoRa packets, using blocks similar to the example below. Instead of using a channel model, implement this using pluto radio blocks.

<figure class="image mx-auto" style="max-width: 750px; height: auto;">
   <img src="{% link assets/lab_07/lora_flow.png %}" style="display: block; margin: auto;">
   <figcaption style="text-align: center;"><strong>Example GNU Radio Flowgraph for LoRa Transmission and Reception</strong></figcaption>
</figure>

Once complete, you’ll be able to transmit and receive real LoRa packets, taking advantage of the resilient modulation scheme you’ve just implemented and explored.


## Testing
Upload your code here for testing, specifying which step (1-3) you test.

## Resources
- [Chirp Spread Spectrum](https://en.wikipedia.org/wiki/Chirp_spread_spectrum)
- [One of the first papers to explore the LoRa modulation scheme](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=8067462)
- [Video that reverse engineers the entire LoRaPHY](https://www.youtube.com/watch?v=NoquBA7IMNc)
- [BYU Research Explaining CSS](https://netlab.byu.edu/assets/WUWNet-2024.pdf)
