# tcpdump 过滤器表达式深度解析 (Filter Expressions)

`tcpdump` 最强大的功能之一就是其灵活的过滤表达式，它允许您精确地捕获您所需要网络流量，过滤掉海量的无关数据。这篇指南将深入解析其工作原理。

过滤器表达式是一个或多个**原语 (primitives)** 的组合。一个原语通常由一个**标识 (id)**（名称或数字）和一个或多个**限定词 (qualifiers)** 组成。

共有三种不同类型的限定词：

1.  **`type`**: 指定标识名称或数字所指代的类型。例如 `host`, `net`, `port`。
2.  **`dir`**: 指定相对于标识的传输方向。例如 `src`, `dst`。
3.  **`proto`**: 限制匹配的协议。例如 `tcp`, `udp`, `icmp`。

如果一个原语没有指定限定词，则默认使用 `host`。

## 目录
- [1. 基本原语 (Primitives)](#1-基本原语-primitives)
    - [host](#host-hostname)
    - [net](#net-network)
    - [port](#port-portnumber)
    - [portrange](#portrange-start-end)
    - [gateway](#gateway-hostname)
- [2. 按方向过滤 (Direction Qualifiers)](#2-按方向过滤-direction-qualifiers)
    - [src](#src)
    - [dst](#dst)
    - [src or dst](#src-or-dst)
    - [src and dst](#src-and-dst)
- [3. 按协议过滤 (Protocol Qualifiers)](#3-按协议过滤-protocol-qualifiers)
    - [ether, ip, ip6, arp, rarp, tcp, udp, icmp 等](#ether-ip-ip6-arp-rarp-tcp-udp-icmp-等)
- [4. 组合表达式 (Combining Expressions)](#4-组合表达式-combining-expressions)
    - [逻辑与 (and / &&)](#逻辑与-and--)
    - [逻辑或 (or / ||)](#逻辑或-or--)
    - [逻辑非 (not / !)](#逻辑非-not--)
    - [组合使用](#组合使用)
- [5. 高级过滤：访问数据包字节](#5-高级过滤访问数据包字节)
    - [语法](#语法)
    - [常用示例：过滤TCP标志位](#常用示例过滤tcp标志位)

---

## 1. 基本原语 (Primitives)

这些是构成过滤表达式的最基本元素。

### `host [hostname]`
捕获与指定主机 `hostname`（可以是 IP 地址或主机名）相关的所有流量（作为源或目的）。

**示例:**
```bash
# 捕获所有与主机 192.168.1.1 相关的流量
sudo tcpdump host 192.168.1.1

# 捕获所有与 google.com 相关的流量 (tcpdump 会自动解析DNS)
sudo tcpdump host google.com
```

### `net [network]`
捕获与指定网络 `network` 相关的所有流量。可以使用 CIDR 表示法。

**示例:**
```bash
# 捕获 192.168.1.0/24 网段的所有流量
sudo tcpdump net 192.168.1.0/24

# 捕获 10.0.0.0/8 网段，但只捕获源于该网段的流量
sudo tcpdump src net 10.0.0.0/8
```

### `port [portnumber]`
捕获源端口或目的端口为 `portnumber` 的流量。

**示例:**
```bash
# 捕获所有与80端口（HTTP）相关的TCP流量
sudo tcpdump tcp port 80

# 捕获所有与53端口（DNS）相关的UDP流量
sudo tcpdump udp port 53
```

### `portrange [start-end]`
捕获源端口或目的端口在 `start-end` 范围内的流量。

**示例:**
```bash
# 捕获端口范围在 21 到 23 之间的TCP流量 (FTP, Telnet)
sudo tcpdump tcp portrange 21-23
```

### `gateway [hostname]`
捕获将 `hostname` 作为网关的数据包。即数据包的以太网源地址或目的地址是 `hostname`，但IP源地址或目的地址不是。

**示例:**
```bash
# 捕获所有通过网关 my-router 的流量
sudo tcpdump gateway my-router
```

---

## 2. 按方向过滤 (Direction Qualifiers)

限定词 `src` 和 `dst` 可以明确指定流量的方向。

### `src`
源地址。

**示例:**
```bash
# 只捕获源IP地址为 10.0.2.15 的流量
sudo tcpdump src host 10.0.2.15

# 只捕获源端口为 443 的TCP流量
sudo tcpdump src port 443
```

### `dst`
目的地址。

**示例:**
```bash
# 只捕获目的网段为 192.168.0.0/16 的流量
sudo tcpdump dst net 192.168.0.0/16

# 只捕获目的端口为 22 (SSH) 的流量
sudo tcpdump dst port 22
```

### `src or dst`
源地址或目的地址。这是 `host`, `net`, `port` 的默认行为。

**示例:**
```bash
# 以下两条命令等价
sudo tcpdump host 8.8.8.8
sudo tcpdump src or dst host 8.8.8.8
```

### `src and dst`
源地址和目的地址。

**示例:**
```bash
# 捕获源和目的IP都是 192.168.1.10 的流量 (例如，发给自己的包)
sudo tcpdump src and dst host 192.168.1.10
```

---

## 3. 按协议过滤 (Protocol Qualifiers)

限定词 `proto` 用来指定协议。常见的协议包括 `ether`, `ip`, `ip6`, `arp`, `rarp`, `tcp`, `udp`, `icmp`。

如果未指定协议限定词，则所有与原语匹配的协议都会被捕获。

**示例:**
```bash
# 只捕获ICMP流量 (通常是 ping)
sudo tcpdump icmp

# 只捕获目的主机为 8.8.8.8 的TCP流量
sudo tcpdump tcp and dst host 8.8.8.8

# 捕获所有IPv6流量
sudo tcpdump ip6
```

---

## 4. 组合表达式 (Combining Expressions)

可以使用逻辑运算符将多个原语组合起来，构建更复杂的过滤器。

### 逻辑与 (`and` / `&&`)
表示"并且"。

**示例:**
```bash
# 捕获源主机为 192.168.1.100 并且目的端口为 80 的流量
sudo tcpdump 'src host 192.168.1.100 and dst port 80'
```
> **注意**: 当表达式中包含特殊字符时，最好用单引号 `' '` 将整个表达式括起来，以避免shell的解释。

### 逻辑或 (`or` / `||`)
表示"或者"。

**示例:**
```bash
# 捕获目的端口为 80 或 443 的流量
sudo tcpdump 'dst port 80 or dst port 443'
```

### 逻辑非 (`not` / `!`)
表示"不是"。

**示例:**
```bash
# 捕获所有流量，但排除SSH流量（端口22）
sudo tcpdump 'not port 22'

# 捕获所有发往 192.168.1.0/24 网段的流量，但排除主机 192.168.1.1
sudo tcpdump 'dst net 192.168.1.0/24 and not dst host 192.168.1.1'
```

### 组合使用
可以使用括号 `()` 来组织复杂的逻辑关系。**注意，括号在shell中有特殊含义，必须使用反斜杠 `\` 转义，或者将整个表达式用引号括起来**。

**示例:**
```bash
# 捕获主机 a.com 与 b.com 或 c.com 之间的流量
sudo tcpdump 'host a.com and (host b.com or host c.com)'

# 捕获所有非广播或非多播的HTTP或HTTPS流量
sudo tcpdump '(port 80 or port 443) and not broadcast and not multicast'
```

---

## 5. 高级过滤：访问数据包字节

`tcpdump` 允许你直接检查数据包特定位置的字节值，这是其最强大的功能之一。

### 语法
```
proto [ expr : size ]
```
- `proto`: 协议名称，如 `ip`, `tcp`, `udp`, `icmp`。
- `expr`: 从协议头开始的字节偏移量。
- `size`: (可选) 要检查的字节数，可以是1, 2, 或 4。默认为1。

**示例:**
```bash
# 捕获所有IP分片包（IP头第6个字节的 "More Fragments" 位被设置）
# ip[6] & 0x20 != 0
sudo tcpdump 'ip[6] & 32 != 0'
```

### 常用示例：过滤TCP标志位
TCP头部第13个字节（从0开始计数）包含了控制标志位 (Flags)。

`CWR | ECE | URG | ACK | PSH | RST | SYN | FIN`

我们可以通过位运算来过滤特定的标志组合。`tcpdump` 提供了一些方便的别名：`tcp-fin`, `tcp-syn`, `tcp-rst`, `tcp-push`, `tcp-ack`, `tcp-urg`。

**示例:**
```bash
# 捕获所有TCP三次握手中的第一个包 (SYN)
# tcp[13] 的值为 2 (二进制 00000010)
sudo tcpdump 'tcp[13] = 2'

# 更具可读性的写法：
sudo tcpdump 'tcp[tcpflags] & tcp-syn != 0 and tcp[tcpflags] & tcp-ack = 0'

# 捕获所有SYN/ACK包 (三次握手的第二个包)
# 标志位为 SYN 和 ACK，值为 18 (二进制 00010010)
sudo tcpdump 'tcp[13] = 18'
# 或者使用位运算
sudo tcpdump 'tcp[tcpflags] & (tcp-syn|tcp-ack) = (tcp-syn|tcp-ack)'

# 捕获所有RST（连接重置）包
sudo tcpdump 'tcp[tcpflags] & tcp-rst != 0'

# 捕获所有包含PUSH标志的包（通常表示有数据传输）
sudo tcpdump 'tcp[tcpflags] & tcp-push != 0'
```
通过熟练掌握这些过滤表达式，您可以将 `tcpdump` 从一个简单的抓包工具，变成一个强大、精准的网络诊断和分析利器。 