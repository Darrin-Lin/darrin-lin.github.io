---
title: NTU SP Process Enivroment
date: 2024-12-15 0:00:00.000000000 +0800 CST
tags: [NTUCSIE, System Programming]
categories: [NTU CSIE SP]
math: true
---

## intro
### two key atrribute
- Logical control flow
	- 每一個 process 看起來是獨自使用 CPU
	- impliment by context switch(multitask)
- Private address space
	- main memory 只有自己可以處理
	- impliment by virtual memory

## main function
### strating point
對 programmer 來說 C 的 staring point 是 main（但在 windows 有時候在特殊環境下要稍微改名）\
對 System 的 starting point 是 start-up routine

### start-up routine
compiler 去寫的，會由 compiler 的 linker 連起來\
main start 是 start-up routine 去 call，return 也是給 start-up routine\
start-up routine
```
initialize
Get Environment Vars
	Command-line
	arguments
Call to main
		> main
Clean up
```
`_start` 會去 call `main()`

#### nm
`nm` 是個 command 可以看 object file 的 symbols\
symbol 包括 function, variable, setcion(code, stack...), linked lib\
`-g` 是只有 external symbols

#### readelf
顯示 ELF 檔的資訊\
ELF 的 magic byte 代表是執行檔\
可以用 `-a` 列出所有資訊\
可以看到 Entry point address 是 `_start` 的 address

#### objdump
顯示 object file 的資訊\
`-d` 可以反組譯（disassemble）\
Opcode 是 CPU 看得懂的語言，會在 ISA 中\
machine code 會對應有自己的 assembly\
compiler 用相對位置比較不會出錯，所以會在 jmp 時用一個位址 + 相對位址，同時也是因為在 link 時會比較好處理

### process termination
- Normal termination
	- retrun from main() (`exit(main(argc,argv));`)
	- call `exit()`
	- call `_exit()`, `_Exit`
	- return of last thread（基本上就是 main thread） from its start routine
	- call `pthread_exit` from last thread
- Abnormal termination（都跟 signal 相關）
	- `abort()`
	- terminate by signal
	- last trhead get a cancellation request
## ELF(Executable and Linkable Format) file
Standard library format for object file
- Relocatable object files (`.o`)
- Executable object file （執行檔）
- Shared object files (`.so`)
Generic name（通稱）：ELF binaries\
以前 `a.out` 也是個 format，ELF 相對擁有更好的 support 給 shared libraries
### format
- ELF header
- program header table
- .text section
- .data section
- .bss section
- .symtab
- .rel.txt
- .rel.data
- .debug
- Section header table(require for relocatables)

#### ELF header
- Magic number(\177ELF)
- Type(.o/.so/exec)
- machine
- byte ordering(endien)
- entry point
- size
etc.

#### program header table
- page size
- virtual addresses memory [segments/sections]
- segment size
就是記接下來 section 的資訊\
section 通常是在指 ELF file，segment 通常是用在 VM
#### .text
放 function code 的位置
#### .data
intitialized (static/global) data 會放這邊
#### .bss
uninitialize (static/global) data 會放這\
Block Start by Symbol 所以簡稱 bss，又被笑稱是 Better Saved Space\
就算有資料也是 ELF file 要用，因為沒有初始化，所以也不用存在 ELF 檔案中，所以不用空間
#### .symtab
variable names, procedures names, section names 跟 location 都在這邊紀錄\
這邊每一區以及其他的東西都是 symbol，用 `nm` 可以去看\
可以知道 symbol 在執行時 virtual memory 中的位置，以及在 ELF 的 address\
也就是紀錄 .text section 中每個 symbol 的位置都會在這邊\
在 compiler 中最麻煩的部份

#### .rel.text
.text section 的 relocation info\
有些東西 address 會是解不開（assebly 中會放 0 作為 address，以後 link 時再去解）\
就是紀錄 relocation （有哪些是填 0 的）的，之後 link 時會去找對應的 address\
for code
#### .rel.data
.data section 的 relocation info\
有些東西 address 會是解不開（assebly 中會放 0 作為 address，以後 link 時再去解）\
就是紀錄 relocation （有哪些是填 0 的）的，之後 link 時會去找對應的 address\
for variable
#### .debug
放 debug information\
如 `gcc -g` 的資訊就放這

### linking example
變數有兩個資訊 definitions 跟 references\
definitions 就是在檔案中直接定義就是\
reference 就是外面連過去，有 local 跟 extern 兩種方式，local 基本上就是在同一個檔案就看得到 definition，extern 則是在其他檔案連過去（在 .h 用 extern 來導入其他 .c 的變數或是直接在 .c 寫 extern）這在檢查一開始不會有資訊，在 relocate 才知道，就只能 check prototype\
通常全域 definitions 的通常就會宣告成 static 來避免被外部使用\
refernces 在 compile 時就需要 relocation 
### relocate
就是在 compile 時不知道 address，所以會放 0，等到 link 時再去解\
在 link 前會分開成不同 .o 檔，然後 link 時會依據 section 去拼成一個完整的 executable ELF file\
解不掉就會出現 undefined reference

### compile
1. C preprocessor
2. C compiler
3. Assembler
4. Linker
#### preprocessor
`gcc -E`\
會得到 `.i` 檔\
就是會把 include 的東西都貼進去還有 MACRO 解開
#### compiler
`gcc -S`\
會得到 `.s` 檔\
把 c file compile 成 assembly
#### assembler
`gcc -C`\
會得到 `.o` 檔\
把 assembly 編成 `.o` 檔
#### linker
把 object file 跟 library link 起來\
除非是 dynamic linking，就要把所有 reloaction info 解掉\
把各個 `.text`, `.data`, `.bss` 的資訊取出來合成一份新的完整 ELF 檔（結構參照上面）
## share library vs. static library
### static library
static library 會透過 linker 連到 executive file\
static libary 在 Linux 都是 `.a`\
優點是可以直接跨電腦執行，缺點是檔案大小太大跟更新都要重新編譯\
可以用 `ar` 把多個 .o archive 成一個 static libarary file (.a)
### share libary
不同的程式 call 的都是同一份\
會在執行時一啟動就會連過去\
缺點是在執行時去連的時候 cost 比較高\
gcc 會去優先找 shared library\
gcc 對 share libaray 優先是用 dynamic link，要 static 就要加 `-static`\
有兩種 load 的方法，一種是啟動時就會 link，一種是用到 funtion 再把用到的 library link 進來\
如果要 compile 成 share libary 在 compile 時 `gcc` 除了 `-c` 還要 `-pic` 設定成 position independent code，最後會連成 `.so`\
在 dynamic link 時會有內建的尋找順序，然後也可以用參數加上要去找的地方\
可以用 system call `dlopen` 來自己去找 share library\
load 完基本上就會去放在記憶體\
通常會用 mmap 做

## Virtual Memory
| Virtual Memory |
|---|
| (High Address) |
| kernel space |
| argc, argv, env (in stack)|
| Stack |
| ↓ |
| (share lib, mmap) |
| ↑ |   
| Heap |
| un-initialized data(bss) |
| initialized data |
| text |
| (Low Address) |

### Text segment
來自 ELF file 的 .text section\
exec 時會 Copy .text section 的值\
Read-only usually & sharable（fork 時）\
放 Code 的地方
### Initialize Data segment
來自 ELF file 的 .bss section\
exec 時會 Copy .bss section
放已經 init 好的資料
### Uninitialize Data segment
在 exec 時會把原本空的開出一塊然後 init 成 0
### Stack frame
Call function 就會長出一塊\
裡面要記 return address, passesd parameter（傳遞的參數）, local variable 跟 CPU 裡面要用的 saved registers\
兩個沒有 local variable 然後還沒 return 的 function 吃同一塊 stack（setjmp 達成）不會出事\
jmp 時會儲存的只有在 register 的也就是當 compile 完假設 normal(automatic) 跟 register type 是有限的資源時才會存在 setjmp 的 `jmp_buf`\
`alloca` 是可以在 stack frame allocate 的 system call，也就時不用 free\
stack 最上方會放 command line arguments 跟 enviroment variables，因為 main 會吃這些\
main 最後一個參數的 `char **envp` 指向的是個 string list(array for pointer)，會放 Enviroment variable，裡面的沒有正式定義，都是約定成俗\
`char *getenv(const char *name)` 可以去拿某個 name 的 env string\
`int putenv(const char *name-value)`\
`int setenv(const char *name, const char *value, int rewrite)` rewrite == 0 是不會去改已經有的 name，rewrite == 1 是會對已經存在的 overwrite\
`int unsetenv(const char *name)` 去拿掉某個 name，如果沒有那個 name 不會有 error msg\
如果改了 envp 連到的 env string 就會把去改使得大小變大的東西移到 heap，如新增一項 env 就會把 envp 移到 heap，把 env string 的改長就會把那項移到 heap 然後改 envp 上的 pointer\
env 的格式都是 `Name=value` 
### heap
dynamic allocate memory\
allocate 時通常會考慮 aliment\
在 allocate 時會去要一大塊，然後由 C 去維護 malloc pool\
真正要 heap 的是 `sbrk()` 這個 system call 去條大條小 heap 大小\
`reallocate` 可能會調整 pointer，因為可能後面有東西要換位置
基本上 free 也不會還給系統，會存起來用
### stack & heap 中間
mmap 跟 share library 就會放這

### loader
執行時會把檔案 load 到 memory\
UNIX 會是 `ld.so` 這隻程式去進行

### physical memory
一開始會把一些 physical memory 給 OS 自己用\
每個檔案 4G 這樣多個 process 不可能就使用如此之多的空間，所以會去切成多個 pages\
physical page 跟 virtual 要等同\
不同 process 的個別 virtual page 可能會共用 physical page\
OS 會 Control 怎麼把 virtual memory address 用 page table 來 mapping 到 physical memory address\
因為 virtual memory page 一定遠大於 physical pages，所以不會是一對一\
在 physical page 不夠時，有時後會在 page table 把 virtual page 指向 disk\
過程是會在發現 physical page 不夠時，會把之前的某塊掃到硬碟，接著使用那塊，要用再去掃回來
### address transltion
virtual address 轉成 physical address 會透過硬體跟軟體合作\
硬體存取 page table 本身，軟體去管理內容\
page offset 是紀錄 address 是在 (physical?) page 中要 offset 多少\
每塊 virtual address 最後會留一塊給 page offset，會根據整塊的大小來決定幾個 bit，如 4K = 2^12 就會是 12 bit，指的是 page 的 offset\
page offset 在 virtual address 跟 physical address 都會有\
兩個 address 的長度不一定一樣大\
page table 就是 virtual address 長度扣掉 page address 的長度，如 32bit - 12 bit(4K per page) = 2^20\
virtual address 的前面就是紀錄在 page table 的位置，也就是 Virtual page number(VPN)\
physical address 的前面就會紀錄 physical page number(PPN) 跟 page offse，PPN 的大小就是 physical address 長度扣掉 page address 長度，如 30bit - 12 bit(4K per page) = 2^18\
在 page table 的每一塊會有 valid, access, 跟 PPN\
vaild 是紀錄在 VM 中是否有效（有被用）或是有沒有 map 到硬碟,access 是紀錄權限 rwx，然後就可以用 PPN 算出 physical address 的位置

有時候會宣告變數是 atomic 如 sig_atomic_t 也就是不會跨 page?

