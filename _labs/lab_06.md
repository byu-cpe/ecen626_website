---
title: Error Mitigation
number: 6
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

However, this method has a low **code rate**, or ratio of data bits to total bits sent. Specifically in this example, **the code rate is 1/3**, meaning that we send an **extra 80 bits** for the message `hello`. This scales very poorly and is not ideal for large messages. Furthermore, if the noise is too high, the only solution is to increase the number of repetitions, which further reduces the code rate.

#### Hamming Codes

Another method is to use **Hamming codes**, which are a type of **forward error correction (FEC)** code. Hamming codes are a family of linear error-correcting codes that can detect and correct single-bit errors and detect two-bit errors. Named after Richard Hamming, who introduced them in 1950, these codes take a message and project the message into a higher-dimensional space. This allows the receiver to correct errors by finding the closest valid codeword to the received message. For a good explanation of this concept, see [this video](https://www.youtube.com/watch?v=X8jsijhllIA) and [its sequel](https://www.youtube.com/watch?v=b3NxrZOu_CE).

For the purposes of our lab, we will consider the Hamming algorithm to be a black box, which takes a $$k$$-bit message and encodes it into a $$n$$-bit codeword where:

$$
n = 2^m - 1
$$

and

$$
k = n - m
$$

where $$m$$ is the number of parity bits you would need defined by:

$$
2^m \geq k + m + 1
$$

For example, a Hamming(7,4) code has 7 bits total, 4 of which are data bits and 3 of which are parity bits. The parity bits are calculated such that the codeword has a specific Hamming distance, which is the minimum number of bit flips required to change one codeword into another. This allows the receiver to correct errors by finding the closest valid codeword to the received message.

To determine the parity bits, we can use the following expression:

$$
P_i = \bigoplus_{j \in S_i} D_j
$$

where $$S_i$$ can be defined generally as:

$$
S_i = \{j \in \mathbb{N} \mid j \text{ has a 1 in the i-th bit of its binary representation}\}
$$

For example, for the Hamming(7,4) code, a codeword will take the following form:

```python
[P1, P2, D3, P4, D5, D6, D7]
```

The parity bits are calculated as follows:

$$
P_1 = D3 \oplus D5 \oplus D7
$$

<pre>
<s>000</s>  # Ignore this combination
00<u>1</u>  # 1st bit (P1)
010
01<u>1</u>  # 3nd bit (D3)
100
10<u>1</u>  # 5th bit (D5)
110
11<u>1</u>  # 7th bit (D7)
</pre>

$$
P2 = D3 \oplus D6 \oplus D7
$$

<pre>
<s>000</s>  # Ignore this combination
001
010  # 2nd bit (P2)
011  # 3rd bit (D3)
100  
101  # 4th bit
110
111  # 8th bit
</pre>


$$
P4 = D5 \oplus D6 \oplus D7
$$

<pre>
<s>000</s>  # Ignore this combination
001  
010
011  # 2nd bit
100
101  # 4th bit
110
111  # 8th bit
</pre>

To decode the message, the receiver can calculate the **syndrome**, or the difference between the received message and the closest valid codeword. The syndrome will be zero if the received message is correct, and the receiver can correct the message by flipping the bit at the position indicated by the syndrome:

$$
S1 = P1 \oplus D1 \oplus D2 \oplus D3
$$

$$
S2 = P2 \oplus D2 \oplus D3 \oplus D4
$$

$$
S3 = P3 \oplus D1 \oplus D2 \oplus D4
$$

### Synchronization Issues

### Characterizing Error Patterns

## Objectives

1. Improve resistance to burst errors through interleaving.
2. Ensure balanced symbol distribution using whitening.
3. Enhance error recovery with forward error correction techniques.

## Requirements

- Download source files for TX and RX modules [here]().
- Referring to the tutorial [here](), create a block that successfully implements interleaving, whitening, and FEC, specifically [Hamming(7,4)]().
- Run the message `"Hello World!"` through the block and connect the output to a gaussian noise channel with a signal-to-noise ratio (SNR) of 10 dB.
- Create a corresponding receiver block that can decode the received message, effectively correcting any errors with Hamming, dewhitening, and deinterleaving.

## Testing

## Resources

- [Overview of Error Correction and Detection](https://en.wikipedia.org/wiki/Error_detection_and_correction)
- [Hamming Code](https://en.wikipedia.org/wiki/Hamming_code)
-
