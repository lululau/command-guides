# Socat 完整指南

## 名称
**socat** - 多用途中继器（SOcket CAT）

## 概述
```bash
socat [选项] <地址> <地址>
socat -V
socat -h[h[h]] | -?[?[?]]
filan
procan
```

## 描述

Socat 是一个基于命令行的实用工具，它建立两个双向字节流并在它们之间传输数据。由于流可以从大量不同类型的数据源和目标构建（参见地址类型），并且可以将许多地址选项应用于流，因此 socat 可用于许多不同的目的。

### 生命周期

socat 实例的生命周期通常包含四个阶段：

1. **初始化阶段**：解析命令行选项并初始化日志记录
2. **打开阶段**：socat 打开第一个地址，然后打开第二个地址  
3. **传输阶段**：socat 通过 select() 监视两个流的读写文件描述符，当一侧有数据可用且可以写入另一侧时，读取数据并写入另一个流
4. **关闭阶段**：当其中一个流到达 EOF 时，开始关闭阶段

### 基本示例
```bash
# 简单的 TCP 连接
socat - TCP4:www.example.com:80

# 端口转发
socat TCP4-LISTEN:8080,fork TCP4:localhost:80

# 文件传输
socat -u FILE:input.txt TCP4:remote:1234
```

## 命令行选项

### 版本和帮助选项

#### -V
打印版本和可用功能信息到标准输出并退出。

```bash
socat -V
```

#### -h | -?
打印帮助文本到标准输出，描述命令行选项和可用地址类型，然后退出。

```bash
socat -h
```

#### -hh | -??
类似 -h，但额外显示所有可用地址选项的短名称列表。

```bash
socat -hh
```

#### -hhh | -???
类似 -hh，但额外显示所有可用地址选项名称的列表。

```bash
socat -hhh
```

### 调试和日志选项

#### -d
启用调试模式。没有此选项时，只打印致命、错误和警告消息；应用此选项还会打印通知消息。

```bash
# 基本调试信息
socat -d TCP4-LISTEN:8080,fork TCP4:localhost:80
```

#### -d0
只打印致命和错误消息。

```bash
socat -d0 TCP4-LISTEN:8080,fork TCP4:localhost:80
```

#### -dd | -d2  
打印致命、错误、警告和通知消息。

```bash
socat -dd TCP4-LISTEN:8080,fork TCP4:localhost:80
```

#### -ddd | -d3
打印致命、错误、警告、通知和信息消息。

```bash
socat -ddd TCP4-LISTEN:8080,fork TCP4:localhost:80
```

#### -dddd | -d4
打印致命、错误、警告、通知、信息和调试消息。

```bash
socat -dddd TCP4-LISTEN:8080,fork TCP4:localhost:80
```

#### -D
在传输阶段开始前记录有关文件描述符的信息。

```bash
socat -D TCP4-LISTEN:8080,fork TCP4:localhost:80
```

### 日志记录选项

#### -ly[<facility>]
将消息写入 syslog 而不是 stderr。

```bash
# 写入默认 daemon 设施
socat -ly TCP4-LISTEN:8080,fork TCP4:localhost:80

# 写入指定设施
socat -lylocal0 TCP4-LISTEN:8080,fork TCP4:localhost:80
```

#### -lf <logfile>
将消息写入指定的日志文件。

```bash
socat -lf /var/log/socat.log TCP4-LISTEN:8080,fork TCP4:localhost:80
```

#### -ls
将消息写入 stderr（默认行为）。

```bash
socat -ls TCP4-LISTEN:8080,fork TCP4:localhost:80
```

#### -lm[<facility>]
混合日志模式。启动时消息打印到 stderr，当 socat 开始传输阶段时切换到 syslog。

```bash
socat -lm TCP4-LISTEN:8080,fork TCP4:localhost:80
```

#### -lh
在日志消息中添加主机名。

```bash
socat -lh TCP4-LISTEN:8080,fork TCP4:localhost:80
```

#### -lu
将错误消息的时间戳扩展到微秒分辨率。

```bash
socat -lu TCP4-LISTEN:8080,fork TCP4:localhost:80
```

### 数据监控选项

#### -v
将传输的数据不仅写入目标流，还写入 stderr。输出格式为文本，带有一些可读性转换，前缀为 ">" 或 "<" 表示流向。

```bash
# 监控 HTTP 请求/响应
socat -v - TCP4:www.example.com:80
```

#### -x
将传输的数据以十六进制格式写入 stderr。可以与 -v 结合使用。

```bash
# 十六进制监控
socat -x - TCP4:www.example.com:80

# 同时使用文本和十六进制格式
socat -v -x - TCP4:www.example.com:80
```

### 数据转储选项

#### -r <file>
将从左到右地址流动的原始（二进制）数据转储到指定文件。

```bash
# 保存发送的数据
socat -r sent_data.bin - TCP4:www.example.com:80
```

#### -R <file>
将从右到左地址流动的原始（二进制）数据转储到指定文件。

```bash
# 保存接收的数据
socat -R received_data.bin - TCP4:www.example.com:80
```

### 性能选项

#### -b<size>
设置数据传输块大小。每步最多传输 <size> 字节。默认为 8192 字节。

```bash
# 设置 4KB 块大小
socat -b4096 - TCP4:www.example.com:80

# 设置 64KB 块大小以提高大文件传输性能
socat -b65536 FILE:large_file.dat TCP4:remote:1234
```

### 错误处理选项

#### -s
默认情况下，当发生错误时 socat 终止以防止进程在某些选项无法应用时运行。使用此选项，socat 对错误不严格并尝试继续。

```bash
socat -s TCP4-LISTEN:8080,fork TCP4:localhost:80
```

### 超时选项

#### -t<timeout>
当一个通道到达 EOF 时，另一个通道的写入部分被关闭。然后，socat 在终止前等待 <timeout> 秒。默认为 0.5 秒。

```bash
# 设置 5 秒超时
socat -t5 - TCP4:www.example.com:80
```

#### -T<timeout>
总不活动超时：当 socat 已在传输循环中且在 <timeout> 秒内没有任何活动时，它终止。

```bash
# 设置 30 秒不活动超时
socat -T30 UDP4-LISTEN:5000 -
```

### 单向模式选项

#### -u
使用单向模式。第一个地址仅用于读取，第二个地址仅用于写入。

```bash
# 文件到网络的单向传输
socat -u FILE:input.txt TCP4:remote:1234

# 网络到文件的单向传输
socat -u TCP4-LISTEN:1234 FILE:output.txt
```

#### -U
使用反向单向模式。第一个地址仅用于写入，第二个地址仅用于读取。

```bash
# 与 -u 相反的方向
socat -U FILE:input.txt TCP4:remote:1234
```

### 网络版本选项

#### -4
如果地址没有隐式或显式指定版本，则使用 IP 版本 4。

```bash
socat -4 TCP-LISTEN:8080,fork TCP:remote:80
```

#### -6
如果地址没有隐式或显式指定版本，则使用 IP 版本 6。

```bash
socat -6 TCP-LISTEN:8080,fork TCP:remote:80
```

#### -0
不偏好特定的 IP 版本，这让被动地址在某些平台上同时服务两个版本。

```bash
socat -0 TCP-LISTEN:8080,fork TCP:remote:80
```

### 其他选项

#### -g
在地址选项解析期间，不检查选项在给定地址环境中是否被认为有用。

```bash
socat -g TCP4:remote:80,bind=/dev/ttyS0 -
```

#### -L<lockfile>
如果锁文件存在，则退出并出错。如果锁文件不存在，则创建它并继续，退出时删除锁文件。

```bash
socat -L/tmp/socat.lock TCP4-LISTEN:8080,fork TCP4:localhost:80
```

#### -W<lockfile>
如果锁文件存在，则等待直到它消失。当锁文件不存在时，创建它并继续，退出时删除锁文件。

```bash
socat -W/tmp/socat.lock TCP4-LISTEN:8080,fork TCP4:localhost:80
```

#### --experimental
必须明确启用使用此选项来启用未经充分测试或将来可能发生变化的新功能。

```bash
socat --experimental NETNS:namespace TCP4:remote:80
```

#### --statistics | -S
在终止 socat 之前记录传输统计信息（两个方向的字节和块计数器）。

```bash
socat --statistics TCP4-LISTEN:8080,fork TCP4:localhost:80
```

## 地址规范

地址规范通常包含：
- 地址类型关键字（如 TCP4、EXEC、UNIX）
- 零个或多个必需的地址参数，用 ':' 分隔
- 零个或多个地址选项，用 ',' 分隔

### 地址规范格式
```
地址类型:参数1:参数2,选项1=值1,选项2=值2
```

### 示例
```bash
# TCP 连接
TCP4:www.example.com:80

# 带选项的 TCP 监听
TCP4-LISTEN:8080,fork,bind=127.0.0.1

# 执行程序
EXEC:/bin/cat,pty,setsid
```

## 主要地址类型

### 文件相关地址

#### CREATE:<filename>
使用 creat() 打开文件名用于写入。这是只写地址。

```bash
# 创建新文件
socat - CREATE:newfile.txt

# 创建带权限的文件
socat - CREATE:newfile.txt,mode=644
```

#### OPEN:<filename>
使用 open() 系统调用打开文件名。

```bash
# 打开现有文件读写
socat - OPEN:existing_file.txt

# 以只读方式打开
socat - OPEN:existing_file.txt,rdonly

# 以追加模式打开
socat - OPEN:log_file.txt,append
```

#### GOPEN:<filename>
通用打开，尝试有效处理除目录外的任何文件系统条目。

```bash
# 通用打开文件
socat - GOPEN:some_file

# 通用打开设备
socat - GOPEN:/dev/ttyS0
```

### 网络地址

#### TCP:<host>:<port>
建立到指定主机和端口的 TCP 连接。

```bash
# 连接到 Web 服务器
socat - TCP:www.example.com:80

# 连接到本地服务
socat - TCP:localhost:22

# 使用 IPv4
socat - TCP4:192.168.1.1:80

# 使用 IPv6
socat - TCP6:[::1]:80
```

#### TCP-LISTEN:<port>
在指定端口上监听 TCP 连接。

```bash
# 基本 TCP 监听
socat TCP-LISTEN:8080 -

# 允许多个连接
socat TCP-LISTEN:8080,fork -

# 绑定到特定接口
socat TCP-LISTEN:8080,bind=127.0.0.1 -

# 设置积压队列
socat TCP-LISTEN:8080,backlog=10,fork -
```

#### UDP:<host>:<port>
连接到指定主机和端口的 UDP 端点。

```bash
# UDP 客户端
socat - UDP:remote:5000

# UDP 广播
socat - UDP-DATAGRAM:255.255.255.255:9999,broadcast

# UDP 多播
socat - UDP-DATAGRAM:224.0.0.1:9999,ip-add-membership=224.0.0.1:eth0
```

#### UDP-LISTEN:<port>
等待 UDP 数据包并连接回发送者。

```bash
# UDP 服务器
socat UDP-LISTEN:5000 -

# UDP 服务器，每个客户端一个进程
socat UDP-LISTEN:5000,fork -
```

### 进程执行地址

#### EXEC:<command-line>
分叉子进程并执行指定程序。

```bash
# 执行 shell 命令
socat - EXEC:/bin/bash

# 执行带参数的命令
socat - EXEC:'/bin/ls -la'

# 使用 pty
socat - EXEC:/bin/bash,pty,setsid,ctty

# 重定向 stderr
socat - EXEC:/bin/bash,stderr
```

#### SYSTEM:<shell-command>
使用 system() 分叉子进程并执行指定程序。

```bash
# 系统调用
socat - SYSTEM:'echo Hello World'

# 复杂的 shell 命令
socat - SYSTEM:'ls -la | grep txt'
```

#### SHELL:<shell-command>
使用配置的 shell 执行命令。

```bash
# 使用默认 shell
socat - SHELL:'echo $SHELL'

# 复杂的 shell 脚本
socat - SHELL:'for i in {1..5}; do echo $i; done'
```

### 伪终端地址

#### PTY
生成伪终端并使用其主端。

```bash
# 创建 PTY
socat PTY,link=/tmp/mypty -

# 等待从端打开
socat PTY,link=/tmp/mypty,wait-slave -

# 设置权限
socat PTY,link=/tmp/mypty,mode=666 -
```

### UNIX 域套接字地址

#### UNIX-CONNECT:<filename>
连接到 UNIX 域套接字。

```bash
# 连接到 UNIX 套接字
socat - UNIX-CONNECT:/tmp/socket

# 绑定到本地地址
socat - UNIX-CONNECT:/tmp/socket,bind=/tmp/local_socket
```

#### UNIX-LISTEN:<filename>
在 UNIX 域套接字上监听连接。

```bash
# UNIX 域套接字服务器
socat UNIX-LISTEN:/tmp/socket -

# 多客户端支持
socat UNIX-LISTEN:/tmp/socket,fork -

# 设置权限
socat UNIX-LISTEN:/tmp/socket,mode=666,fork -
```

### SSL/TLS 地址

#### OPENSSL:<host>:<port>
建立 SSL 连接。

```bash
# 基本 SSL 连接
socat - OPENSSL:secure.example.com:443

# 使用客户端证书
socat - OPENSSL:secure.example.com:443,cert=client.pem

# 验证服务器证书
socat - OPENSSL:secure.example.com:443,cafile=ca.pem,verify=1
```

#### OPENSSL-LISTEN:<port>
在端口上监听 SSL 连接。

```bash
# SSL 服务器
socat OPENSSL-LISTEN:8443,cert=server.pem,fork -

# 要求客户端证书
socat OPENSSL-LISTEN:8443,cert=server.pem,cafile=ca.pem,verify=1,fork -
```

## 地址选项

### FD 选项组

#### cloexec[=<bool>]
设置 FD_CLOEXEC 标志。

```bash
socat - TCP:remote:80,cloexec
```

#### nonblock[=<bool>]
尝试以非阻塞模式打开或使用文件。

```bash
socat - TCP:remote:80,nonblock
```

#### user=<user>
设置流的用户（所有者）。

```bash
socat - CREATE:file.txt,user=nobody
```

#### group=<group>
设置流的组。

```bash
socat - CREATE:file.txt,group=nogroup
```

#### mode=<mode>
设置流的模式（权限）。

```bash
socat - CREATE:file.txt,mode=644
```

### SOCKET 选项组

#### bind=<sockname>
使用 bind() 系统调用将套接字绑定到给定的套接字地址。

```bash
# 绑定到特定接口
socat - TCP:remote:80,bind=192.168.1.100

# 绑定到特定端口
socat - TCP:remote:80,bind=:12345
```

#### connect-timeout=<seconds>
在指定秒数后中止连接尝试。

```bash
socat - TCP:remote:80,connect-timeout=5
```

#### so-keepalive
在套接字上启用发送保持活动。

```bash
socat - TCP:remote:80,so-keepalive
```

#### so-reuseaddr
允许其他套接字绑定到已在使用的地址。

```bash
socat TCP-LISTEN:8080,reuseaddr,fork TCP:remote:80
```

### 应用程序选项组

#### cr
将默认行终止符 NL 转换为/从 CR。

```bash
socat - TCP:remote:80,cr
```

#### crnl
将默认行终止符 NL 转换为/从 CRNL。

```bash
socat - TCP:remote:80,crnl
```

#### ignoreeof
当此通道上发生 EOF 时，socat 忽略它并尝试读取更多数据。

```bash
socat - FILE:growing_file.log,ignoreeof
```

## 高级用法示例

### 端口转发
```bash
# 简单端口转发
socat TCP4-LISTEN:8080,fork TCP4:backend:80

# 带日志的端口转发
socat -d -d TCP4-LISTEN:8080,fork TCP4:backend:80

# SSL 到普通 TCP 的转发
socat OPENSSL-LISTEN:8443,cert=server.pem,fork TCP4:backend:80
```

### 文件传输
```bash
# 发送文件
socat -u FILE:file.txt TCP4:remote:1234

# 接收文件
socat -u TCP4-LISTEN:1234 CREATE:received_file.txt

# 加密文件传输
socat -u FILE:file.txt OPENSSL:remote:8443,cert=client.pem
```

### 串行通信
```bash
# 连接到串行端口
socat - /dev/ttyS0,raw,echo=0

# 串行端口桥接
socat /dev/ttyS0,raw,echo=0 TCP4-LISTEN:54321,fork
```

### 调试和监控
```bash
# 监控 HTTP 流量
socat -v TCP4-LISTEN:8080,fork TCP4:www.example.com:80

# 十六进制转储
socat -x TCP4-LISTEN:1234,fork -

# 保存传输数据
socat -r sent.dat -R received.dat TCP4-LISTEN:1234,fork TCP4:remote:80
```

### 伪终端应用
```bash
# 创建虚拟调制解调器
socat PTY,link=/dev/vmodem0,raw,echo=0 TCP4:modem.server.com:23

# SSH 通过 PTY
socat PTY,link=/tmp/ssh_pty EXEC:'ssh user@remote',pty,setsid,ctty
```

### UNIX 域套接字
```bash
# 代理 X11 连接
socat UNIX-LISTEN:/tmp/.X11-unix/X1,fork TCP4:remote:6000

# 本地套接字转发
socat UNIX-LISTEN:/tmp/local.sock,fork TCP4:remote:80
```

### 多播和广播
```bash
# 加入多播组
socat - UDP4-DATAGRAM:224.0.0.1:9999,ip-add-membership=224.0.0.1:eth0

# 发送广播
socat - UDP4-DATAGRAM:255.255.255.255:9999,broadcast

# 监听广播
socat UDP4-RECV:9999,broadcast -
```

## 环境变量

### 输入变量
- `SOCAT_DEFAULT_LISTEN_IP`: 设置默认监听 IP 版本
- `SOCAT_PREFERRED_RESOLVE_IP`: 设置首选解析 IP 版本

### 输出变量
- `SOCAT_VERSION`: socat 版本
- `SOCAT_PID`: socat 进程 ID
- `SOCAT_PEERADDR`: 对等地址
- `SOCAT_PEERPORT`: 对等端口
- `SOCAT_SOCKADDR`: 本地套接字地址
- `SOCAT_SOCKPORT`: 本地套接字端口

## 诊断

### 消息严重性级别
- **FATAL**: 需要无条件立即程序终止的条件
- **ERROR**: 阻止正确程序处理的条件
- **WARNING**: 某些功能未正确运行的情况
- **NOTICE**: 程序的有趣操作
- **INFO**: 程序执行的描述
- **DEBUG**: 程序工作方式的描述

### 退出状态
- 0: 由于 EOF 或不活动超时而终止
- 正值: 错误
- 负值: 致命错误

## 常见用法模式

### 1. 网络调试
```bash
# 监听并显示接收到的数据
socat -v TCP4-LISTEN:1234 -

# 连接并发送数据
echo "test data" | socat - TCP4:target:1234
```

### 2. 服务代理
```bash
# HTTP 代理
socat TCP4-LISTEN:8080,fork TCP4:www.example.com:80

# SOCKS 代理
socat TCP4-LISTEN:1080,fork SOCKS4:proxy:target:port
```

### 3. 数据转换
```bash
# 行终止符转换
socat - TCP4:target:80,crnl

# 字符编码（需要外部工具）
socat - EXEC:'iconv -f utf8 -t gbk' | socat - TCP4:target:80
```

### 4. 安全隧道
```bash
# SSL 客户端
socat - OPENSSL:secure.server.com:443,verify=0

# SSL 服务器
socat OPENSSL-LISTEN:8443,cert=server.pem,fork EXEC:/bin/bash
```

### 5. 系统管理
```bash
# 远程 shell
socat TCP4-LISTEN:2222,fork EXEC:/bin/bash,pty,setsid,ctty

# 文件同步
socat -u FILE:source.txt TCP4:remote:1234
```

---

*本指南涵盖了 socat 的主要功能和用法。对于更复杂的用例和详细的技术解释，请参考 socat-extended-concepts-guide.md。*
