---
title: NTU SP Chapter 03 - File I/O
date: 2024-12-15 0:00:00.000000000 +0800 CST
tags: [NTUCSIE, System Programming]
categories: [NTU CSIE SP]
math: true
---

### ANSI C
在跟作業系統無關，希望在各個 OS 都可以運行的

### POSIX
通常跟作業系統有關，希望可以在各個 OS 可以動

## Unbuffer I/O
不是沒有 buffer 的 I/O，每次都會進行一次 system call，是沒有在 C lib 幫忙 allocate 的 buffer\
call read 到 kernel 然後如果沒有在記憶體就到硬碟拿，每拿一次都要 trap in system


### HDD
track 是每一個碟盤中內部儲存資料的一圈結構，裡面由 sectors 組成，而數個 sectors 組成 cluster\
cost 最高的是讀寫頭的運動（seek time）\
轉的可以很快所以影響不是最大（Rotation Latency）\
因為只為了一個 byte 去進行讀寫頭的移動跟 disk 轉動 cost 太大，所以一次做至少要一個 block（數個 cluster）

### SSD
也是一個 block 一個 block 在讀\
但最小單位是更小的 page\
但寫入在 page 上 cost 很高，因為除非那個 page 沒資料，不然要把整個 block 清掉，所以通常開一塊新的然後 mapping

### buffer cache
也是一塊記憶體\
把 read 過的東西，讀完後先在 kernel 存在 buffer，之後如果需要就不用進行一次 I/O（太花時間）
read 執行步驟：先 call read system call ，因為一開始為在 kernel 的 buffer cache 什麼都沒有，所以就去硬碟讀，之後放到 buffer cache，接著 copy buffer cache 的資料到 memory，接著 retuen N（讀到的 byte 數），之後如果 buffer cache 有就會從那邊拿，所以即使 trap in system 也不一定會碰到 disk，所以這樣**從 buffer cache 拿不需要 context switch**

| per FILE object | per process | kernel(only one) | disk |
|---|---|---|---|
| buffer I/O | unbuffer I/O | unbuffer I/O | unbuffer I/O |
| FILE \* fp  file object(file descriptor, **buffer**（unbuffer 指的 buffer）) C standard I/O lib. | Open file desc. table | Open file table, i-node table, buffer cache | disk |

### meaning 
unbuffer I/O 因為那些 I/O, API 沒有 C Lib 幫程式 create 的 buffer


UNIX 裡所有都視為 file\
在一開始就有把 `stdin` `stdout` `stderr` 開好了\
處理 file descriptors


#### open file desc. table
預設會從 parent process 繼承，所以基本上長得很像，是位在 kernel mode 且是每個 process 分開


#### open file table
open file table 只要每 open 一次就會有一個 entry\
被指到一次就會 + 1，歸零就會消失\
content 會存在 buffer chache\
Reference Count(RC) 在 Open file table 跟 i-node table 裡的每一項都有，紀錄被指到的地方，當歸 0 就把他砍掉

buffer I/O 是只說不用 trap in system，不用 system call 的處理，如 C 的 `fread`，所以速度更快\
virtual memory 有 user space 跟 kernel space

### file desc
最開始的 file descriptor 一般會是 3，因為 0 1 2 是 `stdin` `stdout` `stderr`（約定成俗）\
最多開到 `OPEN_MAX`(compile-time limits)\
檔名不屬於 metadata 也不屬於 content，是位在 dir block，裡面是 i-node number 跟檔名\
referenced by the kernel\
per-process base\
shell fork 出你要執行的程式，而 0 1 2 是 shell 開的\
file descriptor 是在 file descript table，每一個 process 就會有一個\
每個 process 的 file descriptor 都是分開的

## Open Openat
```c
int open(const char *pathname,int oflag,.../* mode_t mode */)
```

每個程式會開 cwd（紀錄現在 program 位置的 func）

每個檔案開一次，在同一個 process 裡也會用不同的 system file table，但是會是在同一個 i-node

umask 是個參數可以在進行前設定好不想要被開的權限設定好，可以避免不小心開錯

`O_RDONLY` `O_WRONLY` `O_RDWR` `O_EXEC` file access mode\
`O_CREAT` `O_TRUNC` `O_EXCL` file creation (TRUNC 是會初始化，EXCL 是檔案不存在會 error)\
`O_APPEND` append to the end in each write\
`O_NONBLOCK` non-blocking\
`O_DSYNC` `O_RSYNC` `O_SYNC`

`openat`
要先帶一個 dirfd （open dir 拿到的 fd\*）\
time-of-check-to-time-of-use(TOCTTOU)\
第三版課本有的一些 at func\
在 openat 這類保證 TOCTTOU 一定相同
### prefetch 
就是在把要讀的地方的整個 page 都讀進 buffer cache\
Open 可以設定要不要 block mode，預設開啟\
block mode：把要求的的讀完才 return，但是可能在 buffer cache 沒有，所以可能會因為要讀資料所以會被踢出 CPU\
non-block mode：不要因為要讀資料所以被踢出 CPU，只讀在 buffer cache 的資料\
Synchronized I/O：一定要寫進 disk 才 return 

### Open file table
會記錄 file status flags(read, write, appenf, sync, nonblocking) 像是 read write 就是開了不能改\
也有 file descriptor flag\
fork 出 child process 預設會指到相同的 open file table，所以會共用 file status flags, current file offset, v-node pointer\
open file table 也有 reference count\
同一 process open 多次同一個 file 會指到 system file table 的不同位置。

### i-node
disk 上的 metadata 在被 open 時的時候會搬來 i-node table 的 entry\
buffer cache 是 cache file content\
v-node 是在對應到 file sys 的虛擬層有的統一介面，由 bell lab 當初搞的\
Linux 下沒有 v-node，用的是硬體上的 i-node\
i-node table 是針對檔案，所以多個 process open 同個 file 在 i-node table 只有一個\
open file table 會從 fork 出 child process 繼承（未特別設定），而對應到的是同一個 system file table，所以 system file table 會可能有多個 referneces\
也會有 refernce count\
而被指到的 system file table 在 UNIX 可以重新導向，如用 `>` \
檔案長度及其他除了檔名的 metatdata 都記在這\
metadata(or attributes)\
v-node table == i-node table 專門在放 metadata 的 entry，只要一個檔案 open 就會被存 metadata 在那，不論 open 幾次都會只會有 1 個 entry\
i-node 的 i 是 index\
每個 i-node table 的 entry 都是一個 i-node

### lseek
有些東西不能 random access（隨意讀取）如 pipe, FIFO\
有些會允許負數的 offset，通常是開的 file 是記憶體，有些是因為把第一個 bit 當作 User/kernel 的 bool

system call return >= 0 都會是成功\
`/etc/motd` 是 login 的歡迎訊息

在把有洞的 file redirect 到 另一個 file 會把 null 填成 '\0' 到另一個檔案，所以會增加 hole 的空間\
創造有洞的 file 方法：用 lseek 跳到後面 write\
在 ls -ls 裡面可以看到有洞的 file 的 disk block 會用得比較少

terminal 是要 enter 按下去才會輸進去\
建議 file 讀失敗(-1) 就要去看 errno

goto 在不傷到編譯後的最佳化的情況下可以放心用

### delayed write
會先把 read write 的處理先在 kernel buffer cache 處理，所以在寫入時可以減少 Disk I/O，而 UNIX 的 update daemon 會定期去把 buffer cache 讀寫進去 disk


## I/O Efficiency
`> /dev/null` 可以把 output 丟到一個無底洞\
read 的 buffer size 要開大一點，可以減少 context switch，但有極限（一個 page）\
STDIN 跟 STDOUT 當作讀寫的作用再進行轉向會比較有彈性\
Disk 讀出來可以算在 System CPU time，也能不算，因為現在有 DMA 技術不會動到 CPU 所以傾相不算\
因為不保證 kernel buffer cache 一定連續，所以一次搬的會是一個 page 的大小，不會再降下去\
系統的 CPU time 下降趨勢要看下降比例是否一樣\
clock time 下降到一定程度就停了，而在 User + sys CPU time 跟 clock time 差距越來越大，因為來不及先幫忙去處理 prefetch 的效果消失

## Atomic Operation
目的：不要有與預期不同的狀況產生\
如果寫了兩個 seek 到最後面寫 10 byte 的程式\
因為 context switch 不能控制，所以萬一在 lseek 後跳到另一隻程式進行 lseek 並寫入，再去寫第一個的，會覆蓋掉\
SP 的測試要把 loading 拉很重如 10000 process 才可能發生

所以要 atomic operation 確保 TOC TOU 相同，保證不會被切掉\
pread/pwrite \
另一種被切掉就停掉
```c
if ((fd=open(pathname, O_WRONLY)) < 0)
	if (errno == ENOENT) {
		if ((fd = creat(pathname, mode)) < 0)
			err_sys(“creat err”);
} else err_sys(“open err);
```

### dup dup2
`dup2` 是可以把 fd 的指向 copy 到另一個 fd 上，所以會指到同一個 system file table 的位置，也就是 fd 進行移動 newfd 也會跟著
```c
dup2(fd,newfd);
```
而 dup 跟 dup 2 會是 atomic operation

### sync
可以在 open 的時候設定，每次 write 都寫進去 Disk ，或是每幾次寫下去
#### when Open 
`O_DSYNC` 是只顧 content，metadata 在被影響到才會寫進\
`O_SYNC` 會把 content 跟 meatdata 去寫進去\
`O_RSYNC` 是在 read 的時候 sync\
寫完才會 return
#### sync fsync fdatasync
在 write 完 call，會把 buffer cache 丟到 disk 寫入的 waiting queue 就 return\
`sync` 是把全部的 data 跟 metadata 都寫進去\
`fsync` 是把指定的 fd 的 data 跟 metadata 都寫進去\
`fdatasync` 只寫指定的 fd 的 data，不寫 metadata

#### update daemon
會定期 call sync (defalt 30s)

#### 實驗會考

### fcntl
會去動 open file desc. table, sysytem file table 等地方的 file status flag 跟 file descriptor flag\
可以用 `F_DUOFD` 去做 dup 的 command
### ioctl
可以對很多東西 control，如 disk, mag, socket, terminal
### /dev/fd
可以對已經 open 的檔案進行 open，但權限不能凌駕原本檔案
```c
fd=open(“/dev/fd/0”,mode)
// ==
fd=dup(0)
```
