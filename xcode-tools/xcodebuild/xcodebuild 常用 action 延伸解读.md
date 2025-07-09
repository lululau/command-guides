# xcodebuild 常用 action 详解

## build
编译 target 或 scheme。

**示例：**
```shell
xcodebuild build -scheme MyScheme
```

## test
编译并运行测试。

**示例：**
```shell
xcodebuild test -scheme MyScheme -destination 'platform=iOS Simulator,name=iPhone 14'
```

## archive
归档 scheme，生成 .xcarchive 包。

**示例：**
```shell
xcodebuild archive -scheme MyScheme -archivePath ./MyApp.xcarchive
```

## clean
清理构建产物。

**示例：**
```shell
xcodebuild clean -scheme MyScheme
```

## build-for-testing
编译测试目标和相关 target，生成 .xctestrun 文件。

**示例：**
```shell
xcodebuild build-for-testing -scheme MyScheme -destination 'platform=iOS Simulator,name=iPhone 14'
```

## test-without-building
在不重新编译的情况下运行测试。

**示例：**
```shell
xcodebuild test-without-building -scheme MyScheme -destination 'platform=iOS Simulator,name=iPhone 14'
```

## analyze
静态分析代码。

**示例：**
```shell
xcodebuild analyze -scheme MyScheme
```

## installsrc
复制源码到源目录（SRCROOT）。

**示例：**
```shell
xcodebuild installsrc -scheme MyScheme
```

## install
编译并安装到目标目录（DSTROOT）。

**示例：**
```shell
xcodebuild install -scheme MyScheme
```

## docbuild
编译文档。

**示例：**
```shell
xcodebuild docbuild -scheme MyScheme
``` 