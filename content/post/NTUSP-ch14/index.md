---
title: NTU SP Advanced I/O
date: 2024-12-15 0:00:00.000000000 +0800 CST
tags: [NTUCSIE, System Programming]
categories: [NTU CSIE SP]
math: true
---


## blocking vs non-blocking
Blocking 是要等完成後才 return\
Nonblocking 就是以待在 CPU 為主，能讀多少就讀多少，寫入時雖然一般來說不會去即時寫入，但是在 buffer cache 滿了的情況\
 nonblocking 就回傳失敗，除了 time slice 以外基本上不會離開 CPU

### slow system calls
可能沒有辦法完成的 system call\
read networking 就是例子，就是可能會沒有辦法完成的，所以就要用 system call

stdout 要 '\n' 或 fd 的 buffer 滿了才會輸出，stderr 沒有 buffer 所以會直接輸出

terminal 有兩個 mode ，一個是按 enter 才會動，一個是即時反應（raw mode）

## I/O multiplexing
#### busy waiting
non-blocking 對 CPU 是一個浪費，雖然不會出現要等別人的代價，但就是會比較浪費記憶體

#### multiing tasking 
會造成 CPU 負荷很大

#### I/O multiplexing
在有人 I/O 等待的時候可以解決
### select
先在 `select` 對於 readfds, writefds, exceptfds 個去開個 set 去看每個有沒有需求進行讀寫，就會是 0 or 1\
可以用 `FD_ZERO` 來把指定 set 進行初始化成 0\
要用 `FD_SET` 去指定要 ON 的 number，`FD_CLR` 去取消

```
select(nfds, readfds, writefds, errofds, timeouts)
```
要等到有人 ready 才會 retrun，但沒說有人 ready 就立刻 return\
nfds 會在 0 ~ nfds-1 檢查 ready 的數量後 reaturn，所以還需要用 `FD_ISSET` 去確認\
其他參數就是 pointer，timeout 可以設成 NULL 就會等到有回應為止\
要注意會把 set 改掉，所以要 copy 一份

### poll
`select` 的改良版
```
struct pollfd {
int fd; // file descriptor to check, or <0 to ignore
short events; // bit mask: each bit indicates an event of interest on fd (more than select)
short revents; // bit mask: each bit indicates an event that occurred on fd
}
```
就是只記錄要用的東西，
不會改掉 set
```
poll( fdarray[], nfds, timeout )
```
### 結果
只有省在 I/O waiting 的部分

## aio_系列
非同步 I/O 的 read write

## lock
### three way lock
`flock` 會把整個檔案鎖起來\
`fcntl` 作業要練的\
`lockf` 用 fcntl 來實作的

### types of Locks
對於不同 process 才成立，同 process 不成立
#### shared read lock
可以多個去同時拿，但會把 write lock 鎖起來，不過因為有可能在系統實作時會檢查有沒有人在等 write lock，會使得在那情況使得新的人去拿 readlock 會暫時拿不到
#### Eclusive write lock
只有一個人可以去拿到

### fcntl lock
`F_SETLK` 如果有辦法就會去拿，沒辦法就 return \
`F_SETLKW` 如果沒辦法就等到拿到為止\
`F_GETLK` 不要再要存取時先去確認，因為不一定同步

#### Lockptr 
```
Lockptr
struct flock {
	short l_type; /* F_RDLCK, F_WRLCK, or F_UNLCK */
	off_t l_start; /* offset in bytes, relative to l_whence */
	short l_whence; /* SEEK_SET, SEEK_CUR, or SEEK_END */
	off_t l_len; /* length, in bytes; 0 means lock to EOF ( 會更新到最後 ) */ 
	pid_t l_pid; /* returned with F_GETLK */
}
```

process 在結束掉會把 lock 都還回去

lock 跟著 file 跟 proess，跟 file desc. 無關\
close 檔案會把 lock 拿掉（不管 file desc.，所以像是同一 process dup 複製 file desc. 到其他 new fd，或是 open 兩次，close new fd 也會把 fd 的 lock 放掉
### Advisory lock 
fcntl 只能管理透過 fcntl 進行 write 的操作，所以不透過 fcntl 就可以寫入，但是也可以透過檔案權限管理來防止不透過 fcntl 的寫入或讀取，可以兼顧安全跟效率。\
因為這樣只要鎖住限制只能透過程式來進行存取，來使得透過指定的帳號進行操作，且使用 fcntl
### Mandatory lock
不管怎麼樣 read write 都要去去進到 kernel mode 去確認可不可以寫，但是 cost 很高