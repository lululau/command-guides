# xcodebuild Address Sanitizer/Thread Sanitizer/Undefined Behavior Sanitizer 详解

## Address Sanitizer（ASan）

Address Sanitizer 是一种内存错误检测工具，可以检测内存越界、Use-After-Free 等问题。

**启用方式：**
```shell
xcodebuild -scheme MyScheme -enableAddressSanitizer YES
```

## Thread Sanitizer（TSan）

Thread Sanitizer 用于检测多线程数据竞争问题。

**启用方式：**
```shell
xcodebuild -scheme MyScheme -enableThreadSanitizer YES
```

## Undefined Behavior Sanitizer（UBSan）

Undefined Behavior Sanitizer 用于检测 C/C++/Objective-C 代码中的未定义行为。

**启用方式：**
```shell
xcodebuild -scheme MyScheme -enableUndefinedBehaviorSanitizer YES
```

## 典型用法

- 在调试构建时启用消毒器：
  ```shell
  xcodebuild -scheme MyScheme -configuration Debug -enableAddressSanitizer YES
  ```
- 结合测试：
  ```shell
  xcodebuild test -scheme MyScheme -enableThreadSanitizer YES
  ```

## 注意事项
- 启用消毒器会影响性能，建议仅在 Debug 或测试阶段使用。
- 需在 Xcode 工程设置中也启用对应 Sanitizer 以获得最佳效果。 