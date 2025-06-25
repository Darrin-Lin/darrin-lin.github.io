---
title: NTNU OS Persistence
date: 2025-06-01 0:00:00.000000000 +0800 CST
tags: [NTNUCSIE, Operating System]
categories: [NTNU CSIE OS]
math: true
---

## I/O Device
- memory-mapped I/O
- explicit I/O instructions
### DMA(Direct Memory Access)
硬體去做 I/O 的時可以透過 DMA 來把 memory 搬到 I/O 中，不用 CPU 去做，可以用來減少 turnaround time
## introduction to FS
process 開啟檔案會產生一個 file descriptor，OS 那邊會有 open file table（oft）紀錄被誰開啟跟對應的 inode，inode 會再對應到 file
### Directory tree
就是從 root 的 directory 開始，下面可以連到 directory 或是 file
### data vs metadata
#### metadata
i-node 包含了部分 metadata，裡面有 file size、owner、permission、time stamp、link count、data block address（man inode）
所有非 data 本身但是為了取得檔案的資料都是 metadata，包括檔名，所以並非所有 metaadta 都在 inode
### name types
regular、directory、symbolic
#### regular
創建時會有一個 link count 連到自己，當每多一個 hardlink 就會 +1 linkcount
#### directory
在創建時，metadata 就會有 2 個 link(. \& ..)，下面新增檔案就會加一個 link，本身創建時會有兩個 link count，一個是 parent dir 一個是自己。

#### symbolic
本身就是一個 file，
應用情境如在 WSL 要從 linux 要去存 Windows FS 的 folder，這時後因為是跨 FS 所以不能用 hard link，這時候就會用 symbolic link，或是用在 directory 上。或是當 package manager 的 libary name 不同，或是沒提供，要自己去用時，可以用 symbolic link 去把它連到正確的地方
### hard link
不能跨 file system 且不能用在 directory 上，因為會造成無窮迴圈，會把 link count +1
### soft link
## file system layout
### HDD
每一圈就是 track，然後上面會被切成很多段的 sector，一個 sector 通常是 512 bytes，一個 blocｋ通常由 4 個 sector 組成（2 KB），然後可以分為多個 region（如 inode table、data region）

### file system
| superblock | i-node bitmap(i-bmap) | data bitmap(d-bmap) | i-node table | data region |
|------------|---------------|-------------|--------------|-------------|

bitmap 是用一個 bit 來記錄對應到的位置是 free 還是 used
### directory
directory 的 data 就是存檔名跟 inode number，也就是
### open
open 會 parse path 去找到對應的 inode 然後生成 fd 並設定 open file table。
不管接下來是要 read 還是 write 都會更新 inode，因為 read 也會更新 access time
### hard disk time(HHD)
因為 hard disk 讀取很慢，比如說 7200 PRM 這樣最久 8.3ms 才能讀取到，所以需要軟硬體協做如 FFS 
### disk physical properties optimization(HDD)
可以把 inode 跟 data block 放在同一圈，這樣可以減少讀寫頭動的距離
### FFS
把硬碟放資料的位置讓 I/O 時可以減少 seek time 但 SSD 就不適用
## xv6 file descriptor application
### example - pipeling
就是 pipe （ls | less） 這個，會去用一個 ring buffer 來連接 input 跟 output，來達到 FIFO 的效果，並用 fd 去 access ring buffer as file，
xv6 實作 command line 的 pipe 是把 fd 1 close 掉然後 dup(fd[1]) 去進行，非 dup2（stdin (fd = 0) 亦同）
#### ring buffer
在 pipe.c 的 pipealloc 有負責 ring buffer 的實作
#### pipe write
在 call write system call 時 OS 會檢查是 write 到 file 還是 pipe，來決定要用 write 還是 pipewirte 寫入。
pipewrite 有用 spinlock 來去防止 race condition
#### fd
在 file 基本上 fd 就是 file descriptor，processstat 裡面 ofile 這個 openfile table 的 array 的 index（見 proc.h）
#### implementation of write
先 lock 然後會寫到要寫的都寫進去，如果滿了就 wake up the channel 然後 sleep（會同時 release lock），然後等到有空位再去寫入，當有空間則會寫完並 wake up 並 release lock 

## sleep lock
### channel
用來分辨不同 Lock 等待條件，當條件達成時要等的條件，然後當 sleep 時會去對自己的 process 狀態上 lock 再去  relase 指定的 lock 然後把自己的 channel 設定好，並且 stat 改成 SLEEPING，被 waked up 再去 unlock，爲了避免被其他 core 在 sleep 時去改變狀態\
如果要 channel 是 unique 的話，multi core 的 channel 是 global，所以也會變成 critcal section，那要 implement 的話就很麻煩，所以就決定讓 channel 不一定得要 unique，就只是用來被叫醒的 interger ID，如果撞到的話被叫醒再去 sleep 就好。

### context switch
當 sleep 時，會 call sched context switch 到 scheduler thread
#### sched vs scheduler
sched 會 context switch 到 scheduler thread\
scheduler 會 context switch 當前的 thread 到 ready queue 的 thread
### spin lock vs sleep lock
spin 雖然比較耗資源（busy waiting），但是不會遇到被永遠睡下去的問題

## maintaining crash consistency
### crash inconsistency
file truncate 
當要縮小 file 時，需要做 update bitmap 去把沒用到的 data block 設成 free，然後更新 inode 的 addr 跟其他資訊，但是這兩個不是 atomic，所以萬一做一個然後系統 crash 會造成 space leak(更新好 inode 但還沒更新 bitmap)，或是 double reference（更新 bitmap 但還沒更新 inode）因為 bitmap 那邊是空的，所以其他也可以用到，這時就會有兩組 inode 有相同的 data block 的控制權
### solution(logging/journaling)
把這兩個動作做成一個 transaction，從軟體面下手，先去寫接下來要做的計劃寫到 disk（log region），然後做完再去 commit 來把 transaction 標註為完成（加到 log 後面的一個 block），當 reboot 時，就把沒有 commit 的 transaction 做一遍，但不保證不會 data loss
#### xv6 implementation
只有一組 transaction，不過是 group transaction 放在 log region，整組裡面有一塊 header 區塊，標注後面的 transaction 的數量（包成一個 group transaction 一次操作，裡面有很多個 system call 的步驟），然後完成一個後就會 + 1，這樣就可以從第 i 個 transaction 開始復原，做完就會 commit 把 group transaction 清空
#### run in xv6
有一個 global 的 number of outstanding system call 用來記錄寫入，當開始寫入就 +1，完成後 -1，當歸零可以知道沒有任何 thread 在寫入，這時候就會去 log commit(log.c: start\op \& end\_op) 清空 log，不同 thread 也會在同一個 log 裡面，就是裡面步驟可以是不同的 thread 對不同檔案操作。

#### in hardware
有一塊 memory 作為 block cache(bcache) 對應到 disk 的 block，在 xv6 中確保一定是 1 to 1，一個 memory cache block 不論 readwrite 只會對應到一個 disk block，來確保 consistency，如果在 log write 時發現這個 block 已經在 log 的話就會只更新那個 log 來增加效率

| file descriptor | pathname | directory | inode | logging | buffer cache | disk |
|-----------------|----------|-----------|-------|---------|--------------|------|

## buffer cache
也就是一個有 buffer structure array，裡面是用來放 linked-list 的 node ，來讓 buffer 這個 linked-list 不用 allocate ，然後也有一個結構是會指向 LRU 的 head 
然後有一個 head 作為 dummy node
```c
bget
bread
bwrite
brelse
```
有這些可以對 buffer 進行操作
### bget
會去檢查 buffer cache 裡面有沒有這個 block，如果有就直接回傳，沒有的話就會去 disk 上面讀取，然後放到 buffer cache 裡面
### bread
會 bget，然後會把 buffer cache 的內容 locked 後 return
### bwrite
會把 buffer cache 的內容寫入到 disk 上面，最好要對 locked 的 buffer cache 寫入

## xv6 virtual memory
### high-radix tree
| EXT | L2 | L1 | L0 | offset |
|-----|----|----|----|--------|
| 25 | 9 | 9 | 9 | 12 |

L2 L1 L0 是用來構成 high-radix tree，用來切個第幾大塊的第幾小塊的哪一塊（L2 大 $\rightarrow$ L0 小），也就是分成三層去 index，當該分層的沒有用到，就可以不用寫到 disk

要確保每個 Page Directory 大小要跟 page table 對齊，所以 4KB，
造成 12 bit 的 offset，所以 39 - 12 = 27，
### satp register
裡面有一塊就是 PTBR
### PDE
在裡面有 PPN 跟 flag，用來表示是不是最後結果跟其他資訊，flag 因為儲存資訊的關係，會是 10 bit，然後因為會是 8 byte (64 bit) 作為總長度，$4KB \div 8B = 2^{12} \div 2^3 = 2^9$，所以 L2, L1, L0 都是 9 bit
| PPN | flag |
|-----|------|
| 44  | 10   |

### PTE
| PPN | offset |
|-----|--------|
| 44  | 12     |

$2^{56}$ = 64 PB，所以 xv6 的 physical memory 最大可以用到 64 PB

### SV39 in RISC-V
在 RISC-V Address space 是 39 bit，也就是 2^39 = 512 GB