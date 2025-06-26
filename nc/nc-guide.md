# nc (netcat) 完整指南

## 名称
nc – 任意 TCP 和 UDP 连接和监听

## 概述
```
nc [-46AcDCdhklnrtUuvz] [-b boundif] [-i interval] [-p source_port]
   [-s source_ip_address] [-w timeout] [-X proxy_protocol] [-x proxy_address[:port]]
   [--apple-delegate-pid pid] [--apple-delegate-uuid uuid] [--apple-ext-bk-idle]
   [--apple-nowakefromsleep n] [--apple-ecn mode] [hostname] [port[s]]
```

## 描述
nc（或 netcat）实用程序可用于涉及 TCP 或 UDP 的几乎任何用途。它可以打开 TCP 连接、发送 UDP 数据包、监听任意 TCP 和 UDP 端口、进行端口扫描，并处理 IPv4 和 IPv6。与 telnet(1) 不同，nc 可以很好地用于脚本编写，并将错误消息分离到标准错误而不是发送到标准输出。

### 常见用途包括：
- 简单的 TCP 代理
- 基于 shell 脚本的 HTTP 客户端和服务器
- 网络守护进程测试
- ssh(1) 的 SOCKS 或 HTTP ProxyCommand
- 以及更多更多功能

## 选项详解

### 网络协议选项

#### `-4` 强制使用 IPv4
仅强制 nc 使用 IPv4 地址。

**示例：**
```bash
# 强制使用 IPv4 连接到 Google DNS
nc -4 8.8.8.8 53
```

#### `-6` 强制使用 IPv6
仅强制 nc 使用 IPv6 地址。

**示例：**
```bash
# 强制使用 IPv6 连接到 Google DNS
nc -6 2001:4860:4860::8888 53
```

#### `-u` 使用 UDP
使用 UDP 而不是默认的 TCP。

**示例：**
```bash
# 创建 UDP 监听器
nc -u -l 1234

# 连接到 UDP 服务
nc -u example.com 53
```

### 监听和连接选项

#### `-l` 监听模式
指定 nc 应该监听传入连接而不是发起到远程主机的连接。

**示例：**
```bash
# 监听端口 8080
nc -l 8080

# 监听 UDP 端口
nc -u -l 1234
```

#### `-k` 保持监听
强制 nc 在当前连接完成后继续监听另一个连接。只能与 `-l` 选项一起使用。

**示例：**
```bash
# 创建持续监听的服务器
nc -k -l 8080

# 每次连接断开后会自动等待新连接
```

#### `-p source_port` 指定源端口
指定 nc 应该使用的源端口。

**示例：**
```bash
# 使用端口 31337 作为源端口连接
nc -p 31337 example.com 80

# 从特定端口发起连接（需要权限）
sudo nc -p 80 example.com 8080
```

#### `-s source_ip_address` 指定源 IP
指定用于发送数据包的接口 IP。

**示例：**
```bash
# 使用特定本地 IP 地址连接
nc -s 192.168.1.100 example.com 80

# 在多网卡系统中指定出口 IP
nc -s 10.0.0.5 192.168.1.1 22
```

### 超时和间隔选项

#### `-w timeout` 连接超时
如果连接和标准输入空闲超过指定秒数，则静默关闭连接。

**示例：**
```bash
# 5 秒超时连接
nc -w 5 example.com 80

# 快速端口检查，1 秒超时
nc -w 1 -z google.com 80
```

#### `-i interval` 发送间隔
指定发送和接收文本行之间的延迟时间间隔。也会在多端口连接之间产生延迟。

**示例：**
```bash
# 每行之间延迟 1 秒发送
echo -e "line1\nline2\nline3" | nc -i 1 example.com 80

# 端口扫描时每个端口间隔 0.5 秒
nc -i 0.5 -z example.com 20-30
```

#### TCP 保活选项
- `-G conntimeout`: TCP 连接超时（秒）
- `-H keepidle`: 初始 TCP 保活超时（秒）
- `-I keepintvl`: 重复 TCP 保活超时间隔（秒）
- `-J keepcnt`: 重复 TCP 保活数据包次数

**示例：**
```bash
# 设置详细的 TCP 保活参数
nc -G 30 -H 600 -I 60 -J 3 example.com 80
```

### 扫描和调试选项

#### `-z` 端口扫描
指定 nc 应该只扫描监听守护进程，而不向它们发送任何数据。

**示例：**
```bash
# 扫描单个端口
nc -z example.com 80

# 扫描端口范围
nc -z example.com 20-30

# 详细端口扫描
nc -v -z google.com 80 443 22
```

#### `-v` 详细输出
让 nc 提供更详细的输出。

**示例：**
```bash
# 详细连接信息
nc -v google.com 80

# 详细端口扫描
nc -v -z example.com 1-100
```

#### `-n` 禁用 DNS 查询
不对任何指定的地址、主机名或端口进行 DNS 或服务查询。

**示例：**
```bash
# 直接使用 IP 地址，不解析域名
nc -n 8.8.8.8 53

# 快速端口扫描，跳过 DNS 解析
nc -n -z 192.168.1.1 22
```

### 数据处理选项

#### `-d` 不读取标准输入
不尝试从标准输入读取。

**示例：**
```bash
# 仅发送一次性数据后关闭
echo "GET / HTTP/1.0\r\n\r\n" | nc -d example.com 80
```

#### `-c` 发送 CRLF
发送 CRLF 作为行结束符。

**示例：**
```bash
# 与需要 Windows 风格换行的服务器通信
nc -c example.com 25
```

#### `-t` Telnet 协商
导致 nc 对 RFC 854 DO 和 WILL 请求发送 RFC 854 DON'T 和 WON'T 响应。

**示例：**
```bash
# 与 telnet 服务器交互
nc -t example.com 23
```

### 代理选项

#### `-X proxy_protocol` 代理协议
请求 nc 在与代理服务器通信时使用指定的协议。

**示例：**
```bash
# 通过 SOCKS5 代理连接
nc -X 5 -x proxy.example.com:1080 target.com 80

# 通过 HTTP 代理连接
nc -X connect -x proxy.example.com:3128 target.com 443
```

#### `-x proxy_address[:port]` 代理地址
请求 nc 通过指定代理连接到主机名。

**示例：**
```bash
# 通过 SOCKS 代理连接
nc -x socks-proxy.com:1080 target.com 80

# 通过 HTTP 代理连接（SSH ProxyCommand）
nc -x http-proxy.com:3128 -X connect ssh-server.com 22
```

### Unix 域套接字

#### `-U` Unix 域套接字
指定使用 Unix 域套接字。

**示例：**
```bash
# 创建 Unix 域套接字监听器
nc -lU /tmp/socket

# 连接到 Unix 域套接字
nc -U /tmp/socket
```

### 其他选项

#### `-r` 随机端口
指定源和/或目标端口应该随机选择。

**示例：**
```bash
# 随机端口扫描
nc -r -z example.com 1-1000
```

#### `-A` 设置 SO_RECV_ANYIF
在套接字上设置 SO_RECV_ANYIF。

#### `-b boundif` 绑定接口
指定绑定套接字的接口。

**示例：**
```bash
# 绑定到特定网络接口
nc -b en0 example.com 80
```

#### macOS 特定选项
- `--apple-delegate-pid pid`: 为指定 PID 委托套接字
- `--apple-delegate-uuid uuid`: 为指定 UUID 委托套接字
- `--apple-ext-bk-idle`: 标记套接字为扩展后台空闲时间
- `--apple-nowakefromsleep n`: 防止端口在系统睡眠时被查询
- `--apple-ecn mode`: 设置 ECN 模式

## 实用示例场景

### 客户端/服务器模型

#### 基本聊天服务器
```bash
# 服务器端
nc -l 1234

# 客户端
nc 127.0.0.1 1234
```

#### 持久聊天服务器
```bash
# 服务器端（支持多次连接）
nc -k -l 1234

# 客户端可以随时连接和断开
```

### 数据传输

#### 文件传输
```bash
# 接收端
nc -l 1234 > received_file.txt

# 发送端
nc target_host 1234 < file_to_send.txt
```

#### 目录传输
```bash
# 接收端
nc -l 1234 | tar -xvf -

# 发送端
tar -cvf - /path/to/directory | nc target_host 1234
```

### 与服务器通信

#### HTTP 请求
```bash
# 获取网页
echo -n "GET / HTTP/1.0\r\n\r\n" | nc google.com 80

# POST 请求
echo -n "POST /api HTTP/1.1\r\nHost: example.com\r\nContent-Length: 12\r\n\r\nHello World!" | nc example.com 80
```

#### SMTP 邮件发送
```bash
nc localhost 25 << EOF
HELO localhost
MAIL FROM: <sender@example.com>
RCPT TO: <recipient@example.com>
DATA
Subject: Test Email

This is a test email.
.
QUIT
EOF
```

### 端口扫描

#### 基本端口扫描
```bash
# 扫描常见端口
nc -z -v example.com 22 80 443

# 扫描端口范围
nc -z -v example.com 20-30

# 快速扫描（无 DNS 解析）
nc -n -z -v 192.168.1.1 1-1000
```

#### 服务识别
```bash
# 获取服务横幅
echo "QUIT" | nc example.com 20-30

# 获取 SSH 版本
nc example.com 22
```

### 网络代理

#### 简单 TCP 代理
```bash
# 在本地 8080 端口创建到远程服务的代理
nc -l 8080 | nc remote_host 80
```

#### 双向代理（需要命名管道）
```bash
# 创建双向代理
mkfifo /tmp/proxy
nc -l 8080 < /tmp/proxy | nc remote_host 80 > /tmp/proxy
```

### 系统管理

#### 远程 shell（仅限受信任环境）
```bash
# 监听端（危险：仅在安全环境中使用）
nc -l 4444 -e /bin/bash

# 连接端
nc target_host 4444
```

#### 系统监控
```bash
# 监控端口是否开放
while true; do
    nc -z -w 1 example.com 80 && echo "Port 80 is open" || echo "Port 80 is closed"
    sleep 5
done
```

## 注意事项

1. **UDP 端口扫描限制**: UDP 端口扫描总是会成功（即报告端口为开放），使得 `-uz` 标志组合相对无用。

2. **安全考虑**: 使用 `-e` 选项（如果可用）创建反向 shell 时要格外小心，仅在受信任的环境中使用。

3. **防火墙**: 某些防火墙可能会阻止或限制 nc 连接，特别是在端口扫描时。

4. **权限**: 某些操作（如绑定到特权端口 < 1024）需要管理员权限。

## 相关命令
- cat(1) - 连接和打印文件
- ssh(1) - 安全 Shell 远程登录

## 作者
- 原始实现：*Hobbit* ⟨hobbit@avian.org⟩
- IPv6 支持重写：Eric Jackson ⟨ericj@monkey.org⟩
