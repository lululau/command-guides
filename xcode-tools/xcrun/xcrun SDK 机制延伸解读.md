# xcrun SDK 机制详解

## 什么是 SDK？

SDK（Software Development Kit，软件开发工具包）是为特定平台（如 macOS、iOS、watchOS、tvOS）提供的头文件、库和工具的集合。

## xcrun 如何选择 SDK？

- 默认使用最新可用的 SDK。
- 可通过 `--sdk` 选项或 `SDKROOT` 环境变量指定。
- `--sdk` 优先级高于 `SDKROOT`。

## 常见 SDK 名称
- macosx
- iphoneos
- iphonesimulator
- watchos
- watchsimulator
- appletvos
- appletvsimulator

## 列出所有可用 SDK
```shell
xcodebuild -showsdks
```

## 示例

### 1. 查找 iOS SDK 下的 clang 路径
```shell
xcrun --sdk iphoneos --find clang
```

### 2. 使用指定 SDK 编译
```shell
xcrun --sdk macosx clang test.c
```

此时 clang 会自动使用 macosx SDK。

## 进阶说明
- 某些工具（如 clang）会自动识别 SDKROOT 环境变量。
- SDK 机制让你可以方便地为不同平台交叉编译。 