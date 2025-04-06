---
title: NTNU OS Virtualization
date: 2025-04-07 0:00:00.000000000 +0800 CST
tags: [NTNUCSIE, Operating System]
categories: [NTNU CSIE OS]
math: true
---

## memory virtualization
現代因為安全化的關係將 variable 在 vm 的 address 隨機化，ALSR\
可以用 `setarch` 來模擬沒有 ALSR 的程式
```sh
setarch $(uname -m) --addr-no-randomize <program>
```
但不同 process 的 vm 是「理論上」分離的，且不能存取其他的（詳見 SP Process control）

## CPU Virtualization
用 context switch 來可以多個 process 共用

## mulitasking
- ready state (Ready queue)
- running state
- blocked state (Event queue)
Blocked state 是透過 DMA 這個元件去處理 I/O\
SP p.43\
沒在做事時就會把 idle process
### Process in CPU

#### direct execution
直接在 CPU 去進行執行

problems:
- 如果當 process 在 CPU 跑，可以直接獲取 CPU 的完整控制權
- 要如何去切換到其他 process

可以用 restricted operations 解決第一項，用 context switch 解決第二項

- mechanisins(how to choose) => restricted operation & context switch
- policyes (which process to choose in ready state) => algorithm

## restricted operation
user space & kernel space
## mechanisins
### example-RISC-V
- user mode(U-mode)
- supervisor mode(S-mode)
- machine mode(M-mode)
S-mode 跟 M-mode 都可以跑 instruction，U-mode 則是不行
#### CSR register(mstatus)
其中兩個 bit 是紀錄權限（MPP）
- U-mode 00
- S-mode 01
- M-mode 11
	同時也是 privilege 的排序

#### space
U-mode 都在 user space\
另外兩個都在 kernel space\
當需要控制硬體時就會去進到 kernel mode
When the system boot up, it will run \texttt{entry.s} to initialize the \texttt{stack pointer}, then run \texttt{start.c},
set some bits of \texttt{mstatus} register to initialize, such as set MPP to \texttt{supervisor mode($01_2$)} to let the \texttt{mret} as to trap return to \texttt{supervisor mode}.
Also set the \texttt{mepc} register to the address of \texttt{main} function
So \texttt{start.c} will use \texttt{mret} to trap into \texttt{supervisor mode} to run \texttt{main.c}.\
In \texttt{main.c}, it will initialize many things, including memeory paging, process table, inode cache, buffer cache, etc.\
it will set the \texttt{epc} register (program counter) to 0, which means the user program counter is initialized to 0, and also set the stack pointer.
Then it will run \texttt{userinit()} function to set up the environment of \texttt{init} process and set it to user mode.\
Finally, it will run \texttt{scheduler()} function to make itself become the \texttt{scheduler} process and let the system run the first user space process, \texttt{init} process.
#### boot-up
一開始會是 M-mode，然後會跑開機的 machine code 來 Introlization 跟 重要的 table\
接著會設定 CSR 提升到 S-Mode\
然後會去跑第一支程式
接著會去 raise mstatus MPP 到 U-mode

在 boot-up 完第一個跑的 U-mode 程式是 shell\

#### system call
如 SP 所述\
System call 就是個功能讓 User mode 可以存取 kernel mode 的功能\

#### trap
1. save state(PC, registers)
2. raise privilege level
3. jump to correct kernel code

return:
1. lower privilege level
2. reduce states of calling process
3. jump back to PC after trap in user code
### exception vs interrupt
exception 是 sync，錯誤產生的中斷\
interrupt 是 async，外部要求

### switch in diffrernt process
1. Process A trap in system
2. call switch() to Process B(context switch) 同時也會把 kernel 狀態存起來
3. return from trap
### kernel space 
直接控制硬體
## policy(schedualing)
- Performance metrics
- schdualing policies (algorithm)
  - FIFO
  - SJF
  - STCF
  - RR
  - MLFQ
### assumptions
1. set of processes all arrive arrive at once.(所有 process 同時抵達)
2. each just use CPU
3. each has some execution time
4. know execution time

turnarround time: time process start to finish\
turnarround time > execution time(CPU time)\
response time: time process start to start execute
### alogrithm
這邊都是不考慮 I/O time，所以 I/O 要不要 context switch 是看 implementation\
Linux implent 可以看 sched 的 manual
#### FIFO algorithm
如其名，First in First out，執行完才會換下一個\
在執行時間差很多的情況，很容易造成平均時間增加（因為後方要等待較長執行時間的 process 的 turrnarround time 也會被拉長\
#### SJF
Shortest-job first\
因為是照著執行時間排序，所以這會是 average trunarround time 最短的演算法\
但是在 arrive time 不同的情況下，如果先跑到執行時間長的 process，中途來執行時間比較短的 process 會造成 average turnaround time 上升
#### STCF
shrotest time-to-complete first\
就是有 process 進來就比較執行時間看要不要 context switch
### RR
Round Robin\
就是用 time slice 的 algo\
amotise（均攤），time slice 要設定多少，會決定 responce time 跟增加的 context switch 時間

#### MLFQ
Multi Level Feedback Queue\
通常是 framework，可以根據上面的演算法去細調
其中一個想法
- Idea: start with RR
  - 分成多個 waiting queue，然後會從高 priorty 到低 priorty，當被從 running state 踢出時就會降低他的 priorty 丟到較低 priorty 的 queue，跑一段時間就會 reset 來把所有 process 丟回 high priorty 的 waiting queue
#### lottery scheduling
example:
- Process A (40% CPU time)
- Process B (60% CPU time)
draw a lot(抽籤) 如 0~100，在每個時間抽個籤，如果是 40 以下就去跑 A，否則跑 B，當有新的 process 進來就重新分配比例然後繼續抽籤\
實作中可以用 linklist 來編號出每一個點，然後抽到哪個點就可以知道是要跑哪個
#### stride scheduling
devide a large number by the number of ticketd for process\
然後從距離起比較近的去做
example:
- Process A (40% CPU time)
- Process B (60% CPU time)
A: 3600/40 = 90\
B: 3600/60 = 60\
A 每次跑走 90，B 每次跑 60\
A(60) => B(90) => A(120) => B(180) => A(180) => A/B

### performance matrics
#### proportional share(fairness)
分配不同重要程度的 CPU resource/time usage

## Memory Virtualization
### motivation
easy to use
### mechanism
address space
P1(base 1024 for physical memory)
```
ld 100, r1
```
address = base + offset = 1K(1024) + 100 = 1124
P2(base 2048 for physical memory)
```
ld 100, r1
```
address = base + offset = 2K(2048) + 100 = 2148

if P1 want load 1200 to read P2's Virtual memory，OS will know because of the address out of P1's range

會有兩個 register 一個存 base，一個存 bound，當 context switch 時就會存到 PCB 中

### strategy
#### segmentation
不用太注重細節，因為現在沒有人在用\
把 memory 分成不同的 segment，每個 segment 有自己的 base & bound，用 index 來存取\
internal fragmentation：當 memeory 的某 page 塞下某個 process 然後還有剩餘的空間\
internal fragmentation：當 memeory 的 pages 剩下的空間小於能使用的部份\
- bast fit
  - 找當前剩餘空間最小且可以放入目前 segment 的 memory pages
- worst fit
  - 找最多空間得來放入
- first fit
  -找到第一個有辦法某塊 segment 放入的 memory，用 data structure 如 link-list 搭配演算法來去找哪個 segment 最適合放入

#### paging
PMP(Physical memory Protection) register，會存 page bundery，然後有多個 register 存 page table
如果要存取某一塊 page 就會找那個上下的 PMP regiser，一個作為 base，一個作為 bound\
這樣就可以達到 Physical memory Protection\
每一塊 fragment 都是一樣大小的 page

## memory paging
### address translation
會把 Virtual adress 的 offset copy 到 physical address，然後把 VPN 轉換成 PFN\
pyhsical address = | PFN | offset |\
virtual address = | VPN | offset |\
不一定相等\
會有個 register PTBR(Page Table Base Register) 來去存 page table 的 base address\
entry in page table = PTBR + VPN \* (size of PTE) 
PA = PFN << (bits of offset) + offset\

會有 valid bit 來判斷這個 page 是不是有被使用，沒有使用就會被設成 invalid(0)，可以省空間。

### part of Hardware involvement
1. extract VPN from virtual address (fast)
2. compute address of PTE using PTBR (fast)
3. fetch PTE (slow)
4. exract PFN and form physical address (fast)
5. fetch physical address to register (slow)
這樣看起來會比 Base & Bound 慢，因為 Base & Bound 只需要一次 fetch，但是 paging 需要兩次，但是可以用 cache 來解決

### part of OS involvement
- new process
- grow process
- process termination
- context switch(update PTBR)

#### example
example: 64 byte Address space, 16 byte pages\
Virtual page
|Adress Sapce| address | VPN(Virtual page number) | offset |
|---| ---| ---| ---|
| code | 0(16) | 0 | 0~15 |
| data | 16(32) | 1 | 16~31 |
|  | 32(48) | 2 | 32~47 |
| stack | 48(64) | 3 | 48~63 |

PFN == Page Frame Number == Physical Frame Number\
physical memory
| Physical memory | PPN(Physical page number) / PFN (Physical frame number) |
|---| ---| 
|  | 0 | 
| code | 1 |
|  | 2 |
| heap | 3 |
|  | 4 |
| stack | 5 |
| | 6 |
| ... | ... |

##### Content in the virtual page(VPN = 0)
每個 virtual page 需要 4 bits 來表達地址（2^4 = 16）\
所以 64 byte 的 address space 需要 6 bits\
在 VPN = 0 所以前兩 bits 是 `00`\
後面 4 bits 是 `0000` ~ `1111`，所以 16 個 address\
也就是
- VPN
  - Virtual address 前兩個 bits
- offset
  - Virtual address 後面 4 bits

##### translate VPN to PFN
VPN = 0 => PFN = 1\
VPN = 1 => PFN = 4\
VPN = 2 => invalid\
VPN = 3 => PFN = 5

mapping table in risc-v:
|VPN|PFN| otherinfo(3 bits for rwx (PTE(page table entry)) in risc-v) |
|---|---|---|
|0|1|5|
|1|4|7|
|2|invalid|0|
|3|0|7|
```
load 17 R1
```
17 = 010001\
=> VPN = 1, offset = 0001\
in page table |1|4|7|\
=> PFN = 4\
=> physical address = |PFN|offset| = |0100|0001| = 65

### TLB(Table Lookaside buffer)
TLB content 
| VPN | PFN | other bits | valid bit | ASID(address space)|
| --- | --- | --- | --- | --- |
| 1 (process 1) | A | .. | 1 | 1 |
| 1 (process 2) | A | .. | 0 | 2 |

TLB 的 valid 跟 Page Table 不同（是 allocate 好了的意思）\
TLB 的意思是其他人可寫入，減少清空的時間（1 不可用，0 可用）

1. extract VPN from VA
2. check TLB
  a. have value Valid mapping(TLB hit)
    (1). fetch PFN form TLB and form PA
    (2). fetch PA to register
  b. don't have valid mapping(TLB miss)
    (0). raise some expception trap into OS
    (1). use PTBR to locate PTE
    (2). fetch PFN from PTE
    (3). update TLB with translation info
    (4). retry the local instruction
#### locality
temporal locailty: 短時間內會動到同樣的位置\
spatial locality: 附近的位置會被再次存取\
因為 programer 的習慣，TLB 會很有效
amotized analysis（均攤複雜度）後會很有效

### OS involvement
- 空間不足時需要清掉
- context switch VPN 就不同，需要移掉

#### flush
當空間不足，會需要清掉

#### memory size affect
32 bit 電腦因為 register 只能存 4GB 的位址（4 * 2^30），所以 Physical memory 會最多 4GB 

### example
32 bit address space\
32 bit computer

#### why not 8 bit AS
因為這樣最多只能放 256 byte 的 data

然後 4KB page（經驗法則）\
offset 是 12 bits（4KB），20 bits VPN(32-12)\
也就是會有 2^20 個 page = 1M number of virtual page\
1M * 4 (for 32 bit int -> 4 byte) \
=> per process need 4MB for page table
我們這樣 500 個 process 就要 2GB 用在 page table => 需要改善 
#### MiB vs MB
MiB 是 (Mebi byte) 中間 i 是代表 binary\
雖然說應該要用 MiB 但是 MB 比較好稱呼，所以通常就默認 define MB MiB

### linear page table
每一個 address space 的 virtual page 都會有一個 physical page

### multi-level page table
把 page table 切成多個 unit，當那一塊 unit 沒有被使用就不會存到 memory 中\
（unit number != page number）
現在第一個步驟不是 PTBR 了，會先去 page table directory，然後再去 page table\
然後不用 PTBR 而是用 page table directory 的 base address，稱作 PDBR(Page Directory Base Register)

簡單來說就是前面的 page number 現在可以分成好幾塊，第一層是第一個 page directory 去獲得下一層 page table 的位置，最後一層是 page table 裡到指定 memory page 的位置，概念就像是 multi-level 的 i-node table。然後 page table directory 還是有 valid bit 來判斷是否有被使用

1. PDE addr = PDBR + (PDE index $\times$ size of PDE)
2. PTE addr = (PDE.PFN << (SHIFT)) + (PT index $\times$ size of (PTE))
3. PA = (PTE.PFN << (SHIFT)) + offset

也有 valid bit，用來判斷該 entry 對應到的 page table 裡的 page 是否有被使用。

功能是因為每個 page 都需要有 page table，而光是花在 page table 就會需要很多空間。不過用這個方法就可以省下那些閒置的 page table，又因為是第二層或更多，所以所需要的數量就更少。
#### page table directory
PDBR=6
|valid|PFN(Page Frame Number)|
|---|---|
|1|7|
|0|-|
|0|-|
|1|10|

page table PFN = 7
|VPN|PFN|
|---|---|

#### example

- 32 KB Adress space
- 64 byte pages
- 4 byte PTE
  
$32KB = 2^{15} = 15 bits$\
15 - 6 = 9 bits for VPN\
$2^9$ virtual pages = 512\
page table size = $512 \times 4 = 2^{11}$ = 2KB\
number of page table entries = $2^{11} \div 2^{6} = 2^5$\
$\Rightarrow$ page dir index = 5 bits\
$\Rightarrow$ PT index = 4 bits\
$\Rightarrow$ offset = 6 bits



VA(15 bits)
|VPN (page dir index) (5)| VPN(page table index) (4)|offset(6)|
|---|---|---|

-> page dir index -> Page table dir\


Adress space
| |
|---|
|VA|
| |
| |

phy memory
| |
|---|
| |
| |
|PA|
| |

page table
|VPN|PFN|page table unit|
|---|---|---|
|0|1|1|
|1|5|1|
|2|4|1|
|3|2|1|
|4|3|2|
|5|9|2|
|6|8|2|
|7|7|2|
|8|10|3|
|9|11|3|
|10|6|3|
|11|15|3|
|12|12|4|
|13|14|4|
|14|13|4|
|15|16|4|

page table directory\
PDE(Page Directory Entry) 整欄
|valid|PFN|
|---|---|
|1|1|
|1|2|
|1|3|

## memory replacement
### mechanism
Hard Disk 會切一塊給 OS manage 的 swap space
| PFN | Phy memory |
|---|---|
| 0 | $P_2$ $VPN_0$ |
| 1 | $P_1$ $VPN_2$ |
| 2 | $P_2$ $VPN_3$ |
| 3 | $P_1$ $VPN_1$ |

| Block | swap space in HD |
|---|---|
| 0 | $P_2$ $VPN_1$ |
| 1 | $P_1$ $VPN_0$ |
| 2 | $P_2$ $VPN_2$ |
| 3 | $P_1$ $VPN_5$ |

1. extract VPN
2. check TLB for VPN
  a. hit
    (1). fetch PFN from TLB form PA
  b. miss
    (1). caculate addr for PTE
    (2). check is in hard drive in PTE

PTE
| PFN | valid | prot rwx | present |
|---|---|---|---|

way handle page fault

```=
PFN = FindPhysicalPage()
  if (PFN == -1) # no free page
    PFN = evictPage() # kick out some page from memory 
DiskRead(PTE, DiskAddr, PFN)
PTE.present = 1; PTE.PFN = PFN
RetryInstruction() # give a TLB miss
```
就是如果去 page table 裡面找 target page，valid bit 是 0（page fault）且 phyical memory 滿了，就會去把一個 victim page 從 physical memory 踢到 disk（evict page），然後把要找的 target page load 到 physical memory 中，把 Page table 的 target page 的 valid bit 設為 1，victim page 的 valid bit 設為 0。\
然後 evict page 是如果 victim page 的 dirty bit 是 1（不在 disk 時有被改過，像 code 就會是 0）就會把 victim page 去寫回 disk，可以減少 I/O time
#### page-buffering algorithm - water marking
會有 watermark 分別是 Low, High 來標註 memory 使用的程度\
當 memory 使用超過 High watermark 就會開始 evict page\
當 memory 使用超過 Low watermark 就會開始 evict 一些少用的 page\
用來降低 page fault rate

### policy
#### view point
conception framework of page replacement
- phy mem $\xrightarrow{backup}$ swap\
- phy mem $\xleftarrow{cache}$ virtual mem

#### AMAT (Average Memory Access Time)
AMAT = (Time for cache hit) $\cdot$ (probability of cache hit) + (Time for cache miss) $\cdot$ (probability of cache miss) = $T_M + P_{miss} \cdot T_D$\
D for disk\
if 
$
\left\{****
\begin{aligned}
T_M & = 100 ns = 10^2 ns\\
T_D & = 10 ms = 10^7 ns\\
\end{aligned}
\right.
$

if miss raate = 10 % $\Rightarrow$ P_{miss} = 0.1\
AMAT = $10^2 + 0.1 \cdot 10^7 \approx 1ms$\
now if miss rate = 0.1 % $\Rightarrow$ P_{miss} = 0.001\
AMAT = $10^2 + 0.001 \cdot 10^7 \approx 10^4 ns$

不過如果換成 SSD 可以降低 $T_D$ 25 % 到 50 %\ 

#### Belady's anomaly
理論上應該 miss rate 會隨著 cache size 增加而減少，但在某些演算法的特定情況下，會出現 miss rate 增加的情況

#### optimal
理論上的演算法，就是會去把最久沒有被使用的 page 踢出去\
example 中，hit rate = 6/11，miss rate = 5/11，\
但是大部分情況基本上不知道未來的狀況，除非有特別去預測\
會被視為最佳的情況，用來比較用

#### FIFO
新開一個 replace queue，紀錄 page 進入的順序，當 memory 滿了就會把最早進入的 page 踢出去\
#### secode-chance
為了記錄最近使用的 page，又不想新增太多空間，會在 PTE 有一個 use bit\
硬體：每次使用就會去把 use bit 設為 1\
OS：
lazy approach: if use bit = 1, clear bit = 0 increase the the clock pointer; else evict the page, set dirty bit\
就是跟 FIFO 很像，但有一個 use bit(referece bit) 來記錄最近使用的 page。會去從最舊的開始找，如果 use bit 是 1 就會把 use bit 設為 0，然後把 clock  pointer 往下移（移到最後面），然後繼續找，如果 use bit 是 0 就會把那個 page 踢出去，當跑完一圈還沒有找到就重新跑一次，這次就會找到。\
也會去在一段時間把所有 use bit 清掉重來

#### random
就 random 選擇一個 page 踢出去，有時候會有效率
#### LRU(Least Recently Used)
把最久沒用到的 page 踢出去\
沒有 Belady's anomaly（因為 n - 1 的會是 n 的 subset）

##### count implementation
每個 page 有一個 counter，每次使用就會把 counter 設成 clock\
當 memory 滿了就會去找 counter 最小的 page 踢出去\
但這樣需要遍歷 page table，所以會很慢，而且如果要紀錄時間，會需要精度很高，所以空間需求很大

##### stack implementation
用 link-list stack 來記錄最近使用的 page，每次使用就會把用到的 page 放到 stack 最上面\
當 memory 滿了就會把 stack 最下面的 page 踢出去\
會需要 6 pointer 的轉換，有點慢
##### LRU-aprrox implementation
當有被用到時，就把最前面的 bit 設為 1，指定每幾次操作進行一次 >>，然後有指定紀錄的是幾個 bit，當 memory 滿了就會去找數值最小的 page 踢出去。\
1, 5, 7, 2, | 4, 5, 3, 2
| page number | reference bit (t=0) | reference bit (t=1) |
|---|---|---|
| 1 | 1000 | 0100 |
| 2 | 1000 | 1100 |
| 3 | 0000 | 1000 |
| 4 | 0000 | 1000 |
| 5 | 1000 | 1100 |
| 6 | 0000 | 0000 |
| 7 | 1000 | 0100 |

#### counting-based replacement algorithm
##### LFU (Least Frequently Used) 
LFU 會去找最少使用的 page 踢出去，但是會有一個問題，如果有一個 page 一直被使用，但是在某個時間點不再使用，但是因為一直被使用，所以 counter 會一直增加，所以會一直被留在 memory 中\
##### MFU (Most Frequently Used)
MFU 會去找最多使用的 page 踢出去，效果並不好，因為在 temporal locality 的情況下，會一直 miss

#### locality
temporal locality: 最近使用的 page 會在未來被使用\
spatial locality: 附近的 page 會在未來被使用\
這兩項可以增加 hit rate\
#### example
4 VPN, 3 cache entries(phy mem)

page fault(not in cache yet)

time line from top to down
| page access | hit / miss(page fault) | evict | resulting cache state |
| --- | --- | --- | --- |
| 0 | miss | - | 0 |
| 1 | miss | - | 0 1 |
| 2 | miss | - | 0 1 2 |
| 0 | hit | - | 0 1 2 |
| 1 | hit | - | 0 1 2 |
| 3 | miss | 2(optimal) | 0 1 3 |
| 0 | hit | - | 0 1 3 |
| 3 | hit | - | 0 1 3 |
| 1 | hit | - | 0 1 3 |
| 2 | miss | 3(optimal) | 0 1 2 |
| 3 | hit | - | 0 1 2 |

policy 就會在丟進去時，原本的已經滿了，會要決定要丟哪個 page
在 loop 中 LRU 跟 FIFO 都沒有用，因為會在回去之把第一個丟掉，所以會一直 miss