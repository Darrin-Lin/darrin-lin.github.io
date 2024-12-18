---
title: NTU SP Process Control
date: 2024-12-15 0:00:00.000000000 +0800 CST
tags: [NTUCSIE, System Programming]
categories: [NTU CSIE SP]
math: true
---

## process ID

從 0 到 max prcess ID，然後再 recycle 回來\
ID 回收後不會立刻發放，短期內就不會去發放那個 pID

`tmpnam` 去要一個不會跟他人衝突的暫存檔名，可能會把 pID 放在其中

## Bootstrapping
1. process of starting up computer 會把電腦 cold boot
2. init 所有元件 (ROM 的 code?)
3. 把 Read Only Memory(ROM) 裡的 boot program 讀起來
4. boot program 開始接手（作業系統開始接手）
5. 把開機必須要的 kernel process 開起來
6. kernel process 去檢查硬體
7. kernel process init system process
8. 把系統開成 single user mode(root)
9. 設定系統資訊（如 hostname, 時間）
10. 開啟 multi user mode
11. 開啟 init process 

init process 會長出所有 process\
init 是 root 的 process 且是 user space 的 process

### PID 0 Swapper or Scheduler
不同 UNIX 下不一樣，有些是 swapper 有些事 Schedular\
Swapper: memory 不夠把東西移到 disk\
Schedular: 去決定哪個 process 先跑

### PID 1 init
由 PID 0 開起來\
root 的 process\
bring up multi user\
有很多功能
#### 常見功能
N User 要在開機時跑起來的東西也會由 init 跑起來

通常會 fork inetd 來去管網路連線，所有網路基本上都是由 inetd 去 fork 出 process 去長\
不管是 remote login 還是 consol login 都會長出 shell，可能是 inetd 的 child process 去開或是 dtlogin 的 child process

## Process Control Block PCB
process table 上面就是有 PCB\
裡面會有第一章提到的 process state\
program counter （紀錄 program 跑到哪）\
CPU registers（紀錄 process 被踢出前的狀態） \
CPU Scheduling Info（有沒有 priority）\
Memory-management information（memory 要管的 data structure, 去紀錄程式在 memory 的哪裡）\
Accounting information（CPU time, real time）\
I/O status information（open file, open device）

都是為了執行 process 所需要記的東西，可以說是 process 的 metadata\
sys/proc.h 的 proc(C structure) 裡面就是 PCB

proc 只有記 process 的 parent process，沒有記 child，因為不一定是固定的，所以要宣告成靜態的固定大小 array 這樣會浪放記憶體空間\
所以要找 child process 要遍歷所有 process，去檢查 process 的 parernt

### CPU process switch
在 context switch 時把 state 記到 PCB，並且把要去跑的 process 的 PCB 讀進去，中間 CPU 是沒在執行東西\
processs(其 PCB) 可能會同時在不同的 queue(Job, Ready, Device) 裡面
#### Process scheduling queue
- Job queue: 所有 process 的 queue
- Ready queue: 可以執行的 process
- Device queue: 等待 I/O 的 process

### Process 的 metadata & content
Metadata 就是 PCB\
content 是 virtual memory\
program counter 也是個 register 但是比較特別的 register 會去記 Virtual memory 的 text 裡面程式碼跑到哪裡，register 則是指到 virtual memory 的 stack top

## Process control system call
這邊的 function 不能出錯，所以一定不會 return -1 的情況
```c
pid_t getpid(void)
pid_t getppid(void)
uid_t getuid(void)
uid_t geteuid(void)
gid_t getgid(void)
gid_t getegid(void)
```
## Process Creation & Termination
fork 出一個 child process，通常就是要用 wait 等到 child process terminate 才會繼續\
child process exit 完 UNIX 會送一個 signal 給 parent process，wait 就會被喚醒\
child exit PCB 還會留存，要等到 parent wait 完才會結束，所以當 parent 不 wait 或是 parent exit 會出現 zombie process 的問題
```c
if ((pid=fork()==0))
{
	// child process
}
else
{
	// parent process
}
exit(0);
```
`fork()` return > 0 會拿到 child 的 pid， return = 0 會是 child process\
fork 完會 child 會從 fork 處開始跑\
PCB 不會繼承，Virtual memory 會複製\
PCB 會複製部分資訊\
上面的程式碼哪個會先跑不會知道

## process 在硬體上
process 的 VM 在 block 上會指到 Physical Memory 某個 page 上，只需要有用到的部分指到 PM 上\
page 是在硬體上習慣的單位\
`fork` process 的 VM 會跟原本的 VM 用 copy on write 指到同一個 page 上來，在 write 才寫到新的 page 上，來降低 cost，同時也可以使得一些 read-only 的資料不用 copy 

fork 完的程式如果在 fork 前有程式 printf，但是把輸出轉向，會因為不是 terminal 所以會變成 fully buffer，所以 buffer 也被複製過去，此時裡面還有東西，所以 printf 的東西也會在 child 的檔案中\
kernel space 不會去記 user space 有多少個東西指到它，所以 open file desc. table 只會有一個指到\
parernt 的 stdout 也會繼承給 child

## fork 特性
### normal case
parent 會 wait 到 child complete\
each go their own way
### inherited
real UID **Effective UID**, working dir, set uid/gid flag...
Real uid, effective uid, real gid, effective gid, supplementary gid,
process group id, session id, controlling terminal, set[ug]id flag,
current working dir, root dir, file-mode creation mask, signal mask
& dispositions, close-on-exec flag, environment, attached shared
memory segments, resource limits

### 不會繼承
```
Returned value from fork, process id, parent pid, tms_[us]time,tms_c[us]time, file locks, pending alarms, pending signals
```
CPU 時間就不會繼承，signal 的對象是 process，所以會是不會繼承，PID 也當然不會繼承，lock 不會繼承也很重要

### fork fail 原因
太多 process\
超過 read uid 的 process limit（`CHILD_MAX`）

### fork usage
同時跑不同部分的 process 如 networking 可以用到\
spawn(exec + fork)

## `vfork`
在還沒有 copy on write 跟 VM 的時代 BSD 派的產物\
讓 child 跟 parent 用同一個 page（也就是 VM 相同）\
Vfork 完 child 要先執行，執行完 exit() 或 exex() 才能去跑 parent\
現代除非要 share memory，因為有 copy on write ，所以否則不會去用這個指令，因為太多東西共用，很容易會不小心造成問題\
vfork 建議要用 _exit(0) 關閉不要用 exit()，因為 exit 會把 file object close 掉，因為是共用記憶體，所以也會把 file stream 順便 close 掉

open source 常在 select 完 non-blocking 為了防止 context-switch

## deadlock
兩個以上要取用對方擁有的資源，造成卡死\
但其實有四個條件同時滿足，才算 deadlock\
vfork 會有機會造成 deadlock，如 vfork 的 child 要等到 parent 達成某一個條件才會跳出，但 vfork 要等 child exit。

## process termination
### Normal termination 
- return from main() (exit(main(exit(argc,argv))))
- call exit
- call _exit _Exit
- start routine 的最後一個 thread return
-  最後一個 thread calling pthread_exit

### Abnormal termination
`abort()` 會送 SIGABRT 給自己\
terminal signal 只能由 UNIX 送\
異常終止只要是收到 signal 終止就算是，無論是自己要求，還是別人送過來

### C program start and termination
從 exec 開始執行\
在 exit() 的時候會去 call exit handler call，跟 standard I/O cleanup\
_exit 則是直接 exit 不做處理\
exit 跟 _exit 不會 return

### zombie process
當 child process return 但是 parent 沒有用 wait 或者是離開，導致 child 的 PCB 沒有人拿，這時候就是 zombie process

### `atexit()`
題外話：要引用別的檔案的變數要用 extern 引用，static 不能被用\
註冊在 exit() 後會去執行的

### `wait` `waitpid`
把結束的 child process 從 Z(Zombie process) 離開，child process 只要結束就會變成 Z\
wait 是 block mode\
waitpid 是可以設定成 non-blocking\
default 會是 ignore
#### calling wait/ waitpid 的 state
Block（等待 return）\
Return with the termination status of a child\
Return error

#### wait 的 return 條件
是在等待狀態改變
- 程式結束
- 變成 stop 之類的（如在背景跑，然後是 scanf）


#### `pid`
pid == -1: wait any child \
pid > 1: wait for the child with pid
#### `op`
op = WNOHANG\
就是不去等待，可以搭配 SIGCHID 去檢查有沒有收到訊號，但會有 TOCTOU 不同可能會造成問題的問題

### zombie & orphan process
#### zombie
The process has terminated, but its parent has not yet waited for it.\
所以在 child terminate 當下瞬間會是 Zombie\
會留存最少需要的資訊，(e.g., process ID, termination status, and CPU time taken) 用來 retunrn 給 parent\
會吃少量資源（PCB），但是當有大量的話會造成有一堆 loading，另外有可能會因此佔掉一堆的 process 的名額，導致 loading 沒有很多，但是沒辦法新增 process

#### orphan process 
process 的 parent terminate 的話就會變 orphan process\
init 永遠會去接手 orphan process 使得 parent process 變為 1(init)\
init 會在 child process 變為 zombie 的時候去 wait 來使得 zombie process 基本上不會在 init 的 child 出現

#### zombie state 
因為要使得 parent process 知道 child 的 termination status\
而且要知道 child process 的狀態（使得不會出現 child process terminate 後，其他的 process 與 child 相同 pid，使得 parent 卻去接收該 process 的資料）

#### trouble of zombie
會佔用 pid 的額度

#### double fork
一個使得 process 不用去 wait 也不會產生 zombie process\
先 fork 一個生 process 的 child process 再讓 child 去 fork 出 grandchild(second child)\
接下來 child process terminate，grandchild 的 parent 會變成 init\
記得還是要 wait child process\
因為不能確定 child 先 exit 還是先 grandchild 執行完 exit，所以建議先讓 grandchild 先一直 while(getppid != 1) sleep

在 open source 會很常出現一開始就 fork 完 exit\
另外
```
$ ./a.out &
會顯示 pid 後在 backgroup 跑
```
session & group 在 ch.9 有，但課上不到
### `wait3` `wait4`
可以拿到 PCB 的所有資訊\
wait3 是可以拿到所有 PCB 資訊的 wait\
wait4 是可以拿到所有 PCB 資訊的 waitpid

## race condition
多個 process 要去對同個資源做事，會出現照執行的順序對資源操作\
基本上就是要根據 context switch 去思考是否有 race condition

## `exec`
把程式寫入到 memory 的 text 區段，然後把 global variable 重新設定，並把 stack 跟 heap 清掉\
不會動到 PCB
```
main(int argc, char **argv, char **envp)
```
envp 是可以把 .env 的傳進去，傳進去的 string 像是 `HOME=/home/a\0`, `PATH=/bin:/usr/bin:/usr/local/bin:.\0`\
PATH 推薦把 `.` 放進去，然後要放到最後面，因為會照著順序找，所以不會把系統預設 command 吃掉\
會記錄 .env 的在 stack 的上方區塊，這在 exec 不會清掉，會被繼承\
env 新增會在 heap 中去找一個區塊去放新的 table，然後更改 memory 記的 envp\
但因為如果在 main funtion 丟進去的 envp 在 stack 中不會改，所以在 env 在執行時更改時會出問題\
出錯才會 return
### `execl`
用 pathname\
把 arg 用 list 的方式傳，最後用 (char \*) 0 結尾
### `execv`
把 arg 用 vector 的方式傳，最後用 argv[]
### `execle`
有 envp 版本的 execl
### `execve`
有 envp 版本的 execv
### `execlp`
用 filename
### `execvp`
用 filename

如果沒有設定 exec 要傳的 env 就會把自己的 env 傳過去
```
execlp             execl         execle
  |build argv       |build argv   |build argv
  V    try each     V      use    V
execvp    ->       execv    ->   execve
       PATH prefix         env
```

### Inherited from the calling process
pid, ppid, real [ug]id, supplementary gid,
proc gid, session id, controlling terminal,
time left until alarm clock, current working dir,
root dir, file mode creation mask, file locks,
proc signal mask, pending signals, resource
limits, tms_[us]time, `tms_cutime`, `tms_ustime`

#### `FD_CLOEXEC` 
可以對每一個 fd flag 記錄要不要在 exec 時 close open 的 file\
預設是 0（不會去 close）\
exec 唯一可能動的 kernel space

### Requirements & Changes
Closing of open dir streams, effective user/group ID\
可以在執行 exec 前做這些事
## UID
如 ch4 所說 save setuid 的功能
| ID | exec & set-UID bit off | exec & set-UID bit on | setuid(uid) & superuser | setuid(uid) & N-user |
|---|---|---|---|---|
| real user ID | unchanged | unchanged | set to uid | unchanged |
| effective uer ID | unchanged | set from user ID of program file | set to uid | set to uid |
| save set-user ID | unchanged |copied from effective user ID | set to uid | unchanged |

要拿 save set-UID 有一個方法是在 exec 完就去檢查 effective user ID

### utility of saved set-user ID
#### example `tip`(BSD) or `cu`(SV)
tip 跟 UUCP 都會透過數據機，一個是去 access 遠端的系統，一個是 copy 遠端的檔案\
因為 tip 要去搶 owner 是 `uucp` 的 lock，所以要去 setuid 在存取 lock 時更換 e-UID
```
step 1: exec tip
	r-UID == Our uid
	e-UID == uucp
	save set-user id == uucp
step 2: access filelock:
step 3: setuid(getuid()) 來回到一般的 permission
	r-UID == Our uid
	e-UID == Our uid
	save set-user ID == uucp
step 4: call setuid(uucpuid) 改 e-UID	
	r-UID == Our uid
	e-UID == uucp
	save set-user ID == uucp
step 5: release lock
```
如果在 step 3 去 spawn 一個 shell save set-UID 會是 Our uid，另兩個也是，因為 exec 了

## Interprete Files
檔案第一行有 `#!<'執行檔絕對路徑> <參數>` 就是個 Interpreter File，是個明文執行檔，絕對路徑的就是 Interpreter\
`#` 在 interperter file 代表註解
```
#!/bin/sh
```
shell script 就是其中一個\
第一行的長度在不同系統所允許的最大長度不同\
對 exec 來說是個不能執行的執行檔，要執行要用 p 的版本如 `execlp`\
用 execl 因為系統看不懂所以吃 #!後面的東西（interpreter），然後，接下來丟 Interpreter 的參數，把在 exec 的 argument 一個個丟進去，第 0 個 argument（execl 的第二個 aargument，shell 裡面第一個）會把他帶成相對路徑，剩下的塞進去 argument
```sh
a.sh x y
---
#!/bin/sh
echo "Hello, World!"
---
exec 的話就會長得像：
execl("/path/to/a.sh","a.sh", "x", "y", (char *)0);// 第二個參數 a.sh 會被解成 /path/to/a.sh
在 shell 就會解成
/bin/sh /path/to/a.sh x y
也就是會在 exec 時用 /bin/sh 執行 a.sh 然後參數帶 x y
第一行因為有 # 所以在 執行時又會被視為註解，所以不會再去執行
UNIX 從頭到尾就不用知道語言是怎麼處理
所以像是 awK 就會用 `#!/bin/awk -f` 因為 awk -f 是會去執行 awk program
---
#!/bin/awk -f
{print $1}
---
./a.awk x y
argv[0] = /bin/awk
argv[1] = -f
argv[2] = /path/to/a.awk
argv[3] = x
argv[4] = y
然後一樣丟到 exec 裡面
execl("/bin/awk", "awk", "-f", "/path/to/a.awk", "x", "y", (char *)0);
就會變成
/bin/awk -f /path/to/a.awk x y
```
### pro
可以指定執行的方式\
同時也造成比較有效率，因為不用經過先從 /bin/sh -c 執行然後到看不懂的去 fork & exec，可以直接從 fork & exec 那邊開始\
不會只能用 bon shell，因為 execlp 只任 bon shell，可以用 awk language 或是其他種的 shell languages
### against
額外還要去辨認 #! 的 interpreter\
但相對好處這微不足道

### system(cmd)
```
fork();
execl("bin/sh", "sh", "-c", cmd, (char *)0);
wait(); or waitpid();
```
可以看課本知道 signal 怎麼處理

## time
### process time
測 process 執行時間\
`clock_t times(struct tms *buffer);`\
```c
struct tms{
	clock_t tms_utime; // user CPU time
	clock_t tms_stime; // system CPU time
	clock_t tms_cutime; // user CPU time terminated child
	clock_t tms_cstime; // user CPU time terminated child
}
```
會是相對某一點的相對時間，所以不論是 reseponse time 或計算經過時間，都要用減的\
算的是 clock ticks\
### calender time
會算 1970 1/1 UTC 0:00 到現在過的秒數\
用 `time_t`\
有很多內建 function 處理日光節約時間、時區、閏年、閏秒問題\
`time_t time(time_t *calptr)`\
可以用 `gmtiem` 轉成 `struct tm` 然後用 fuction 進行轉成 string 或 format string 的處理，也可以用 `mktime` 轉回來
