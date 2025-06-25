---
title: NTNU OS Concurrency
date: 2025-06-01 0:00:00.000000000 +0800 CST
tags: [NTNUCSIE, Operating System]
categories: [NTNU CSIE OS]
math: true
---

## thread
現代有些 GNU/Linux 可能沒有做 process，使用 thread implement
### Process vs thread
| | Process | Thread |
|---|---|---|
| Address space| different | share |

thread 是用 stack 分開來做，heap 是 share 的
## pthread
POSIX thread，很多程式語言都是用 pthread 去 implement 內部的 thread
## problem
因為 share memory 所以會有 race condition，就要對 critical section 做好防護

## x64 RIP register

### mutual exclusion(mutex)
[mutex](https://darrin.cc/p/ntu-sp-thread/#mutex)
互斥，
可以讓一個 thread 進入 critical section 時，其他 thread 不能進入\
可以用 xchg 指令來 implement atomic check and set\
要硬體才能去控制 context switch 的時機點，所以 atomic 要硬體實做

#### test-and-set
就是atomic 的上 lock（如 SP），如果不能上 lock 就會等到能上或是直接退出，實做是 spin lock，
需要 hadrware-software codesign（hardware: atomic; software: policy）
```
test lock == 0
set lock = 1
# atmoic
```
#### spin lock
一直去 busy waiting 去 test（所以非 blocking）

#### spining(busy waiting)
如 SP
#### yielding
就是當確定拿不到就會 block 在那邊，在 schedule 到時，會放棄直接讓給其他 thread，直到拿到條件
#### sleep wile waiting
就是如 SP 所述，會在 block 在那邊，不會被 schedule 到，直到被 wake up

## synchronization primtives
### locks 
相對 cv 比較 low level，但還是會搭配 cv 使用
### condition variables
如 SP 所述，可以用 signal 跟 wait 加上 atomic 來實作
#### mesa semantics
簡單來說就是 while(condition) cv\_wait，比較好實作
會先送 signal 喚醒，再去 unlock，而醒來的會先確認 condition 是否成立，成立就進入 critical section，否則繼續睡。
#### hoare semantics
就是用 if (condition) cv\_wait; critcal，且是 atomic 執行 cv_wait 跟 critical，中間不會 context switch
不好 implement
### semaphores
可以用這個去 implement lock 跟 cv，
會想用來解決 atomic execution, event ordering, producer-consumer\
因為是在 thread，所以是把變數存在 heap 來在同一個 adress space 中 share

是藉由觀察某一個變數的操作，有 semaphore wait 跟 semaphore post 
#### unnamed semaphore(memory-based)
就是在 heap 中的 semaphore，會用在 thread 之間的 communication，
一般來說 unamed semaphore 不能跨 process，但也有可以用 shared memory 的 semaphore\
在 sem\_init 有一個 pshared 來決定是否可以跨 address space\
也就是有 intra-process semaphore（same process）跟 inter-process semaphore（memory mapped）

#### named semaphore(file-based)
就是在 file system 中的 semaphore，會用在 process 之間的 communication，有 sem\_init 等外，
還有 sem\_open, sem\_close, sem\_unlink 等，但就是會比較慢。\
有 intra-process semaphore 跟 inter-process semaphore 的特性（for all seasons）

#### semaphore wait
1. S-- (S is the semaphore variable)
2. the calling thread will block if S < 0

#### semaphore post
1. S++
2. wake up one of the blocking threads (waiting thread)

## deadlock
P(deadlock) $\Rightarrow$ Q(necessory condition)
not Q $\Rightarrow$ not P
### necessary condition
- circular waiting
- hold and wait
- only loker-holder can unlock
- mutual exclusion
#### circular waiting
A waits for B, B waits for A\
唯一在 lock 中可以改變的
#### hold and wait
hold 1 lock and wait for another lock

## non-deadlock bug
### atomicity violation
就是在對同一個 critical section 中不是 atomic 的，造成的 race condition
### ordering violation
因為執行順序因為 context switch 而反了，造成的 race condition。如先 free 再去 allocate

## share memory
share memory 在 fork 時還會對應到同一塊 physical memory，不會改，
因為在 memory 所以所以會比 file I/O 快，但缺點是要用的需求比較複雜