---
title: NTU SP Introduction
date: 2024-12-15 0:00:00.000000000 +0800 CST
tags: [NTUCSIE, System Programming]
categories: [NTU CSIE SP]
math: true
---

ISA Instruct/kereerererererion Set Architecture 用來在 Machine Language 跟 microachitecture 溝通\
作業系統主要是用 C 寫的\
以前的 OS 是用組合語言寫的，為了可以對應到不同平臺\
廣義的系統程式包含 OS, Compiler, Editor, Command interpreter


kernel 指的就是作業系統

system calls 就是 API 用來從 application 跟 OS 溝通\
要用 system calls 來跟 kernel 溝通\
POSIX 是在訂定 system call 規定

gcc -Wall 很重要（但習慣 -Wall -Wextra）\
system call 主要就是 call OS 的 function\
不同 OS 有相同的 system calls 的標準，就是為了 application layer 可以跨平臺

## function call vs. system call
### porcess
一個執行檔執行下去就是一個 process，process 就會吃資源，而在同一 process 就會有一樣權限

### function call 
可信度在 User 跟 function 相同，因為在同一個 process

### system call 
在 call 的時候會跳到 OS 那去執行 OS 的程式\
OS 的 function 是可信的，但 User 不一定，因為 OS 不可信就不該用那個 OS\
OS 權限比較高，但 User 碰不到（只有 OS 直接碰得到硬體）\
資源基本上就是 OS 給的權限

### mode
OS 基本上就是要有至少兩個 mode 可以區分權限\
如 non-privileged(User mode) & privileged(kernel mode)\
而 User mode 會要分離不同的 User 存取權\
User mode 不能存取其他 process，只能存取 user space

## mode switch
平常就在 User mode\
System call 會進到 kernel mode 然後檢查權限，有則執行，無則丟回 User mode

## space
在 user mode 跑的 process 可以 access 的記憶體就是 user space，不論在記憶體的哪\
只能在 kernel mode 可以 access 的 space 是 kernel space

malloc 是 call sbrk 來進行 memory 的操作

User CPU time 是指說在 User mode 所在 User CPU 執行的時間，會比較快\
Sys CPU time 是在 kernel 那邊 CPU 執行的時間\
Real time 是在從 enter 按下去到跑完的時間，>= User + sys 因為有 I/O, be forked & exec

System call 在 UNIX 會做適當的保護，所以不會因權限提升造成 crash

有時候優化可以去看 CPU time 是否造成執行時間是否過長
可以用一些 system function 來減少 CPU 的浪費

在 C system call 是從尾端的參數開始 push，然後才 call，接下來丟到 register，然後 trap（中斷），接下來儲存狀態來預備 rollback ，之後切到 kernel mode，接著進行檢查權限，再進行執行，接著 return 回 User mode，所以吃的資源比較多，接下來 return 後就從 stack 裡 pop 掉\
C 有些 function 就是提供 system call

## OS
### 垂直面
負責提供 interface 使得可以比較可以容易操作\
擋住麻煩的部分使得可以安全的執行及提供服務

### 水平面
管理資源包括 CPU, memory, I/O device...\
管理程式使用資源的時間（CPU）以及空間（memory）
#### Service for Application Programs
user identification, process management, memory management, inter-process Cummumication, Signal, I/O Management(Terminal, Network...)...
#### OS Kernel
CPU Scheduling, Virtual Memory, File System, Protection, Security, Synchronization, I/O Control...

## Unix Architecture（結構）

| user interface | users |
| --- | --- |
| system call interface | Shells, Compilers, Application programs, ... |
| UNIX Kernel | CPU scheduling, signal handling, virtual memory, paging, swapping, file system, disk drivers, caching/buffering... |
| Kernael interface to the hardware | terminals, physical memory, hard disks, printers... |
