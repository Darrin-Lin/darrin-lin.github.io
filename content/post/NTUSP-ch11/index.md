---
title: NTU SP Thread
date: 2024-12-15 0:00:00.000000000 +0800 CST
tags: [NTUCSIE, System Programming]
categories: [NTU CSIE SP]
math: true
---

## what's thread
用同一個 Virtual Memeory\
可以平行執行\
被稱為 lightweight process\
除了 不同的 process 會 contecxt switch，在 thread 也會有

### pre thread item
- progtam item
- register
- stack
- scheduling policy
- errno process
- signal mask

### process vs. thread
#### process
通常是不同 user own\
因此考量到安全性，會不太能share resource(memory)\
用 fork 這個 expensive 的來 create new process\
context switch cost high
#### thread
own by same user\
shared resource，所以要自己保護\
所有 thread 是平等關係除了 main thread\
cost of thread creation and switch is low\
通常稱作新增（等同在 process fork+exec）的動作叫做 spawn

### memory(stack)
有一種做法是把 stack 分成一小塊一小塊，分給不同 thread，每個 stack 的就是只能用到那塊的底部
### 特性
#### beneft
asynchronous event(如 signal) 處理上就很容易\
處理要 share 的 memory 會很方便\
可以在 blocking system call 更方便\
thrughput（單位時間可以處理的工作量）在把問題切得很多部份可以 improve\
可以降低在處理 block system call 就被 suspend 的狀況，可以降低 response time
#### drawback
要避免 memeory race condition\
不好 debug 因為不會一行行跑\
效能的運用會比較差，假設沒有足夠的 CPU core，
### OS
對 OS 來說還是同一個 process，分不出不同 thread\

## user-level thread
就在 user space 的 prcess 中有一個 run-time system\
run-time system 會 schedueling thread\
每個 run-time system 有 thread table 裡面會記 TCB(thread control block)

### implement
要對於 read 這些 blocking system call 要處理，避免整個 process suspend\
通常會在 run-time system 攔截 read 之類的 system call\
要擁有 preemptive，實作上可以用 signal 的 interrupt 特性來寫，

#### preemptive
OS 會擁有強制中斷 process 執行的能力，通常都是這種，為了做 context switch
#### non-preemptive
系統無法強制中斷 process 在 CPU 中執行，windows 3.1 3.2 就是

## kernel-level thread
kernel space 除了 process table 還有 thread table
Windows XP 2000 就是這種
切換 thread 的 cost 就會比較高
所以通常結束 thread 不會把那塊 TCB 還回去，而是 unmark 來使得之後要使用的時候不用再要一次，且可以減少刪除的 cost

## hybrid thread
在 process 中有 kernel thread 然後，kernel thread 裡面也有 user thread
Solaris

## POSIX thread
其他系統如 Windows 有自己的 Win32 thread\
是在 user-space implment 所以不是 system call\
如果沒有去建立就會是 single thread，從 main 開始跑，然後會是 main thread
### per thread their own
Thread ID, stack, registers, signal mask, errno value, scheduling properties, thread specific data
### creation
return < 0 都是有問題，且不止 -1\
`int pthread_create( pthread_t *tidp, pthread_attr_t *attr, void *(*start_rtn)(void *), void *arg )`

- `tidp`: pointer for thread id/handle\
- `attr`: customized thread attribute (NULL: default) 
- `start_rtn`: starting function for the created thread 輸入是任意指標，輸出也是任意指標
- `arg`: argument for the starting function pointer to a structure 丟進去的那項指標 arg
任何 thread 都可以接收 terminate thread 的 TCB\
main thread terminate 會結束一切\
create 完誰先跑跟 fork 一樣是看 CPU scheduling

### thread id
`pthread_t pthread_self ( void )` 可以拿到 thread id(非 pointer)
`int pthread_equal ( pthread_t tid1,pthread_t tid2 )` 可以比較不同 thrad id 是否相同
### thread attributes
`int pthread_attr_init(pthread_attr_t *attr);`
`int pthread_attr_destroy(pthread_attr_t *attr);`
設定屬性在 create 時用\
要先 allocate 一個 `pthread_attr_t` 的變數\
規定 init 的 atrribute 一定要 destroy，因為可能會去額外 allocate resource\
init 會把 `pthead_attr_t` 設定成 invaild 來避免誤用。

- `detachstate` 比較常見，設定成 detatch 的話不用回收 terminated status
- `guardize` 設定 thread stack 的 end 後的一塊 buffer 用來防止壓到其他 stack
- `stackaddr` 可以設定 thread 在 stack 中的 thread stack 的 low address
- `stacksize` 可以設定 thread 在 stack 中的 thread stack 的 size
`int pthread_attr_setdetachstate(pthread_attr_t *attr, int detachstate);`
`int pthread_attr_getdetachstate(const pthread_attr_t *attr, int *detachstate);`

### detatch
`int pthread_detach(pthread_t thread);`\
可以用 function 設定指定 id 的 thread（含自己）的 detach stat\
除了 create 時設定跟用 `pthread_detach` 外，也可以用 `pthread_join()` 去回收 terminated 的 tcb

### resource share
有些 implementation 會用 light process implement 所以有時候會有 share problem\
正常是使用時的時候會最好不要有 share resource 的問題，因為會共用\
如果遇到傳入參數的要避免被 return 所以 invalid\
不同 thread 最好也不要共用參數，因為不知道哪個先執行

### terminate
#### whole process
在以下任一條件，process 會結束
- 任一 thread call exit 系列
- 收到會 terminate 的 signal
- main thread 的 main() return
#### only thread
- 從 start routine return
- call `pthread_exit`
- 被其他 thread cancel（異常死亡）

#### `void pthread_exit(void *retval);`
不會 close file 跟 flush\
`retval` 是可以設定 return value，等同於 start routine return\
main thread 如果 使用會 block 直到所有 thread 結束\
main 也可以在結束所有任務，但其他 thread 還要 run 的時候可進行 `pthread_join` 來去等其他 thread terminated\
return 也可以把 int 轉成 (void \*) 在接收時傳回去，在 open source 也很常見\
回傳也要注意不要傳 local variable 的 ptr

### join
`int pthread_join(pthread_t tid, void **rval_ptr);`\
異常死亡也要去 join\
只能去等 non-detach 的 thread，並且會 block 直到該 thread terminated\
如果拿到 canceled thread 會收到 `PTHREAD_CANCELED`\
如果是去拿 detached  的 thread 會 return `EINVAL`\
`rtval_ptr` 設定成 NULL 帶癟不回收資訊\
回收完會 detach，所以假設另一個 thread 要去 join 就會失敗\
`int pthread_timedjoin_np(pthread_t thread, void **retval, const struct timespec *abstime);`\
是 non-block 的版本，超過時間就會 return `ETIMEDOUT`\
`int pthread_tryjoin_np(pthread_t thread, void **retval);`\
是除非可以回收否則就是回傳失敗

### cancel
`int pthread_cancel(pthread_t tid);`\
效果如同 `tid` 的 thread 去 call `thread_exit(PTHREAD_CANCELED)`\
要記得 join 或 detach 來使得 tcb 回收\
是會丟一個 request 給 tid\
要知道成功完成 cancel 可以用 join 來知道

`int pthread_setcancelstate(int state, int *oldstate);`\
可以設定自己要不要被 cancel
- `PTHREAD_CANCEL_ENABLE`(default)
	- 不會馬上 terminated
	- 執行到 cancelation point 才會 cancel
- `PTHREAD_CANCEL_DISABLE`
	- 會 pending cancel request 直到被設定成 enable 然後是 cancelation point 才會 cancel
cancelation point 通常是 system call

`int pthread_setcanceltype(int type, int *oldtype);`\
設定 cancelability type\
- `PTHREAD_CANCEL_DEFERRED`(Default)
	- 到 cancelation point 才 cancel
- `PTHREAD_CANCEL_ASYNCHRONOUS`
	- 收到立刻結束

#### cancelation point
`void pthread_testcancel(void);`\
等同於 cancelation point\
因為 asynchronously cancel 會對一些存取 share resource 的狀況出問題，可以用加上 lock 來處理\
通常 cancelation point 都是 block system call

### cleanup
就是像是 atexit	的作用\
`void pthread_cleanup_push(void (*routine)(void *), void *arg);`
`void pthread_cleanup_pop(int execute);`
- `pthread_exit`
- canceled
- call `pthread_cleanup_pop()`，且 argument 不為 0，只會執行最上面的 function

兩個一定要成對出現不然會出錯\
return from start routine 不會執行，因為代表該執行的都執行完了

### thread synchronization
兩個 thread 做 i++，因為實際上是很多動作，所以有可能會是 A 先 +1 然後 context switch 到 B 然後 + 1 ，這樣兩個值在使用時都會變成 i+2 => race condition\
有三個方法可以解決：
- Mutex lock
- readers-writer lock
- condition variable
memory lock 會需要跟著 lock 的 memory
### Mutex
**Mut**ual-**ex**clusion interfaces\
advisery lock\
lock memory 的 resource\
用之前要 lock，用完要 unlock\
如果要用的被 lock 會 block\
本身就是個 object\
在存取的東西 unlock，其他所有要存取同樣資源 block 住的資源的 thread 就會變 runable 然後被丟到 ready queue 從請求 lock 繼續，先 schedule 到的會拿到，其他在執行時又會 block\
child process 會繼承，因為是 VM 分開\
要防止 deadlock 的一個方法是用 non-blocking 、排 order 、granukarity（多個情況共用 lock） 或是用 condition variable\
怎麼去分配 lock 數是個學問，因為太多 lock 雖然可以增加平行化，但一直請求 lock 會造成 performance 降低，且有 dead lock 的問題
#### init and destroy
`int pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *mutexattr);`\
也可以 static allocte: `pthread_mutex_t mlock = PTHREAD_MUTEX_INITIALIZER`設成 default attribute（但是 static 就不能改）\
dynamic allocate 就是用 init 然後記得要 destroy，因為可能會額外 allocate\
`int pthread_mutex_destroy(pthread_mutex_t *mutex);`\
有 process-shared attribute 跟 type attributeL，設定為 NULL 就會設成 default\
在三種可以解決 share resource 的 方法中只有 mutex 有 type attribute
#### attribute
`int pthread_mutexattr_init(pthread_mutexattr_t *attr);`\
`int pthread_mutexattr_destroy(pthread_mutexattr_t *attr);`\
也是要動態宣告跟 destroy\
可以在這邊設定可以跨 process 存取同一塊 memory
- `PTHREAD_PROCESS_SHARED` 可以跨 process share memory
- `PTHREAD_PROCESS_PRIVATE` (default) 不行跨 process share memory
而可以用 MACRO 在 compile time check 有沒有提供這個功能，或是在 runtime 用 sysconf（check 跟 file 無關的，跟 file 有關就是 pathconf） 來 check
用各個 function 去改\
type attribute 有四種
- normal
	- 不會 error check 跟 deadlock detection
- errcheck
	- 會 error check 跟 deadlock detection
- recuresive
	- 單個 thread 可以多次 lock，但也要用相同次數的 unlock（有 lock count），也會 error check 跟 deadlock dection
- default
	- 看系統（大部分是 Normal）
可能的 error 如 lock 住自己 lock 過的，會 deadlock、unlock 別人的 lock(undefine)，unlock 已經被 unlock 的
```c
int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_trylock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);
```
trylock 就是 unblock mode 的 lock\
比 readerwriter lock 好 implement，效率較高，但只能同時讀寫一個
### ReaderWriter lock
可以多個去拿 readlock，write lock 只能有單獨一個，就是跟 file lock 的概念一樣\
只有一個 attribute 就是 process-share\
attribute 的宣告也有 static 跟 dynamic\
一樣有 init, destroy, unlock\
lock 跟 trylock 有 rdlock 跟 wrlock 兩個版本

### Thread Pool
有個 work queue 會排很多 job，Master Thrad 會去 update\
程式會 maintain 一個 pool 裡面有 worker threads，稱之為 thread pool\
worker thread 當作完事情，會去 work queue 看有沒有跟目前閒置的 thread id 符合的並領取，然後去把 work queue 上的那個 job remove\
job queue 通常是 link-list\
有一個方式是上 read lock 去確認，有了再去用 write lock 更新，可以比 mutex 增加平行度，但會有當 queue 空了的話會 busy waiting 的狀況\
更好的方式是用 condotion variable

### condition variable(cv)（一定會考）
\*-free 代表沒有什麼\
會是 race-free（不會不定期的一直 recheck，減少資源的競爭）
用來解 job queue 空，但每個 thread 都在 busy waiting 來去查找 jib queue\
一定會搭配 mutex 用\
允許一個明確的通知事件（exclicit event notification）\
動 condition 一定要取得 mutex\
基本上就是兩個概念：wait 跟 signal\
`int pthread_cond_init(pthread_cond_t *cond, pthread_condattr_t *cond_attr);`\
`int pthread_cond_destroy(pthread_cond_t *cond);`

### attribute
只有 `PTHREAD_PROCESS_SHARED` 會用到
### func
`int pthread_cond_signal(pthread_cond_t *cond);`\
是可通知單個在 wait 的 cv\
`int pthread_cond_broadcast(pthread_cond_t *cond);`\
通知所有在 wait 的 cv 然後看誰先搶到 mutex 誰就先 return，其他人則是會去等 mutex lock

`int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex);`\
1. sleep 直到收到 cv signal
2. 睡前 relese mutex lock（以上為 atomic
3. 去拿 mutex lock 然後 return（等到 lock 才會 return）
`int pthread_cond_timedwait(pthread_cond_t *cond, pthread_mutex_t *mutex, const struct timespec *abstime);`\
有 timeout 的版本，但是是要帶時間點進去，也就是要在 `gettimeofday()` 的 return 去 加上要等的時間

### usage
1. 拿 mutex lock 對要查的 condition
2. 看 conditon
3. 用 condition wait 來放掉 mutex lock 然後 sleep，並且使用 condition variable
4. 當其他人發現條件達成，用 codition signal 通知指定 cv，然後當是被叫的那個的話就拿回 mutex lock 再 return 
5. 做事然後最後 unlock mutex lock

在 2. 通常會是用 `while` 來避免當通知前，通知的 thread 放掉 mutex 然後被改了，可以繼續睡下去
```c
// Waiting for x==y condition
pthread_mutex_lock(&m);
while (x != y) // 用 while 很重要
	pthread_cond_wait(&v, &m);
// do something here if necessary
pthread_mutex_unlock(&m);

//Notifying waiting thread that x is incremented
pthread_mutex_lock(&m);
x++;
pthread_mutex_unlock(&m);
// 可能中間 context switch 造成 x != y
pthread_cond_signal(&v);
```

opensource 通常會先 signal 再去 unlock，不過反過來在一些狀況也不會影響\
這樣做是因為 wait 會先解除 sleep 然後再去 wait mutex lock，假設是 signal 完 context switch 就會變成 wait 的在那邊等 lock 然後送 signal 的 thread 再去 unlock mutex，這樣條件再沒有其他人先去請求 mutex 的時候就會就不會被改到\
反過來先 unlock 再送 signal 的話在一些有 priorty 的 thrad pools 會出現問題
```
priority A > B
A 在等工作然後拿 mutex 之後執行 wait 把 mutex 放掉
此時 main thread 拿 lock 直到 work queue 空了 unlock
然後 B 做完事情去拿 mutex lock 拿到了
main thrad signal 
A 接收到但卡在 mutex
B 這時候就會在 while 那邊因為條件達成。所以直接去執行變得 B 會比 A 先

如果是先 signal 再 unlock 的話，A B 都會在等 mutex 的狀況
這時因為 priorty A 比較大，所以進到 ready queue 時會是 A 拿到 lock 先開始執行
```

## memory mapping(ch 14.9)
除了 `shmget` 的另一個方法\
把檔案 map 到記憶體\
好處是不用像 buffer/unbuffer I/O 一樣搬來搬去\
會把檔案 map 到 physical memory 然後再 link 到 VM stack 跟 heap 中間

`void *mmap(void *addr, size_t len, int prot, int flags, int fildes, off_t off);`

為了對應到 page 的開頭，`off` 通常 會希望是 page size 的倍數，在 VM 上的 start addr(`addr`) 也是，len 也通常會是，page 大小可以用 `sysconf` 來取得，通常 `addr` 會設定成 NULL 讓系統決定\
系統會希望剛好 map 到剛好完整在 physical memory 的 page 上，也就是會在 page 的開頭，所以通常會開到整數的 page 數，剩下空的補 0\
`prot` 會設定 無 跟 讀寫執行的權限，用 OR 開\
mmap 可以設定 flag 成 `MAP_SHARED` 來使得兩個 process 可以 shared 同一塊 mapping 的記憶體（physical memory），也可以設定成 `MAP_PRIVATE` 來限制\
performance 的分析會根據系統不同\
可以開一個不存在的檔案拿到 fd mmap 來造成 memory shared 的效果，如 `/dev/null`，然後 fork 來讓兩個 process 來 share memory\
或是可以 mmap fd = -1 然後 mmap 參數加上 `MAP_ANOYMOUS` 來使得沒有檔案沒有關係\
多次呼叫 mmap fd = -1 不會有問題，就是開多塊 memory

## Virtual memory
gcc 預設是 dynamic link\
一般的程式碼是 static link\
dynamic 的存放位置會 load 在 stack 跟 heap 中間的區域\
UNIX VM 通常習慣 1G 給 kernel 3G 給 user\
thread 執行時也會用到 stack 跟 heap 中間的區域

## Thread vs. fork()
fork 完假設有 memory lock 在 原本的 VM 中在 fork 時會被繼承，但是 thread 會被清空，所以會造成 dead lock\
exec 完因為會清掉 VM 所以會把 memory lock 清掉\
fork 完 child 剩下的 thread 就是 fork 它的 thread

`int pthread_atfork(void (*prepare)(void), void (*parent)(void), void (*child)(void))`\
`prepare` 是在 call fork 時會去執行的，`child` 是在 child process fork 完 return 前會執行的，`parent` 是 fork 完 return 前會去執行的\
可以在處理 fork 時用 `prepare` 把所有的 memory lock 上上去，再去 fork，並且在 child 可以在執行前先把所有東西 unlock，確保不會在 fork 時造成有無法解鎖的 lock\
atfork 執行的順序在 prepare 是 stack，child 跟 parent 是 queue，是因為了可以對稱，比如說 lock 跟 unlock 就會成對

## thread signal
sigaction 對所有的 thread 都是一樣，所以不論是 ignore 或是設定 signal handler 都是整個 process 的 thread 都有效\
signal mask 每個 thread 分開\
signal 在 thread 一般來說不像 process 那邊是 asynchronous，因為是多半是平行化跑，不會有被中斷的問題，所以會是 synchronous\
thread 的 context 是 signal handler 本身\
通常會是一個或多個 thread 去處理，然後通常會用 while 去不停 wait\
如果宣告了 sigaction（process handle）然後同時又用了 sigwait（thread handle）會由系統決定誰去處理，通常不會這樣找麻煩

### generate
- synchronously
	- 執行了一定會發生
	- system 因 excpetion 而產生
		- 應該要 debug
	- 某個 thread(internal) 用 kill 送給另一個 thread
		- 該個 signal 收
- asynchronously
	- 外部 (external) 的 process 用 kill 送的（包括 ^C）
	- 系統任挑一個沒有 block 的
### dedicated signal handling thread
特定用來處理 signal 的 thread\
把其他 thread 對要處理的 signal block 起來，只讓要處理的 thread unblock
### inherit
signal mask 會繼承 spawn 出自己的 thread 的 mask
### mask
`int pthread_sigmask(int how, const sigset_t *set, sigset_t *oldset);`\
要去處理就會呼叫，參數跟 sigpromask 一樣

`int sigwait(const sigset_t *restrict set, int *restrict signop);`\
跟 sigsuspend 概念一樣，可以去改 mask 然後 wait，但是參數不一樣\
參數不像 sigsuspend 是替換 mask（block 的 signal），sigwait 是放要等的 signal（unblock 的 signal）\
因為 signal handler 是 thread 本身，所以不會 handler call 完才 return，會在 signal 來就 return\
return 完接著就會去處理 signal\
會 atomic 的處理 mask 跟 pause 然後 return 前會恢復 sigmask，然後把接到的 signal 寫到 signop

### implement
在操作時通常會讓 spawn 前先把 mask 關好，等處理好初始狀態再交給 thread unblock\
一開始就是處理給 sigwait 用的 signal mask 跟其他東西，然後再去用 while loop 包住 sigwait 跟 signal handler 的程式，因為 sigwait return 前會還原 mask 所以會有跟 sigaction 在 handler 執行時 block 住 signal 相同的效果。

## thread safe/atomic I/O
因為 fd 是共用的，所以不能用一般的 read write & lseek\
`ssize_t pread(int fildes, void *buf, size_t nbyte, off_t offset);`\
`ssize_t pwrite(int fildes, const void *buf, size_t nbyte, off_t offset);`\
pread pwrite 是 atomic 所以在 unbuffer I/O 要用這個

## thread-safe function
因為要避免在 function 到一半因為要跑其他 thread，然後又被呼叫一次造成的問題
條件是 thread 不能有 static variable\
通常是在 function 後面加上一個 `_r` 來代表 thread-safe
### thread-safe vs. async-signal safe
在多個 threads 中被中斷時重複呼叫不會出問題 vs. 在 signal handler 中間被中斷時重複呼叫不會出問題\
負面表列（但通常還是有另外提供 thread safe 版本）vs. 正面表列\
可以在 multi-thread reentrant vs. 在 async-signal handler 可以 reentrant\
thread-safe 不代表 async-signal safe：\
如 malloc 在裡面上 mutex lock 然後這樣在 thread 中可行，但在 async-signal handler 就會 deadlock\
也就是用 mutex 保護的可以 thread-safe 但是在 async-signal 不會是 safe 
在沒有惡意處理的情況下（如把 global variable 複製到 local）async-signal safe 通常是 thread-safe

## thread-safe FILE object
通常 C 的 Buffer-I/O function 會呼叫 flockfile 使得是 thread-safe\
`void flockfile(FILE *filehandle);`\
`int ftrylockfile(FILE *filehandle);`\
`void funlockfile(FILE *filehandle);`\
就是可以把 file stream 上 lock，通常 fprintf, fscanf 會難見呼叫\
是 recurive lock（有 lockcount）\
為了提升如用 for loop print 一個個 char 的狀況可以用 `_unlocked` 版本，手動 lock & unlock