升级 Android
====

本文介绍了如何修改 Android 项目来从 旧版的 Cordova 升级。大部分指令适用于与旧的 `cordova` CLI 工具创建的项目。有关 CLI 内容，可以参阅[命令行界面（CLI）](../docs/cli.md)

## 升级至 4.0.0

升级至 4.0.0 有些特殊的步骤，首先，
对于非 CLI 项目，运行:

        bin/update path/to/project

对于 CLI 项目:

1. 升级 `cordova` CLI 版本，见[命令行界面（CLI）](../docs/cli.md)

2. 在现有的项目中运行 `cordova platform update android`。

### 升级 Whitelist

所有的 whitelist 功能现在都通过插件实现，当升级至 4.0.0 后，将不会受 whitelist 保护。 Cordova 有2个 whitelist 插件，来提供不同级别的保护：

1. `cordova-plugin-whitelist` 插件 *(推荐)*
  * 这个是值得推荐的，因为比之前的 whitelist 版本更加安全和可配置。
  * 详见 [cordova-plugin-whitelist](https://github.com/apache/cordova-plugin-whitelist)
  * 运行: `cordova plugin add cordova-plugin-crosswalk-webview`

2. `cordova-plugin-legacy-whitelist` 插件
  * 与之前的 whitelist 版本类似，见 [cordova-plugin-legacy-whitelist](https://github.com/apache/cordova-plugin-legacy-whitelist)
  *不会有配置上的变化，但比推荐的插件少一些保护
  * 运行: `cordova plugin add cordova-plugin-legacy-whitelist`

### 使用 Crosswalk WebView

默认，应用继续使用设备提供的系统 WebView。若想替代之，使用 Crosswalk 插件:

    cordova plugin add cordova-plugin-crosswalk-webview

### 升级至 Splashscreen 插件

若想使用闪屏，需使用该插件，配置方法没有变，只需如下方法添加插件:

    cordova plugin add cordova-plugin-splashscreen

## 其他旧版升级

可参阅 <http://cordova.apache.org/docs/en/latest/guide/platforms/android/upgrade.html>
