# xcrun 中文手册与使用指南

## 名称

**xcrun** - 运行或定位开发工具及属性

## 概述

```shell
xcrun [--sdk <SDK 名称>] --find <工具名>

xcrun [--sdk <SDK 名称>] <工具名> ... 工具参数 ...

<工具名> ... 工具参数 ...
```

## 描述

`xcrun` 提供了一种在命令行查找或调用开发工具的方法，无需用户修改 Makefile 或为支持多个 Xcode 工具链而采取不便措施。

- `xcode-select(1)` 用于设置系统默认的活跃开发者目录，可通过 `DEVELOPER_DIR` 环境变量覆盖（见“环境变量”）。
- 默认搜索最新可用的 SDK，可通过 `SDKROOT` 环境变量或 `--sdk` 选项指定（`--sdk` 优先级更高）。
- 当用于调用其他工具时，xcrun 会在环境变量 `SDKROOT` 中提供所选 SDK 的绝对路径。

### 用法说明

- 使用 `--find` 参数时，如 `xcrun [--sdk <SDK 名称>] --find <工具名>`，会打印该工具的绝对路径。
- 不带 `--find` 时，需指定工具名，xcrun 会执行该工具并传递参数。
- 作为符号链接目标时，会根据调用名推断工具名并执行。

## 选项与参数

### -v, --verbose
显示工具查找过程的详细信息。

**示例：**
```shell
xcrun -v --find clang
```

### -n, --no-cache
查找时不使用缓存，强制刷新缓存。

**示例：**
```shell
xcrun -n --find clang
```

### -k, --kill-cache
清除缓存，强制所有值重新缓存。

**示例：**
```shell
xcrun -k --find clang
```

### --sdk <SDK 名称>
指定查找工具时使用的 SDK。

**示例：**
```shell
xcrun --sdk iphoneos --find texturetool
```

> **延伸解读：**
> 见《xcrun SDK 机制延伸解读.md》

### --toolchain <工具链>
指定查找工具时使用的工具链。

**示例：**
```shell
xcrun --toolchain com.apple.dt.toolchain.Swift_5_7 swiftc --version
```

### -l, --log
打印被调用的完整命令行。

**示例：**
```shell
xcrun -l clang --version
```

### -f, --find
启用“查找”模式，仅打印工具路径而不执行。

**示例：**
```shell
xcrun --find xcodebuild
```

### -r, --run
启用“运行”模式，执行工具（默认模式）。

**示例：**
```shell
xcrun -r clang --version
```

### --show-sdk-path
打印所选 SDK 的路径。

**示例：**
```shell
xcrun --sdk macosx --show-sdk-path
```

### --show-sdk-version
打印所选 SDK 的版本号。

**示例：**
```shell
xcrun --sdk iphoneos --show-sdk-version
```

### --show-sdk-build-version
打印所选 SDK 的构建版本号。

**示例：**
```shell
xcrun --sdk macosx --show-sdk-build-version
```

### --show-sdk-platform-path
打印所选 SDK 的平台路径。

**示例：**
```shell
xcrun --sdk macosx --show-sdk-platform-path
```

### --show-sdk-platform-version
打印所选 SDK 的平台版本号。

**示例：**
```shell
xcrun --sdk macosx --show-sdk-platform-version
```

## 环境变量

### CPATH
当未显式指定 SDK 且未传递 -nostdinc/-nostdsysteminc 时，xcrun 会将 `/usr/local/include` 加入 CPATH。

### DEVELOPER_DIR
覆盖活跃开发者目录。

### LIBRARY_PATH
当未显式指定 SDK 且未传递 -Z 给链接器时，xcrun 会将 `/usr/local/lib` 加入 LIBRARY_PATH。

### SDKROOT
指定默认 SDK。xcrun 调用工具时也会设置该变量为 SDK 的绝对路径。

**示例：**
```shell
xcrun --sdk macosx clang test.c
# clang 会自动使用 macosx SDK
```

### TOOLCHAINS
指定默认工具链。

### xcrun_log
等同于 --log。

### xcrun_nocache
等同于 --no-cache。

### xcrun_verbose
等同于 --verbose。

## 示例

- 查找 clang 路径：
  ```shell
  xcrun --find clang
  ```
- 查找 iOS SDK 下的 texturetool 路径：
  ```shell
  xcrun --sdk iphoneos --find texturetool
  ```
- 打印当前 macOS SDK 路径：
  ```shell
  xcrun --sdk macosx --show-sdk-path
  ```
- 执行 git status：
  ```shell
  xcrun git status
  ```

## 诊断与调试
- `--log` 和 `--verbose` 可用于调试工具查找过程。
- `--no-cache` 可跳过缓存，但会影响性能。
- 若 xcrun 作为其他工具的替身被调用，则不能使用 xcrun 选项，此时可用环境变量控制。

## 相关命令
- [xcodebuild(1)](../xcodebuild/xcodebuild-man.txt)
- [xcode-select(1)](../xcode-select/xcode-select-man.txt)

---

> **延伸解读：SDK 机制、工具链机制等详见对应延伸解读文件。** 