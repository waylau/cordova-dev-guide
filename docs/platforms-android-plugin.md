Android 插件
====

本节详细介绍了如何在 Android 平台上的实现原生插件代码。在读这篇文章前，请参阅应用程序插件关于插件的结构和共同的 JavaScript 接口的概述。本节展示“回声”插件示例，来实现 Cordova webview 与原生平台的通信交互。该示例也可见
[CordovaPlugin.java](https://github.com/apache/cordova-android/blob/master/framework/src/org/apache/cordova/CordovaPlugin.java).

基于 Cordova-Android 的 Android 插件，是 Android 的 WebView 与连接到其挂钩。插件被表示为在 config.xml 文件类的映射。一个插件至少有一个Java类，是继承了 CordovaPlugin 类，覆盖它的 `execute`方法之一。作为最佳实践，该插件还应该处理任何消息的插件之间传递的`[pause](../../../cordova/events/events.pause.html)` 和 `[resume](../../../cordova/events/events.resume.html)` 事件。插件长时间运行的请求，后台活动，如媒体播放，监听器，或内部状态应该实现 `onReset()`方法为好。它在 `WebView` 导航到新页面或刷新时执行，从而重新加载 JavaScript。

## 插件 类 映射

插件的 JavaScript 接口使用 `cordova.exec` 方法如下:

        exec(<successFunction>, <failFunction>, <service>, <action>, [<args>]);

请求从 WebView 发起到 Android 端，调用`service` 类的  `action` 方法，传递 `args`数组参数。

发布一个插件，无论是 Java 文件或者是 _jar_ 文件，插件必须在你的 Cordova-Android 应用的 `res/xml/config.xml` 文件指定。见应用插件如何使用`plugin.xml` 文件来插入`feature` 元素:

        <feature name="<service_name>">
            <param name="android-package" value="<full_name_including_namespace>" />
        </feature>

service 名称在 JavaScript 的 `exec` 调用中相匹配。该值是 Java 类的完全限定命名空间的标识。否则，该插件即使可以编译，仍无法在 Cordova 使用。

## 插件的初始化及有效期

插件对象实例由每个 `WebView` 的有效期内创建。插件不会初始化，直到第一个引用被 JavaScript 调用, 除非 `config.xml` 的 `<param>` 中的 `onload`
的`name`属性设置为 `"true"`，如:

    <feature name="Echo">
        <param name="android-package" value="<full_name_including_namespace>" />
        <param name="onload" value="true" />
    </feature>

插件的 `initialize` 方法启动如下：

    @Override
    public void initialize(CordovaInterface cordova, CordovaWebView webView) {
        super.initialize(cordova, webView);
        // your init code here
    }

## 编写 Android Java 插件

插件类的 `execute` 方法，实现方式一般如下:

        @Override
        public boolean execute(String action, JSONArray args, CallbackContext callbackContext) throws JSONException {
            if ("beep".equals(action)) {
                this.beep(args.getLong(0));
                callbackContext.success();
                return true;
            }
            return false;  // Returning false results in a "MethodNotFound" error.
        }

JavaScript `exec` 方法的 `action` 参数对应私有类的可选参数的调用。

当捕捉异常并返回错误，JavaScript 的错误应该尽可能与 Java 异常相匹配。

## 线程

插件的 JavaScript 并不运行在 `WebView` 接口的主线程，而是运行在 `WebCore`线程的 `execute` 方法。如果需与其他用户界面交互，使用方法修改如下 :

        @Override
        public boolean execute(String action, JSONArray args, final CallbackContext callbackContext) throws JSONException {
            if ("beep".equals(action)) {
                final long duration = args.getLong(0);
                cordova.getActivity().runOnUiThread(new Runnable() {
                    public void run() {
                        ...
                        callbackContext.success(); // Thread-safe.
                    }
                });
                return true;
            }
            return false;
        }

当不需在主界面线程运行，又不想阻塞 `WebCore`线程，使用如下:

        @Override
        public boolean execute(String action, JSONArray args, final CallbackContext callbackContext) throws JSONException {
            if ("beep".equals(action)) {
                final long duration = args.getLong(0);
                cordova.getThreadPool().execute(new Runnable() {
                    public void run() {
                        ...
                        callbackContext.success(); // Thread-safe.
                    }
                });
                return true;
            }
            return false;
        }

## 添加依赖库

如果插件需额外的库，需要在 `config.xml` 添加，

方法1 通过 _Gradle_ 引用，比如:

        <framework src="com.android.support:support-v4:+" />

详情可参见 _Gradle_ 的依赖管理，参考[《Gradle 2 用户指南》](https://github.com/waylau/Gradle-2-User-Guide)

方法2  _JAR_ 文件替换插件的文件夹使用 `lib-file` 来链接,比如:

        <lib-file src="src/android/libs/gcm.jar"/>

用此方法的前提是你非常清楚你特定插件的依赖 jar 不会被其他插件所使用，不然会代码构建的问题。

## Echo Android Plugin 示例

本例演示 JavaScript 接口的 _echo_ 功能，使用 `plugin.xml` 来注入 `feature` 到 `config.xml` 文件:

        <platform name="android">
            <config-file target="config.xml" parent="/*">
                <feature name="Echo">
                    <param name="android-package" value="org.apache.cordova.plugin.Echo"/>
                </feature>
            </config-file>
        </platform>

添加 `src/org/apache/cordova/plugin/Echo.java` 文件:

        package org.apache.cordova.plugin;

        import org.apache.cordova.CordovaPlugin;
        import org.apache.cordova.CallbackContext;

        import org.json.JSONArray;
        import org.json.JSONException;
        import org.json.JSONObject;

        /**
         * This class echoes a string called from JavaScript.
         */
        public class Echo extends CordovaPlugin {

            @Override
            public boolean execute(String action, JSONArray args, CallbackContext callbackContext) throws JSONException {
                if (action.equals("echo")) {
                    String message = args.getString(0);
                    this.echo(message, callbackContext);
                    return true;
                }
                return false;
            }

            private void echo(String message, CallbackContext callbackContext) {
                if (message != null && message.length() > 0) {
                    callbackContext.success(message);
                } else {
                    callbackContext.error("Expected one non-empty string argument.");
                }
            }
        }

文件继承 `CordovaPlugin`，并覆盖 `execute()` 方法，来接收 `exec()`的消息。`execute()` 方法首先测试`action`的值，在这种情况下，只有一个有效 `echo`值。任何其他操作都返回 `false`，并导致一个`INVALID_ACTION` 错误，给 JavaScript 回调。

接下来，该方法检索使用 `args` 对象的 `getString` 方法，指定传递给该方法的第一个参数。值传递给私有  `echo`  方法后，对参数进行检查，以确保它不是 `null`或空字符串，在这种情况下 `callbackContext.error()` 调用 JavaScript 的错误回调。如果各种检查都通过，`callbackContext.success()` 把原始 `message` 字符串返回到 JavaScript 的成功回调作为参数。

## Android 集成

Android 功能 `Intent` 允许与其他进程进行交互。插件访问
`CordovaInterface` 对象，而该对象可以访问应用的 Android `Activity`。这个`Context` 需要启动一个新的 Android `Intent`。 `CordovaInterface` 允许插件来启动一个 `Activity` 用于处理结果，并当`Intent` 返回给应用时设置回调插件。

Cordova 2.0  插件不能直接访问 `Context`, 遗留的`ctx`成员不再推荐使用。所有的 `ctx` 方法存在于 `Context`, 所以 `getContext()`和`getActivity()`可以返回所需的对象。

## Android 权限

Android 权限是在安装时而不是在运行时的进行处理。这些权限需要在使用时在应用程序中声明，并且这些权限需要被添加到 Android Manifest。这可以通过使用在config.xml 注入在 AndroidManifest.xml 文件中来实现这些权限。下面的示例为使用联系人权限。

        <config-file target="AndroidManifest.xml" parent="/*">
            <uses-permission android:name="android.permission.READ_CONTACTS" />
        </config-file>

## Android 权限 (Cordova-Android 5.0.x 及跟高版本)

Android 6.0 "Marshmallow" 引入了新的权限模型，可以在需要时启动后者关闭权限。他意味着应用程序处理这些权限更改必须是面向未来的，这是Cordova-Android 5.0 版本的重点。

需要在运行时处理权限可以在 [Android 开发者文档](http://developer.android.com/guide/topics/security/permissions.html#perm-groups)中找到。

至于一个插件而言，权限请求通过权限方法，该方法的签名如下：

        cordova.reqquestPermission(CordovaPlugin plugin, int requestCode, String permission);

为减少冗长，标准的做法是分配一个本地静态变量：

    public static final String READ = Manifest.permission.READ_CONTACTS;

同时定义一个请求编码，如下:

    public static final int SEARCH_REQ_CODE = 0;

在 exec 方法,权限将会被检查:

            if(cordova.hasPermission(READ)) {
                search(executeArgs);
            }
            else
            {
                getReadPermission(SEARCH_REQ_CODE);
            }

此时，我们只需调用 requestPermission:

    protected void getReadPermission(int requestCode)
    {
        cordova.requestPermission(this, requestCode, READ);
    }

这会调用 activity 并会提示要求权限。若用户有权限，则结果必须被 onRequestPermissionResult 方法处理。这是每个插件必须覆盖的。示例如下:

    public void onRequestPermissionResult(int requestCode, String[] permissions,
                                             int[] grantResults) throws JSONException
    {
        for(int r:grantResults)
        {
            if(r == PackageManager.PERMISSION_DENIED)
            {
                this.callbackContext.sendPluginResult(new PluginResult(PluginResult.Status.ERROR, PERMISSION_DENIED_ERROR));
                return;
            }
        }
        switch(requestCode)
        {
            case SEARCH_REQ_CODE:
                search(executeArgs);
                break;
            case SAVE_REQ_CODE:
                save(executeArgs);
                break;
            case REMOVE_REQ_CODE:
                remove(executeArgs);
                break;
        }
    }

权限也可以用数组形式，下面是 Geolocation 插件:

    String [] permissions = { Manifest.permission.ACCESS_COARSE_LOCATION, Manifest.permission.ACCESS_FINE_LOCATION };

当请求权限时，只需如下处理:
    
    cordova.requestPermissions(this, 0, permissions);

这就要求在数组中指定权限。这是一个好主意，提供可公开访问的权限数组，因为这可以由使用你的插件作为依赖插件，虽然这不是必需的。

## 调试 Android 插件

Android 的调试可以使用 Eclipse 或 Android Studio 来完成，虽然推荐使用 Android Studio。由于 Cordova-Android 目前用作一个库项目，并且插件支持源代码，这使得在 Cordova 应用中调试 Java 代码就像是原生的 Android 应用。

