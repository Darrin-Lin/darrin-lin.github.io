---
title: NTU SP Signal
date: 2024-12-15 0:00:00.000000000 +0800 CST
tags: [NTUCSIE, System Programming]
categories: [NTU CSIE SP]
math: true
---

system call 在某些條件下結果可能會不如預期
## what is signal  
無論是自己或是他人要求的，都是透過 OS 送的\
像是 Ctrl + C 或是 SIGPIPE 都是 signal 的一種
也算一種 IPC 的方式
### signal v.s. interrupt
#### interupt（狹義）
對 CPU(Hardware)\
內部指令可能會間接或者本身就想要去直接中斷，或是外部硬體接腳 如 PIC(Programable interrurpt controller) 這邊就是連到很多周邊 device 並接收 request 再接收 IRQ 去中斷 CPU\
CPU 被中斷會 run ISR(interupt service routine) 是用來記錄 event
#### signal
對 process\
會去處理 ISR 記錄的 event，由 OS 去解釋 event（不一定會是單一個 signal\
signal == 0 是 NULL signal\
signal 會是正整數\
不同系統會有不同 signal number\
OS 等有空會去處理紀錄的 Event 然後轉換成 signal 傳送到 process\
signal 的定義是來自 kernel，也是由 kernel 送給 process，不論 event 是什麼，解釋權都在 signal

## Synchronous v.s. asynchronous signal
### Synchronous 同步
因為執行某個 instruction 就發生\
只要一發生就會產生，如 divided by zero 或 segmentation fault
### Asynchronous 非同步
不知道 run 到什麼會發生\
如 killed by other process, SIGCHILD(child termination)

## Generate Signal
### Terminal-generated signals
DELETE key
#### SIGINT(2)
Ctrl-C
### Signals from exceptions
#### SIGFPE(8)
divided by 0
#### SIGSEGV(11)
illegal memory access
### Fuction kill
(need owner or superuser)
### Shell command kill
#### SIGKILL(9)
kill -9 pid
### Signals because of software conditions
#### SIGPIPE
reader of the pipe terminated
#### SIGALRM
expiration of an alarm clock

## 對 Signal 的反應（action）
SIGKILL and SIGSTOP 不能被設定反應
### 設定 Ingore signals
SIGKILL and SIGSTOP 不能被 ignored\
有些 signal 沒有設定對 ignore 的反應，所以當設定成 ignore 的後果要自負，如 SIGFPE

### 設定 Catch signals
SIGKILL and SIGSTOP 不能被 catch\
當 signal 來要 call 哪個 function\
call 的 function 就是 signal hadler
如 waitpid 就是

### Apply the default action
Terminate（多數） or ignore or stop\
沒有設定就會去變成 default

## Example of Signals
w/core 指到是 core dump 就是終止後會把 memory copy 一份下來
```
Terminate w/core – not POSIX.1
No core file:
non-owner setuid process
non-grp-owner setgid process
no access rights at the working dir
file is too big (RLIMIT_CORE)
core.prog (4.3+BSD)
```
### SIGABRT
- terminate w/core
call `abort()` 來自己終止自己，會送 SIGABORT 給自己
### SIGALRM
- terminate
Call `setitimer()`, `alarm()` 這兩個是用來設定每幾秒就會送一個 SIGALRM 訊號過來，如果沒設為 catch 去處理，會使得程式終止
### SIGBUS 
- terminate w/core
Hardware 去 implement 的情形
### SIGCHLD
- ignore
sent to the parent whenever a child process terminates or stops
### SIGCONT
- continue/ignore
Continue a stopped process
### SIGFPE
- terminate w/core
Divid-by-0, floating point overflow, etc.
### SIGHUP
- terminate
以前是實體接線斷掉，會接到 SIGHUP\
通常 deamon(background process) 會有 comunication file（組態檔）來根 process 溝通\
現在是對 deamons 更改組態檔後通知 deamons 要進行重新讀取組態檔有關（因為通常不會隨邊停止）
### SIGINT
- terminate
DELETE key or CTRL-C
### SIGIO
- terminate/ignore
跟做 asynchronous I/O 的 event 有關\
比如說要一直讀，讀到 buffer 去，作業系統在讀完把 data copy 過來後就會收到
### SIGPIPE
- terminate
reader of the pipe/socket terminated
### SIGPROF
- terminate
A profiling timer expires (setitimer)
### SIGPWR
- ignore (SVR4)
當沒電 UPS 運作時會 call 這個 signal 給 init
### SIGQUIT
- terminate w/core
CTRL-\ triggers the terminal driver to send the signal to all foreground processes
### SIGSEGV
- terminate w/core
Invalid memory access
### SIGSTOP
- stop process (like SIGTSTP)
Can not be caught or ignored
### SIGSYS
- terminate w/core
Invalid system call
### SIGTERM
- terminate
Termination signal sent by kill command
### SIGTSTP
- stop process
Terminal stop signal (CTRL-Z) to all foreground processes
### SIGURG
- ignore
Urgent condition (e.g., receiving of out-of-band data on a network connection)
### SIGUSR1
- terminate
User-defined
### SIGUSR2
- terminate
User-defined

## system call of set signal
```c
`#include <signal.h>`\
void (*signal(int signo, void (*func)(int)))(int)
```
**以前註冊只有一次有效** \
設定 `signo` 的對應要用哪一個 function 處理\
return 會是回傳之前的 function address\
輸入輸出的 function 都要是 `void func(int signo)` 的格式\
`func` 有由系統 define address 是 -1 是 `SIG_ERR` 0 是 default `SIG_DFL` 1 是 ignore `SIG_IGN`，因為不會有 address 那麼前面
因為可以很多個 signal id 共用同一個 function 所以會接去判斷接到的 id ，會大概長得像
```c
void function(int signal)
```
`pause()` 會等到收到沒有被設定為 ignore 的 signal\
可以使用 `signal()` 去確定設定的種類，就是設定成想確認的結果就會知道之前是什麼
```c
int sig_int();
if (signal(SIGINT, SIG_IGN) != SIG_IGN)
signal(SIGINT, sig_int);
```
### exec
呼叫 exec 會把所有 signal 的設定條回 default，除非該項被設為 ignore
### tips of signal
系統會自動把 backgroup process 的只有 foregroud 接收到相關資訊的 signal 設為 ignore
### interrupt
可以在 slow systemcall 會被 forever block 的情況可以被 interrupted\
errno 會是 EINTR\
所以可以用這點來打斷 forever block\
BSD 有設計被 interuptted 就會自動 restart\
不同系統有些可以設定，或是寫死，或是不會 autorestart
## reentrant function
有正面表列出 reentrant 的 function

### call twice
有時候處理 signal 的 function 用到被 interrupt 的 function，裡面又有 static 出現，會出現問題\
static 會被 compiler 放在 global variable，但存取會被 compiler 擋住

### non-reentrant function
不能被 call twice 的 function，如裡面有 static, global, heap\
也就是不會在 call twice 遇到被存取到相同位置

### Example of non-reentrant function
```c
Those which use static data structures
Those which return a pointer to static data
Those which call malloc() or free()
Those which are part of the standard I/O library – usage
of global data structures (e.g., printf(), ctime(), gmtime())
Those which are POSIX.1 system database functions
(e.g., getgrgid(), getgrnam(), getpwuid() & getpwnam())
and POSIX.1 process environment functions (e.g.,
getlogin(), ttyname())
Those which call non-reentrant functions
```
signal 去對 errno 的防止 reentrant 的方法是，會先去存 errno，並在 handler return 前回復\
longjump 也是 non-reentrant\
如 printf 是 non-reentrant 但因為發生機率極低，所以還是用它來 debug

### reentrant function solution
把要用到的變數由使用者提供 pointer 來存取，這樣在 reentrant 時就不會改到另一個 call 這個 function 那邊的變數

## reliable signals
### unreliable signals
signal could get lost\
指的是感覺沒有 catch 到\
像是因為以前 signal 只有註冊一次有效，而在處理前再次設定前又收到一個，所以就以預設處理\
或是先把收到訊號用 global variable 去存起來，之後再處理，會遇到當開始去確認有沒有訊號時還沒有訊號，但是此時 context switch 收到，然後接回等待 signal(pause) 時，就等不到 signal 造成 deadlock\
也就是 non-atomic 有 Race condition 造成問題
### reliable signal 的做法
signal generated 時會先 set flag 然後等排到 CPU 才會去把 signal deliver 給 process，中間的時間就被稱為 pending\
通常是在 context switch 間的時間去 generate\
可以有很多 process 認領 signal 但至少要一個\
reliable signal 的處理方式是在 deliver 的時候 block 住，等到 unblock 或 ignore
#### `sigpending()`
可以去看哪些 signal 被 block
#### signal mask
和 cwd 跟 umask 一樣在 process 中記錄

### deliver times
標準沒說多個相同的 signal 會怎麼在 unblock 時去 deliver，通常實作是只會一次\
多個不同的則會去把最嚴重的先 deliver

## kill and raise
`#include <sys/types.h>`
`#include <signal.h>`

### `int kill(pid_t pid, int signo)`

會把 signal 送給指定 process\
`pid`  == 0 會給所以有 process 有相同 gid\
`pid` < 0 則是送給所有 gid = |pid| 的\
`pid` == -1 broadcast signals under SVR4 and 4.3+BSD\
如 Worktation shutdown 前可以送 signal 到所有線上 User\
r-UID **或** e-UID 要一樣才能送，也有些系統會去確認 r-UID 或 saved set-UID\
`signo` == 0 (NULL signal) 是用來確認知道 process 是不是 alive 但會有 TTC != TTU 的問題，如果不是 alive 會 return -1\
signal 送給自己且沒被 block 會在 return 前去 check 有沒有被 pending 的狀態（包含剛剛送的 signal），直到至少一個 deliver 才會 return
### `int raise(int signo)`
送 signal 給自己

## alarm and pause
`#include <unistd.h>`
### `unsigned int alarm(unsigned int secs)`
幾秒要送一個 SIGALRM 給自己\
會把之前設定的 alarm 取消掉\
會把前面鬧鐘剩餘秒數 return\
會比設定的多一些時間，所以設定要保守些\
alarm(0) 可以 reset\
要設定接收的反應，因為預設是 termination

### `int pause(void)`
會暫停直到收到 沒被 blocking 也非 ignore 的 signal\
都會 return -1，errno = EINTR

## setjump and longjump
`#include <setjmp.h>`\
可以 non local goto
### Stack Frame
fucntion 的 那塊區域\
每 call 一次 funtion 就會出現\
會存 return address, passed parameters, save registers（CPU 之前的狀態）, local variable\
return adress 的概念基本上就是會落在 call fuction 的下一行\
裡面的 local variable 如果有做初始化，會由 compiler 產生的組合語言把它填成設定的值\
compiler 會去優化跟預測接下來的行為來最佳化，所以亂用 goto 會讓 compiler 的預測效率降低\
沒有 initialized 的 global variable 在 exec 會把整塊清零\
設定 local variable 是 compiler\
return 完 pop 掉其實不會被真正刪掉，只會動 pointer，但在重新 push（不論是不是同一個）時會重新設定\
所以當 set file pointer 的 buffer 在 function 中宣告，會出現又 call function 時會出問題，但是不會有 error

### `jmp_buf`
是 machine-dependent\
通常會包含 CPU registers, stack pointer 以及 return address
### `int setjmp(jmp_buf env)`
會 return 0 當 call 自己，當被 longjmp 會 return longjmp 的 val\
通常會把 env 設為 global variable\
### `int longjmp(jmp_buf env, int val)`
jump 到 有 setjmp 為 env 的 function 並在那邊 return val\
可以用來一次跳回很多層，但是 stack 的 local vairable 基本上是不會回復\
不能回到已經 return 的 function（要在 VM 的 stack 中）

#### sleep2 實驗
```c
static void sig_alrm(int signo)
{
	longjmp(env_alrm, 1);
}
unsigned int sleep2(unsigned int nsecs)
{
	if(signal(SIGALRM, sig_alrm) == SIG_ERR)
		return(nsecs);
	if(setjmp(env_alrm) == 0)
	{
		alarm(nsecs);
		pause();
	}
	return(alarm(0));
}
```
先註冊 signal 然後當 alarm 響了會跳到 `sig_alrm`，不管 pasue 是否執行到，會到 if 所以不會被 pause 影響到（不會從 pause return）\
可以解決 race condition，因為不論是什麼情況都會跳回 return\
假設收到 SIGINT 也會中斷\
但會造成中間假設有在處理其他的 signal hadler，會被中斷，然後假設處理到一半，因為處理 SIGALRM 的 longjmp 使得另一邊的 signal hadler 還沒跑完就結束了。\
還是有 open source 會用，因為以邏輯上沒出錯，然後對於這個問題不重視（如 error number 沒被還原）

## slow system call 遇到 signal
### read 遇到 alarm
如果還沒 read 到東西，收到 SIGALRM 會使得 read return -1，errno == EINTR
### auto restart
BSD 有提出一些 slow system call 被特定 signal interrupt 會 auto restart\
所以遇到需要自己設定 timeout 會需要確定系統沒有對用到的 system call 沒有 auto restart

## Signal Mask
每個 process 有自己的 signal mask\
是一個類似 bit string 的東西\
希望有一段時間可以把 signal block 住，因為那時處理的事情會跟 signal handler 衝突\
預設都是 0 
### signal sets
`#include <signal.h>`
`sigset_t` 是用來存 signal numbers 的變數型態
```c
int sigemptyset(sigset_t *set);
int sigfillset(sigset_t *set);
int sigaddset(sigset_t *set, int sig_no);
int sigdelset(sigset_t *set, int sig_no);
int sigismember(const sigset_t *set, int sig_no);
```
### `int sigprocmask(int how,const sigset_t *set, sigset_t *oset)`
如果 oset 不是 NULL 會先把目前的 sigset 複製到這\
再來 set 如果不是 NULL 會去 check how 是什麼，set 是用來傳參數的 mask\
然後會把 return 前所有 unblock 的 pending signal 至少 deliver 一個後再去把 set 設定到 sigset 中然後 return\
所以如果 unblock 的話會立即去處理 signal
#### `SIG_BLOCK`
把 set 的跟 sigset 做聯集後去設定到 sigset
#### `SIG_UNBLOCK`
做交集
#### `SIG_SETMASK`
直接 replace
### `int sigpending(sigset_t *set)`
會把 set 設定成正在 pending 的 signal\
不一定會是 block 的 signal，可能是沒被 block 單純只是還沒被 deliver\
但通常單純的 pending 不會太久，所以通常還是 block
### inherited
#### fork
signal mask 跟 dispositions（signal 的 signal handlers）會繼承\
peding signal 跟 alarm 因為對象是 process 所以不會繼承
#### exec
signal mask, pending signals, alarm 的 time left 會繼承\
diposition(signal handler) 因為會把原本的程式區塊清掉所以不會清掉

### `sigaction(int signo, const struct sigaction *act, struct sigaction *oact);`
用來取代 `signal()`，改正了不能確認之前的設定跟在處理時額外 block 的效果，也可以一次註冊多次有效\
如果 act 不是 NULL 會改 signal hadler\
如果 oact 不是 NULL 會回傳現在註冊的狀態\
現在 signal 會用 sigaction implement
#### `struct sigaction`
```c
struct sigaction {
	void (*sa_handler)(int);
	sigset_t sa_mask;
	int sa_flags;
	/* alternate handler */
	void (*sa_sigaction)
	(int, siginfo_t *, void *);
};
```
`sa_mask` 是設定把在跑 sigaction 時會額外 block 的 signal，也會再去 block 住自己的 signal\
`sa_flag` 是可以設定要怎麼處理\
`sa_sigaction` 是可以用不同參數的 signal handler 可以用特定 `sa_flag` 啟用\

#### `sa_flag`
|**`SA_INTERRUPT`**|`SA_NOCLDSTOP`|**`SA_NOCLDWAIT`**|**`SA_NODEFER`**|`SA_ONSTACK`|**`SA_RESETHAND`**|**`SA_RESTART`**|**`SA_SIGINFO`**|
|---|---|---|---|---|---|---|---|
|被 interrupt 的 sysytem call 不會 auto restart|不會去接收產生的 `SIGCHILD`|自己的 child 不會變 zombie，用 wait 時，要所有的 child 才會 return，等同 SIGCLD|不會在呼叫時把自己用 mask block 起來| |呼叫一次 sigaction 一次有效|會 auto restart 被 signo interrupt 的 system call|使用 `sa_sigaction` 當作 signal handler|

System V 有一個 SIGCLD 是可以不用 wait，然後 wait return 是所有 child 的結束才會 return

#### `sa_sigaction`
接收的 `siginfo_t` 可以接收資訊
每個版本不一樣，但可以列出一些大家都有的
```c
 struct siginfo {
  int    si_signo;  /* signal number */
  int    si_errno;  /* if nonzero, errno value from <errno.h> */
  int    si_code;   /* additional info (depends on signal) */
  pid_t  si_pid;    /* sending process ID */
  uid_t  si_uid;    /* sending process real user ID */
  void  *si_addr;   /* address that caused the fault */
  int    si_status; /* exit value or signal number */
  long   si_band;   /* band number for SIGPOLL */
  /* possibly other fields also */
};

```
#### signal implement by sigaction
後面很多 signal 就是使用 sigaction impement 的
```c
#include "apue.h"

/* Reliable version of signal(), using POSIX sigaction(). */
Sigfunc *
signal(int signo, Sigfunc *func)
{
    struct sigaction    act, oact;

    act.sa_handler = func;
    sigemptyset(&act.sa_mask);
    act.sa_flags = 0;
    if (signo == SIGALRM) {
#ifdef SA_INTERRUPT
       act.sa_flags |= SA_INTERRUPT;
#endif
    } else {
#ifdef  SA_RESTART
        act.sa_flags |= SA_RESTART;
#endif
    }
    if (sigaction(signo, &act, &oact) < 0)
        return(SIG_ERR);
    return(oact.sa_handler);
}
```

## sigsetjmp & siglongjmp
```c
int sigsetjmp(sigjmp_buf env, int savemask);
void siglongjmp(sigjmp_buf env, int val);
```
會可以設定之後恢復 sigmask 不管是不是 empty\
把 sigmask 存在 `env` 當 savemask != 0

### jmp 的處理
當 signal handler 設定會 jmp 到 setjmp 但此時收到時 setjmp 還沒設定好會出問題\
有時候就會設定 global variable 來去在 longjmp 前確認 set 了沒，然後在 setjmp 完設定

## `int sigsuspend(const sigset_t *sigmask)`
把 sigprocmask  跟 pause 做成 atomic\
也就是會把 sigmask 設定成輸入的值然後 wait\
一定會 return -1 然後 errno == EINTR\
return 會恢復成之前的 sigmask\
所以如果在 sigsuspend 前設定了 sigprocmask 來擋 critical reigion 的話（sigsuspend 開始等），要記得用在 sigsuspend 後把原本怕被 call 的 signal 從 sigmask 中移除後重新設定
### critical reigion
signal 絕對不能被 deliver 的區塊
## signal IPC
如果有需要在一開始先去 block 住 signal 的話，要在 parent 的 sigmask 設定好後再 fork 使得 child 繼承，才不會出現在剛 fork 完就收到 signal 的狀況

## sleep
因為要不要保留前面的 alarm 秒數是 implementation issue，所以這邊不處理\
會把前面的 signal handler 暫時存起來，再去 set alarm 然後處理，當中間被叫醒就把回傳值設定成剩下秒數，最後再把把之前的 signal handler 恢復，也就是假設有之前的 alarm 的 hadler 會不處理

```c
#include "apue.h"

static void
sig_alrm(int signo)
{
    /* nothing to do, just returning wakes up sigsuspend() */
}

unsigned int
sleep(unsigned int nsecs)
{
    struct sigaction    newact, oldact;
    sigset_t            newmask, oldmask, suspmask;
    unsigned int        unslept;

    /* set our handler, save previous information */
    newact.sa_handler = sig_alrm;
    sigemptyset(&newact.sa_mask);
    newact.sa_flags = 0;
    sigaction(SIGALRM, &newact, &oldact);

    /* block SIGALRM and save current signal mask */
    sigemptyset(&newmask);
    sigaddset(&newmask, SIGALRM);
    sigprocmask(SIG_BLOCK, &newmask, &oldmask);

    alarm(nsecs);

    suspmask = oldmask;
    sigdelset(&suspmask, SIGALRM);    /* make sure SIGALRM isn't blocked */
    sigsuspend(&suspmask);            /* wait for any signal to be caught */

    /* some signal has been caught,   SIGALRM is now blocked */

    unslept = alarm(0);
    sigaction(SIGALRM, &oldact, NULL);  /* reset previous action */

    /* reset signal mask, which unblocks SIGALRM */
    sigprocmask(SIG_SETMASK, &oldmask, NULL);
    return(unslept);
}
```


## abort
會送一個 SIGABRT\
當 user 有註冊 SIGABRT 的 signal handler 會先執行然後還不會 return，然後再換成 default call SIGABRT 來結束程式（不 return）\
會出現問題在 longjmp 到其他地方（因為就不會繼續往下跑）、exit（因為要異常 terminate 變正常 terminate）\
ANSI C 是說 file stream flush 跟是否要砍掉暫檔是 implementation issue\
POSIX 是定義不允許 block 跟 ignore SIGABRT，且會 flush file stream\
abort 在原先 signal handler 有 longjmp 的話會逃掉
```c
void
abort(void)         /* POSIX-style abort() function */
{
    sigset_t           mask;
    struct sigaction   action;

    /*
     * Caller can't ignore SIGABRT, if so reset to default.
     */
    sigaction(SIGABRT, NULL, &action);
    if (action.sa_handler == SIG_IGN) {
        action.sa_handler = SIG_DFL;
        sigaction(SIGABRT, &action, NULL);
    }
    if (action.sa_handler == SIG_DFL)
        fflush(NULL);           /* flush all open stdio streams */

    /*
     * Caller can't block SIGABRT; make sure it's unblocked.
     */
    sigfillset(&mask);
    sigdelset(&mask, SIGABRT);  /* mask has only SIGABRT turned off */
    sigprocmask(SIG_SETMASK, &mask, NULL);
    kill(getpid(), SIGABRT);    /* send the signal */
    /* 
     * 因為 kill 會在 return 前至少 deliver 一個包含自己剛剛送出的 non-block & non-ignore signal 才會 return
     * 所以把其他都 block 掉，然後等待 SIFABRT 抵達
    */
    
    /*
     * If we're here, process caught SIGABRT and returned.(not default handler)
     */
    fflush(NULL);               /* flush all open stdio streams */
    action.sa_handler = SIG_DFL;
    sigaction(SIGABRT, &action, NULL);  /* reset to default */
    sigprocmask(SIG_SETMASK, &mask, NULL);  /* just in case origin handler block it again */
    kill(getpid(), SIGABRT);                /* and one more time */
    exit(1);    /* this should never be executed ... */
}
```
### ANSI C
如果成功跑完會 flush buffer 然後 delete 所有 temp file
### POSIX
如果成功終止程式，會 fclose 所有 file
### implement
如果原本 SIGABRT 是 ignore 會把它設定成 default
然後 fflush
## variable type（補充）
### normal
就是一般的變數，在最佳化可以被操作
### static
在 function 內部時會把值在跳出時依然存下來（用全域變數實踐）\
在 module（c 檔）中作為非 body(function) 變數不能被外部存取（區域變數）
### const
常數，只可讀不可寫
### register
推薦放 variable 到 register，但只有不要放在 register 的才能絕對確定\
當 register 有空會放到 register，但就會沒有 address
### volatile
宣告說不論在哪邊改數字，跟判斷中間可能都會被更改（不要放到 regiter 去）\
也就是讓 compiler 知道部會只看 function 直接跑\
一定有 address，且不能被 compiler 自行判斷有關這個的結果因爲前面結果而取消
```c
static int foo;
void bar (void){
foo = 0;
	while(foo != 255)
		;

}
```
可能會被 compiler 優化改成
```c
static int foo;

void bar (void){
	foo = 0;
	while(true)
		;
}
```
如果是中間接到 signal 改了 foo 會出問題
### restrict
指的 source 是不能被用兩次，目前是給 programmer 看，compiler 不一定要實作
也就是不能有 overlaping
