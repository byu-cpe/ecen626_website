---
title: Lab 8 (MAC Lab)
number: 8
---

## Overview
This lab is about learning how to take a payload message and package that message inside a header. The resultant sequence of bytes is called a packet. Or frame. Depends on who you ask. I'll refer to it as "packet" for this lab.

You will read in some input text from an input file, packetize that data, send it to a server using ZeroMQ sockets, then parse that received packet and strip the header off to write the payload to an output file.

Some source code is provided for you. Begin by downloading that code. You should have 3 .py files (header, client, and server) along with 3 input .txt files (in_small, in_med, in_big) to use for testing your client/server. Within the .py files, you should only need to add code where there are comments "# <your code here>", but you are welcome to modify any of the code you would like or start from scratch. Just be sure that in the end, your client and server are able to communicate with each other using ZMQ and following this lab's provided packet structure.

The packet structure you should implement is shown in the table below.

| SRC      | DST      | LEN     | PAYLOAD       | CRC     |
| :-----:  | :-----:  | :----:  | :----------:  | :----:  |
| 6 bytes  | 6 bytes  | 2 bytes | 0-8192 bytes  | 2 bytes |

## Objectives
1) Packetize a payload of bytes
2) Send/Receive packets using ZMQ sockets (REQ/REP style)
4) Parse a packet's header
5) Strip the header to recover the data

## Required Materials
1) Your own computer to act as a client and server
2) python (I used 3.13.0)
3) ZMQ library; run the following in your virtual environment
```bash
pip install pyzmq
```

SDRs will not be used in this lab.

## header.py
This file contains two classes: one that takes in a payload and constructs header data for it, and another that takes in a packet and deconstructs the header from the payload. This file is where the focus on header construction/deconstruction lies. Further details are found in comments within the .py file. These classes are intended to be used in client.py and server.py to make your code look cleaner.

## client.py
This file focuses on implementing a ZMQ REQ (request) socket. In this file, you should read in payload data from some input .txt file, packetize that payload, then send the packet to a server (which you will implement in server.py). Further details are found in comments within the .py file. The client can be run with the following command:
```bash
python3 client.py -f [INPUT_FILE] -d [DESTINATION_MAC_ADDRESS]
```
You can also get "help" info about proper usage of this command by running:
```bash
python3 client.py -h
```

## server.py
This file also focuses on ZMQ, but instead you will implement a REP (reply) socket. This script should receive packets from client.py, parse those packets to verify their validity and retrieve the payload, then write the payload contents to some output file. Further details are found in comments within the .py file. The server can be run with the following command:
```bash
python3 server.py
```

## Resources
* ZMQ docs:
https://pyzmq.readthedocs.io/en/latest/api/zmq.html
