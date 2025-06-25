# tcpdump 中文指南

这份指南是 `tcpdump` man page 的中文翻译版本，并附带了更多示例和解释，希望能帮助你更好地理解和使用这个强大的网络工具。

## 目录
- [名称 (NAME)](#名称-name)
- [概要 (SYNOPSIS)](#概要-synopsis)
- [描述 (DESCRIPTION)](#描述-description)
- [选项 (OPTIONS)](#选项-options)
- [表达式 (EXPRESSION)](#表达式-expression)
- [示例 (EXAMPLES)](#示例-examples)
- [输出格式 (OUTPUT FORMAT)](#输出格式-output-format)
- [另见 (SEE ALSO)](#另见-see-also)
- [作者 (AUTHORS)](#作者-authors)
- [BUG反馈 (BUGS)](#bug反馈-bugs)

## 名称 (NAME)

**tcpdump** - 转储（抓取并显示）网络上的流量。

## 概要 (SYNOPSIS)

```bash
tcpdump [ -AbdDefhHIJKlLnNOpqStuUvxX# ] [ -B buffer_size ]
           [ -c count ] [ --count ] [ -C file_size ]
           [ -E spi@ipaddr algo:secret,...  ]
           [ -F file ] [ -G rotate_seconds ] [ -i interface ]
           [ --immediate-mode ] [ -j tstamp_type ] [ -k (metadata_arg) ]
           [ -m module ]
           [ -M secret ] [ --number ] [ --print ]
           [ -Q packet-metadata-filter ] [ -Q in|out|inout ]
           [ -r file ] [ -s snaplen ] [ -T type ] [ --version ]
           [ -V file ] [ -w file ] [ -W filecount ] [ -y datalinktype ]
           [ -z postrotate-command ] [ -Z user ]
           [ --time-stamp-precision=tstamp_precision ]
           [ --micro ] [ --nano ]
           [ expression ]
```

## 描述 (DESCRIPTION)

`tcpdump` 会打印出网络接口上，与所给的布尔表达式匹配的数据包内容的描述。默认情况下，描述信息前会带有一个时间戳，格式为时、分、秒和午夜以来的小数秒。

`tcpdump` 也可以使用 `-w` 标志运行，这将把原始数据包数据保存到文件中以供以后分析。或者使用 `-r` 标志，这将使其从已保存的数据包文件（savefile）中读取，而不是从网络接口实时捕获。还可以使用 `-V` 标志，这将使其读取一个已保存数据包文件的列表。在所有情况下，只有匹配 `expression` 的数据包才会被 `tcpdump` 处理。

如果不使用 `-c` 标志，`tcpdump` 将持续捕获数据包，直到被 `SIGINT` 信号（例如，通过键入中断字符，通常是 `Ctrl+C`）或 `SIGTERM` 信号（通常通过 `kill(1)` 命令生成）中断。如果使用 `-c` 标志，它将捕获数据包，直到被 `SIGINT` 或 `SIGTERM` 信号中断，或者已处理了指定数量的数据包。

当 `tcpdump` 完成捕获数据包后，它将报告以下统计信息：

- **packets "captured"**：`tcpdump` 接收并处理的数据包数量。
- **packets "received by filter"**：其含义取决于 `tcpdump` 运行的操作系统及其配置。如果在命令行上指定了过滤器，在某些操作系统上，它会计算所有数据包，无论它们是否被过滤器表达式匹配。在其他操作系统上，它只计算被过滤器表达式匹配的数据包。
- **packets "dropped by kernel"**：由于操作系统的数据包捕获机制缺少缓冲区空间而丢弃的数据包数量。如果操作系统不报告此信息，则报告为 0。

在支持 `SIGINFO` 信号的平台（如大多数BSD系统，包括macOS）上，当接收到 `SIGINFO` 信号时（通常通过 `Ctrl+T` 发送），它将报告这些统计信息并继续捕获数据包。在不支持 `SIGINFO` 信号的平台上，可以使用 `SIGUSR1` 信号实现相同的目的。

将 `SIGUSR2` 信号与 `-w` 标志一起使用将强制将数据包缓冲区的内容刷新（写入）到输出文件中。

从网络接口读取数据包可能需要您具有特殊权限（例如 root 权限）。读取已保存的数据包文件则不需要特殊权限。

## 选项 (OPTIONS)

> **注意**: 运行 `tcpdump` 通常需要 `root` 权限（或使用 `sudo`），因为它需要访问网络接口。

---

#### `-A`

以ASCII格式打印每个数据包（不包括其链路层头部）。这对于捕获和分析基于文本的协议（如HTTP、SMTP）非常方便。

**示例:**
```bash
# 捕获80端口的HTTP流量，并以ASCII显示内容，方便查看HTTP请求头和HTML内容
sudo tcpdump -A -i en0 'port 80'
```

---

#### `-b`

在BGP（边界网关协议）数据包中以 ASDOT 表示法而不是 ASPLAIN 表示法打印 AS（自治系统）号。

**示例:**
```bash
# 捕获BGP流量，并以ASDOT格式显示AS号，这对于网络工程师分析BGP路由信息很有用
sudo tcpdump -b -i any 'proto bgp'
```
> **延伸**: BGP是互联网的核心路由协议。AS号是用于标识互联网上各个网络的唯一编号。ASDOT 和 ASPLAIN 是表示32位AS号的两种不同格式。

---

#### `-B buffer_size` / `--buffer-size=buffer_size`

设置操作系统捕获缓冲区的大小，单位为KiB (1024字节)。

**示例:**
```bash
# 将内核捕获缓冲区大小设置为4096 KiB (4MB)
# 在高速网络上，增加缓冲区可以减少因缓冲区来不及处理而丢包的情况
sudo tcpdump -B 4096 -i en0
```

---

#### `-c count`

在接收到 `count` 个数据包后退出。

**示例:**
```bash
# 在en0接口上捕获10个数据包后自动停止
sudo tcpdump -c 10 -i en0

# 也可以使用 skip,count 的形式，跳过前5个包，再抓10个包
# （注意：此功能在所有版本或平台上可能不完全支持）
tcpdump -c 5,10 -r existing_capture.pcap
```

---

#### `--count`

在读取捕获文件时，仅在标准错误输出（stderr）上打印数据包的计数，而不是解析和打印数据包内容。如果指定了过滤器，则只计算匹配的数据包。

**示例:**
```bash
# 快速统计pcap文件中端口为80的数据包数量，而不显示包内容
tcpdump --count -r capture.pcap 'port 80'
```

---

#### `-C file_size`

在向抓包文件（savefile）写入原始数据包之前，检查文件是否大于 `file_size`。如果超过，则关闭当前文件并打开一个新文件。后续文件的命名会在 `-w` 指定的文件名后附加一个从1开始的数字。`file_size` 的单位是百万字节（1,000,000字节）。

**示例:**
```bash
# 捕获流量并保存到文件，每个文件大小达到10MB时，自动创建新文件
# 这会生成 capture.pcap, capture.pcap1, capture.pcap2, ...
sudo tcpdump -i en0 -w capture.pcap -C 10
```
> **延伸**: 此功能常与 `-W` 结合使用，用于创建循环覆盖的日志文件。

---

#### `-d`

将编译后的数据包匹配代码（BPF指令）以人类可读的形式转储到标准输出并停止。这对于调试复杂的过滤器表达式非常有用。

**示例:**
```bash
# 查看 'host example.com and port 80' 这个过滤器表达式编译后的指令
tcpdump -d 'host example.com and port 80'
```

---

#### `-dd`

将数据包匹配代码转储为C程序片段。

**示例:**
```bash
tcpdump -dd 'ip and tcp'
```

---

#### `-ddd`

将数据包匹配代码转储为十进制数（前面带有计数）。

**示例:**
```bash
tcpdump -ddd 'udp'
```

---

#### `-D` / `--list-interfaces`

打印系统上可用的网络接口列表，`tcpdump` 可以在这些接口上捕获数据包。每个接口会显示一个编号和一个名称，有时还附带描述。接口名称或编号可以提供给 `-i` 标志。

**示例:**
```bash
# 列出所有可用的网络接口
tcpdump -D
# 输出可能如下:
# 1.en0 [Up, Running] (Wi-Fi)
# 2.lo0 [Up, Running, Loopback] (Loopback)
# 3.eth0 [Up, Running] (Ethernet)
```

---

#### `-e`

在每一行转储中打印链路层头部信息。例如，在以太网上，这会显示源和目的MAC地址。

**示例:**
```bash
# 捕获ICMP（ping）流量，并显示MAC地址
sudo tcpdump -e -i en0 'icmp'
# 输出会包含类似 "00:11:22:33:44:55 > 66:77:88:99:aa:bb" 的MAC地址信息
```

---

#### `-E spi@ipaddr algo:secret,...`

用于解密IPsec ESP数据包。这是一个高级功能，用于分析加密的VPN流量。

**示例:**
```bash
# 解密发往192.0.2.1，SPI为0x12345678的ESP数据包
# 使用aes-cbc算法和十六进制密钥
sudo tcpdump -E 'spi@192.0.2.1 aes-cbc:0xdeadbeef...'
```
> **警告**: 在命令行上直接提供密钥存在安全风险。

---

#### `-f`

以数字形式打印"外部"IPv4地址，而不是符号形式（主机名）。这可以避免因DNS反向解析缓慢或失败而导致的 `tcpdump` 挂起。

**示例:**
```bash
# 捕获与外部主机通信的流量，并直接显示IP地址，不进行DNS查询
sudo tcpdump -f -i en0
```

---

#### `-F file`

使用 `file` 的内容作为过滤器表达式的输入。命令行上给出的其他表达式将被忽略。

**示例:**
```bash
# 创建一个包含过滤规则的文件 my_filters.txt
echo "host 192.168.1.1 and tcp port 443" > my_filters.txt

# 使用文件中的规则进行捕获
sudo tcpdump -F my_filters.txt -i en0
```

---

#### `-g` / `--apple-oneline`

(Apple 添加) 在详细模式下，IP报头后不插入换行符，以便于解析。

---

#### `-G rotate_seconds`

如果指定，每隔 `rotate_seconds` 秒轮换一次由 `-w` 选项指定的转储文件。保存文件的名称应包含 `strftime(3)` 定义的时间格式。如果没有指定时间格式，每个新文件将覆盖前一个文件。

**示例:**
```bash
# 每3600秒（1小时）创建一个新的抓包文件
# 文件名将包含时间信息，如 capture-2023-10-27_14:00:00.pcap
sudo tcpdump -w 'capture-%Y-%m-%d_%H:%M:%S.pcap' -G 3600
```

---

#### `-h` / `--help`

打印 `tcpdump` 和 `libpcap` 的版本字符串，显示用法消息，然后退出。

**示例:**
```bash
tcpdump -h
```

---

#### `--version`

打印 `tcpdump` 和 `libpcap` 的版本字符串，然后退出。

**示例:**
```bash
tcpdump --version
```

---

#### `-H`

尝试检测 802.11s 草案的 mesh 头部。用于无线网状网络分析。

---

#### `-i interface` / `--interface=interface`

指定要监听的网络接口。如果未指定，`tcpdump` 会在系统接口列表中搜索编号最小、已配置且"up"的接口（不包括环回接口）。

**示例:**
```bash
# 在接口 en0 上进行捕获
sudo tcpdump -i en0

# 在所有接口上进行捕获 (Linux)
sudo tcpdump -i any

# 使用 -D 命令输出的接口编号进行捕获
sudo tcpdump -i 1
```

---

#### `-I` / `--monitor-mode`

将接口置于"监控模式"（monitor mode）。仅在IEEE 802.11 Wi-Fi接口上受支持，并且仅在某些操作系统上受支持。监控模式允许捕获该区域内所有的Wi-Fi流量，而不仅仅是连接到你的接入点的流量。

**示例:**
```bash
# 将Wi-Fi接口置于监控模式进行捕获
# 这对于Wi-Fi安全分析和故障排除非常有用
sudo tcpdump -I -i wlan0
```
> **警告**: 在监控模式下，网卡会与其当前连接的网络断开。

---

#### `--immediate-mode`

在"立即模式"下捕获。在此模式下，数据包一到达就立即传递给 `tcpdump`，而不是为了效率而进行缓冲。当数据包被打印到终端而不是保存到文件时，这是默认行为。

**示例:**
```bash
# 强制使用立即模式，确保实时看到每一个包
sudo tcpdump --immediate-mode -i en0 | grep 'some-pattern'
```

---

#### `-j tstamp_type` / `--time-stamp-type=tstamp_type`

设置捕获的时间戳类型。可用的类型取决于平台和接口。使用 `-J` 选项可以查看支持的类型。

**示例:**
```bash
# 设置时间戳类型为 adapter_unsynced
sudo tcpdump -j adapter_unsynced -i en0
```

---

#### `-J` / `--list-time-stamp-types`

列出接口支持的时间戳类型并退出。

**示例:**
```bash
# 查看 en0 接口支持哪些时间戳类型
tcpdump -J -i en0
```

---

#### `--time-stamp-precision=tstamp_precision`

设置捕获的时间戳精度。支持的值为 `micro`（微秒）和 `nano`（纳秒）。默认为微秒。

**示例:**
```bash
# 以纳秒级精度捕获数据包时间戳
sudo tcpdump --time-stamp-precision=nano -i en0
```

---

#### `--micro` / `--nano`

`--time-stamp-precision=micro` 或 `--time-stamp-precision=nano` 的简写形式。

---

#### `-k metadata_arg`

(Apple 添加) 控制数据包元数据的显示。

---

#### `-K` / `--dont-verify-checksums`

不尝试验证IP、TCP或UDP校验和。这对于那些在硬件中执行部分或全部校验和计算的接口很有用，否则，所有传出的TCP校验和都可能被标记为坏的。

**示例:**
```bash
# 禁用校验和验证，避免在某些网卡上看到错误的校验和警告
sudo tcpdump -K -i en0
```

---

#### `-l`

使 stdout 行缓冲。如果您想在捕获数据的同时看到数据，这将非常有用。

**示例:**
```bash
# 使用行缓冲，通过管道将输出实时传递给 tee 命令
sudo tcpdump -l -i en0 | tee capture.log
```

---

#### `-L` / `--list-data-link-types`

列出接口已知的链路层类型并退出。这对于使用 `-y` 选项很有用。

**示例:**
```bash
# 查看 en0 接口支持的链路层类型
tcpdump -L -i en0
```

---

#### `-m module`

从文件 `module` 加载SMI MIB模块定义。此选项可以多次使用以加载多个MIB模块。主要用于SNMP分析。

---

#### `-M secret`

使用 `secret` 作为共享密钥，用于验证带有TCP-MD5选项（RFC 2385）的TCP段中的摘要。

---

#### `-n`

不将地址（例如，主机地址、端口号等）转换为主机名或服务名。使用此选项可以避免DNS查询，从而加快输出速度。

**示例:**
```bash
# 捕获流量，直接显示IP地址和端口号，而不是主机名和服务名
sudo tcpdump -n -i en0 'host 8.8.8.8'
# 输出将是 "8.8.8.8.53" 而不是 "google-public-dns-a.google.com.domain"
```

---

#### `-N`

不打印主机名的域名限定。例如，如果您使用此标志，`tcpdump` 将打印 `nic` 而不是 `nic.ddn.mil`。

**示例:**
```bash
# 只显示主机名，不显示完整的域名
sudo tcpdump -N -i en0
```

---

#### `-#` / `--number`

在行的开头打印一个可选的数据包编号。

**示例:**
```bash
# 在每行输出前加上数据包的序列号
sudo tcpdump -# -i en0 -c 5
# 输出:
# 1 14:30:00.123456 IP ...
# 2 14:30:00.123457 IP ...
```

---

#### `-O` / `--no-optimize`

不运行数据包匹配代码优化器。仅当您怀疑优化器中存在错误时才有用。

---

#### `-p` / `--no-promiscuous-mode`

不将接口置于混杂模式。请注意，接口可能由于其他原因而处于混杂模式；因此，`-p` 不能用作 `ether host {local-hw-addr} or ether broadcast` 的缩写。

**示例:**
```bash
# 只捕获发往本机、从本机发出或广播的数据包
sudo tcpdump -p -i en0
```
> **延伸**: 混杂模式（Promiscuous Mode）允许网卡捕获网络上所有流经它的数据包，无论这些数据包的目的地址是否是本机。默认情况下 `tcpdump` 会开启此模式。使用 `-p` 则关闭它。

---

#### `--print`

即使原始数据包正通过 `-w` 标志保存到文件中，也打印已解析的数据包输出。

**示例:**
```bash
# 在将流量保存到文件的同时，也在屏幕上查看解析后的输出
sudo tcpdump -w capture.pcap --print -i en0
```

---

#### `-q`

快速（安静？）输出。打印更少的协议信息，使输出行更短。

**示例:**
```bash
# 使用简洁模式输出，只显示最关键的信息
sudo tcpdump -q -i en0
```

---

#### `-Q direction` / `--direction=direction`

选择要捕获的数据包的发送/接收方向。可能的值为 `in`、`out` 和 `inout`。并非在所有平台上都可用。

**示例:**
```bash
# 只捕获进入 en0 接口的数据包
sudo tcpdump -Q in -i en0

# 只捕获从 en0 接口发出的数据包
sudo tcpdump -Q out -i en0
```

---

#### `-r file`

从 `file` 中读取数据包（该文件是由 `-w` 选项或其他写入 pcap 或 pcapng 文件的工具创建的）。如果 `file` 是 ``-``，则使用标准输入。

**示例:**
```bash
# 从之前保存的 capture.pcap 文件中读取并显示内容
tcpdump -r capture.pcap

# 对文件中的内容进行过滤
tcpdump -r capture.pcap 'host 192.168.1.1'
```

---

#### `-S` / `--absolute-tcp-sequence-numbers`

打印绝对的TCP序列号，而不是相对的。默认情况下，`tcpdump` 会将TCP序列号转换为相对于对话开始的相对值。

**示例:**
```bash
# 显示原始的、绝对的TCP序列号
sudo tcpdump -S -i en0 'tcp'
```

---

#### `-s snaplen` / `--snapshot-length=snaplen`

从每个数据包中截取 `snaplen` 字节的数据，而不是默认的262144字节。因为快照长度有限而被截断的数据包在输出中会以 `[|proto]` 表示。设置为 `0` 等同于设置为默认值。

**示例:**
```bash
# 只捕获每个数据包的前96个字节
# 这可以减少捕获文件的大小和处理时间，但可能丢失高层协议的详细信息
sudo tcpdump -s 96 -i en0

# 捕获完整的数据包
sudo tcpdump -s 0 -i en0
```
> **延伸**: 如果你只关心IP和TCP/UDP头部，一个较小的 `snaplen`（如128字节）通常就足够了。如果你需要分析应用层数据（如HTTP内容），则需要一个更大的 `snaplen`，甚至使用默认值。

---

#### `-T type`

强制将由 "expression" 选择的数据包解释为指定的 `type`。这在 `tcpdump` 无法自动识别协议时很有用。

**示例:**
```bash
# 将UDP 12345端口的流量强制解释为RADIUS协议
sudo tcpdump -T radius -i en0 'udp port 12345'
```

---

#### `-t`

不在每行转储上打印时间戳。

**示例:**
```bash
# 输出不带时间戳
sudo tcpdump -t -i en0
```

---

#### `-tt`

在每行转储上打印自1970年1月1日00:00:00 UTC以来的秒数和该时间的微秒/纳秒数（取决于精度设置）。

**示例:**
```bash
# 以Unix时间戳格式显示时间
sudo tcpdump -tt -i en0
```

---

#### `-ttt`

在每行转储上打印当前行和上一行之间的增量（微秒或纳秒，取决于精度）。

**示例:**
```bash
# 显示每个包与上一个包之间的时间差
sudo tcpdump -ttt -i en0
```

---

#### `-tttt`

在每行转储上打印一个时间戳，格式为时、分、秒和秒的小数部分，前面加上日期。

**示例:**
```bash
# 显示包含日期的完整时间戳
sudo tcpdump -tttt -i en0
# 输出: 2023-10-27 14:30:00.123456 IP ...
```

---

#### `-ttttt`

在每行转储上打印当前行和第一行之间的增量。

**示例:**
```bash
# 显示每个包与捕获开始时的时间差
sudo tcpdump -ttttt -i en0
```

---

#### `-u`

打印未解码的NFS句柄。

---

#### `-U` / `--packet-buffered`

使打印的数据包输出"数据包缓冲"。即，每打印一个数据包的描述，它就会被写入标准输出，而不是等到输出缓冲区填满时才写入。与 `-l`（行缓冲）类似但不同。当使用 `-w` 保存文件时，此选项也会使写入操作变为数据包缓冲，确保数据包被立即写入文件。

**示例:**
```bash
# 确保每个数据包被立即写入文件，这在需要实时监控抓包文件的场景中很有用
sudo tcpdump -U -w capture.pcap
```

---

#### `-v`, `-vv`, `-vvv`

当解析和打印时，产生（稍微更）详细的输出。
*   `-v`: 显示更多细节，如IP包中的TTL、ID、总长度和选项。
*   `-vv`: 更加详细，例如，会完整解码SMB包。
*   `-vvv`: 最详细的输出，例如，会完整打印telnet的 `SB ... SE` 选项。

**示例:**
```bash
# 以非常详细的模式捕获ICMP流量
sudo tcpdump -vv -i en0 'icmp'
```

---

#### `-V file`

从 `file` 中读取文件名列表。如果 `file` 是 ``-``，则使用标准输入。这允许你一次性分析多个抓包文件。

**示例:**
```bash
# 创建一个包含pcap文件列表的文件 pcap_files.txt
echo "day1.pcap" > pcap_files.txt
echo "day2.pcap" >> pcap_files.txt

# 分析列表中的所有文件
tcpdump -V pcap_files.txt 'port 80'
```

---

#### `-w file`

将原始数据包写入 `file` 而不是解析和打印它们。之后可以使用 `-r` 选项来打印它们。如果 `file` 是 ``-``，则使用标准输出。

**示例:**
```bash
# 将 en0 接口的所有流量捕获并保存到 a.pcap 文件中
sudo tcpdump -i en0 -w a.pcap

# 捕获指定主机的流量
sudo tcpdump -i en0 -w host-1.pcap 'host 1.2.3.4'
```

---

#### `-W filecount`

与 `-C` 选项结合使用时，这将把创建的文件数量限制为指定的数字，并从头开始覆盖文件，从而创建一个"循环"缓冲区。

**示例:**
```bash
# 创建一个循环缓冲区，最多10个文件，每个文件10MB
# 当 capture.pcap9 写满后，下一个文件会覆盖 capture.pcap0
sudo tcpdump -w capture.pcap -C 10 -W 10
```

---

#### `-x`, `-xx`

以十六进制打印每个数据包的数据（不包括其链路层头部）。`-xx` 包含链路层头部。

**示例:**
```bash
# 同时显示协议解析和十六进制内容
sudo tcpdump -x -i en0 'port 53'
```

---

#### `-X`, `-XX`

以十六进制和ASCII码打印每个数据包的数据。`-XX` 包含链路层头部。这对于分析未知协议或在数据中查找特定字符串非常有用。

**示例:**
```bash
# 以十六进制和ASCII码显示DNS请求的详细内容
sudo tcpdump -X -i en0 'udp port 53'
```

---

#### `-y datalinktype` / `--linktype=datalinktype`

设置捕获数据包时使用的数据链路类型。使用 `-L` 查看可用类型。

**示例:**
```bash
# 在一个PPI（Per-Packet Information）接口上捕获，并指定链路层类型为802.11
# 这在某些无线网卡和特殊驱动下需要
sudo tcpdump -y IEEE802_11_RADIO -i wlan0
```

---

#### `-z postrotate-command`

与 `-C` 或 `-G` 选项结合使用，这会使 `tcpdump` 在每次轮换后关闭保存文件时运行 `postrotate-command file`。

**示例:**
```bash
# 在每次创建新的10MB抓包文件后，使用gzip压缩它
sudo tcpdump -i en0 -w capture.pcap -C 10 -z gzip
```

---

#### `-Z user` / `--relinquish-privileges=user`

如果 `tcpdump` 以 root 身份运行，在打开捕获设备或输入保存文件后，但在打开任何输出保存文件之前，将用户ID更改为 `user`，并将组ID更改为 `user` 的主组。这是一个很好的安全实践。

**示例:**
```bash
# 以root启动，但打开接口后立即放弃权限，切换到 'nobody' 用户
sudo tcpdump -i en0 -Z nobody
```

## 表达式 (EXPRESSION)

表达式用于选择哪些数据包将被转储。如果没有给出表达式，网络上的所有数据包都将被转储。否则，只有表达式为"真"的数据包才会被转储。

关于表达式的语法，请参见 `pcap-filter(7)` 手册页。这是一个非常强大的功能，允许你精确地定位你感兴趣的流量。

> **延伸**: 我们将准备一份单独的 `tcpdump-filter-expressions-guide.md` 来详细解释过滤表达式的构建。

## 示例 (EXAMPLES)

> 许多基础示例已合并到上面的 [选项 (OPTIONS)](#选项-options) 部分。这里提供一些更复杂的组合示例。

**打印所有进出 `sundown` 主机的数据包:**
```bash
tcpdump host sundown
```

**打印 `helios` 和 `hot` 或 `ace` 之间的流量:**
```bash
tcpdump host helios and \( hot or ace \)
```
> 注意：括号需要用 `\` 转义以防止 shell 解释它们。

**打印 `ace` 和除 `helios` 之外的任何主机之间的所有IP数据包:**
```bash
tcpdump ip host ace and not helios
```

**打印所有通过网关 `snup` 的ftp流量:**
```bash
tcpdump 'gateway snup and (port ftp or ftp-data)'
```

**打印涉及非本地主机的每个TCP会话的开始和结束数据包（SYN和FIN包）:**
```bash
tcpdump 'tcp[tcpflags] & (tcp-syn|tcp-fin) != 0 and not src and dst net localnet'
```

**打印所有到80端口的IPv4 HTTP数据包，且不包括SYN、FIN和纯ACK包（即只看有数据的包）:**
```bash
tcpdump 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'
```
> 这是一个高级示例，通过直接访问IP和TCP头部中的字段来计算数据包的载荷长度。

**打印所有不是ping包的ICMP包:**
```bash
tcpdump 'icmp[icmptype] != icmp-echo and icmptype != icmp-echoreply'
```

## 输出格式 (OUTPUT FORMAT)

`tcpdump` 的输出是协议相关的。

> **延伸**: `tcpdump` 的输出格式非常丰富，包含了时间戳、链路层、IP层、传输层和应用层的信息。我们将准备一份 `tcpdump-output-format-guide.md` 来详细解读不同协议的输出格式。

## 另见 (SEE ALSO)

stty(1), pcap(3PCAP), bpf(4), nit(4P), pcap-savefile(5), pcap-filter(7), pcap-tstamp(7)

## 作者 (AUTHORS)

最初的作者是：Van Jacobson, Craig Leres and Steven McCanne。
目前由 tcpdump.org 维护。
当前版本可通过 HTTPS 获取：https://www.tcpdump.org/

## BUG反馈 (BUGS)

要报告安全问题，请发送电子邮件至 security@tcpdump.org。
要报告错误和其它问题，请参阅 tcpdump 源代码根目录中的 `CONTRIBUTING` 文件。
