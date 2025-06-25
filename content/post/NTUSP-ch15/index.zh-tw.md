---
title: NTU SP Chapter 15 - IPC
date: 2024-12-15 0:00:00.000000000 +0800 CST
tags: [NTUCSIE, System Programming]
categories: [NTU CSIE SP]
math: true
---

## Intrerprocess Communication(IPC)

|ICP types|
|---|
|pipes (half duplex)|
|FIFOs|
|Stream pipes(full duplex)|
|named stream pipes|
|message queues|
|semaphores 重要|
|shared memory 重要，最快|
|sockets|
|streams|

### 不同 computer 的通訊
socket\
stream

## UNIX Pipe
可以跨 process\
能用同一個 pipe 的 process 必須要有共同祖先，因為是用 fork 會繼承 fd 的特性\
使用 pipe 會傳送到 kernel 的一塊 string 的空間\
只能一端寫進去，一端讀出來，不能 random acess，可以多個人寫跟讀\
讀完就會被清掉，所以兩 process 不會讀到相同的東西（讀會把東西 pop 掉）\
可以用來在 parent 跟 child 間溝通，把 parent 跟 child 的各自其中要關掉的讀 / 寫關掉，就可以用來當作兩個 process 溝通的橋樑\
當沒有人讀的時候，寫的人就會收到 SIGPIPE 的 signal \
如果寫的人比 pipe 的 size 還要小，寫入的過程一定是 atomic\
不用的要記得 close 掉會浪費資源以及會不小心操作到\
當寫超過 pipe 的 buffer size 會 suspend\
**不把不用的東西 close 掉在作業跟考試會扣分**

### stream pipe
可以雙向的 pipe
### name pipe
給予名字，可以突破不同祖先

### name stream pipe
集合兩個特點


## FIFOs
概念是有名字的 pipe，所以讀完就會清掉\
在讀跟寫沒有人 oepn 就會把資料都清掉
### `int mkfifo(const char *pathname, mode_t mode)`
mode 是在管理 rwx \
會在 pathname 創一個 fifo 檔案\
open 時可以設為 `O_NONBLOCK`
#### 沒有人讀的判斷
在寫入到一個沒有被 open for reading 會收到 `SIGPIPE`\
建議要在送出去的資料前面加入自己 client 的資訊好判斷來源
#### 沒有人寫
會收到 EOF ，所以建議要在讀取是 open for wr
