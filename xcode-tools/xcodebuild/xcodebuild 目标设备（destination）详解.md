# xcodebuild 目标设备（destination）详解

## 什么是 destination？

destination（目标设备）用于指定 xcodebuild 构建或测试时的目标平台、设备或模拟器。

## 格式

destination 由一组 key=value 对组成，多个 key 用逗号分隔。例如：

```
platform=iOS Simulator,name=iPhone 14,OS=16.4
```

常用 key：
- platform：平台（如 macOS、iOS、iOS Simulator、tvOS、tvOS Simulator 等）
- name：设备名称（如 iPhone 14、My iPad）
- id：设备唯一标识符（UDID）
- OS：系统版本（如 16.4、latest）
- arch：架构（如 arm64、x86_64，常用于 macOS）

## 常见用法示例

### 指定 iOS 模拟器
```shell
xcodebuild -scheme MyScheme -destination 'platform=iOS Simulator,name=iPhone 14'
```

### 指定物理 iOS 设备（通过 id）
```shell
xcodebuild -scheme MyScheme -destination 'platform=iOS,id=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX'
```

### 指定 macOS 架构
```shell
xcodebuild -scheme MyScheme -destination 'platform=macOS,arch=arm64'
```

### 指定多个 destination 并发测试
```shell
xcodebuild test -scheme MyScheme \
  -destination 'platform=iOS Simulator,name=iPhone 14' \
  -destination 'platform=iOS Simulator,name=iPhone 14 Plus'
```

## 进阶说明
- destination 可指定多次，xcodebuild 会在多个设备/模拟器上并发测试。
- 某些平台（如 watchOS）需要配对的 iOS 设备 destination。
- 可用 destination 可通过 `-showdestinations` 查询：
  ```shell
  xcodebuild -scheme MyScheme -showdestinations
  ```

## 参考
- [官方文档：xcodebuild -destination](https://developer.apple.com/documentation/xcode/reference-xcodebuild-destination-specifier) 