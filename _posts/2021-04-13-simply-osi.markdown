---
layout: post
title:  "Simply OSI"
date:   2021-04-13 16:21:21 -0400
categories: [Networking]
tags: networking
permalink: /networking
---

One of the most important concepts in networking is understanding the _'life of a packet'_. Whether you are in software development or the infrastructure data center side of things, at some point you'll be interacting with the OSI stack.  

## **What is the OSI Model**
**OSI** stands for Open Systems Interconnection model. If you are curious about the history, check out the [wiki](https://en.wikipedia.org/wiki/OSI_model#History).

The OSI layer is the universal language for computer networking and it is based on the concept of splitting up a communication system into seven abstract layers stacked upon each other.

![Image](/assets/images/osimodel.png)

The OSI model is useful for troubleshooting network problems and helps break down the issue one layer at a time rather than diving into to the larger scope of the network.

As data travels through the OSI model, it is also important to note what a [PDU](https://en.wikipedia.org/wiki/Protocol_data_unit) (Protocol Data Unit) is. A PDU is a specific block of information transferred over a network. 

From top to bottom, here are the seven layers. 

## **7. The Application Layer**

This layer interacts directly with data from the user. Software applications like web browsers and email clients rely on this layer to initiate communications. 
> **To be clear, client software applications are not part of the application layer, rather it is responsible for the protocols and data manipulation that the software relies on to present meaningful data to the user.**
 
Some of the most common application layer protocols are [HTTP](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol) _(Hypertext Transfer Protocol - allows users to communicate data on the World Wide Web)_ and [SMTP](https://en.wikipedia.org/wiki/Simple_Mail_Transfer_Protocol#:~:text=The%20Simple%20Mail%20Transfer%20Protocol,send%20and%20receive%20mail%20messages.) _(Simple Mail Transfer Protocols - enables email communications)_

## **6. The Presentation Layer**

This layer is responsible for preparing data to be used by the application layer along with the translation, encryption, decryption, and compression of data. 
> **A good example of this would be accessing your bank account via the Internet, where you used a secure connection provided by the presentation layer and also encrypted your account login information prior to transmission. On the financial institution's internet server, the presentation layer decrypts your account login information for processing.**

## **5. The Session Layer**

This layer is responsible for opening and closing communication lines between end-user application processes. The time between when the communication is opened and closed is known as the **_session_**.
> **It allows logical linking of two software application processes and allows them to exchange data over a preiod of time. A good analogy would be establish a telephone call between two people.** 

## **4. The Transport Layer**

Layer 4 is responsible for end-to-end communication between two devices. This includes taking data from the session layer and breaking it up into chunks called **_segments_** prior to sending it down to Layer 3 (Network Layer). 
> **A good example would be to associate this layer with the  Transmission Control Protocol (TCP) and User Datagram Protocol (UDP), built on top of the Internet Protocol TCP/IP which work at Layer 3 (Network Layer)**

## **3. The Network Layer**

This layer is responsible for logical device addressing. At this layer an IP header is encapsulated to the information being transferred, which at this point it is now called a **_packet_**. 

>**Encapsulation - the process in which some extra information is added to the data item to add some features to it.**

The IP header contains source and destination IP addresses and these are used by a Layer 3 device (router) in order to "route" packets to the correct network where the destination machine resides. 

## **2. The Data Link Layer**

This layer facilitates data transfer between two devices on the same network. It takes packets from the network layer and breaks them into smaller pieces called **_frames_**. It also ensures that the data is formatted the correct way and takes care of error detection to make sure that the data is delivered reliably. 
> **The Ethernet protocol uses MAC addresses to identify unique devices on the network. To do this, switches keep a MAC address table mapping them to individual switch ports.**

## **1. The Physical Layer**

This layer includes the physical equipment involved in the data transfer, such as cables and switches. This layer is where the data gets converted into a **_bit stream_**, which are strings of 1s and 0s. 
> **The physical layer provides actual connections between devices such as Ethernet and fiber optic cables.**