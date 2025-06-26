# ngrep - 网络 grep 工具完整指南

## 名称
ngrep - 网络 grep

## 概述
```bash
ngrep <-hNXViwqpevxlDtTRM> <-IO pcap_dump> <-n num> <-d dev> <-A num> <-s snaplen> <-S limitlen> <-W normal|byline|single|none> <-c cols> <-P char> <-F file> <match expression> <bpf filter>
```

## 描述
ngrep 致力于提供 GNU grep 的大部分常用功能，并将其应用到网络层。ngrep 是一个支持 pcap 的工具，允许您指定扩展正则表达式来匹配数据包的数据载荷。它目前可以识别以太网、PPP、SLIP、FDDI 和空接口上的 TCP、UDP 和 ICMP，并以与更常见的数据包嗅探工具（如 tcpdump(8) 和 snoop(1)）相同的方式理解 bpf 过滤器逻辑。

## 选项详解

### 基础选项

#### `-h` - 显示帮助信息
显示帮助/使用信息。

**示例：**
```bash
ngrep -h
```

#### `-V` - 显示版本信息
显示版本信息。

**示例：**
```bash
ngrep -V
```

### 匹配和搜索选项

#### `-i` - 忽略大小写
忽略正则表达式的大小写。

**示例：**
```bash
# 匹配 "HTTP" 或 "http" 或 "Http" 等
ngrep -i "http"
```

#### `-w` - 按单词匹配
将正则表达式作为单词进行匹配。

**示例：**
```bash
# 只匹配完整的单词 "GET"，不匹配 "FORGET" 中的 "GET"
ngrep -w "GET"
```

#### `-v` - 反向匹配
反转匹配；仅显示不匹配的数据包。

**示例：**
```bash
# 显示所有不包含 "200 OK" 的 HTTP 响应
ngrep -v "200 OK" port 80
```

#### `-X` - 十六进制匹配
将匹配表达式视为十六进制字符串。

**示例：**
```bash
# 匹配十六进制模式
ngrep -X "DEADBEEF"
ngrep -X "0x474554"  # 匹配 "GET" 的十六进制表示
```

### 显示和输出选项

#### `-q` - 安静模式
保持安静；除了数据包头和其载荷（如果相关）外，不输出任何信息。

**示例：**
```bash
# 静默搜索，只显示匹配的数据包内容
ngrep -q "password" port 80
```

#### `-x` - 十六进制和 ASCII 显示
以十六进制和 ASCII 格式转储数据包内容。

**示例：**
```bash
# 同时显示十六进制和 ASCII 内容
ngrep -x "POST" port 80
```

#### `-l` - 行缓冲
使 stdout 行缓冲。

**示例：**
```bash
# 实时查看输出，适合管道操作
ngrep -l "ERROR" | tail -f
```

#### `-W normal|byline|single|none` - 显示模式
指定数据包的替代显示方式（非十六进制模式下）。

**各模式说明：**
- `normal`: 默认模式
- `byline`: 遵循嵌入的换行符，仅在遇到换行符时换行
- `single`: 所有内容（包括 IP 和源/目标头信息）都在一行上
- `none`: 在任何情况下都不换行

**示例：**
```bash
# 适合观察 HTTP 事务的按行显示模式
ngrep -W byline "HTTP" port 80

# 单行显示所有信息
ngrep -W single "GET" port 80
```

#### `-c cols` - 设置控制台宽度
显式设置控制台宽度为 `cols`。

**示例：**
```bash
# 设置显示宽度为 120 字符
ngrep -c 120 "POST" port 80
```

#### `-P char` - 非打印字符替换
指定替代字符来表示显示时的不可打印字符。默认为 `.`。

**示例：**
```bash
# 使用 * 代替不能打印的字符
ngrep -P "*" "login" port 23
```

### 时间戳选项

#### `-t` - 绝对时间戳
每次匹配数据包时打印 YYYY/MM/DD HH:MM:SS.UUUUUU 格式的时间戳。

**示例：**
```bash
# 显示绝对时间戳
ngrep -t "error" port 80
# 输出示例: T 2023/12/07 14:30:25.123456 192.168.1.1:80 -> 192.168.1.100:54321
```

#### `-T` - 相对时间戳
打印 +S.UUUUUU 格式的时间戳，表示数据包匹配之间的增量。指定两次则表示自第一个数据包匹配以来的增量。

**示例：**
```bash
# 显示相对时间戳
ngrep -T "GET" port 80

# 显示自第一个匹配以来的时间
ngrep -TT "POST" port 80
```

### 网络接口选项

#### `-d dev` - 指定网络接口
强制 ngrep 监听指定接口 dev。

**示例：**
```bash
# 监听 eth0 接口
ngrep -d eth0 "SSH"

# 监听 wlan0 接口
ngrep -d wlan0 "HTTP"

# 监听所有可用接口
ngrep -d any "FTP"
```

#### `-p` - 非混杂模式
不将接口设置为混杂模式。

**示例：**
```bash
# 只捕获发送到本机的数据包
ngrep -p "telnet" port 23
```

### 数据包处理选项

#### `-s snaplen` - 设置捕获长度
设置 bpf caplen 为 snaplen（默认 65536）。

**示例：**
```bash
# 只捕获每个数据包的前 1500 字节
ngrep -s 1500 "GET" port 80
```

#### `-S limitlen` - 设置检查长度上限
设置 ngrep 检查的数据包大小上限。用于只查看数据包的前 N 字节。

**示例：**
```bash
# 只检查每个数据包的前 200 字节
ngrep -S 200 "password" port 80
```

#### `-n num` - 限制匹配数量
总共只匹配 num 个数据包，然后退出。

**示例：**
```bash
# 只显示前 10 个匹配的数据包
ngrep -n 10 "login" port 22
```

#### `-A num` - 显示后续数据包
匹配数据包后转储 num 个尾随上下文数据包。

**示例：**
```bash
# 显示匹配数据包后的 3 个数据包
ngrep -A 3 "error" port 80
```

### 特殊显示选项

#### `-e` - 显示空数据包
显示空数据包。通常空数据包会被丢弃，因为它们没有可搜索的载荷。

**示例：**
```bash
# 显示所有数据包，包括空载荷的数据包
ngrep -e "" port 80
```

#### `-N` - 显示子协议编号
显示子协议编号以及单字符标识符（观察原始或未知协议时有用）。

**示例：**
```bash
# 显示详细的协议信息
ngrep -N "data" 
```

### 文件操作选项

#### `-I pcap_dump` - 读取文件
将 pcap_dump 文件输入到 ngrep。适用于任何与 pcap 兼容的转储文件格式。

**示例：**
```bash
# 从文件中搜索模式
ngrep -I capture.pcap "GET"

# 从 tcpdump 生成的文件中搜索
tcpdump -w traffic.pcap port 80
ngrep -I traffic.pcap "POST"
```

#### `-O pcap_dump` - 输出到文件
将匹配的数据包输出到与 pcap 兼容的转储文件。此功能不会干扰正常的 stdout 输出。

**示例：**
```bash
# 将匹配的数据包保存到文件
ngrep -O matches.pcap "error" port 80

# 同时在屏幕上显示并保存到文件
ngrep -O suspicious.pcap "password" port 23
```

#### `-F file` - 从文件读取 BPF 过滤器
从指定文件名读取 bpf 过滤器。这是为熟悉 tcpdump 的用户提供的兼容性选项。

**示例：**
```bash
# 创建过滤器文件
echo "tcp port 80 or tcp port 443" > web_filter.txt
ngrep -F web_filter.txt "GET"
```

### 高级选项

#### `-D` - 回放时间间隔
读取 pcap_dump 文件时，按其记录的时间间隔回放（模拟实时）。

**示例：**
```bash
# 按原始时间间隔回放捕获文件
ngrep -D -I old_capture.pcap "HTTP"
```

#### `-R` - 不降低权限
不尝试将权限降低到 DROPPRIVS_USER。

**示例：**
```bash
# 以 root 权限运行（谨慎使用）
sudo ngrep -R "sensitive_data" port 443
```

#### `-K num` - 终止 TCP 连接
终止匹配的 TCP 连接（类似 tcpkill）。数字参数控制发送多少个 RST 段。

**示例：**
```bash
# 发现恶意连接时终止它们（发送 3 个 RST 包）
ngrep -K 3 "malware" port 80
```

## 匹配表达式

匹配表达式可以是扩展正则表达式，或者如果指定了 `-X` 选项，则是表示十六进制值的字符串。

### 正则表达式示例
```bash
# 匹配 HTTP GET 请求
ngrep "GET /" port 80

# 匹配邮箱地址
ngrep "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" port 25

# 匹配 IP 地址
ngrep "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}"
```

### 十六进制表达式示例
```bash
# 匹配特定的十六进制模式
ngrep -X "DEADBEEF"
ngrep -X "0x474554"  # "GET" 的十六进制
```

## BPF 过滤器

BPF 过滤器用于选择要转储的数据包。如果未给出 bpf 过滤器，将转储在选定接口上看到的所有 IP 数据包。

### 基本过滤器类型

#### 主机过滤器
```bash
# 特定主机
ngrep "GET" host 192.168.1.1
ngrep "POST" src host 192.168.1.100
ngrep "HTTP" dst host google.com
```

#### 网络过滤器
```bash
# 特定网络
ngrep "ssh" net 192.168.1.0/24
ngrep "ftp" src net 10.0.0.0/8
```

#### 端口过滤器
```bash
# 特定端口
ngrep "GET" port 80
ngrep "login" src port 22
ngrep "email" dst port 25

# 多个端口
ngrep "web" port 80 or port 443
```

#### 协议过滤器
```bash
# TCP 流量
ngrep "data" tcp

# UDP 流量  
ngrep "dns" udp port 53

# ICMP 流量
ngrep "ping" icmp
```

### 复合过滤器示例
```bash
# HTTP 请求到特定服务器
ngrep "GET" host 192.168.1.1 and port 80

# 来自特定网络的 SSH 连接
ngrep "SSH" src net 192.168.0.0/16 and port 22

# 非 HTTP 的 TCP 流量
ngrep "." tcp and not port 80 and not port 443

# 大于 1000 字节的数据包
ngrep "." greater 1000

# 小于 64 字节的数据包  
ngrep "." less 64
```

## 实际使用案例

### 网络调试
```bash
# 监控 HTTP 请求
ngrep -W byline "GET|POST" port 80

# 查看 DNS 查询
ngrep -x "" udp port 53

# 监控 SSH 登录尝试
ngrep -i "password|login" port 22
```

### 安全监控
```bash
# 检测可疑登录
ngrep -i "failed|invalid|denied" port 22

# 监控敏感数据传输
ngrep -i "password|credit.*card|ssn" port 80

# 检测端口扫描
ngrep -n 100 "." tcp and dst port 1:1024
```

### 性能分析
```bash
# 监控慢查询
ngrep -t "SELECT.*FROM" port 3306

# 分析 HTTP 响应时间
ngrep -T "200 OK|404|500" port 80

# 监控大文件传输
ngrep "." greater 10000 port 80
```

## 退出状态
- `0`: 匹配到一个或多个帧
- `1`: 未匹配到帧
- `2`: 发生错误
- `3+`: 地狱开始结冰，快跑！

## 作者
由 Jordan Ritter <jpr5@darkridge.com> 编写。

## 报告错误
请将错误报告到 ngrep 的 GitHub Issue Tracker：
http://github.com/jpr5/ngrep/issues

## 注意
ALL YOUR BASE ARE BELONG TO US.
