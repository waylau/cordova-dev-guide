Android 配置
====

`config.xml` 文件控制应用在各个应用和 CordovaWebView 实例的基本设置。本文只涉及 Android 构建部分。详见 [config.xml 文件](../docs/config_ref.md)。

- `KeepRunning` (boolean, 默认`true`): 确定项目是否在触发 `[pause](../../../cordova/events/events.pause.html)`事件后在后台运行。设置为 `false` 并不会在 `[pause](../../../cordova/events/events.pause.html)` 事件后杀掉应用,应用程序是在后台而只是暂停 cordova webview 的执行。

        <preference name="KeepRunning" value="false"/>

- `LoadUrlTimeoutValue` (数值是毫秒, 默认值是 `20000`,
  20 秒): 当加载页面时，超时会抛出错误。下面是一个设置为10秒的例子

        <preference name="LoadUrlTimeoutValue" value="10000"/>

- `SplashScreen` (string, 默认是`splash`): 该文件减去其在`res/drawable`目录扩展的名称。各种资源必须在不同的子目录共享这一名字。

        <preference name="SplashScreen" value="mySplash"/>

- `SplashScreenDelay` (数值是毫秒, 默认值是 `3000`): 闪屏图像显示的时间。

        <preference name="SplashScreenDelay" value="10000"/>

- `InAppBrowserStorageEnabled` (boolean, 默认是 `true`): 控制InAppBrowser 默认的浏览器打开的页面是否可以访问相同的 localStorage 和 WebSQL 存储。

        <preference name="InAppBrowserStorageEnabled" value="true"/>

- `LoadingDialog` (string, 默认是`null`): 如果设置，加载应用程序的第一页时，对话框显示指定的标题和消息，以及一个旋转器。标题和消息之间用逗号分离出来，并在显示对话框之前这个逗号被删除。

        <preference name="LoadingDialog" value="My Title,My Message"/>

- `LoadingPageDialog` (string, 默认是`null`): 与`LoadingDialog`类似,
  但在第一个页面之后加载所有页面

        <preference name="LoadingPageDialog" value="My Title,My Message"/>

- `ErrorUrl` (URL, 默认是 `null`):如果设置，则会显示一个错误页面来代替"Application Error"对话框.

        <preference name="ErrorUrl" value="myErrorPage.html"/>

- `ShowTitle` (boolean, 默认是`false`): 显示标题在屏幕上方

        <preference name="ShowTitle" value="true"/>

- `LogLevel` (string, 默认是`ERROR`): 设置日志级别，值包括`ERROR`, `WARN`, `INFO`, `DEBUG`, 和 `VERBOSE`.

        <preference name="LogLevel" value="VERBOSE"/>

- `SetFullscreen` (boolean, 默认是`false`): 与全局配置文件 xnl 中的`Fullscreen` 参数类似。该属性在未来版本将会移除，不推荐使用

- `AndroidLaunchMode` (string, 默认是`singleTop`): 设置 Activity
  `android:launchMode` 属性。这个会在应用图标启动或者已经运行在运行时发生变化，值包括 `standard`, `singleTop`, `singleTask`, `singleInstance`.

        <preference name="AndroidLaunchMode" value="singleTop"/>

- `DefaultVolumeStream` (string, 默认是`default`, 在 cordova-android 3.7.0 版本加入): 设置硬件音量按钮链接到哪个音量。默认情况下，这是“call”链接到手机，而“media”链接的是平板电脑。设置“media”到你的应用程序的音量按钮将会随时改变媒体音量。需要注意的是使用 Cordova 的媒体插件时，当任何媒体对象是活动时，音量按键会动态变化，以控制媒体音量。

- `OverrideUserAgent` (string, 默认不设置):如果设置，该值将取代 webview 的老用户代理。从应用程序/浏览器请求远程页面时，对于确定该请求是有用的。请谨慎使用这个设置，因为可能会导致与 Web 服务器的兼容性问题。对于大多数情况下，使用AppendUserAgent 代替。

        <preference name="OverrideUserAgent" value="Mozilla/5.0 My Browser" />

- `AppendUserAgent` (string, 默认不设置):如果设置，该值将追加到的 webview 的老用户代理的尾部。当 OverrideUserAgent 使用，该值将被忽略。

        <preference name="AppendUserAgent" value="My Browser" />


