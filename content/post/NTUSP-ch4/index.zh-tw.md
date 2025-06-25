---
title: NTU SP Chapter 04 - Files & Directories
date: 2024-12-15 0:00:00.000000000 +0800 CST
tags: [NTUCSIE, System Programming]
categories: [NTU CSIE SP]
math: true
---

主要在講檔案屬性跟 directory tree

## type.h stat.h
### `stat(const char *pathname, struct stat *buf)`
把 i-node 的東西讀出來
### `fstat(int filedes, struct *buf)`
用 file descriptors 打開
### `lstat(const char *pathname, struct stat *buf)`
可以用來讀捷徑檔這種（用 stat 會讀到目的的 i-node 位置）的檔案，會讀捷徑檔本身

### struct stat
```c
dev_t     st_dev     // Device ID of device containing file.
ino_t     st_ino     // File serial number.
mode_t    st_mode    // File type & mode(permissions)
nlink_t   st_nlink   // Number of hard links to the file. 多少檔案指到這個 i-node
uid_t     st_uid     // User ID of file.
gid_t     st_gid     // Group ID of file.
dev_t     st_rdev    // Device ID (if file is character or block special).
off_t     st_size    // For regular files, the file size in bytes. 
time_t    st_atime   // Time of last access. 
time_t    st_mtime   // Time of last data modification. 
time_t    st_ctime   // Time of last status change. 
blksize_t st_blksize // A file system-specific preferred I/O block size for this object. In some file system types, this may vary from file to file. 檔案在作業系統的最佳 block size 
blkcnt_t  st_blocks  // Number of blocks allocated for this object.
```

## `st_mode`
### mode
有記錄 file type \
在 `<sys/stat.h>`\
有 MACRO 可以去確認檔案形式
| `S_ISREG()` | `S_ISDIR` | `S_ISCHR` | `S_ISBLK` | `S_ISFIFO` | `S_ISLNK` | `S_ISSOCK` |
| --- | --- | --- | --- | --- | --- | --- |
| regular file | directory file | character special file | block special file | pipe or FIFO | symbolic link | socket |

是對 `st_mode` 做 & 運算

### mask
有 `S_(W | R | X 進行組合)(USR(U) | GRP(G) | OTH(O))` 的 macro\
檔案的讀寫執行權限很好理解，就很直觀
#### Directory
R: 可以 list flies under it(ls)\
W: 可以更新 dir，如刪除及新增檔案，不需要檔案本身的權限，因為是把檔案 unlink reference count -1，但是需要資料夾的執行權限，是在動 table 中的東西 \
X(執行): 可以去 search 目錄，去檢查檔案，所以創檔案跟刪檔案都要這個權限，要能 read only 也是要這個權限

## `st_uid` / `st_gid`
處理如要更改如 passwd 的檔案，但是不能改其他人的部分，這時候就要去借其他人（如 root）的權限去讀寫程式，所以 passwd 只需要 root 有 rw，其他都只需要 r\
`set_uid` 是在處理把 user 權限借給其他 user 的 process 的部分
- who you really are(**process** ownser 就算拿到其他 e-UID 用 exec 去執行其他程式，還是原本的 r-UID)
	- real user ID
	- read group ID
- used for file access **permission checks**
	- effective user ID
	- effective group ID
	- supplementary group IDs （因為 user 可以有很多 group，這是 sub-group）
- saved by `exec` functions（用來存 effective 的 UID GID）or super user use `setuid`（沒辦法用 system call 得到）
	- saved set-user-ID
	- saved set-group-ID
EUID 是透過 system call 來讓 kernel space 處理，所以 kernel space 的原理跟 effective UID 無關。
### set-User-ID
在執行檔的 permission 的隱藏（user 的 rwx 前面）bit 如果 on 起來就會把 effective uid 變成**執行檔**的 owner\
`int setuid(uid_t uid);` 如果是 superuser 就會把 r-UID, e-UID, save set-UID 設成 `uid`，否則就只能把 `uid` 設定成 real-UID 或是 save set-UID，不然會噴 error (errno == EPERM)

BSD 以前在沒有 save set-user ID 有用 `int setreuid(uid_t ruid, uid_t guid)` 跟 `int setregid(uid_t rgid, uid_t egid)` 交換 r-UID 跟 e-UID，後來 save set-user ID 出來後也可以交換 e-UID 跟 save set-user ID

`int seteuid(uid_t uid);`\
`int setegid(uid_t gid);`\
可以去設定 e-UID，對 N-user 跟 setuid 一樣，但對 superuser 是可以只設定 e-UID
### set-Group-ID
與 set-User-ID 同理，是在處理 main group 的權限
### 權限最前面 3 bit
第一個 bit 是 set User ID，第二個是 set Group ID，第三個是 sticky bit（之後講）\
如果 on 起來，在原本 user/group x 的位置會變成 s，如果可以執行，那就會是小寫 s，不行就是大寫 S，sticky bit 則是 t，**s** 通常是代表 security

可以運用這個功能讓執行檔案設定其他人的權限有 set UID，目錄只開 x，這樣只有知道目錄的人可以執行，也可以在程式中加一層密碼防護

### access permission
權限在打開後就不會再檢查，所以只要在權限關掉前 open 檔案，還是可以對檔案進行操作\
在檢查 user/group 只檢查 effective 和要操作檔案的要求是否相同

#### `int access(const char *pathname, int mode)`
會去檢查 process 的 r-UID 是否對 mode 有權限，如果有就 return 0，否則 return -1\
mode 是 `R_OK`, `W_OK`, `X_OK`, `F_OK`\
`F_OK` 是檢查檔案是否存在

### 新增檔案的 ownership
UID 是 process 的 E-UID

GID 則不一定，有些看 process E-GID，有些是看 dir 的 GID or dir 是否有開 set-GID，有就去繼承 GID，裡面也有分為要檢查 dir 的 set-GID 跟不需要\
另外，有些系統會是把 set GID 開起來然後把 x 關掉，有 mandertory lock 開起來的功用\
這些設定每個系統都不一樣

## `umask(mode_t cmask)`
把 off 掉想要擋掉的 rwx，在 create mode 前先用 `umask` 歸零比較不會出問題\
process 只會改到自己的 mask，child process 會繼承 parent process 的 mask\
umask 會是個 built-in(internal) command 在 shell，因為要使得 shell 的值也會跟著改，但在其他地方會是 external command（不會改到其他 process）\
umask 用完要記得還原成 0
```c
umask(S_IRGRP | S_IWGRP )
create("bar", S_IRUSR | S_IWUSR | S_IRGRP | S_IWGRP)
```
會創一個 -rw------- 的檔案

## chmod fchmod
chomd 只會動 i-node table 的資料，所以不會動到 disk\
如果用 chmod 對檔案開啟 set-GID/set-UID ，且檔案的 GID 與創檔案的 e-GID 不同，系統會把 set-UID 關掉，但不會報錯

### sticky bit(saved-text bit)
以前是用來管是否在 memory，可以加速，現在因為有 virtual memory 然後檔案讀取也變快，所以已經無此功能\
以前的用意是把它 swap 一份到 memory

現在的功能是在 dir 上 on 起來會使得要對底下檔案進行**刪除**檔案，除了要有 wx 還需要是（資料夾的 owner | 檔案 owner | root）\
tmp 目錄就是例子

有沒有 sticky bit 可以看 other 的 x 是不是 t

## chown fchown lchown
l 就是不會 follow symbolic link 
### limit of ANSI C, POSIX, XPGS3, FIPS
ch.2\
用 macro 來看有沒有定義，如果沒有，就要去在 runtime 看有沒有辦法得到，再不行就自己猜
### owner
只有 root 可以更換
### group
只能更換到自己所屬的 group

## File size
### `st_size`
regular files: 0~max(`off_t`)\
directory files: 16 / 512 的倍數\
Symbolic links: pathname length

實際大小在有 Hole 的會比邏輯大小小，而 du -s 出來的是實際大小，ls -l 出來的是邏輯大小\
當然實際大小也可能比邏輯大小大

## truncate ftruncate
把檔案尾巴砍掉，輸入是要留下的長度\
在有些系統也可以把的大小輸入的比目前大小大，可以製造 hole

fctnl 允許用 `F_FREESP` 把檔案的某些地方 free 掉，來製造 hole


當 funtion call 不管用可以用 fcntl，再不行用 IOcontrol

## UNIX File and Directory
### File
sequence of bytes
### Directory
有如何連往其他檔案的資訊的檔案
### System V
disk 裡面有不同的 partition\
裡面有 file system 裡面開頭有 boot blocks 跟 super block 後面接著很多個的 Cylinder group\
Cylinder group 裡面開頭是 super block 的 copy、cg info、i-node map、i-nodes、Data blocks
### i-node and data blocks
#### i-node
除了檔案的 metadata 外，還有很重要的一點是要記錄檔案所在的 data blocks\
檔案資訊除了檔名外，還會在最前面有 hard link 到 i-node 的 i-node number，然後 i-node 會有 reference count 紀錄有多少檔案連到它
> soft link 是捷徑的 link

檔名跟 i-node 也存在 dir block 中 \
在格式化的 cylinder 裡面可以設定上限的 i-node 數，而這也可能造成 i-node 滿但是 data blcok 還沒滿，所以設定 i-node 是一門學問\
ls -i 可以看 i-node number\
每個 dir 至少會指到兩個 hard-link . 跟 ..\
current path 只需要記 device ID 跟 i-node number

### move file
**在同一個 file system 內**，只需要移動 entry，把 hard link 移動過去

可以把 i-node 串 i-node table 這樣可以增加小檔案能存放的數量，且因為讀取第二層 i-node 會把 data 一起存到 buffer，所以也不會特別慢

### hard link vs soft link
#### hard link
hard link 需要有一個名字跟著\
只有 superuser 可以去創一個連到 dir 的 hard link，N-user 只能創指到 file 的 hard-link\
不能跨 file system

#### soft link(Symbolic link)
捷徑檔\
不限制能指到的地方\
可以跨 file system\
是絕對路徑到指定檔案\
被指到的不知道有沒有被指到，所以可能造成 diangling link\
在系統沒限制的時候可能造成 cycle 無窮迴圈

### link unlink rename remove
link 跟 unlink 是在處理 hard link

#### 檔案刪除的條件
當 reference count 降到 0 且沒有被 open 才會被刪掉\
暫存檔可以運用這個特性把檔案 open 完 unlink，可以使得檔案無法被 ls 到，且程式結束後會自動刪除

#### rename
如果遇到新檔名的檔案已經存在，且有 wx 權限：\
舊檔案和新檔案都是檔案，會先 unlink 掉舊檔案，再去 rename 新檔案\
舊檔案和新檔案都是目錄，且舊檔案是空目錄（只有 . & ..），會先 unlink 掉舊目錄，再去 rename 新檔案，新檔名不能是舊檔名的 prefix（父資料夾）

## symbolic links

#### sybolink readlink
unistd.h\
有些 system link 會 follow link 有些不會，open 就會（連帶 read write 也會）

## file times
| Field | Description | Example | ls-option |
| --- | --- | --- | --- |
| `st_atime` | last-access-time | read | -u |
| `st_mtime` | last-modification-time | write | default |
| `st_ctime` | last-i-node-change-time | chmod chown | -c |

### `int utime(const char *pathname, const struct utiimebuf *times)`
utime.h

當 times 設為 NULL，會設定成現在時間，否則設定成的要設定的時間\
當 times 是 NULL 只需要 (E-UID == file owner UID) or w 權限\
但設定成指定時間要 (E-UID == file owner UID) and w 權限\
在 `time_t` 是 32bit int 所以在 2038 會遇到年份可以記錄的極限\
i-node 的時間無法在這更改

## mkdir rmdir
dir 無法使用 open 的手段還原，因為當可以用 rmdir 已經是空的了

## opendir readdir rewinddir closedir
dirent.h\
`DIR *opendir(const char *pathname)` 可以開啟一個遍歷 dir 的 cursor，但沒有規定一定會對開啟時的外部更改有反應\
`struct dirent *readdir(DIR *dp)` 當遍歷完會 return 通知，因為 dir 是 set 所以不會照字典順序\
rewinddir 會回到開頭\
最好確保 dir 不會動

## chdir fchdir getcwd
如前面所說，cwd 只是記錄 device ID 跟 i-node number\
cd 必須要是 bulit-in command 才能更動 shell 的 cwd\
chdir 在遇到 symbolic link 會被影響，而 getcwd 因為是 follow hrad link 往上走，所以不會被影響\
當 cd 到 `/usr/a/test` 但是 `/usr/a` 是個 symbolc link 指到 `/var/b`，那這樣 cwd 會變 `/var/b/test`

## `st_ino`(i-node number)
可以去看 i-node number
