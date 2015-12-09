Android shell 工具
====

本文展示了如何使用以平台为中心的 shell 工具来开发 Android 应用。这些工具允许你创建、构建和运行 Android 应用。

## 创建项目

执行`create` 命令，在 Mac/Linux 和 Windows 的语法如下：

```
$ /path/to/cordova-android/bin/create /path/to/project com.example.project_name ProjectName

C:\>\path\to\cordova-android\bin\create.bat \path\to\project com.example.project_name ProjectName
```

## 构建

执行这个，先会清空项目而后才构建。


Debug, 在 Mac/Linux 和 Windows 的语法如下：

```
$ /path/to/project/cordova/build --debug

C:\>\path\to\project\cordova\build.bat --debug
```
    
Release, 在 Mac/Linux 和 Windows 的语法如下：

```
$ /path/to/project/cordova/build --release

C:\>\path\to\project\cordova\build.bat --release
```

## 运行

`run`命令可以接受如下可选参数：

* 指定 Target，包括： `--emulator`, `--device`, 或 `--target=<targetID>`
* 指定 Build，包括： `--debug`, `--release`, 或 `--nobuild`

```
$ /path/to/project/cordova/run [Target] [Build]

C:\>\path\to\project\cordova\run.bat [Target] [Build]
```

请确保你至少创建一个 Android Virtual Device,，否则系统会提示您用`android`命令这样做。如果有多个 AVD 可作为一个目标，系统会提示您选择一个。默认情况下运行命令检测已连接的设备，或当前正在运行的模拟器（如果没有设备被发现）。

## 对应用签名

Android 应用签名的需求，可查看 <http://developer.android.com/tools/publishing/app-signing.html>

签名一个应用，需要如下参数：

* Keystore (--keystore): 存储 key 的编译文件的路径
* Keystore password (--storePassword): keystore 密码
* Alias (--alias): 指定用于签名的私钥 id
* Password (--password): 指定密钥的密码
* Type of the keystore (--keystoreType): pkcs12, jks (默认值：自动检测基于文件扩展名)

这些参数可以在 `build` 或 `run` 脚本里面指定。

另外，您也可以使用（`--buildConfig`）参数在构建配置文件（build.json）中指定。下面是一个构建配置文件的示例：

```
{
     "android": {
         "debug": {
             "keystore": "..\android.keystore",
             "storePassword": "android",
             "alias": "mykey1",
             "password" : "password",
             "keystoreType": ""
         },
         "release": {
             "keystore": "..\android.keystore",
             "storePassword": "",
             "alias": "mykey2",
             "password" : "password",
             "keystoreType": ""
         }
     }
 }
```
 
对于发行的签名，密码可以被排除在外，构建系统会发出提示，要求输入密码。

也有支持混合和匹配的命令行参数和 build.json 文件的参数。命令行参数的值将获得优先权。这对于在命令行上指定密码是有用的。

## 日志

```
$ /path/to/project/cordova/log

C:\>\path\to\project\cordova\log.bat
```
    
## 清除

```
$ /path/to/project/cordova/clean

C:\>\path\to\project\cordova\clean.bat
```

## 使用 Gradle 构建

自 cordova-android@4.0.0，项目的构建使用 [Gradle](http://www.gradle.org/)。对于 Gradle 的学习，可以参阅[《Gradle 2 用户指南》](<https://github.com/waylau/Gradle-2-User-Guide>) 。


### Gradle 属性

下面[属性](http://www.gradle.org/docs/current/userguide/tutorial_this_and_that.html)可以在构建时自定义：

* cdvBuildMultipleApks (默认: false)

如果这样设置，那么多个 APK 文件，将生成：每一个库项目（X86，ARM等）支持本地平台生成一个。这可能是重要的，如果你的项目使用了大量本地库，它可以显着提高所产生的 APK 的大小。

如果没有设置，则一个单个的 APK 将生成可以在所有设备中使用。

* cdvVersionCode

覆盖 AndroidManifest.xml 中的 versionCode。

* cdvReleaseSigningPropertiesFile (默认: release-signing.properties)

包含签名信息的发布构建的 `.properties` 文件路径，类似于:

```
storeFile=relative/path/to/keystore.p12
storePassword=SECRET1
storeType=pkcs12
keyAlias=DebugSigningKey
keyPassword=SECRET2
```

`storePassword` 和 `keyPassword` 是可选的，如果省略会提示。

* cdvDebugSigningPropertiesFile (默认: debug-signing.properties)

和 cdvReleaseSigningPropertiesFile 一样,但是为了调试构建。在需要和其他开发者分享签名密钥时非常有用

* cdvMinSdkVersion

覆盖 AndroidManifest.xml 中的 minSdkVersion。当创建基于 SDK 版本多个 APK 时非常有用

* cdvBuildToolsVersion

重载自动检测 android.buildToolsVersion 值

* cdvCompileSdkVersion

重载自动检测 android.compileSdkVersion 值

### 扩展 build.gradle

自定义 build.gradle, 不是直接编辑，而是应该创建一个同级文件名为`build-extras.gradle`。该文件将被包含在主`build.gradle`。下面是一个例子：:

```
# Example build-extras.gradle
# This file is included at the beginning of `build.gradle`
ext.cdvDebugSigningPropertiesFile = '../../android-debug-keys.properties'
# When set, this function allows code to run at the end of `build.gradle`
ext.postBuildExtras = {
    android.buildTypes.debug.applicationIdSuffix = '.debug'
}
```

注意插件也可以包含进来，通过在 `build-extras.gradle` 文件设置:

```
<framework src="some.gradle" custom="true" type="gradleReference" />
```

### 构建示例

```
export ORG_GRADLE_PROJECT_cdvMinSdkVersion=14
cordova build android -- --gradleArg=-PcdvBuildMultipleApks=true
```