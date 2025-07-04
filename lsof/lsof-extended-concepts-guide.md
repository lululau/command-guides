# lsof 延伸解读

本文档旨在对 `lsof` 手册中一些较为复杂或需要更详细解释的概念进行延伸解读，并提供更丰富的示例。

---

## lsof 的阻塞、超时和内核交互

### 为什么 lsof 会阻塞？

`lsof` 通过查询内核来获取有关打开文件的信息。它使用了一些标准的系统调用，如 `stat(2)`、`lstat(2)` 和 `readlink(2)`。在某些情况下，这些内核函数可能会被 "阻塞" 或 "挂起"，导致 `lsof` 看起来没有响应。

最常见的原因是 `lsof` 试图获取一个通过网络挂载的文件系统（如 NFS）的信息，而该网络文件系统的服务器变得无法访问（例如，服务器崩溃、网络中断）。当 `lsof` 对该挂载点上的任何文件执行 `stat(2)` 操作时，内核会不断尝试联系无响应的服务器，从而导致 `lsof` 进程被阻塞。

### 如何应对阻塞？

`lsof` 内置了一些机制来处理这种情况：

1.  **超时 (`-S [t]`)**: `lsof` 为这些可能阻塞的内核调用设置了一个定时器。如果调用在指定的时间 `t` (默认为 15 秒，最小为 2 秒) 内没有返回，`lsof` 会中断该操作并继续执行。这可以防止 `lsof` 无限期地挂起。

    **示例：**
    ```bash
    # 设置一个 5 秒的超时
    lsof -S 5
    ```

2.  **避免阻塞调用 (`-b`)**: `-b` 选项告诉 `lsof` 完全避免使用 `lstat(2)`, `readlink(2)`, 和 `stat(2)` 这些可能阻塞的函数。这是一个更强的措施，但有一些副作用：
    *   `lsof` 将无法解析符号链接。
    *   `lsof` 可能无法获取文件的设备号和 inode 号，除非系统提供了备用机制（见下文）。
    *   你不能通过文件名来查找文件，除非该名称是一个文件系统挂载点。

    **示例：**
    ```bash
    # 在一个可能存在问题的 NFS 环境中运行 lsof
    lsof -b
    ```

3.  **绕过保护机制 (`-O`)**: 这个选项会禁用 `lsof` 的超时和子进程保护策略。这会减少 `lsof` 的启动开销，但如果遇到阻塞情况，`lsof` 将会完全挂起。**请谨慎使用此选项。**

### 备用设备号 (Alternate Device Numbers)

当使用 `-b` 选项或发生超时时，`lsof` 无法通过 `stat(2)` 获取文件系统的设备号。但是，在某些系统中，`lsof` 可以从系统挂载表 (如 `/etc/mtab`) 中获取一个备用的设备号。这使得即使在无法直接与文件系统交互的情况下，`lsof` 仍然能够识别和列出在该文件系统上打开的文件。

---

## lsof 的内核名称缓存 (Kernel Name Cache)

为了提高性能，操作系统内核会缓存最近访问过的文件路径组件。`lsof` 在某些操作系统上可以利用这个缓存来重构一个打开文件的完整路径，即使该文件的路径没有被直接提供。

### lsof 如何显示路径？

当 `lsof` 能够从缓存中找到路径信息时，它会在 `NAME` 列显示它。如果 `lsof` 只能找到部分路径，它会显示文件系统名，然后是 ` -- `，最后是它能找到的路径组件。

**示例输出可能如下：**
```
COMMAND   PID   USER   FD   TYPE DEVICE SIZE/OFF   NODE NAME
sshd    12345   root  cwd    DIR    1,2     4096      2 / -- /usr/bin
```
这表示 `lsof` 知道文件在根文件系统 `/` 上，并且从缓存中找到了路径 `/usr/bin`。

### 为什么路径信息有时不准确或不完整？

*   **缓存刷新**: 内核名称缓存是动态的。其他进程的活动可能会导致缓存条目被替换，所以 `lsof` 在重复模式下运行时，同一文件的路径信息可能会变化。
*   **不支持的文统**: 像 AFS 这样的文件系统不使用标准的内核名称缓存，所以 `lsof` 无法为它们报告路径组件。
*   **不准确的缓存**: 在极少数情况下（例如，在一个快速变化的文件系统上，inode 号被重用），内核缓存可能会包含过时的信息，导致 `lsof` 显示错误的路径。

### 如何禁用名称缓存查找？

使用 `-C` 选项可以完全禁止 `lsof` 使用内核名称缓存。

---

## lsof 的设备缓存文件 (Device Cache File)

### 什么是设备缓存文件？

在启动时，`lsof` 需要扫描 `/dev` 目录下的所有设备节点，以收集设备号、inode 号和路径等信息。这是一个耗时的操作，而且这些信息通常不会频繁改变。

为了加速启动，`lsof` 会创建一个名为 `.lsof_hostname` 的缓存文件（通常在用户的家目录下），存储这次扫描的结果。在下一次运行时，`lsof` 只需读取这个缓存文件，而不是重新扫描整个 `/dev` 目录，从而大大提高了性能。

### 如何管理设备缓存？

你可以使用 `-D` 选项来控制 `lsof` 对设备缓存的行为：

*   **`-Di` (忽略)**: 这是最有用的选项之一。如果你怀疑缓存文件已损坏或过时，或者你想获取最实时的设备信息，可以使用此选项。`lsof` 将忽略缓存文件，直接从 `/dev` 扫描。

    **示例：**
    ```bash
    # 忽略缓存，强制重新扫描设备
    lsof -Di
    ```

*   **`-Dr` (只读)**: `lsof` 会读取缓存文件，但如果发现文件有问题，它不会尝试重建它。

*   **`-Db` (构建)**: 强制 `lsof` 重新构建一个新的缓存文件。

*   **`-D?` (查询)**: 显示 `lsof` 将要使用的缓存文件的路径信息。

### 安全注意事项

`lsof` 会对缓存文件进行完整性校验（如 CRC 校验），以防止意外或恶意的修改。如果校验失败，`lsof` 会发出警告并尝试重建缓存。缓存文件通常以 `0600` 的权限创建，只有文件所有者可以读写。
