---
title: Lab 4
number: 4
---

## Overview
In previous labs, you have learned how to use GNU Radio and have seen how it can make some tasks very easy. But there are also some things GNU Radio can’t do. There are also some steps, such as header generation or FEC that could be done with more detail in Python. In these situations, you would need a way to get data in or out of GNU Radio. In this lab, you will learn how to integrate ZeroMQ (ZMQ) with GNU Radio to pass data between different programs. Additionally, you will explore using XMLRPC to modify GNU Radio variables dynamically.

### ZMQ
ZeroMQ is an interprocess messaging library. It allows you to send data between processes through sockets. GNU Radio has several ZMQ blocks that provide different methodologies for communication such as: Publisher-Subscriber, Push-Pull, and Request-Reply. You may have come across these methodologies in other contexts, but you should at least review the diagrams for each method from the links below.

[Publisher-Subscriber](https://learning-0mq-with-pyzmq.readthedocs.io/en/latest/pyzmq/patterns/pubsub.html)  
[Push-Pull](https://learning-0mq-with-pyzmq.readthedocs.io/en/latest/pyzmq/patterns/client_server.html)  
[Request-Reply](https://learning-0mq-with-pyzmq.readthedocs.io/en/latest/pyzmq/patterns/pushpull.html)  

## Task 1
Let’s get started with ZMQ without using GNU Radio. We will be using the publisher-subscriber model. For this task your goal is to read a text file into zmq_pub.py, send it to the subscriber with ZMQ, and write it to output.txt.

- Create zmq_pub.py and zmq_sub.py.
- Aquire a text file to send, or use the declaration of independance from [here](../files/declaration.txt). 
- In zmq-pub.py read in the text file and break it up into chunks of 50 characters.
- Publish each chuck with ZMQ to the topic "text_send"
- Write code in zmq_sub.py to listen for messages from zmq_pub.py, and write them to an output file


If you are working in your radioconda environment, ZMQ python package should already be installed for you. Otherwise, install it using `pip install pyzmq`.

## Task 2
Now lets take GNU Radio and put it in between your publisher and subscriber. Use [this](../files/send_through.py) GNU Radio flow graph . It uses OFDM blocks because they make it easy to send and receive characters. This will be an easy way for us to send text data without getting too far into the weeds of GNU Radio. 

- In the GRC flow graph, remove the vector source block, and replace it with the ZMQ Subscriber block. 
- Try starting the flow graph and then running your task1_pub.py. Take a screenshot of what your GNU Radio GUI looks like while the data from the text file is being sent.
- Add a ZMQ Publisher to the end of the flow graph.
- You will have to update the port numbers and topics used for ZMQ as needed
- Launch your GNU Radio flow and send the entire text file from your publisher to your subscriber.

The idea of ZMQ is that you could add more pre-processing and post-processing work to your python scripts, and use GNU Radio for over the air transmission. 

## Task 3
In task 2, you sent a stream of characters to GNU Radio through ZMQ. But sometimes it is better to use PDUs. A PDU is a Protocol Data Unit and is defined as a pair of metadata and data in GNU Radio. One example of why this could be useful is if you want to allow for a variable packet length. Right now your packet length is set to 50 in GNU Radio, which is why your zmq_sub script should be receiving text in blocks of 50. But putting it in a PDU provides metadata such as packet length so gnu Radio can handle one packet all together. 


- Change two blocks in GNU Radio: the ZMQ sub block and the stream to tagged stream block. 
- Modify your zmq_pub.py to break the text file into variable sized chucks
- Turn your chunks into PDUs before publishing them with ZMQ
    - Use use the ptm library for this process
    - Create a PMT dictionary for metadata
    - Create a PMT u8vector with the message (chunk of text)
    - Use pmt.cons to pair the metadata and message
    - Serialize it the paired PDU and send to with ZMQ



Take another screenshot of your GNU Radio GUI during transmission. This time, you should be able to see that the packet_len tag isn’t always the same value. Submit your zmq_pub.py, zmq_sub.py and both of your screenshots.

## XMLRPC Blocks
Now you know how do pass data in and out of GNU Radio. But what if you want to access GNU Radio variables from another program? That is what the XMLRPC block are for. XMLRPC stands for XML-based Remote Protocol Control. It uses HTTP transport and allows a client to use SET commands to change parameters on a server or use GET commands to obtain the value of parameters on the server. It transmits the parameters in an XML format. You can read the GNU Radio documentation [here](https://wiki.gnuradio.org/index.php/Understanding_XMLRPC_Blocks). That site also links two more sites, which give more information on how XMLRPC works, and how you could create your own clients and servers in other applications (not necessarily GNU Radio).

[What is XMLRPC](https://xmlrpc.com/)  
[Python Documentation](https://docs.python.org/3.8/library/xmlrpc.html)  

## Task 4
Create a simple GNU Radio flow to test out XMLRPC. It can be as simple as you want, or a flow that you have already made. The one requirement is that it must use at least one variable that will cause visible change in the GUI if it is changed in real time. 

- Add an XMLRPC server to your flow
- Create a python script that changes this variable through the XMLRPC server
- Also make your the value of samp_rate from your GNU Radio flow, and print it out.

Make sure that it works by launching your GNU Radio flow and then running your python script. You should see your variable change in real time. Submit your xmlrpc_test.py and your simple GNU Radio flow. 