# xcodebuild 中文手册与使用指南

## 名称

**xcodebuild** – 构建 Xcode 项目和工作区

## 概述

```shell
xcodebuild [-project name.xcodeproj] [[-target targetname] ... | -alltargets] [-configuration configurationname] [-sdk [sdkfullpath | sdkname]] [action ...] [buildsetting=value ...] [-userdefault=value ...]

xcodebuild [-project name.xcodeproj] -scheme schemename [[-destination destinationspecifier] ...] [-destination-timeout value] [-configuration configurationname] [-sdk [sdkfullpath | sdkname]] [action ...] [buildsetting=value ...] [-userdefault=value ...]

xcodebuild -workspace name.xcworkspace -scheme schemename [[-destination destinationspecifier] ...] [-destination-timeout value] [-configuration configurationname] [-sdk [sdkfullpath | sdkname]] [action ...] [buildsetting=value ...] [-userdefault=value ...]

xcodebuild -version [-sdk [sdkfullpath | sdkname]] [infoitem]

xcodebuild -showsdks

xcodebuild -showBuildSettings [-project name.xcodeproj | [-workspace name.xcworkspace -scheme schemename]]

xcodebuild -showdestinations [-project name.xcodeproj | [-workspace name.xcworkspace -scheme schemename]]

xcodebuild -showTestPlans [-project name.xcodeproj | -workspace name.xcworkspace] -scheme schemename

xcodebuild -list [-project name.xcodeproj | -workspace name.xcworkspace]

xcodebuild -exportArchive -archivePath xcarchivepath -exportPath destinationpath -exportOptionsPlist path
xcodebuild -exportNotarizedApp -archivePath xcarchivepath -exportPath destinationpath

xcodebuild -exportLocalizations -project name.xcodeproj -localizationPath path [[-exportLanguage language] ...]
xcodebuild -importLocalizations -project name.xcodeproj -localizationPath path
```

## 描述

`xcodebuild` 用于构建一个或多个 Xcode 项目的 target，或构建 Xcode 工作区或项目中的 scheme。

### 用法说明

- 构建 Xcode 项目时，在包含项目（.xcodeproj）的目录下运行 xcodebuild。若目录下有多个项目文件，需用 `-project` 指定要构建的项目。
- 默认构建项目中第一个 target，使用默认构建配置。
- 构建 Xcode 工作区时，必须同时传递 `-workspace` 和 `-scheme`。
- 还有一些用于显示 Xcode 版本或本地项目/工作区信息的选项，如 `-list`、`-showBuildSettings`、`-showdestinations`、`-showsdks`、`-showTestPlans`、`-usage`、`-version`。

## 选项与参数

### -project name.xcodeproj
指定要构建的项目文件。

**示例：**
```shell
xcodebuild -project MyProject.xcodeproj
```

### -target targetname
指定要构建的 target。

**示例：**
```shell
xcodebuild -target MyTarget
```

### -alltargets
构建项目中的所有 target。

**示例：**
```shell
xcodebuild -alltargets
```

### -workspace name.xcworkspace
指定要构建的工作区。

**示例：**
```shell
xcodebuild -workspace MyWorkspace.xcworkspace
```

### -scheme schemename
指定要构建的 scheme。构建工作区时必需。

**示例：**
```shell
xcodebuild -scheme MyScheme
```

### -destination destinationspecifier
指定构建或测试的目标设备。

**示例：**
```shell
xcodebuild -scheme MyScheme -destination 'platform=iOS Simulator,name=iPhone 14'
```

> **延伸解读：**
> 见《xcodebuild 目标设备（destination）详解.md》

### -destination-timeout value
查找目标设备的超时时间（秒），默认 30 秒。

**示例：**
```shell
xcodebuild -destination 'platform=iOS Simulator,name=iPhone 14' -destination-timeout 60
```

### -configuration configurationname
指定构建配置（如 Debug 或 Release）。

**示例：**
```shell
xcodebuild -configuration Release
```

### -arch architecture
指定构建架构（如 arm64、x86_64）。

**示例：**
```shell
xcodebuild -arch arm64
```

### -sdk [sdkfullpath | sdkname]
指定 SDK。

**示例：**
```shell
xcodebuild -sdk iphoneos
```

### -showsdks
列出所有可用 SDK。

**示例：**
```shell
xcodebuild -showsdks
```

### -showBuildSettings
显示项目或工作区的构建设置。

**示例：**
```shell
xcodebuild -showBuildSettings -scheme MyScheme
```

### -showdestinations
列出可用的目标设备。

**示例：**
```shell
xcodebuild -showdestinations -scheme MyScheme
```

### -showBuildTimingSummary
显示构建各步骤的耗时报告。

**示例：**
```shell
xcodebuild -showBuildTimingSummary
```

### -showTestPlans
列出指定 scheme 的测试计划。

**示例：**
```shell
xcodebuild -showTestPlans -scheme MyScheme
```

### -list
列出项目的 target 和配置，或工作区的 scheme。

**示例：**
```shell
xcodebuild -list -project MyProject.xcodeproj
```

### -enableAddressSanitizer [YES | NO]
启用/禁用地址消毒器（Address Sanitizer）。

**示例：**
```shell
xcodebuild -enableAddressSanitizer YES
```

> **延伸解读：**
> 见《xcodebuild Address Sanitizer 延伸解读.md》

### -enableThreadSanitizer [YES | NO]
启用/禁用线程消毒器（Thread Sanitizer）。

**示例：**
```shell
xcodebuild -enableThreadSanitizer YES
```

### -enableUndefinedBehaviorSanitizer [YES | NO]
启用/禁用未定义行为消毒器（Undefined Behavior Sanitizer）。

**示例：**
```shell
xcodebuild -enableUndefinedBehaviorSanitizer YES
```

### -enableCodeCoverage [YES | NO]
启用/禁用测试时的代码覆盖率。

**示例：**
```shell
xcodebuild -enableCodeCoverage YES
```

### -testLanguage language
指定测试时的语言（ISO 639-1）。

**示例：**
```shell
xcodebuild -testLanguage zh
```

### -testRegion region
指定测试时的地区（ISO 3166-1）。

**示例：**
```shell
xcodebuild -testRegion CN
```

### -derivedDataPath path
指定派生数据（Derived Data）目录。

**示例：**
```shell
xcodebuild -derivedDataPath ./DerivedData
```

### -resultBundlePath path
指定测试结果包的保存路径。

**示例：**
```shell
xcodebuild -resultBundlePath ./TestResults
```

### -allowProvisioningUpdates
允许自动签名时与 Apple Developer 网站通信。

**示例：**
```shell
xcodebuild -allowProvisioningUpdates
```

### -allowProvisioningDeviceRegistration
允许注册设备到 Apple Developer 网站。

**示例：**
```shell
xcodebuild -allowProvisioningDeviceRegistration
```

### -authenticationKeyPath
指定 App Store Connect 认证密钥路径。

**示例：**
```shell
xcodebuild -authenticationKeyPath ./AuthKey.p8
```

### -authenticationKeyID
指定认证密钥的 Key ID。

**示例：**
```shell
xcodebuild -authenticationKeyID ABCDE12345
```

### -authenticationKeyIssuerID
指定认证密钥的 Issuer ID。

**示例：**
```shell
xcodebuild -authenticationKeyIssuerID 12345678-1234-1234-1234-123456789012
```

### -exportArchive
导出归档包（archive）。需配合 -archivePath、-exportOptionsPlist、-exportPath。

**示例：**
```shell
xcodebuild -exportArchive -archivePath MyApp.xcarchive -exportPath ./Exported -exportOptionsPlist export.plist
```

### -exportNotarizedApp
导出已 notarize 的归档包。

**示例：**
```shell
xcodebuild -exportNotarizedApp -archivePath MyApp.xcarchive -exportPath ./Exported
```

### -archivePath xcarchivepath
指定归档包路径。

**示例：**
```shell
xcodebuild -archivePath ./MyApp.xcarchive
```

### -exportPath destinationpath
指定导出产品的目标路径。

**示例：**
```shell
xcodebuild -exportPath ./Exported
```

### -exportOptionsPlist path
指定导出选项 plist 文件。

**示例：**
```shell
xcodebuild -exportOptionsPlist export.plist
```

### -exportLocalizations
导出本地化 XLIFF 文件。

**示例：**
```shell
xcodebuild -exportLocalizations -project MyProject.xcodeproj -localizationPath ./localizations -exportLanguage zh-Hans
```

### -importLocalizations
导入本地化 XLIFF 文件。

**示例：**
```shell
xcodebuild -importLocalizations -project MyProject.xcodeproj -localizationPath ./localizations/zh-Hans.xliff
```

### -localizationPath
指定本地化文件或目录路径。

**示例：**
```shell
xcodebuild -localizationPath ./localizations
```

### -exportLanguage language
指定导出本地化的语言。

**示例：**
```shell
xcodebuild -exportLanguage zh-Hans
```

### action ...
指定要执行的操作，如 build、test、archive、clean 等。

**示例：**
```shell
xcodebuild build
xcodebuild test
xcodebuild archive
xcodebuild clean
```

> **延伸解读：**
> 见《xcodebuild 常用 action 延伸解读.md》

### -xcconfig filename
加载指定的 xcconfig 配置文件。

**示例：**
```shell
xcodebuild -xcconfig Debug.xcconfig
```

### -testProductsPath path-to-xctestproducts
指定 xctestproducts 路径。

**示例：**
```shell
xcodebuild build-for-testing -testProductsPath ./MyScheme.xctestproducts
```

### -xctestrun xctestrunpath
指定 xctestrun 文件路径。

**示例：**
```shell
xcodebuild test-without-building -xctestrun MyTestRun.xctestrun
```

### -testPlan test-plan-name
指定测试计划。

**示例：**
```shell
xcodebuild test -scheme MyScheme -testPlan MyTestPlan
```

### -skip-testing test-identifier, -only-testing test-identifier
指定跳过或仅测试的目标。

**示例：**
```shell
xcodebuild test -only-testing MyTests/FooTests/testFooWithBar
```

### -skip-test-configuration test-configuration-name, -only-test-configuration test-configuration-name
指定跳过或仅测试的测试配置。

**示例：**
```shell
xcodebuild test -only-test-configuration Debug
```

### -disable-concurrent-destination-testing
禁用多目标并发测试。

**示例：**
```shell
xcodebuild test -disable-concurrent-destination-testing
```

### -maximum-concurrent-test-device-destinations number
指定同时测试的物理设备数量。

**示例：**
```shell
xcodebuild test -maximum-concurrent-test-device-destinations 2
```

### -maximum-concurrent-test-simulator-destinations number
指定同时测试的模拟器数量。

**示例：**
```shell
xcodebuild test -maximum-concurrent-test-simulator-destinations 2
```

### -parallel-testing-enabled [YES | NO]
启用/禁用并行测试。

**示例：**
```shell
xcodebuild test -parallel-testing-enabled YES
```

### -parallel-testing-worker-count number
指定并行测试的 worker 数量。

**示例：**
```shell
xcodebuild test -parallel-testing-worker-count 4
```

### -maximum-parallel-testing-workers number
限制并行测试 worker 的最大数量。

**示例：**
```shell
xcodebuild test -maximum-parallel-testing-workers 4
```

### -parallelize-tests-among-destinations
在多个目标设备间分配测试类。

**示例：**
```shell
xcodebuild test -parallelize-tests-among-destinations
```

### -test-timeouts-enabled [YES | NO]
启用/禁用测试超时。

**示例：**
```shell
xcodebuild test -test-timeouts-enabled YES
```

### -default-test-execution-time-allowance seconds
指定单个测试的默认超时时间。

**示例：**
```shell
xcodebuild test -default-test-execution-time-allowance 60
```

### -maximum-test-execution-time-allowance seconds
指定单个测试的最大超时时间。

**示例：**
```shell
xcodebuild test -maximum-test-execution-time-allowance 120
```

### -test-iterations number
指定测试重复次数。

**示例：**
```shell
xcodebuild test -test-iterations 5
```

### -retry-tests-on-failure
测试失败时重试。

**示例：**
```shell
xcodebuild test -retry-tests-on-failure
```

### -run-tests-until-failure
一直运行测试直到失败。

**示例：**
```shell
xcodebuild test -run-tests-until-failure
```

### -test-repetition-relaunch-enabled [YES | NO]
每次重复测试是否重启进程。

**示例：**
```shell
xcodebuild test -test-repetition-relaunch-enabled YES
```

### -collect-test-diagnostics [on-failure | never]
指定何时收集测试诊断信息。

**示例：**
```shell
xcodebuild test -collect-test-diagnostics on-failure
```

### -enumerate-tests
枚举将要执行的测试。

**示例：**
```shell
xcodebuild test -enumerate-tests
```

### -test-enumeration-style [hierarchical | flat]
指定测试枚举的组织方式。

**示例：**
```shell
xcodebuild test -test-enumeration-style hierarchical
```

### -test-enumeration-format [text | json]
指定测试枚举的输出格式。

**示例：**
```shell
xcodebuild test -test-enumeration-format json
```

### -test-enumeration-output-path [path | -]
指定测试枚举输出路径。

**示例：**
```shell
xcodebuild test -test-enumeration-output-path ./tests.json
```

### -dry-run, -n
仅打印将要执行的命令，不实际执行。

**示例：**
```shell
xcodebuild -n build
```

### -skipUnavailableActions
跳过无法执行的操作。

**示例：**
```shell
xcodebuild -scheme MyScheme -skipUnavailableActions
```

### buildsetting=value
设置构建参数。

**示例：**
```shell
xcodebuild -target MyTarget OBJROOT=./Obj SYMROOT=./Sym
```

### -userdefault=value
设置用户默认参数。

**示例：**
```shell
xcodebuild -userdefault=MyDefault
```

### -toolchain [identifier | name]
指定工具链。

**示例：**
```shell
xcodebuild -toolchain com.apple.dt.toolchain.Swift_5_7
```

### -quiet
只输出警告和错误。

**示例：**
```shell
xcodebuild -quiet build
```

### -verbose
输出更多状态信息。

**示例：**
```shell
xcodebuild -verbose build
```

### -version
显示 Xcode 版本信息。

**示例：**
```shell
xcodebuild -version
```

### -license
显示 Xcode 及 SDK 许可协议。

**示例：**
```shell
sudo xcodebuild -license
```

### -checkFirstLaunchStatus
检查是否需要安装额外组件。

**示例：**
```shell
xcodebuild -checkFirstLaunchStatus
```

### -runFirstLaunch
安装额外组件并同意许可协议。

**示例：**
```shell
sudo xcodebuild -runFirstLaunch
```

### -downloadAllPlatforms
下载并安装所有可用平台。

**示例：**
```shell
xcodebuild -downloadAllPlatforms
```

### -usage
显示 xcodebuild 用法信息。

**示例：**
```shell
xcodebuild -usage
```

## 目标设备（destination）详解

> 见《xcodebuild 目标设备（destination）详解.md》

## 常用 action 详解

> 见《xcodebuild 常用 action 延伸解读.md》

## Address Sanitizer 等消毒器详解

> 见《xcodebuild Address Sanitizer 延伸解读.md》

## 相关命令
- [xcode-select(1)](../xcode-select/xcode-select-man.txt)
- [xcrun(1)](../xcrun/xcrun-man.txt)
- [ibtool(1)](../../ibtool/ibtool-man.txt)

## 参考资料
- [Xcode Build Settings Reference](https://developer.apple.com/documentation/xcode/build-settings-reference) 