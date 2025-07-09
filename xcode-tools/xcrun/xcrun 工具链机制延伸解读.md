# xcrun 工具链（toolchain）机制详解

## 什么是工具链？

工具链（toolchain）是一组用于编译、链接、打包等的工具集合，通常包括编译器、链接器、调试器等。

## xcrun 如何选择工具链？

- 默认使用系统工具链。
- 可通过 `--toolchain` 选项或 `TOOLCHAINS` 环境变量指定。

## 常见工具链
- com.apple.dt.toolchain.XcodeDefault
- com.apple.dt.toolchain.Swift_5_7

## 示例

### 1. 使用 Swift 5.7 工具链编译
```shell
xcrun --toolchain com.apple.dt.toolchain.Swift_5_7 swiftc --version
```

### 2. 通过环境变量指定
```shell
export TOOLCHAINS=com.apple.dt.toolchain.Swift_5_7
xcrun swiftc --version
```

## 进阶说明
- 指定工具链可用于测试新版本编译器或自定义工具链。
- 某些第三方工具链也可通过 xcrun 调用。 