---
title: NTU Computer Network Ch.1 Introduction
date: 2025-10-20 0:00:00.000000000 +0800 CST
tags: [NTUCSIE, Computer Network]
categories: [NTU CSIE CN]
math: true
---

# edge networks
## circuit switch
去連接需要單獨建立連線，直到斷掉前都會佔用資源在那邊，但可以保證網路頻寬不會被搶走
## packet switching
以 packet 為單位去傳輸資料，會跟其他人搶頻寬，但不會佔用資源，但也有頻寬搶不到時，會有微小機率 drop 掉封包
### store-and-forward
要等整個 packet 都收到後才會傳送到下一個節點，會產生 processing delay 跟 queuing delay
### processing delay
確定封包要送給誰，要對封包進行處理的 delay
### queuing delay
封包在 queue 中等待傳送的 delay（因位進來的封包可以有多個，但出去只有一個）
### FDM(Frequency Division Multiplexing)
頻分多工，把頻寬切成很多小頻寬，分給不同的人使用
### TDM(Time Division Multiplexing)
時分多工，把時間切成很多小時間，分給不同的人使用
### packet switching vs. circuit switching
當使用者數量少時，circuit switching 比較好，傳輸會比較穩定，比較適合 "bursty" data（只在一段時間有傳輸），但會遇到當同時太多人使用，會需要考量 packet delay 跟 buffer overflow 的問題，所以需要設計一些辦法\
當使用者數量多時，packet switching 比較好，因為不會因沒有搶到頻寬而需要長時間等待。

#### example
* 1 Gb/s link
* each user:
    * 100 Mb/s when "active"
    * active 10% of time

**Q: how many users can use this network under circuit-switching and packet switching?**

* **circuit-switching:** 10 users

* **packet switching:** with 35 users, probability > 10 active at same time is less than .0004

**1. Circuit-switching (電路交換) 的計算**

* **概念**：在電路交換中，每個用戶都必須**獨佔**一條固定頻寬的「circuit」。
* **計算**：
    * 鏈路總容量 = 1 Gb/s = 1000 Mb/s
    * 每個用戶需要的容量 = 100 Mb/s
    * 最大用戶數 = (總容量) / (每個用戶需要的容量)
    * 最大用戶數 = 1000 Mb/s / 100 Mb/s = **10 users**
* **結論**：在電路交換下，無論用戶是否 active (active 10% of time)，系統都必須為他保留 100 Mb/s 的頻寬。因此，最多只能有 10 個用戶。

**2. Packet-switching (封包交換) 的計算**

* **概念**：在封包交換中，所有用戶**共享**鏈路。只有當用戶「active」時，他們才會發送封包並使用頻寬。這利用了「統計多工 (statistical multiplexing)」的優勢。
* **問題**：鏈路容量只有 1 Gb/s (1000 Mb/s)。如果**超過 10 個用戶同時 active**（例如 11 個用戶），他們總共會需要 $11 \times 100 \text{ Mb/s} = 1100 \text{ Mb/s}$ 的頻寬，這會超過鏈路負荷，導致網路壅塞。
* **目標**：我們要計算的是：如果我們讓 35 個用戶 (`N=35`) 使用這個網路，發生壅塞（即**超過 10 人同時 active**）的機率有多低？

這是一個典型的**二項分佈 (Binomial Distribution)** 機率問題。

* **$N$** (總試驗次數) = 35 (總用戶數)
* **$p$** (單次成功的機率) = 10% = 0.1 (單一用戶處於「active」狀態的機率)
* **$k$** (成功次數) = 同時 active 的用戶數

我們要計算的是「超過 10 人同時 active」的機率，即 $P(k > 10)$。
$P(k > 10) = P(k=11) + P(k=12) + \dots + P(k=35)$

直接計算這個總和很複雜，所以我們計算它的補集（complement），也就是「最多 10 人 (含) 同時 active」的機率，再用 1 去減。
$P(k > 10) = 1 - P(k \le 10)$
$P(k > 10) = 1 - \sum_{k=0}^{10} P(k)$

二項分佈的機率公式為：
$P(k) = \binom{N}{k} \cdot p^k \cdot (1-p)^{N-k}$
$P(k) = \binom{35}{k} \cdot (0.1)^k \cdot (0.9)^{35-k}$

我們需要計算從 $k=0$ 到 $k=10$ 的機率總和：
$P(k \le 10) = P(k=0) + P(k=1) + \dots + P(k=10)$

（使用機率計算器或軟體計算）
* $P(k=0) \approx 0.0253$
* $P(k=1) \approx 0.0975$
* $P(k=10) \approx 0.0007$

將以上機率相加：
$P(k \le 10) \approx 0.99963$

最後，計算我們想要的「超過 10 人同時 active」的機率：
$P(k > 10) = 1 - P(k \le 10) \approx 1 - 0.99963 = \mathbf{0.00037}$


這意味著，如果使用封包交換，我們可以讓 35 個用戶共享這條鏈路，而發生網路壅塞（即超過 10 人同時需要頻寬）的機率非常非常低（僅約 0.037%）。這就是封包交換相較於電路交換的效率優勢。

## Cable based access

使用 **同軸電纜 (coaxial cable)** 來提供網際網路服務

* 須透過 **Cable Modem Termination System (CMTS)** 與 ISP 核心網路連接
* 傳輸速率取決於共享使用者數量與 ISP 分配的頻寬

### Cable modem

透過 **有線電視網路** 傳輸

* 下行頻寬通常比上行大 (asymmetric)
* 頻寬較寬，但同一區域用戶需 **共享頻寬** (可能發生擁塞)
* 實際速率取決於使用者數量與網路負載


## DSL (Digital Subscriber Line)

利用 **現有電話線 (twisted pair)** 提供數據傳輸

* 每個使用者有獨立的「最後一哩」線路，不必和鄰居共享頻寬
* 須透過 **DSL Access Multiplexer (DSLAM)** 匯入 ISP 網路

### ADSL (Asymmetric DSL)

* 下行速率 > 上行速率 (適合下載需求為主)
* 頻寬較小，但不會與同區使用者競爭
* 傳輸速率與距離有關，距離越遠速率越低

### VDSL (Very-high-bit-rate DSL)

* 速率比 ADSL 快得多，但距離限制更嚴格
* 常搭配光纖到社區 (FTTC) 使用


## Wireless access networks

透過 **無線電波** 提供存取

* 特性：傳輸速率與傳輸距離、頻率範圍存在權衡

### Wireless local area networks (WLANs)
WiFi
### Wide-area cellular access networks
3G, 4G, 5G, satellite

## data center networks
high-bandwidth links 去連接很多的 server
### data center
冷熱通道（舊型）
新型通常用水冷式加上氣冷式（新型）如 H100

## packets
把資料切成 L bits 的 packet，transmission rate R bps（不考慮其他 delay 的情況下）\
packet transmission delay = $\frac{L(bits)}{R(bits/sec)}$ seconds
## media
### guided media
透過實體媒介傳輸，如 copper, fiber（光纖）, coax（同軸電纜）
### Coaxial cable
100 Mbps
### fiber 
錯誤率只有 $10^{-9} ~ 10^{-12}$，速度快，10~100 Gbps
### unguided media
透過無線媒介傳輸，如 radio，目前也有技術是會對指定角度傳輸增價速率跟降低耗電率
### radio
錯誤率到 $10^{-3} ~ 10^{-9}$，速度慢，10~100 Mbps

## internet core
### forwarding
routing 會決定要走哪一條路（透過 routing algorithms），forwarding(switching) 會決定在 router 中要走哪一個 output link 來達成 routing 的決定

# core network
## internet
network of networks
## ISP
Internet Service Providers(ISPs)\
hosts 透過 access ISP 來連上 internet

### global ISP
ISP 會再去連到 regional ISP，再去連到的 global ISP，
global ISP 中間再去透過 IXP（internet exchange point）或是 peering point（兩家關係較好時）連接，
不過因為同一家 ISP 中間連線處理的節點較 IXP 或 peering point 多，所以通常相同 ISP 的內部的連接會比較便宜。\
### content provider network
跟 global ISP 同一層，像 google 這些，或是可以提供服務 archive 服務在上面的公司

# performance
## packet delay and packet loss occur(wire networks)
在 packet queue 中排隊會造成 delay，當 queue 滿了就會造成 loss

## delay
### processing delay
在節點處理封包的時間，包含檢查錯誤、決定路由等
### queuing delay
在 queue 中等待的時間
- a: average arrival rate (packets/sec)
- L: packet length(bits/packets)
- R:  link bandwidth (bit transmission rate) (bits/sec)

traffic intensity $\rho = \frac{L \times a}{R} = \frac{\text{arrival rate of bits}}{\text{service rate of bits}}$

$\rho \approx 0$: avg queuing delay small\
$\rho \rightarrow 1$: avg queuing delay large\
$\rho > 1$: 幾乎是 infinity delay
### transmission delay
把 L bits 的 packet 傳送到 R bps 的 link 上所需要的時間\
= $\frac{L(bits)}{R(bits/sec)}$ seconds
### propagation delay
在媒介中傳播的時間，取決於距離和傳播速度
= $\frac{length\ of\ physical\ link}{propagation\ speed\ in\ medium}$
跟 transmission delay 不同
### total nodal delay
$d_{nodal} = d_{proc} + d_{queue} + d_{trans} + d_{prop}$
### end-to-end delay

### traceroute
測試 delay 的工具，每次送出 3 個封包再往外一個 hub 然後接收回傳（但商用網路有時候不會回），重複直到到目的地（最多往外 30 個 hub）

## loss
可能會由前面一個節點、來源重送，或是沒有動作。

## throughput
從 sender 到 receiver 的 rate(bits/time)，要考量每個節點中間的傳輸量（取最低）
$R_{Server}, R_{Client}, R$ 這三個通常是瓶頸

# security
## packet “sniffing”
在 broadcast 的網路去抓封包（Ethernet, wireless）
## ip spoofing
去偽造 IP src
## Denial of Service(DoS)
瘋狂送請求去塞爆某一臺流量
## lines of defense
### authentication
### confidentiality
### integrity checks
### access restrictions
### firewalls

# layer
每一層都會加一個 header，可以從 header 判讀是跟哪一層溝通
## application
$\rightarrow$**message**\
supporting network applications\
ex: HTTP, IMAP, SMTP, DNS
## transport
$\rightarrow$**segment**\
process-process data transfer\
ex: TCP, UDP
## networks
$\rightarrow$**datagram**\
routing of datagrams\
ex: IP, routing protocol
## link
$\rightarrow$**frame**\
data transfer between neighboring network elements\
ex: Ethernet, 802.11 (WiFi), PPP
## physical
bits “on the wire”
