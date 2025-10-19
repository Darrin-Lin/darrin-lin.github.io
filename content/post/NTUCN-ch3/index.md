---
title: NTU Computer Network Ch.3 Transport Layer
date: 2025-10-20 0:00:00.000000000 +0800 CST
tags: [NTUCSIE, Computer Network]
categories: [NTU CSIE CN]
math: true
---

## transport vs network layer
transport layer 是 process-to-process\
## transport layer protocols
TCP, UDP, ...
### TCP
TCP 有 Source Port, IP 跟 Destination Port, IP
- reliable, in-order delivery
- congestion control
- flow control
- connection setup
#### header
source IP, Dest IP, source port(16 bits), destination port(16 bits), other header fields
### UDP
- unreliable, unordered delivery
- no-frills extension of "best-effort" IP service

## multiplexing and demultiplexing
### multiplexing
會把不同 socket 的
就像高速公路多個閘道匯集到同一條路
### demultiplexing
收到會根據 IP address 跟 port number 送到不同 socket
就是類似高速公路路牌的概念

UDP 一個 port 只會有一個 socket（沒有 connection oriented）\
TCP 一個 port 會有多個 socket（connection oriente，會根據 source IP 去到不同 socket），另外 server 還會有一個額外的 welcome socket

**To be continued...**