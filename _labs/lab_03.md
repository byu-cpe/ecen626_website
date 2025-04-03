---
title: Lab 3
number: 3
---

## Overview

In the previous lab you were introduced to the basics of GNU radio and began to understand just how capable of a tool an SDR can be. However, the design that you implemented is extremely well known, documented, and proven. It's basically the "hello world" of RF design. So how do we get from a simple "hello world" to designing our own version of wifi?

While you probably won't develop something as complex as wifi, this lab will explore a bit more about designing a wireless communication system. To help you now (and later), we will discuss various decisions to be made when creating a wireless protocol to help you come up with questions and ideas to consider when deciding on your own semester-long protocol design projects. We will then have you practice some GNU radio companion again to better prepare you to create your own flowgraphs for your protocol. Unlike the previous lab, you will implement both the transmitter and receiver this time.

## Objectives

By the end of this lab, you will:

- Understand some key factors involved in designing a PHY layer.
- Implement a QPSK transmitter and receiver in GNU Radio Companion.
- Observe and analyze signal quality and potential sources of degradation.

## Wireless Design

Here is a list of considerations that someone might need to take into account when designing a wireless protocol:

- Speed
- Power
- Time Usage
- Frequency Usage
- Error detection, correction
- throughput
- Multiple Access Contorl (MAC)
- Noise and Ranging
- Syncronization

Speed is one of the most prominent features of a wireless protocol. Crazy maximums like [Wifi-7's theorectical maximum of 46 Gbps](https://www.netgear.com/au/hub/wifi/mesh/wifi-7-speed/) and [Starlink's promise to bring 1 Gbps from LEO orbit](https://www.pcmag.com/news/spacex-preps-new-starlink-dishes-including-one-for-gigabit-speeds) are eye-catching and make the future feel like its really coming quickly. On the other end of the speed spectrum, we find "slow" moving protocols like LoRa's 50 kbps average maximum, and zigbee's 250 kbps. All of these protocols have a purpose, and the speeds are a part of the design.

## Requirements

### Protocol Design

You protocol should have the following properties:

1. Uses QPSK to modulate data.
2. Uses differential encoding.
3. Be shapped with an RRC filter.

### Transmitter Design

Your transmitter flow should:

1. Generate random binary data, or use a file as an input.
2. Apply differential encoding to the data.
3. Modulate the data using QPSK.
4. Shape the signal with an RRC filter.
5. Transmit.

### Receiver Design

The receiver flowgraph will:

1. Receive a signal.
2. multiply down to baseband.
3. Perform carrier synchronization.
4. Demodulate the QPSK symbols.
5. Decode the differential encoding.
6. Extract and display the received data, either live or in a file.

## Testing

1. **Over-the-Air Test**: Transmit using antennas and evaluate performance in different environments.
2. **Bit Error Rate (BER) Measurement**: Compare transmitted and received data to estimate BER.

## Resources

- [GNU Radio Documentation](https://wiki.gnuradio.org)