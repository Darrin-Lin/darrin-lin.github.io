---
title: Linux Block layer
date: 2025-10-20 0:00:00.000000000 +0800 CST
tags: [Linux Kernel, Block layer]
categories: [Linux Kernel]
math: true
---
## Block layer
Block layer is an interface that provides a 

## software queue
Different from the document and paper, the software queue is only used if `q->elevator` is not NULL you can see in [blk-mq.c](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/tree/block/blk-mq.c?h=v6.19.10#n2654) 

### elevator_switch
## Reference
* [Linux Block layer](https://www.kernel.org/doc/html/v6.4/block/blk-mq.html)
* Linux block IO: introducing multi-queue SSD access on multi-core systems(https://kernel.dk/blk-mq.pdf)