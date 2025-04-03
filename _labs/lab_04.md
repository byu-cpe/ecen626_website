---
title: Error Mitigation
number: 4
---

## Overview

In this lab, we will explore techniques to mitigate errors in communication systems. We will focus on interleaving, whitening, and forward error correction (FEC) to enhance the reliability of data transmission.

Determing the type of forward error correction (FEC) you need for your transmission is largely dependent on the characteristics of the channel and the nature of the data being transmitted. Different channels introduce different types of errors, and understanding these can help in selecting the appropriate FEC method.

For this lab we will **assume a channel where errors are bursty in nature, meaning that errors tend to occur in clusters rather than being randomly distributed.** This is common in many real-world scenarios, such as when a signal is partially blocked or interfered with by obstacles.

### Noisy Channels

In real-world communication systems, signals often encounter various types of noise and interference. This can lead to bit errors, where the received data does not match the transmitted data. Understanding how to mitigate these errors is crucial for maintaining data integrity.

To illustrate this, we consider a message `hello` that we will transmit through a noisy channel. The goal is to ensure that the message is received correctly despite potential errors introduced by the channel. However, as is true with most communication systems, we cannot guarantee that the message will be received without errors due to the presence of noise. Noise can come in the form of random bit flips, burst errors, or other types of interference. What often results is a received message that is different from the transmitted message.

<figure class="image mx-auto" style="max-width: 100%">
  <img src="{% link assets/err_mitigation/error_mitigation_01.svg %}" alt="A message getting sent through a noisy medium which flips bits and alters the intended payload.">
</figure>

### Forward Error Correction

#### Repetition Codes

There are many methods that can be used to send additional information along with the original method to help the receiver correct errors. For example, a repetition code sends the bits multiple times, and the receiver can use a majority vote to determine the correct bit.

<figure class="image mx-auto" style="max-width: 100%">
  <img src="{% link assets/err_mitigation/repetition_codes.svg %}" alt="A message getting sent through a noisy medium which flips bits and alters the intended payload.">
</figure>

However, this method has a low **code rate**, or ratio of data bits to total bits sent. Specifically in this example, **the code rate is 1/3**, meaning that we send an **extra 80 bits** for the message `hello`. This scales very poorly and is not ideal for large messages. Furthermore, if the noise is too high, the only solution is to increase the number of repetitions, which further reduces the code rate. For most applications, increasing the transmission power is just as reliable as increasing the number of repetitions, but it is not always possible due to regulatory limits or power constraints. Therefore, repetition codes are not preferred for most practical applications.

#### Hamming Codes

Another method is to use **Hamming codes**, which are a type of **forward error correction (FEC)** code. Hamming codes are a family of linear error-correcting codes that can detect and correct single-bit errors or detect two-bit errors. Named after Richard Hamming, who introduced them in 1950, these codes (as well as all other FEC codes) take a message and project the message into a higher-dimensional space. This allows the receiver to correct errors by finding the closest valid codeword to the received message. For a good explanation of this concept, see [this video](https://www.youtube.com/watch?v=X8jsijhllIA) and [its sequel](https://www.youtube.com/watch?v=b3NxrZOu_CE).

We define the Hamming to be an algorithm that does the following: we take a $$k$$-bit message and encodes it into a $$n$$-bit codeword with $$r$$ parity bits such that:

$$
n = k + r
$$

The number of $$r$$ parity bits is determined by the following inequality, which ensures that the code can correct single-bit errors:

$$
2^r \geq k + r + 1
$$

where each parity bit is placed at positions at positions of $$2^i$$ meaning that $$ 0 \leq i \lt r $$. This means that the parity bits are located at positions 1, 2, 4, 8, etc. in the codeword.

For example, a Hamming(7,4) code has 7 bits total, 4 of which are data bits and 3 of which are parity bits. The parity bits are calculated such that the codeword has a specific Hamming distance, which is the minimum number of bit flips required to change one codeword into another. This allows the receiver to correct errors by finding the closest valid codeword to the received message.

To determine the parity bits $$P_i$$, we can use the following expression:

$$
P_i = \bigoplus_{j \in C_i} D_j
$$

where $$C_i$$, the set that includes all indices of the data bits defined as:

$$
C_i = \{ j \mid (j \land 2^i) \neq 0, \quad 2^i \lt j \leq n \}
$$

and $$D_j$$ are the data bits.

For example, for the Hamming(7,4) code, a codeword will take the following form:

```python
[P1, P2, D3, P4, D5, D6, D7]
```

The parity bits are calculated as follows:

$$
P_1 = D_3 \oplus D_5 \oplus D_7
$$

$$
P_2 = D_3 \oplus D_6 \oplus D_7
$$

$$
P_4 = D_5 \oplus D_6 \oplus D_7
$$

To decode the message, the receiver can calculate the **syndrome**, or the difference between the received message and the closest valid codeword. The syndrome will be zero if the received message is correct, and the receiver can correct the message by flipping the bit at the position indicated by the syndrome.

$$
S_i = P_i \oplus \left( \bigoplus_{j \in C_i} D_j \right)
$$

To continue with our Hamming(7, 4) example, we would determine the syndrome by calculating the following:

$$
S_1 = P_1 \oplus D_3 \oplus D_5 \oplus D_7
$$

$$
S_2 = P_2 \oplus D_3 \oplus D_6 \oplus D_7
$$

$$
S_4 = P_4 \oplus D_5 \oplus D_6 \oplus D_7
$$

Finally, we take the syndrome and put it in a vector such that $$S = (S_1, S_2, ... S_r)$$. Then an error $$E$$, as defined by:

$$
E = \bigoplus_{i=1}^{r} S_i
$$

will evaluate to 0 ($$E = 0$$), indicating that the message has no errors. However, if $$ E \neq 0 $$, then the syndrome is non-zero, indicating that there is an error in the received message. The position of the error can be determined by interpreting the syndrome $$S$$ as a binary number.

<figure class="image mx-auto" style="max-width: 100%">
  <img src="{% link assets/err_mitigation/hamming_codes.svg %}" alt="A message getting sent through a noisy medium which flips bits and alters the intended payload.">
</figure>

The **Hamming distance**, as defined by:

$$
d = n - k + 1
$$

describes the minimum number of bit flips required to change one valid codeword into another. For Hamming codes, the relationship between the minimum distance and error correction capabilities is as follows:

$$
\text{Error Correction Capability} = \left\lfloor \frac{d - 1}{2} \right\rfloor
$$

with a code rate defined as:

$$
\text{Code Rate} = \frac{k}{n}
$$

This demonstrates that as the minimum Hamming distance increases, the error correction capability also increases, though this often comes at the cost of a lower code rate. These codes are best suited for channels where single-bit errors are common, such as in memory storage or communication systems with low noise levels.

For bursty errors, Hamming codes may not be sufficient, and more advanced techniques like [Reed-Solomon](https://en.wikipedia.org/wiki/Reed%E2%80%93Solomon_error_correction) codes may be required.

### Interleaving
While Hamming codes are effective for correcting single-bit errors, they may not perform well in the presence of burst errors, where multiple consecutive bits are corrupted.

<figure class="image mx-auto" style="max-width: 100%">
  <img src="{% link assets/err_mitigation/interleave_one.svg %}" alt="A message getting sent through a noisy medium which flips bits and alters the intended payload.">
</figure>

One effective technique to mitigate burst errors is **interleaving**. Interleaving rearranges the order of the bits in the transmitted data, spreading out the bits across multiple codewords. This means that if a burst error occurs, it is less likely to affect consecutive bits in the same codeword, allowing for better error correction. To interleave data, we can use a simple matrix-based approach. The data is arranged in a matrix with a specified number of rows and columns, and then the bits are read out column-wise instead of row-wise. This effectively spreads the bits across the codewords.

<figure class="image mx-auto" style="max-width: 100%">
  <img src="{% link assets/err_mitigation/interleave_two.svg %}" alt="A message getting sent through a noisy medium which flips bits and alters the intended payload.">
</figure>

Upon receive a burst error, the interleaved data can be deinterleaved by reversing the process. The receiver reads the bits in the same column-wise order to reconstruct the original message. This way, even if a burst error affects multiple bits, they are distributed across different codewords, making it easier for the Hamming code to correct them.

<figure class="image mx-auto" style="max-width: 100%">
  <img src="{% link assets/err_mitigation/interleave_final.svg %}" alt="A message getting sent through a noisy medium which flips bits and alters the intended payload.">
</figure>

### Symbol Distribution and Whitening
While forward error correction techniques like Hamming codes help correct errors, they do not address the issue of symbol distribution in the transmitted data. In many communication systems, especially those using modulation schemes like QAM (Quadrature Amplitude Modulation), it is crucial to ensure that the symbols are evenly distributed to improve the overall performance of the system.

Furthermore, if multiple of the same symbol are transmitted in a row, certain physical phenomena such as DC bias can occur, where the average power of the transmitted signal is not zero and can lead to distortion in the received signal and make it more susceptible to errors. To mitigate this, we use a technique called **whitening**.

Whitening is a process that introduces a pseudo-random sequence to the data before transmission. This sequence is designed to ensure that the symbols are more evenly distributed over time, reducing the likelihood of long runs of the same symbol.

<figure class="image mx-auto" style="max-width: 100%">
  <img src="{% link assets/err_mitigation/whitening_one.svg %}" alt="A message getting sent through a noisy medium which flips bits and alters the intended payload.">
</figure>

Whitening is typically achieved by XORing the data with a pseudo-random sequence generated by a linear feedback shift register ([LFSR](https://en.wikipedia.org/wiki/Linear-feedback_shift_register)) or a similar algorithm. This process effectively randomizes the data, making it less predictable and more robust against errors introduced by the channel.

To dewhiten the received data, the receiver must apply the same pseudo-random sequence used for whitening. This ensures that the original data can be recovered correctly, even if it was transmitted through a noisy channel. The dewhitening process is simply the inverse operation of whitening, where the received data is XORed with the same pseudo-random sequence to retrieve the original message.

<figure class="image mx-auto" style="max-width: 100%">
  <img src="{% link assets/err_mitigation/whitening_recv.svg %}" alt="A message getting sent through a noisy medium which flips bits and alters the intended payload.">
</figure>

Determining the appropriate polynomial for the LFSR used in whitening is crucial. The polynomial must be chosen such that it generates a maximum-length sequence, meaning it will produce a pseudo-random sequence that covers all possible states before repeating. This ensures that the whitening process effectively randomizes the data without introducing patterns that could lead to errors.


## Objectives

1. Improve resistance to burst errors through interleaving.
2. Ensure balanced symbol distribution using whitening.
3. Enhance error recovery with forward error correction techniques.

## Requirements

- Download boilerplate and test archive for the lab [here]().
- Create a program that successfully implements:
  - Interleaving and the general Hamming algorithm (not just a specific hard-coded version of it).
  - **NOT** whitening (we will not worry about LFSR and polynomial selection), furthermore, we really don't get any benefit from it while simulating the transmission in code.
  - Deinterleaving and the corresponding Hamming decoding.
- Put your solutions in the `lab.py` file. You'll notice that there is a function called `gaussian_noise`. This will be used to simulate the transmission channel.
- Run your program using a range of different message lengths and bit error probability rates. Create a few graphs that illustrate the impact of interleaving and different Hamming codes on your transmission. What conclusions can you draw about the effectiveness of these techniques in improving error resilience?
- Test your implementation thoroughly to ensure it meets the specified requirements.

## Testing

## Resources

- [Overview of Error Correction and Detection](https://en.wikipedia.org/wiki/Error_detection_and_correction)
- [Hamming Code](https://en.wikipedia.org/wiki/Hamming_code)
- [Whitening Filters](https://www.allaboutcircuits.com/technical-articles/whitening-filters-help-low-power-radios-tackle-issues-caused-by-long-identical-bit-sequences/)
- [Linear-feedback shift register](https://en.wikipedia.org/wiki/Linear-feedback_shift_register)
