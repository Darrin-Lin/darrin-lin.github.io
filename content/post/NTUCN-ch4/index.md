---
title: NTU Computer Network Ch.4 Network Layer - Data Plane
date: 2025-12-15 0:00:00.000000000 +0800 CST
tags: [NTUCSIE, Computer Network]
categories: [NTU CSIE CN]
math: true
---

## data plane and control plane
### data plane
負責選擇傳送給哪個輸出\
local per-router
### control plane
負責 end-to-end routing
network-wide logic\
#### approaches to control plane
- traditional routing algorithms
  - implemented in routers
- software-defined networking (SDN)
  - control plane 從 router 分離出來，集中在一個 (remote) servers(controller) 裡面
  - controller 會有 network-wide 的 view，可以做更好的決策
## router architecture
### routing process
負責決定封包的路由路徑
### high-speed packet fabric
負責快速轉發封包，連接著 input/output ports
### input ports
- Line Termination
  - 負責 physical layer bit-level reception
- Link-layer Protocols(receive)
  - 負責 link-layer frame 的處理
- lookup, forwarding, queuing
  - lookup: 檢查封包標頭中的欄位值（在傳統路由器中，這只看目標 IP 位址 ）。
  - forwarding:拿著這個目標 IP 位址，去查詢儲存在輸入埠記憶體中的「轉發表 (forwarding table)」
  - queuing: 當 output port 忙碌時，將封包暫存起來（Head-of-the-Line blocking）
#### Longest prefix matching
當一個封包的目標 IP 位址同時符合轉發表中的多筆規則時，路由器必須選擇前綴最長（也就是最精確）的那一筆規則來轉發封包 。
#### input port queuing
當 input port 的速率比 switch fabric 的速率還快時，input port 會將封包暫存在佇列中，等待傳送到 switch fabric。\
Head-of-the-Line (HOL) blocking: 當一個封包在 queue 的最前面，但被其他 queue 的 packet 擋住（像是 output port 被佔用），導致整個 queue 都無法前進。
### Switch fabrics
將 packet從 input link 轉移到正確的 output link。
- first generation
  - Switching via memory
  - 封包會先從 input port 被複製到路由器的系統記憶體中，然後 CPU 查詢轉發表，再將封包從記憶體複製到對應的 output port。
  - 速度受到記憶體頻寬的限制，因為一個封包必須兩次跨越系統匯流排（一次寫入記憶體，一次讀出），效能較差 。
- second generation
  - Switching via bus
  - 封包從 input port 的記憶體，透過一條共享的匯流排（bus），直接傳送到 output port 的記憶體。
  - 因為所有封包都必須共用這條匯流排，會產生「匯流排爭用」（bus contention），其交換速度的上限就是匯流排的總頻寬 。例如，Cisco 5600 路由器使用 32 Gbps 的匯流排，這對於存取路由器（access router）來說通常足夠 。
- third generation
  - Switching via interconnection networks
  - 使用多條平行的匯流排或交錯網路（interconnection network）來連接 input port 和 output port。（但是一個出口 / 入口還是只能同時處理一個封包）
### output ports
#### Buffering
當 datagram 抵達 output port 速率超過 link 的傳輸速率時，output port 會將 datagram 暫存在 buffer 中，等待傳送到 link 上。
#### Drop policy
當 buffer 滿了，output port 必須決定要丟棄哪個
#### Datagram lost
Datagrams can be lost due to congestion, lack of buffers
queueing (delay) and loss due to output port buffer overflow!
#### Scheduling discipline
當緩衝區中有多個封包在排隊時，路由器下一個應該先 transmission 哪一個
#### priority scheduling
根據封包的優先權來決定傳送順序
### buffer size
當太大的 buffer 會導致延遲增加，太小的 buffer 會導致封包丟失
#### RFC 3439 rule of thumb
B=RTT×C
#### More recent recommendation
N flows,\
buffer size = $\frac{RTT×C}{\sqrt{N}}$
### buffer management
當 buffer 滿了，決定要丟棄哪個封包
#### drop-tail
當 buffer 滿了，丟棄新到的封包
#### priority
當 buffer 滿了，根據封包的優先權決定丟棄順序
#### marking
當 buffer 快滿了，標記封包，讓接收端知道網路擁塞，然後回傳 ACK 給傳送端，讓傳送端減少傳輸速率\
- ECN (Explicit Congestion Notification)
  - 當 buffer 快滿了，標記封包的 ECN 欄位
- RED (Random Early Detection)
  - 當 buffer 滿了，隨機丟棄封包
### scheduling
#### FCFS
First-Come-First-Served
#### priority scheduling
根據封包的優先權來決定傳送順序，分成不同 queue 去排隊
#### round robin
每個 queue 輪流傳送一個封包
#### weighted fair queueing (WFQ)
每個 queue 根據權重輪流傳送封包
$\frac{w_i}{\sum_j w_j}$

## network layer
path selection and forwarding, IP protocol, ICMP protocol
### ICMP
Internet Control Message Protocol\
用來傳送錯誤訊息和診斷資訊的協定
### IP Datagram format
20 bytes TCP header + 20 bytes IP header = 40 bytes +apps overhead for TCP/IP
## IP addressing
### IPv4 address
32 bits, 分成 4 個 8 bits 的 octet，用十進位表示法來區分不同 host/router interface
### interface
host/router 跟 physical link 連接的地方，通常是一個網卡 (network interface card, NIC)\
host 通常一到兩個 interface(wired/wireless)
#### router 
基本上就是有兩張以上的網卡（interface），把不同網路連接起來（n 張網卡就有 n 個不同的網路）
### subnet
可以在不通過 intervening router 的情況下 physically connect 的一
#### mask / CIDR(Classless Inter-Domain Routing)
/24 = 把前面 24 bits 遮住，後面視為同一個 subnet\
longest prefix matching: 當一個封包的目標 IP 位址同時符合轉發表中的多筆規則時，路由器必須選擇前綴最長（也就是最精確、mask 最大）的那一筆規則來轉發封包 。
### 動態 IP
192.168.x.x
### get IP
- hard-coded in config file(`/etc/rc.config`)
- DHCP (Dynamic Host Configuration Protocol)
  - host 開機時，向 DHCP server 要 IP address
## DHCP
1. host broadcast DISCOVER message(optional)
2. DHCP server respond with OFFER message(optional)
3. host send REQUEST message to DHCP server
4. DHCP server send ACK message to host

除了 IP address，還會給 subnet mask, first-hop router(default gateway), DNS server address\
使用 UDP，有時效（lease）
### Hierarchical addressing
當某一個 static IP 是某個大的 ISP 所有的大塊 IP底下，但是換了 ISP，要不改 IP address 會使用將 broad cast mask 的大小增加，因為 router 是 longest prefix matching，會找最大 mask 的 rule，在剩下範圍比較小的 subnet 就會歸到新的 ISP 底下，而不會去找原本的 ISP
## ICANN
負責管理 全球的 IP address 和 TLD，IP address 分配給 5 個 regional registries
## NAT
Network Address Translation
**NAT** 全名是 **Network Address Translation**（網路位址轉換）。

它的核心概念是：**讓整個區域網路（如家庭或公司）內的所有設備，在對外連線時，共用「同一個」公開的 IPv4 位址**。

這就像是一間大公司只有一個對外的總機號碼，但內部有數百個分機。


### NAT 的運作原理

NAT 路由器透過修改 IP 封包的標頭來運作，具體流程如下（根據您的投影片範例）：

1.  **出去 (Outgoing)**：
    * 當內網的一台電腦（例如 `10.0.0.1`）要傳送封包到網際網路時。
    * NAT 路由器會攔截這個封包，將其**來源 IP** (`10.0.0.1`) 和**來源埠號** (`3345`) 替換成路由器的**公開 IP** (`138.76.29.7`) 和一個**新的埠號** (`5001`)。
    * 這樣，外部世界看到的請求來源就是路由器本身。

2.  **紀錄 (Translation Table)**：
    * 路由器會在內部的 **NAT 轉換表 (NAT translation table)** 中記下一筆資料：
    * `{公開 IP: 138.76.29.7, 埠號 5001} <---> {內網 IP: 10.0.0.1, 埠號 3345}`。

3.  **回來 (Incoming)**：
    * 當外部伺服器回傳資料時，它的目標會是路由器的 `{138.76.29.7, 5001}`。
    * 路由器收到後，查表發現 `5001` 對應的是內網的 `10.0.0.1:3345`。
    * 路由器將封包的**目的 IP** 和**目的埠號**改回內網的數值，然後轉發給該電腦。


### NAT 的優點

1.  **節省 IP address**：只需要向 ISP 申請**一個**公開 IP，就可以讓成千上萬個設備上網。這是解決 IPv4 位址枯竭的主要功臣。
2.  **安全性**：內網的設備對外部網路是不可見的（直接定址），外部無法主動連線到內網設備，提供了一層天然的防火牆保護。
3.  **靈活性**：更換 ISP（更換公開 IP）時，不需要更改內網所有設備的 IP 設定。

### NAT 的爭議與缺點

儘管 NAT 非常普及，但它在網路架構上是有爭議的：

1.  **違反端到端原則 (End-to-End Argument)**：網路層（Layer 3）的設備（路由器）不應該去修改傳輸層（Layer 4）的埠號資訊。
2.  **妨礙 P2P 連線**：如果外部的使用者想要主動連線到內網的某個服務（例如 P2P 下載或架站），會因為沒有公開 IP 而受阻，需要透過「NAT 穿透 (NAT traversal)」等複雜技術來解決。

但無論如何，NAT 已經成為現代網路（家庭、企業、4G/5G）不可或缺的一部分。

## IPv6
**IPv6** (Internet Protocol version 6) 是網際網路協定 (IP) 的最新版本，旨在取代目前主流的 IPv4。

### 1. 為什麼需要 IPv6？ (Motivation)

主要的推動力只有一個：**IPv4 位址不夠用了！**

* **address exhaustion**：IPv4 使用 32 位元位址，總數約 43 億個，已經全數分配完畢 。
* **效能優化**：除了擴充位址，IPv6 也藉機改進了標頭格式，以加速路由器處理封包的速度（forwarding speed） 。
* **服務品質 (QoS)**：IPv6 在標頭中加入了對「資料流 (Flows)」的支援，以利於即時影音等服務 。

---

### 2. IPv6 與 IPv4 的關鍵差異

IPv6 並不僅僅是加長了 IP 位址，它還 **removed** 了許多 IPv4 的舊欄位來簡化處理流程：

| 特性 | IPv4 | IPv6 | 改變原因 |
| :--- | :--- | :--- | :--- |
| **位址長度** | 32 bits | **128 bits**  | 提供幾乎無限的位址空間。 |
| **標頭長度** | 可變 (20 bytes + options) | **固定 40 bytes**  | 固定長度讓硬體處理更快速。 |
| **破碎化 (Fragmentation)** | 路由器可執行 | **不允許**  | 路由器不再負責切分封包，以減輕負擔。如果封包太大，直接丟棄並通知來源端變小。 |
| **檢查碼 (Checksum)** | 有 | **removed**  | 現代鏈路層（如乙太網）和傳輸層（TCP/UDP）都有檢查碼，IP 層再做一次是多餘且耗時的。 |
| **選項 (Options)** | 包含在標頭中 | **removed**  | 改為透過 "Next Header" 指向擴充標頭，主標頭保持精簡。 |

---

### 3. IPv6 封包格式 (Datagram Format)



一個 IPv6 的封包標頭包含以下重要欄位 ：
* **Priority (Traffic Class)**：用來識別封包的優先順序 。
* **Flow Label**：用來標記屬於同一個「資料流」（如視訊串流）的封包 。
* **Next Header**：指出上層協定是什麼（如 TCP, UDP），或是指向下一個擴充標頭 。
* **Hop Limit**：相當於 IPv4 的 TTL (Time To Live)，每經過一個路由器減 1 。
* **Source/Destination Address**：各佔 128 bits 的來源與目的位址 。

---

### 4. 過渡策略：隧道技術 (Tunneling)

由於網際網路太大，不可能在一天之內讓全世界所有路由器同時升級到 IPv6（沒有 "Flag days"） 。因此，我們需要一種機制讓 IPv4 和 IPv6 共存。

**隧道技術 (Tunneling)** 是目前的解決方案：
* **概念**：將整個 IPv6 封包當作「**資料 (Payload)**」，封裝在一個 IPv4 的封包裡面 。
* **運作**：當兩個 IPv6 路由器中間隔著一段 IPv4 網路時，它們會建立一個「隧道」。
    * 入口路由器把 IPv6 封包塞進 IPv4 封包裡，寄給出口路由器。
    * 中間的 IPv4 路由器只會看到 IPv4 標頭，照常轉發。
    * 出口路由器收到後，剝除 IPv4 外殼，取出原本的 IPv6 封包繼續傳送 。

---

### 5. 採用現況

IPv6 的普及速度比預期慢很多（已經推廣超過 25 年） 。
* 主要原因是 NAT 技術延長了 IPv4 的壽命，減少了升級的急迫性。
* 根據 Google 的統計，截至 2023 年，約有 **40%** 的用戶是透過 IPv6 存取服務 。

## flow
每個 router 都會有 flow table，記錄每個 flow 的狀態\
flow 本身會從 link/network/transport layer header 來定義
### match plus action
對 arrival packet 的 bit 進行 match，然後執行 action
#### destination-based forwarding
根據 destination IP address 來 forward
#### generalized forwarding
根據多個 header fields 來決定 action（包括 drop/copy/modify/log packets）\
會 match 多個 fields (e.g., src/dst IP address, src/dst port, protocol type)\
然後執行 action drop, forward, modify, matched packet or send matched packet to controller\
也有 priority，match 多個 rule 時，選擇 priority 高的 rule\
還有 counter 記錄封包/bytes 數量  
### OpenFlow
**OpenFlow** 是一種通訊協定，它是現代 **SDN (軟體定義網路)** 架構的基石。

它的核心概念是將網路設備（如switch、路由器）的「控制權」與「轉發權」分離。OpenFlow 允許一個遠端的** controller  (Controller)** 直接透過軟體指令，來修改網路設備中的**流表 (Flow Table)**，從而決定封包該如何被處理 。

這實現了我們先前提到的「**廣義轉發 (Generalized Forwarding)**」。


### OpenFlow 的核心結構：流表 (Flow Table)
在 OpenFlow 架構下，每一個流表項目 (Flow Entry) 都包含三個主要部分 ：

1.  **Match**：
    * OpenFlow 不像傳統路由器只看「目的 IP」。它可以檢查封包標頭中，從**鏈路層**到**傳輸層**的幾乎所有欄位。
    * **可 Match 欄位**：輸入埠 (Ingress Port)、來源/目的 MAC、VLAN ID、來源/目的 IP、IP 協定、TCP/UDP 埠號等 。

2.  **動作 (Action)**：
    * 當封包符合上述條件時，要執行什麼動作？
    * **轉發 (Forward)**：送到特定的物理埠 。
    * **丟棄 (Drop)**：像防火牆一樣直接擋掉 。
    * **修改 (Modify)**：修改標頭欄位（例如 NAT 修改 IP） 。
    * **Forward to controller**：將封包封裝後送給 controller，詢問該怎麼處理 。

3.  **統計 (Stats/Counters)**：
    * 記錄有多少封包、多少 bytes Match 了這條規則 。

### OpenFlow 帶來的變革：設備角色的統一

透過 OpenFlow，我們可以使用**同一套硬體**，透過安裝不同的流表規則，讓它扮演完全不同的網路設備角色 ：

| role | Match | Action |
| :--- | :--- | :--- |
| **Router** | 最長 IP Longest prefix match | forward to next-hop link  |
| **Switch** | 目的 MAC 位址 | forward or flood |
| **Firewall** | IP 位址 + TCP/UDP 埠號 | permit or deny  |
| **NAT** | IP 位址 + 埠號 | rewrite address and port |

### 範例：網路編排 (Orchestration)

OpenFlow 最強大的地方在於**全網控制**。

舉例來說， controller 可以同時設定好幾個 switch（s1, s2, s3）的流表，強制規定：「**只要是來自 Host h5 或 h6 的流量，不管目的地是哪裡，都必須強制經過 s1 和 s2 這條特定路徑**」 。

這在傳統網路中很難做到（傳統路由通常只選最短路徑），但在 OpenFlow SDN 中，這只是 controller 下達幾條規則就能完成的事。