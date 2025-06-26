# iperf3 指南

`iperf3` 是一款用于进行网络吞吐量测试的工具。本文档是 `iperf3` 手册页的中文翻译，并为每个选项提供了示例。

## 目录
- [名称](#名称)
- [概要](#概要)
- [描述](#描述)
- [通用选项](#通用选项)
- [服务器专属选项](#服务器专属选项)
- [客户端专属选项](#客户端专属选项)
- [示例：验证](#示例验证)
- [延伸阅读：进阶概念](#延伸阅读进阶概念)
- [作者](#作者)
- [另请参阅](#另请参阅)

## 名称

**iperf3** - 执行网络吞吐量测试

## 概要

```bash
iperf3 -s [选项]
iperf3 -c 服务器 [选项]
```

## 描述

`iperf3` 是一个用于进行网络吞吐量测量的工具。它可以测试 TCP、UDP 或 SCTP 的吞吐量。要执行 `iperf3` 测试，用户必须同时建立一个服务器和一个客户端。

`iperf3` 可执行文件包含客户端和服务器两种功能。可以使用 `-s` 或 `--server` 命令行参数来启动 `iperf3` 服务器，例如：

```bash
# 启动 iperf3 服务器
iperf3 -s
```

或者

```bash
iperf3 --server
```

请注意，许多 `iperf3` 参数都有短格式（如 `-s`）和长格式（如 `--server`）。在本节中，我们通常使用短格式的命令行标志，除非只有长格式可用。

默认情况下，`iperf3` 服务器在 TCP 端口 5201 上监听来自 `iperf3` 客户端的连接。可以使用 `-p` 标志指定自定义端口，例如：

```bash
# 在 5002 端口启动服务器
iperf3 -s -p 5002
```

服务器启动后，它将监听来自 `iperf3` 客户端的连接（换句话说，以客户端模式运行的 `iperf3` 程序）。客户端模式可以使用 `-c` 命令行选项启动，该选项还需要一个 `iperf3` 应该连接的主机。主机可以通过主机名、IPv4 地址或 IPv6 地址指定：

```bash
# 连接到名为 iperf3.example.com 的服务器
iperf3 -c iperf3.example.com

# 连接到 IPv4 地址为 192.0.2.1 的服务器
iperf3 -c 192.0.2.1

# 连接到 IPv6 地址为 2001:db8::1 的服务器
iperf3 -c 2001:db8::1
```

如果 `iperf3` 服务器在非默认的 TCP 端口上运行，客户端也需要指定该端口号：

```bash
iperf3 -c iperf3.example.com -p 5002
```

初始的 TCP 连接用于交换测试参数、控制测试的开始和结束以及交换测试结果。这有时被称为"控制连接"。实际的测试数据是通过一个单独的 TCP 连接、一个独立的 UDP 包流或一个独立的 SCTP 连接发送的，具体取决于客户端指定的协议。

通常，测试数据从客户端发送到服务器，用于测量客户端的上传速度。通过在客户端上指定 `-R` 标志，可以测量从服务器下载的速度。这会导致数据从服务器发送到客户端。

```bash
# 从服务器下载数据进行测试（反向模式）
iperf3 -c iperf3.example.com -R
```

结果会同时显示在客户端和服务器上。每个测量间隔（默认为一秒，但可以通过 `-i` 选项更改）至少会有一行输出。每行输出至少包括测试开始以来的时间、间隔期间传输的数据量以及该间隔内的平均比特率。请注意，每个测量间隔的值是从发出该输出的端点的角度获取的（换句话说，客户端上的输出显示了客户端的测量间隔数据）。

测试结束时会有一组统计数据，这些数据显示了发送方和接收方所看到的测试摘要（尽可能详细），并相应地标记了行。回想一下，默认情况下客户端是发送方，服务器是接收方，尽管如上所述，使用 `-R` 标志将反转这些角色。

客户端可以通过指定 `--get-server-output` 标志来获取给定测试的服务器端输出。

客户端或服务器都可以通过传递 `-J` 标志，以 JSON 结构生成其输出，这对于与其他程序集成非常有用。通常，JSON 结构的内容只有在测试完成后才能完全知晓，因此在测试结束前不会发出任何 JSON 输出。通过启用 `--json-stream`，将发出多个对象以提供可实时解析的 JSON 输出。

`iperf3` 有一套（非常）庞大的命令行选项，可用于设置测试的参数。它们在下面的手册页的"通用选项"部分中给出，并在 `iperf3` 的帮助输出中进行了总结，可以通过使用 `-h` 标志运行 `iperf3` 来查看。

## 通用选项

### -p, --port n
设置服务器监听/连接的端口为 n（默认为 5201）。

**示例：**
```bash
# 服务器在端口 8080 上监听
iperf3 -s -p 8080

# 客户端连接到端口 8080
iperf3 -c <server_ip> -p 8080
```
---
### -f, --format [kmgtKMGT]
报告格式：`k`=Kbits/sec, `m`=Mbits/sec, `g`=Gbits/sec, `t`=Tbits/sec。大写字母表示法类似，但单位是 KBytes/sec, MBytes/sec 等。

**示例：**
```bash
# 以 Gbits/sec 为单位显示结果
iperf3 -c <server_ip> -f g

# 以 MBytes/sec 为单位显示结果
iperf3 -c <server_ip> -f M
```
---
### -i, --interval n
两次定期吞吐量报告之间暂停 n 秒；默认为 1，使用 0 禁用。

**示例：**
```bash
# 每 5 秒报告一次吞吐量
iperf3 -c <server_ip> -i 5
```
---
### -I, --pidfile file
写入一个包含进程 ID 的文件，在作为守护进程运行时最有用。

**示例：**
```bash
# 启动服务器并将其 PID 写入 /var/run/iperf3.pid
iperf3 -s -D -I /var/run/iperf3.pid
```
---
### -F, --file name
使用文件作为数据的来源（在发送方）或汇点（在接收方），而不是仅仅生成随机数据或丢弃它。此功能用于查找存储子系统是否是文件传输的瓶颈。它不会将 iperf3 变成文件传输工具。接收文件的长度、属性，在某些情况下内容可能与原始文件不匹配。

**示例：**
```bash
# 在发送端，从 /path/to/largefile 读取数据并发送
iperf3 -c <server_ip> -F /path/to/largefile

# 在接收端，将接收到的数据写入 /dev/null
iperf3 -s -F /dev/null
```
---
### -A, --affinity n/n,m
设置 CPU 亲和性（如果可能）（仅限 Linux、FreeBSD 和 Windows）。在客户端和服务器上，您可以使用此参数的 `n` 形式（其中 n 是 CPU 编号）设置本地亲和性。此外，在客户端，您可以使用 `n,m` 形式的参数覆盖该测试的服务器亲和性。请注意，使用此功能时，进程将仅绑定到单个 CPU（而不是可能包含多个 CPU 的集合）。

**示例：**
```bash
# 将服务器进程绑定到 CPU 2
iperf3 -s -A 2

# 将客户端进程绑定到 CPU 3，并将服务器端测试线程绑定到 CPU 5
iperf3 -c <server_ip> -A 3,5
```
**延伸阅读：** [CPU 亲和性](#)
---
### -B, --bind host[%dev]
绑定到与地址主机相关联的特定接口。如果指定了可选接口，则将其视为 `--bind-dev dev` 的快捷方式。请注意，对于 IPv6 链路本地地址文字，需要百分号和接口设备名称。

**示例：**
```bash
# 客户端通过 192.168.1.100 这个 IP 地址发包
iperf3 -c <server_ip> -B 192.168.1.100

# 绑定到 eth1 接口 (IPv6)
iperf3 -s -B fe80::1234%eth1
```
---
### --bind-dev dev
绑定到指定的网络接口。此选项使用 `SO_BINDTODEVICE`，并且可能需要 root 权限。（在 Linux 和可能的其他系统上可用。）

**示例：**
```bash
# 将客户端绑定到 eth0 接口
sudo iperf3 -c <server_ip> --bind-dev eth0
```
---
### -V, --verbose
提供更详细的输出。

**示例：**
```bash
# 运行测试并显示详细信息
iperf3 -c <server_ip> -V
```
---
### -J, --json
以 JSON 格式输出。

**示例：**
```bash
# 将测试结果输出为 JSON
iperf3 -c <server_ip> -J > results.json
```
---
### --json-stream
以行分隔的 JSON 格式输出。

**示例：**
```bash
# 实时获取 JSON 格式的测试结果
iperf3 -c <server_ip> --json-stream | while read line; do echo "Real-time: $line"; done
```
---
### --logfile file
将输出发送到日志文件。

**示例：**
```bash
# 将服务器日志写入 iperf3.log
iperf3 -s --logfile iperf3.log
```
---
### --forceflush
在每个间隔强制刷新输出。用于在将输出发送到管道时避免缓冲。

**示例：**
```bash
# 强制刷新输出，以便通过管道实时处理
iperf3 -c <server_ip> -i 1 --forceflush | grep 'Gbits/sec'
```
---
### --timestamps[=format]
在每行输出的开头添加时间戳。默认情况下，时间戳的格式由 `ctime(1)` 发出。可选地，可以传递 `=` 后跟格式规范来自定义时间戳，请参阅 `strftime(3)`。如果给出此可选格式，`=` 必须紧跟在 `--timestamps` 选项之后，中间没有空格。

**示例：**
```bash
# 添加默认格式的时间戳
iperf3 -c <server_ip> --timestamps

# 添加自定义格式的时间戳 (例如：2023-10-27 10:30:00)
iperf3 -c <server_ip> --timestamps="%Y-%m-%d %H:%M:%S"
```
---
### --rcv-timeout #
设置活动测试期间接收数据的空闲超时。如果接收方在此毫秒数内未从发送方收到数据，则将停止测试（默认为 120000 毫秒，即 2 分钟）。

**示例：**
```bash
# 如果 10 秒内没有收到数据，则服务器终止测试
iperf3 -s --rcv-timeout 10000
```
---
### --snd-timeout #
设置未确认的 TCP 数据的超时时间（在测试和控制连接上）。此选项可用于在测试期间发生网络分区时强制更快的测试超时。所需参数以毫秒为单位指定，默认为系统设置。此功能取决于 `TCP_USER_TIMEOUT` 套接字选项，并且在不支持它的系统上将无法工作。

**示例：**
```bash
# 如果 TCP 数据在 5 秒内未被确认，则客户端超时
iperf3 -c <server_ip> --snd-timeout 5000
```
---
### --use-pkcs1-padding
此选项仅在使用 iperf3 的身份验证功能时才有意义。3.17 之前的 iperf3 版本在 RSA 加密的凭证中使用 PCKS1 填充，这容易受到可能泄露服务器私钥的旁道攻击。从 iperf-3.17 开始，使用 OAEP 填充，但这是一个不兼容旧 iperf3 版本的重大更改。使用此选项可保留安全性较低但兼容性更好的行为。

**示例：**
```bash
# 当与旧版 iperf3 服务器通信时，客户端可能需要使用此选项
iperf3 -c <old_server_ip> --username myuser --rsa-public-key-path public.pem --use-pkcs1-padding
```
---
### -m, --mptcp
为当前协议使用 MPTCP 变体。这仅适用于 TCP 并启用 MPTCP 使用。

**示例：**
```bash
# 使用 MPTCP 进行 TCP 测试
iperf3 -c <server_ip> -m
```
**延伸阅读：** [多路径 TCP (MPTCP)](#)
---
### -d, --debug
发出调试输出。主要（也许完全）供开发人员使用。

**示例：**
```bash
# 运行并输出大量调试信息
iperf3 -s -d
```
---
### -v, --version
显示版本信息并退出。

**示例：**
```bash
iperf3 -v
```
---
### -h, --help
显示帮助摘要。

**示例：**
```bash
iperf3 -h
```
---
## 服务器专属选项

### -s, --server
以服务器模式运行。

**示例：**
```bash
iperf3 -s
```
---
### -D, --daemon
在后台以守护进程方式运行服务器。

**示例：**
```bash
iperf3 -s -D
```
---
### -1, --one-off
处理一个客户端连接，然后退出。如果设置了空闲时间，服务器将在该时间段内没有连接后退出。

**示例：**
```bash
# 服务器在完成一次测试后自动退出
iperf3 -s -1
```
---
### --idle-timeout n
在 `one-off` 模式下，这是服务器在退出前等待连接的秒数。

**示例：**
```bash
# 如果 60 秒内没有客户端连接，服务器将退出
iperf3 -s -1 --idle-timeout 60
```
---
### --server-bitrate-limit n[KMGT]
在服务器端设置一个限制，如果客户端指定的测试速率超过 n 位/秒，或者客户端发送或接收的平均数据（包括所有数据流）大于 n 位/秒，则测试将中止。默认限制为零，表示没有限制。平均数据速率的间隔默认为 5 秒，但可以通过在比特率说明符后添加"/"和数字来指定。

**示例：**
```bash
# 如果客户端请求的带宽超过 100Mbits/s，服务器将拒绝测试
iperf3 -s --server-bitrate-limit 100M

# 如果客户端在 10 秒内的平均速率超过 100Mbits/s，则中止测试
iperf3 -s --server-bitrate-limit 100M/10
```
---
### --rsa-private-key-path file
用于解密来自客户端的身份验证凭证的 RSA 私钥（未受密码保护）的路径（如果使用 OpenSSL 支持构建）。

**示例：**
```bash
# 服务器使用指定的私钥进行客户端身份验证
iperf3 -s --rsa-private-key-path /path/to/private.pem --authorized-users-path /path/to/credentials.csv
```
**延伸阅读：** [iperf3 身份验证](#)
---
### --authorized-users-path file
包含授权用户凭证以运行 iperf 测试的配置文件的路径（如果使用 OpenSSL 支持构建）。该文件是用户名和密码哈希的逗号分隔列表；有关文件结构的更多信息可以在示例部分找到。

**示例：**
```bash
# 服务器使用指定的用户凭证文件
iperf3 -s --rsa-private-key-path /path/to/private.pem --authorized-users-path /path/to/credentials.csv
```
**延伸阅读：** [iperf3 身份验证](#)
---
### --time-skew-threshold seconds
身份验证过程中服务器和客户端之间的时间偏差阈值（以秒为单位）。

**示例：**
```bash
# 允许服务器和客户端之间最多 10 秒的时间差
iperf3 -s --time-skew-threshold 10 --rsa-private-key-path ...
```
---
## 客户端专属选项

### -c, --client host[%dev]
以客户端模式运行，连接到指定的服务器。默认情况下，测试包括从客户端向服务器发送数据，除非指定了 `-R` 标志。如果指定了可选接口，它将被视为 `--bind-dev dev` 的快捷方式。请注意，对于 IPv6 链路本地地址文字，需要百分号和接口设备名称。

**示例：**
```bash
# 作为客户端连接到 192.168.1.5
iperf3 -c 192.168.1.5

# 通过 eth1 接口连接到 IPv6 链路本地地址
iperf3 -c fe80::1234%eth1
```
---
### --sctp
使用 SCTP 而不是 TCP（FreeBSD 和 Linux）。

**示例：**
```bash
# 使用 SCTP 协议进行测试
iperf3 -c <server_ip> --sctp
```
**延伸阅读：** [流控制传输协议 (SCTP)](#)
---
### -u, --udp
使用 UDP 而不是 TCP。

**示例：**
```bash
# 使用 UDP 协议进行测试，速率为 100 Mbits/sec
iperf3 -c <server_ip> -u -b 100M
```
---
### --connect-timeout n
设置与服务器建立初始控制连接的超时时间，以毫秒为单位。默认行为是操作系统的 TCP 连接建立超时。提供一个较小的值可以加快对故障 iperf3 服务器的检测。

**示例：**
```bash
# 如果在 3 秒内无法连接到服务器，则超时
iperf3 -c <server_ip> --connect-timeout 3000
```
---
### -b, --bitrate n[KMGT]
设置目标比特率为 n 位/秒（UDP 默认为 1 Mbit/sec，TCP/SCTP 无限制）。如果有多个流（-P 标志），则吞吐量限制分别应用于每个流。您还可以在比特率说明符后添加"/"和数字。这被称为"突发模式"。它将发送给定数量的数据包而无需暂停，即使这暂时超过了指定的吞吐量限制。将目标比特率设置为 0 将禁用比特率限制（对 UDP 测试特别有用）。此吞吐量限制在 iperf3 内部实现，并在所有平台上可用。与 `--fq-rate` 标志进行比较。此选项取代了 `--bandwidth` 标志，该标志现已弃用但（至少目前）仍被接受。

**示例：**
```bash
# UDP 测试，目标速率为 10 Mbits/sec
iperf3 -c <server_ip> -u -b 10M

# TCP 测试，限制速率为 500 Kbits/sec
iperf3 -c <server_ip> -b 500K

# 突发模式：以 20Mbits/s 的速率发送 1000 个数据包
iperf3 -c <server_ip> -u -b 20M/1000
```
---
### --pacing-timer n[KMGT]
设置起搏计时器间隔，单位为微秒（默认为 1000 微秒，或 1 毫秒）。这控制了 `-b/--bitrate` 选项的 iperf3 内部起搏计时器。计时器按此参数设置的间隔触发。较小的起搏计时器参数值可以使 iperf3 发出的流量更平滑，但可能由于更频繁的计时器处理而牺牲性能。

**示例：**
```bash
# 使用更精细的 200 微秒起搏计时器来平滑流量
iperf3 -c <server_ip> -b 10M --pacing-timer 200
```
---
### --fq-rate n[KMGT]
设置用于基于公平队列的套接字级调速的速率，单位为比特/秒。此调速（如果指定）将是对 iperf3 内部吞吐量调速（`-b/--bitrate` 标志）的补充，并且可以为同一测试同时指定两者。仅在支持 `SO_MAX_PACING_RATE` 套接字选项的平台上可用（目前仅 Linux）。默认不使用基于公平队列的调速。

**示例：**
```bash
# 使用内核的公平队列调速将速率限制为 50 Mbits/s
iperf3 -c <server_ip> --fq-rate 50M
```
**延伸阅读：** [公平队列调速 (Fair-Queueing Pacing)](#)
---
### --no-fq-socket-pacing
此选项已弃用，将被删除。它等同于指定 `--fq-rate=0`。
---
### -t, --time n
传输时间，单位为秒（默认为 10 秒）。

**示例：**
```bash
# 运行测试 30 秒
iperf3 -c <server_ip> -t 30
```
---
### -n, --bytes n[KMGT]
要传输的字节数（而不是 `-t`）。

**示例：**
```bash
# 传输 100MB 的数据
iperf3 -c <server_ip> -n 100M
```
---
### -k, --blockcount n[KMGT]
要传输的块（数据包）数（而不是 `-t` 或 `-n`）。

**示例：**
```bash
# 传输 1 百万个数据块
iperf3 -c <server_ip> -k 1M
```
---
### -l, --length n[KMGT]
要读取或写入的缓冲区长度。对于 TCP 测试，默认值为 128KB。在 UDP 的情况下，iperf3 会尝试根据路径 MTU 动态确定合理的发送大小；如果无法确定，则使用 1460 字节作为发送大小。对于 SCTP 测试，默认大小为 64KB。

**示例：**
```bash
# 将 TCP 读/写缓冲区大小设置为 256KB
iperf3 -c <server_ip> -l 256K

# 在UDP测试中，手动设置包大小为1400字节
iperf3 -c <server_ip> -u -b 100M -l 1400
```
---
### --cport port
将数据流绑定到特定的客户端端口（仅适用于 TCP 和 UDP，默认为使用临时端口）。

**示例：**
```bash
# 客户端使用 5500 端口进行数据传输
iperf3 -c <server_ip> --cport 5500
```
---
### -P, --parallel n
要运行的并行客户端流的数量。iperf3 将为每个测试流派生一个单独的线程。使用多个流可能会比单个流获得更高的吞吐量。

**示例：**
```bash
# 使用 8个并行流进行测试以提高吞吐量
iperf3 -c <server_ip> -P 8
```
---
### -R, --reverse
反转测试方向，以便服务器向客户端发送数据。

**示例：**
```bash
# 测试从服务器到客户端的下载速度
iperf3 -c <server_ip> -R
```
---
### --bidir
双向测试，客户端和服务器同时发送和接收数据。

**示例：**
```bash
# 同时测试上传和下载速度
iperf3 -c <server_ip> --bidir
```
---
### -w, --window n[KMGT]
设置套接字缓冲区大小/窗口大小。此值将发送到服务器并在该端使用；在两端，此选项都设置发送和接收套接字缓冲区大小。此选项可用于（间接）设置最大 TCP 窗口大小。请注意，在 Linux 系统上，有效最大窗口大小大约是此选项指定的两倍（此行为不是 iperf3 中的错误，而是 Linux 内核的"特性"，如 `tcp(7)` 和 `socket(7)` 所述）。

**示例：**
```bash
# 将 TCP 窗口大小设置为 1MB
iperf3 -c <server_ip> -w 1M
```
**延伸阅读：** [TCP 窗口大小](#)
---
### -M, --set-mss n
设置 TCP/SCTP 最大段大小（MTU - 40 字节）。

**示例：**
```bash
# 手动设置 MSS 为 1460 字节
iperf3 -c <server_ip> -M 1460
```
**延伸阅读：** [MTU 和 MSS](#)
---
### -N, --no-delay
设置 TCP/SCTP 无延迟，禁用 Nagle 算法。

**示例：**
```bash
# 禁用 Nagle 算法，适用于需要低延迟的应用
iperf3 -c <server_ip> -N
```
**延伸阅读：** [Nagle 算法](#)
---
### -4, --version4
仅使用 IPv4。

**示例：**
```bash
# 强制使用 IPv4 进行连接
iperf3 -c <server_ip> -4
```
---
### -6, --version6
仅使用 IPv6。

**示例：**
```bash
# 强制使用 IPv6 进行连接
iperf3 -c <server_ip> -6
```
---
### -S, --tos n
设置 IP 服务类型（ToS）。可以使用通常的八进制和十六进制前缀，即 `52`、`064` 和 `0x34` 都指定相同的值。

**示例：**
```bash
# 将 ToS 字段设置为 184 (0xB8)，通常用于视频流量
iperf3 -c <server_ip> -S 184
```
---
### --dscp dscp
设置 IP DSCP 位。接受数字和符号值。数字值可以以十进制、八进制和十六进制指定（参见 `--tos`）。

**示例：**
```bash
# 使用符号值设置 DSCP 为 AF21 (Assured Forwarding Class 2, Low Drop)
iperf3 -c <server_ip> --dscp AF21

# 使用数值设置 DSCP 为 18 (等同于 AF21)
iperf3 -c <server_ip> --dscp 18
```
**延伸阅读：** [DSCP (Differentiated Services Code Point)](#)
---
### -L, --flowlabel n
设置 IPv6 流标签（目前仅在 Linux 上支持）。

**示例：**
```bash
# 为 IPv6 流量设置流标签
iperf3 -c <server_ipv6> -6 -L 12345
```
---
### -X, --xbind name
使用 `sctp_bindx(3)` 将 SCTP 关联绑定到特定的链接子集。如果指定此标志，则将忽略 `--B` 标志。通常，SCTP 在建立关联时会包括本地主机上所有活动链接的协议地址。指定至少一个 `--X` 名称将禁用此行为。必须为要包含在关联中的每个链接指定此标志，并且 iperf 服务器和客户端都支持此标志（后者通过将第一个 `--X` 参数传递给 `bind(2)` 来支持）。主机名作为参数被接受，并使用 `getaddrinfo(3)` 解析。如果指定了 `--4` 或 `--6` 标志，则不会解析为指定协议族内地址的名称将被忽略。

**示例：**
```bash
# 将 SCTP 关联绑定到两个特定 IP 地址
iperf3 -c <server_ip> --sctp -X 192.168.1.10 -X 192.168.2.20
```
---
### --nstreams n
设置 SCTP 流的数量。

**示例：**
```bash
# 在 SCTP 测试中使用 16 个流
iperf3 -c <server_ip> --sctp --nstreams 16
```
---
### -Z, --zerocopy
使用"零拷贝"方法发送数据，例如 `sendfile(2)`，而不是通常的 `write(2)`。

**示例：**
```bash
# 使用零拷贝方法发送文件以提高效率
iperf3 -c <server_ip> -F /path/to/file -Z
```
**延伸阅读：** [零拷贝 (Zero Copy)](#)
---
### --skip-rx-copy
忽略接收到的数据包数据，在`recv(2)`系统调用中使用`MSG_TRUNC`标志。
**示例：**
```bash
# 在接收端忽略数据包内容，只关心元数据，可用于测量网络处理能力
iperf3 -s --skip-rx-copy
```
---
### -O, --omit n
执行 n 秒的预测试并省略预测试统计信息，以跳过 TCP 慢启动阶段。

**示例：**
```bash
# 运行测试，但忽略前 5 秒的结果，以获得更稳定的吞吐量测量
iperf3 -c <server_ip> -t 30 -O 5
```
---
### -T, --title str
在每行输出前加上此字符串前缀。

**示例：**
```bash
# 在每行输出前添加 "Test-Run-1: "
iperf3 -c <server_ip> -T "Test-Run-1: "
```
---
### --extra-data str
指定要包含在 JSON 输出中的额外数据字符串字段。
**示例：**
```bash
# 在 JSON 输出中添加一个名为 "run_id" 的字段
iperf3 -c <server_ip> -J --extra-data '{"run_id":"abc-123"}'
```
---
### -C, --congestion algo
设置拥塞控制算法（仅限 Linux 和 FreeBSD）。此标志的旧同义词 `--linux-congestion` 被接受但已弃用。

**示例：**
```bash
# 使用 BBR 拥塞控制算法
iperf3 -c <server_ip> -C bbr
```
**延伸阅读：** [TCP 拥塞控制](#)
---
### --get-server-output
获取服务器的输出。输出格式由服务器决定（特别是，如果服务器使用 `--json` 标志调用，输出将是 JSON 格式，否则将是人类可读格式）。如果客户端使用 `--json` 运行，服务器输出将包含在 JSON 对象中；否则，它将附加在人类可读输出的底部。

**示例：**
```bash
# 运行测试并获取服务器端的报告
iperf3 -c <server_ip> --get-server-output
```
---
### --udp-counters-64bit
在 UDP 测试数据包中使用 64 位计数器。使用此选项有助于防止在长时间或高比特率 UDP 测试期间计数器溢出。客户端和服务器都需要至少运行 3.1 版本才能使此选项起作用。它可能会在将来的某个时候成为默认行为。

**示例：**
```bash
# 在高比特率 UDP 测试中使用 64 位计数器以避免溢出
iperf3 -c <server_ip> -u -b 5G --udp-counters-64bit
```
---
### --repeating-payload
在有效负载中使用重复模式，而不是随机字节。iperf2 中使用相同的有效负载（ASCII '0..9' 重复）。它可能有助于测试和揭示具有硬件压缩（包括一些 WiFi 接入点）的网络设备中的问题，其中 iperf2 和 iperf3 仅基于有效负载熵而表现不同。

**示例：**
```bash
# 使用可预测的重复负载，有助于诊断硬件压缩问题
iperf3 -c <server_ip> --repeating-payload
```
---
### --dont-fragment
在传出数据包上设置 IPv4 不分片（DF）位。仅适用于通过 IPv4 进行的 UDP 测试。

**示例：**
```bash
# 在 UDP 包上设置 DF 位，用于路径 MTU 发现
iperf3 -c <server_ip> -u --dont-fragment
```
---
### --username username
用于向 iperf 服务器进行身份验证的用户名（如果使用 OpenSSL 支持构建）。运行时将以交互方式提示输入密码。注意，要使用的密码也可以通过 `IPERF3_PASSWORD` 环境变量指定。如果存在此变量，将跳过密码提示。

**示例：**
```bash
# 使用用户名 "admin" 进行身份验证，将提示输入密码
iperf3 -c <server_ip> --username admin --rsa-public-key-path public.pem

# 从环境变量读取密码
export IPERF3_PASSWORD='secret_password'
iperf3 -c <server_ip> --username admin --rsa-public-key-path public.pem
```
**延伸阅读：** [iperf3 身份验证](#)
---
### --rsa-public-key-path file
用于加密身份验证凭证的 RSA 公钥的路径（如果使用 OpenSSL 支持构建）。

**示例：**
```bash
# 客户端使用指定的公钥来加密凭证
iperf3 -c <server_ip> --username admin --rsa-public-key-path /path/to/public.pem
```
**延伸阅读：** [iperf3 身份验证](#)
---

## 示例：验证

### 认证 - RSA 密钥对
iperf3 的认证功能需要一个 RSA 公私钥对。公钥用于加密包含用户凭证的认证令牌，而私钥用于解密认证令牌。私钥必须是 PEM 格式，并且不能设置密码。公钥必须是 PEM 格式并使用 SubjectPublicKeyInfo 编码。以下是使用 OpenSSL 生成正确格式密钥对的一组 UNIX/Linux 命令示例：

```bash
# 1. 生成带密码保护的私钥
openssl genrsa -des3 -out private.pem 2048

# 2. 从私钥中提取公钥
openssl rsa -in private.pem -outform PEM -pubout -out public.pem

# 3. 生成一个无密码保护的私钥，供 iperf3 服务器使用
openssl rsa -in private.pem -out private_not_protected.pem -outform PEM
```

在这些命令之后，公钥将包含在文件 `public.pem` 中，私钥将包含在文件 `private_not_protected.pem` 中。

### 认证 - 授权用户配置文件
必须向 iperf3 服务器提供一个简单的纯文本文件，以指定授权用户的凭证。该文件是一个简单的逗号分隔的用户名和相应密码哈希列表。密码哈希是字符串 `"{<user>}<password>"` 的 SHA256 哈希。该文件还可以包含注释行（以 `#` 字符开头）。下面是在 UNIX/Linux 系统上生成密码哈希的命令示例：

```bash
# 设置用户名和密码
S_USER=mario
S_PASSWD=rossi

# 生成 SHA256 哈希
echo -n "{$S_USER}$S_PASSWD" | sha256sum | awk '{ print $1 }'
```

下面是一个密码文件示例（包含与上述用户名和密码对应的条目）：
```
> cat credentials.csv
# 文件格式: username,sha256
mario,bf7a49a846d44b454a5d11e7acfaf13d138bbe0b7483aa3e050879700572709b
```

## 延伸阅读：进阶概念
对于本文档中标记为 **延伸阅读** 的一些较为复杂的概念，我们准备了更详细的说明文档：
[**iperf3 进阶概念指南**](./iperf3-extended-concepts-guide.md)

该指南包含了对以下概念的深入解释和示例：
- CPU 亲和性 (`-A`)
- 多路径 TCP (MPTCP) (`-m`)
- iperf3 身份验证
- 流控制传输协议 (SCTP) (`--sctp`)
- 公平队列调速 (`--fq-rate`)
- TCP 窗口大小 (`-w`)
- MTU 和 MSS (`-M`)
- Nagle 算法 (`-N`)
- DSCP (Differentiated Services Code Point) (`--dscp`)
- 零拷贝 (Zero Copy) (`-Z`)
- TCP 拥塞控制 (`-C`)


## 作者
iperf3 贡献者列表可以在位于 https://software.es.net/iperf/dev.html#authors 的文档中找到。

## 另请参阅
libiperf(3), https://software.es.net/iperf
