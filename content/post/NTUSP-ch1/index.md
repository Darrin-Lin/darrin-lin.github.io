---
title: NTU SP Services for Application Programs
date: 2024-12-15 0:00:00.000000000 +0800 CST
tags: [NTUCSIE, System Programming]
categories: [NTU CSIE SP]
math: true
---

## User Identification
### Logging in
記在 /etc/password\
每個 row 代表一個 User，第一項是 user name，第二個是 password，第三個是 user ID，第四個是 main group ID,，第五個是 comment，第六個是 home dir，最後一個是 shell。
```
root:x:0:1:Super-User:/root:/bin/bash
```
UNIX 下認 ID 不認 name\
可以用 `finger` 來看 user 資料\
以前 `/etc/password` 是有放 hash 後的密碼\
現在因為安全性問題不存在那，並且有些工作站為了方便，會存在 NIS DB 以方便同步密碼

可以使用 `passwd` 來改密碼

帳號可以不去用 shell，把它設定為 Null 或 None，又或是設定成 no login 只用來設定權限使得可以指定好定程式所需權限，用 `su` 來附加到指定的 user 上

密碼是用時間產生 salt bit 加進去 `crypt()` 後再把 salt bit 加到最前面再存起來\
避免可以透過改自己密碼來去對應\
UNIX 密碼是 13 個字元，前兩個是 salt bit

## Process Management
一個 program 可以跑一堆 process，process 有自己的 pid，可以用 `ps` `top` 看

### CPU scheduling
一個 CPU core 可以跑一個 process，不同 core 可以 share memory\
`nice` 指令在 n-user 可以調**低** process 的優先度，root 才能任意調
### multitasking
為了使 response time 不要有一些 process 出現有太久的情況，所以在 CPU 夠快，可以一個處理完一部分再處理一個，中間沒感覺有跑到其他程式\
第一步是先進到 ready queue\
ready queue 要在要執行的在 memory 才會進去，否則就是在 event queue 去排 event(I/O) 的處理\
整個 loop 就是 Ready Queue -> CPU -> (Time Slicie（吃太久 CPU，超過 time slice）) / (Event Queue -> Event) -> Ready Queue

#### Context Switch (\*)
context switch: CPU 在 換人執行的過程，儲存或恢復執行的狀態，在 kernel mode，問題有時候會出在這

#### state
Ready State: 在 CPU 執行的時候，離開會因為要 Exit、超過等待時限，要跑等待 Event 這種跑比較慢的部分\
Block State: 在 Event Queue 的狀態\
Ready State: 在 Ready Queue 的狀態\
（start 跟 exit 有時候也會當成各一個 state）

### Shell
也是一隻 Process，shell process 再吃到指令時會去 fork 出一個 child process，而 parent process 會 wait child process exit，若沒有 wait 則會有很多 zombie process\
process 會像一個樹，最上面是 init 的 process

#### Common UNIX Shells
- Bourne shell
	- /bin/sh
	- Bell Labs
- C shell
	- /bin/csh
	- Berkeley
	- Commmand-line editing, history, job,control,...
- KornShell
	- bin/ksh
	- successor of Bourne shell
	- Commmand-line editing, job,control,...

### pipe 
`|` 把 stdout 轉到 stdin 接著跑另一個\
通常會在 pipe 後面加入 filter command 如 `more`, `grep`\
`more` 可以把輸入分頁


## Memory Management
access 到的基本上都是 virtual memory，每一個 process 都有自己的一塊 virtual memory\
系統會在 virtual memory 處理 call stack 跟 heap 還有 initialized，而之後會在 ch7 說明


## File/Directory Management
Dir 都是一個樹

### File Permission 
| Filtype-Permissions| X | Owner | Group | Size | ModifyDate | FileName |
|---|---|---|---|---|---|---|
| lrwxr-xr-x | 1 | test | user | 18 | Aug 28 13:41 | home -> /usr/people/maria |

FIleName 有 `->` 是連結\
Permision 前三個是 Owner 中三個 是 Group 後三個是 Other 的權限

chmod 其實是有四個區塊，但第一個是有關 security (set-UID, set-GID, sticky bit)

---

mode switch 只會在 system call 出現\
mode switch 跟 context switch 完全不相關

multitask 會因為造成了多次的 context switch 所以會比 unitask 還要慢