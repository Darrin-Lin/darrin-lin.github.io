---
title: NTU Computer Network Ch.5 Network Layer - Control Plane
date: 2025-12-15 0:00:00.000000000 +0800 CST
tags: [NTUCSIE, Computer Network]
categories: [NTU CSIE CN]
math: true
---
## Network-layer functions
### forwarding
把 packet 從 input interface 送到 output interface 的過程，Data Plane
### routing
決定 packet 從 source 到 destination 的路徑，Control Plane
#### structuring network control plane
- per-router control
  - traditional
  - 每個 router 都有自己的 control algorithm component
- logical centralization
  - software-defined networking (SDN)
  - remote controller

## routing protocols
目標是找到一個 "good" path 從 source 到 destination
### Routing algorithm classification
又兩個方向：global <--> decentralized, dynamic<-->static
#### global
"link state" algorithm\
所有的 router 都有 complete toplogy 跟所有的 link cost info
#### decentralized
"distance vector" algorithm\
iterative 的過程，每個 router 只知道鄰居的 info（只會跟鄰居交換）
#### dynamic
routes 的變化很頻繁
#### static
routes 很少變化
## Dijkstra's algorithm
- **centralized**
  - 對所有 nodes，network topology 與 link cost 已知
  - link state broadcasts 這些資訊
  - node 的 info 都一樣
- iterative
  - 當 k 次 iteration 結束時，會找到從 source 到 k 個 nodes 的 shortest path
- $O(n^2)$ time complexity，可以透過 priority queue 改成 $O(n log n)$
- $O(n^2)$ message complexity
### Route Oscillations
當有大流量的 traffic 時，link cost 會變高，導致 routing algorithm 重新計算 path\
如果所有的 router 都同時重新計算 path，會導致 route oscillations，也就是大家都擠一起，cost 又變高，然後又重新計算 path，如此反覆\
通常把 link state broadcasts 的時間 randomize 錯開更新時間，可以減少這個問題
## Distance Vector Algorithm
### Bellman-Ford equation
$$d_x(y) = min_v { c(x,v) + d_v(y) }$$
- $d_x(y)$: 從 node x 到 node y 的最短距離
- $c(x,v)$: 從 node x 到鄰居 node v 的 link cost
- $d_v(y)$: node v 宣告的從 v 到 y 的最短距離
### key idea
每個 node 只需要知道鄰居的 distance vector，就可以計算出自己到其他 node 的 cost，如果 cost 有變化，就把新的 distance vector 傳給鄰居
- WAIT for change in local link cost or distance vector from neighbor
- UPDATE distance vector
- NOTIFY neighbors if distance vector changes
### por and cons
- pros
  - good news travels fast
    - 當 link cost 變低時，會很快被傳播出去
- cons
  - bad news travels slow
    - 當 link cost 變高時，會很慢被傳播出去，可能會導致 count-to-infinity 問題
### count-to-infinity problem
當 link cost 變高時，可能會導致 routing loop，造成 distance vector 不斷增加\
example: A-60-B-1-C，A 到 C 的 cost 變成 infinity，但 A 會從 B 那邊收到 cost 61 的資訊，B 會從 A 那邊收到 cost 62 的資訊，如此反覆
### link state vs distance vector
| feature               | link state                | distance vector          |
|-----------------------|---------------------------|--------------------------|
| algorithm type        | global                    | decentralized            |
| information required  | complete topology        | neighbor's distance vector|
| robustness to failures | less robust               | more robust              |
| convergence speed     | fast                      | slow                     |
| message complexity    | $O(n^2)$                   | exchange with neighbors  |
| computational complexity | $O(n^2)$               | slow and not specified     |
## internet approach to routing
### autonomous systems (AS)
把 routers 聚集在一個 region 內，稱為 AS(a.k.a. domains)
- intra-AS routing
  - AS 內部的 routing
  - 由 AS 自己決定使用什麼 routing protocol
  - "edge" 的 routers 會跟其他 AS 的 edge routers 溝通
- inter-AS routing
  - AS 之間的 routing
在 local 時，通常會選擇 shortest path\
在 large scale 的部分，只要不要 worst case 就好\
forwarding table 是由 inter-AS 跟 intra-AS 共同決定的


## intra-ISP routing
常見的有 RIP(Routing Information Protocol), OSPF(Open Shortest Path First), EIGRP(Enhanced Interior Gateway Routing Protocol)
### RIP
- distance vector protocol
- 每 30 秒廣播 distance vector 給鄰居
- 現在沒什麼用
### EIGRP
- Distance vector based
- Cisco proprietary
### OSPF
- link state protocol
  - 可以有多個 link cost metrics (bandwidth, delay, load, reliability)
  - 使用 Dijkstra's algorithm 計算 forwarding table
- IS-IS protocol(ISO standard, not RFC standard) essentially similar as OSPF
- 是 public available 的 protocol(open)
- authentication
- heirarchical routing with areas
  - 把 network 分成多個 area，由一個 backbone area 連接所有 area
    - Link-State Advertisement (LSA) 只在 area 內或 backbone area 廣播
    - 對其他 area 只知道大方向 
  - routers
    - Local routers: 只在同一 area 內
    - Area Border Routers (ABR): 連接 area 跟 backbone area
    - Backbone Routers: 在 backbone area 內
    - Boundary Routers: 連接不同 AS 的 routers
## inter-ISP routing
### BGP(Border Gateway Protocol)
#### provides
- eBGP: neighboring AS 之間的 routing
- Policy: 跟據 policy 決定路徑，而不是單純的 shortest path
- iBGP: same AS 內部的 routing
- Advertise: 透過 eBGP 廣播 network reachability info
#### basics
- BGP session
  - 兩個 BGP routers 之間的 TCP connection
  - 建立 semi-permanent connection（半永久連線）
- advertises
  - 當 AS1 跟 AS2 advertise path AS1, X 代表 promise 可以到達 network X 透過 AS1
#### protocol messages
- OPEN: 建立 BGP session
- UPDATE: advertise 可達的 networks 以及 path attributes
- KEEPALIVE: 當沒有 UPDATE 時，確認 BGP session 還活著，也是 ACKs OPEN messages
- NOTIFICATION: 通報 error 跟 close connection 用
#### path attributes
- AS-PATH: 紀錄從 advertise 的 AS 到目前 AS 經過的 AS 列表
  - 避免 looping
  - policy decisions（可以不要經過某些 AS）
- NEXT-HOP: 下一個 hop 的 IP address
##### Hot Potato Routing
選擇離自己最近的出口 AS，盡快把 packet 傳出去，而不是選擇最短路徑\
也就是只管最低 intra-AS cost，不管 inter-AS cost
##### achieving policy
對 providers 來說， revenue 很重要\
當不是 customer 的 AS，不會把自己拿不到 revenue 的路徑 advertise 出去，customer 收到的路徑就是那些可以帶來 revenue 的路徑\
對 customers，不會想要免費當 transit：
當連接到兩個 providers 時，不會把其中一個 provider 的路徑 advertise 給另一個 provider，避免 provider 把自己當成 transit

### selection of BGP routes
- local preference value attribute: policy decisions
- shortest AS-PATH
- closest NEXT-HOP (hot potato routing)
- additional criteria

