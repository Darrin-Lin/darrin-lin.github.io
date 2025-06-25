---
title: NTU SP Chapter 05 - Buffer I/O
date: 2024-12-15 0:00:00.000000000 +0800 CST
tags: [NTUCSIE, System Programming]
categories: [NTU CSIE SP]
math: true
---

## Standard I/O Library
定義在 ANSI C 的標準，所以不管 OS

### File I/O
會得到一個 FILE & fp 指到 FILE object 在 virtual memory 中，最後還是要透過 Unbuffer I/O\
有 file descriptor, pointer to buffer buffer size, remaining chars, error flag, EOF flag...\
可以在 Linux 下在執行檔前面加 strace，會 trace 所有 buffer I/O 會對應什麼 unbuffer I/O

### Buffer I/O
C allocate 在 heap 的 buffer，使用者不會知道，在 User space，在 standard I/O 的操作會在這邊（standard I/O buffer）拿\
把字串分次 printf 出來對 sys CPU 沒影響，但對 user CPU 有影響

#### freopen
類似 dup 
#### fdopen
可以把 unbuffer I/O 的 file descriptor 丟進去 associate 一個 subset 的 file pointer(buffer I/O)

----

## buffering
### 本章所指的 Actual I/O
只是讀寫 Unbuffer I/O 何時產生，並不需要去在意 disk
### types
#### fully buffered
基本上所有 I/O 都是 fully buffer\
意思是 buffer 滿了才去進行往外讀寫\
當讀到空的東西，才去把東西讀進來\
寫到 buffer 滿了才去寫入

#### line buffered
terminal 通常是這樣\
讀到換行才去輸入輸出，或是 buffer 滿，又或是收到 request 把東西 flush 出去\
flush: 把東西掃出去\
如 terminal print 出 $ 後再接收輸入

#### unbffered
沒有 buffer，即時輸入輸出，通常 stderr 就是這種

### ANSI C requirements
stdin stdout 只要不是 terminal 就是 fully buffered（如重新導向之類的），否則就不是 fully buffered\
通常慣例是 terminal 會寫成 line buffered\
stderr 不能是 fully buffered\
慣例是 unbuffered

### synchronize data
要把東西寫到 disk 要先用 fflush() 寫到 kernel buffer cache，再用 fsync() 同步檔案跟 metadata 或用 fdatasync() 只寫 metadata 到 disk，又或是 sync() 把寫入的動作進行排程


## 自己建 C 的 buffer
在還沒做 read write (I/O) 前要做處理
### setbuf(FILE \*fp, char \*buf)
如果 buf 不是 NULL 就至少要 `BUFSIZE` 的大小（stdio.h 定義），而種類看導向哪裡，如檔案就是 fully buffered\
buf == NULL 就是 unbufferd

### `setvbuf(FILE *fp, char *buf, int mode, size_t size)`
可以設定 buffer type 跟 buffer size\
假設是 `_IOFBF` 或 `_IOLBF` 且 buf 是 NULL，Disk 會幫忙開一個 block 的 buffer\
在還沒 read write 前可以設定

## buffer seek
### fseek
binary file 不支援 seek `SEEK_END` 因為有些系統會因規定某些檔案的大小要多少的整數，剩下填 NULL，所以不一定能找到實際的結尾\
text file 只能用到 `SEEK_SET`，只能移動到 ftell 回傳的那個值(???)
### `void rewind(FILE *fp)`
把 offset 移到最前面
### `long ftell(FILE *fp)`
回傳 offset，在 lseek 要用 lseek 把 offset 設成 0，`SEEK_CUR` 的回傳值來找\
lseek 沒有同樣的功能，要用 lseek(fd, 0, SEEK_CUR) 的 return 來找

----

## Standard I/O Lib type
### Unformatted I/O
沒有 type 的 I/O，可以一次讀一個 char, line, Direct I/O
#### Direct I/O
`int getc(FILE *fp)` 可能會是一個 MACRO\
`int getchar(void)` == `getc(stdin)`\
char 有可能是 signed 或 unsigned，所以 getc 最好用 int 來接，**char 不管是 signed 或 unsigned 都會有可能造成問題**\
`int unget(int c,FILE *fp)`，可以把讀到的東西放回去，但不一定要放原本的字元，但不能是 -1(EOF)，可以用來檢查文字是否為中文（2~3 byte）

### Memeory aligment
會透過 address decoder 一排一排拿，所以 struct 考慮到 row 的大小（8 byte）會提升效率，所以有些系統會加一些 aligment，所以在 binary 的 I/O 有時候會造成問題

snprintf 比較好（如程設所述）
### interleaved R&W restrictions
在 R&W 的 FILE \* 的 Output 跟 Input 間做一些動作使得兩邊 pointer 同步\
Output [fflush | fseek | fsetpos | rewind] Input\
Input [fseek | fsetpos | rewind | EOF] Output\
但每次都做會使 buffer 的好處消失\
可以 W R 分開各開一個 FILE \* 也可以解決，但就是會增加 open 跟 close 的時間

### multibyte file
可以在還沒 read write 時或還沒設定時可以設定，決定一次讀進來的字元寬度，有其他有 w 的相關 function 如 fwprintf
