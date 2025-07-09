# xcode-select 中文手册与使用指南

## 名称

**xcode-select** - 管理 Xcode 和 BSD 工具的活跃开发者目录

## 概述

```shell
xcode-select [-h|--help] [-s|--switch <路径>] [-p|--print-path] [-v|--version]
```

## 描述

`xcode-select` 用于控制 Xcode 及 BSD 开发工具（如 xcrun、xcodebuild、cc 等）所使用的开发者目录的位置。这也会影响 man(1) 命令查找开发工具手册页的位置。

通过该命令，你可以轻松地在不同版本的 Xcode 工具之间切换，或者在 Xcode 移动安装位置后更新路径。

### 用法说明

当系统上安装了多个 Xcode 应用（如 `/Applications/Xcode.app` 和 `/Applications/Xcode-beta.app`）时，可以使用 `xcode-select --switch 路径` 指定你希望命令行开发工具使用的 Xcode。

设置开发者目录后，所有由 xcode-select 提供的开发工具 shim（见下文“文件”部分）会自动调用所选开发者目录下的工具。你自己的脚本、Makefile 及其他工具也可以通过 xcrun(1) 查找活跃开发者目录下的工具，从而方便地在不同 Xcode 版本间切换。

## 选项

### -h, --help
打印帮助信息。

**示例：**
```shell
xcode-select --help
```

### -s <路径>, --switch <路径>
将活跃开发者目录设置为指定路径，例如 `/Applications/Xcode-beta.app`。该命令需要超级用户权限（需用 sudo），会影响系统所有用户。若只想为当前 shell 会话设置路径或不想用超级用户权限，请使用 `DEVELOPER_DIR` 环境变量（见下文“环境变量”）。

**示例：**
```shell
sudo xcode-select --switch /Applications/Xcode-beta.app
```

### -p, --print-path
打印当前选中的开发者目录路径。适合检查当前设置，脚本和工具应优先用 xcrun(1) 查找工具。

**示例：**
```shell
xcode-select --print-path
```

### -r, --reset
取消用户指定的开发者目录，恢复为默认查找机制。需要超级用户权限，会影响所有用户。

**示例：**
```shell
sudo xcode-select --reset
```

### -v, --version
打印 xcode-select 版本信息。

**示例：**
```shell
xcode-select --version
```

### --install
弹出界面请求自动安装命令行开发工具。

**示例：**
```shell
xcode-select --install
```

## 环境变量

### DEVELOPER_DIR
覆盖活跃开发者目录。当设置了 `DEVELOPER_DIR`，其值会被用作开发者目录，而不是系统范围的设置。

> **延伸解读：**
> `DEVELOPER_DIR` 允许你临时指定 Xcode 的开发者目录，适合在脚本或 CI/CD 环境中切换 Xcode 版本，无需更改全局设置。
>
> **示例：**
> ```shell
> export DEVELOPER_DIR="/Applications/Xcode-beta.app"
> xcodebuild -version
> ```
> 或者：
> ```shell
> env DEVELOPER_DIR="/Applications/Xcode-beta.app" xcodebuild -version
> ```

注意：开发者目录通常指的是 Xcode 应用内的 `Contents/Developer` 目录（如 `/Applications/Xcode.app/Contents/Developer`）。你可以将环境变量设置为 Xcode 应用目录或其 Developer 子目录，xcode-select 会自动转换为完整路径。

## 示例

- 选择 `/Applications/Xcode.app/Contents/Developer` 作为活跃开发者目录：
  ```shell
  sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
  ```
- 选择 `/Applications/Xcode.app` 也会自动推断为 `Contents/Developer`：
  ```shell
  sudo xcode-select --switch /Applications/Xcode.app
  ```
- 运行活跃开发者目录下的 xcodebuild：
  ```shell
  /usr/bin/xcodebuild
  ```
- 用 xcrun 查找 xcodebuild：
  ```shell
  /usr/bin/xcrun --find xcodebuild
  ```
- 临时用其他开发者目录运行 xcodebuild：
  ```shell
  env DEVELOPER_DIR="/Applications/Xcode-beta.app" /usr/bin/xcodebuild
  ```

## 文件

- `/usr/bin/xcrun`：用于查找或运行活跃开发者目录下的任意命令。详见 xcrun(1)。
- `/usr/bin/xcodebuild`、`/usr/bin/ibtool`、`/usr/bin/opendiff` 等：会从活跃开发者目录运行对应的 Xcode 工具。
- `/usr/bin/clang`、`/usr/bin/gcc`、`/usr/bin/make`、`/usr/bin/swift` 等：会从活跃开发者目录运行对应的 BSD 工具。

## 相关命令

- [xcrun(1)](../xcrun/xcrun-man.txt)
- [xcodebuild(1)](../xcodebuild/xcodebuild-man.txt)

## 历史

xcode-select 命令最早出现在 Xcode 3.0。

---

> **延伸解读：开发者目录（Developer Directory）**
>
> 开发者目录是 Xcode 应用包中的一个子目录，包含所有开发工具、SDK、头文件等。通常路径为 `/Applications/Xcode.app/Contents/Developer`。切换开发者目录可以让你在不同版本的 Xcode 之间快速切换开发环境。
>
> **示例：**
> - 你有 Xcode 正式版和测试版，想用测试版编译项目：
>   ```shell
>   sudo xcode-select --switch /Applications/Xcode-beta.app
>   ```
> - 只想临时切换，不影响全局：
>   ```shell
>   env DEVELOPER_DIR="/Applications/Xcode-beta.app" xcodebuild
>   ``` 