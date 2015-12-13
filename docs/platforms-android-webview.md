Android WebViews
====

本文演示了如何在大型 Android 应用里面，内嵌 Cordova-enabled WebView 组件。欲了解组件之间如何交互，参见[插件开发指南](../docs/hybrid-plugins.md)。

若你对 Android 还不熟悉，可以先参考[Android 平台](docs/platforms-android.md)，并安装 Android SDK。 Cordova 1.9 开始, Android 平台依赖`CordovaWebView` 组件,该组件是基于 `CordovaActivity`组件构建的。

1. 首先确保有最新的 Cordova 的发布包，从  [cordova.apache.org](http://cordova.apache.org) 下载，并解压  Android 包。

1. 切换到 Android 包的 `/framework` 目录，并运行 `ant jar`，它将创建 Cordova `.jar` 文件，格式如 `/framework/cordova-x.x.x.jar`

1. 拷贝 `.jar` 文件到 Android 项目的`/libs` 目录

1. 添加如下 `/res/xml/main.xml` 文件,设置相应的 `layout_height`, `layout_width` 和 `id`:

        <org.apache.cordova.CordovaWebView
            android:id="@+id/tutorialView"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />

1. 修改 activity 让其实现 `CordovaInterface`，它会实现包含的方法。可以拷贝自 `/framework/src/org/apache/cordova/CordovaActivity.java`,
  或者自己实现。下面的代码片段显示，依赖于接口上的基本应用。请注意引用视图 `id` 与上面显示的 XML 片段指定的id属性如何相匹配:

        public class CordovaViewTestActivity extends Activity implements CordovaInterface {
            CordovaWebView cwv;
            /* Called when the activity is first created. */
            @Override
            public void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setContentView(R.layout.main);
                cwv = (CordovaWebView) findViewById(R.id.tutorialView);
                Config.init(this);
                cwv.loadUrl(Config.getStartUrl());
            }

1. 若应用使用了摄像头，用法如下:

        @Override
        public void setActivityResultCallback(CordovaPlugin plugin) {
            this.activityResultCallback = plugin;
        }
        /**
         * Launch an activity for which you would like a result when it finished. When this activity exits,
         * your onActivityResult() method is called.
         *
         * @param command           The command object
         * @param intent            The intent to start
         * @param requestCode       The request code that is passed to callback to identify the activity
         */
        public void startActivityForResult(CordovaPlugin command, Intent intent, int requestCode) {
            this.activityResultCallback = command;
            this.activityResultKeepRunning = this.keepRunning;
            
            // If multitasking turned on, then disable it for activities that return results
            if (command != null) {
                this.keepRunning = false;
            }
        
            // Start activity
            super.startActivityForResult(intent, requestCode);
        }   
    
        @Override
        /**
         * Called when an activity you launched exits, giving you the requestCode you started it with,
         * the resultCode it returned, and any additional data from it.
         *
         * @param requestCode       The request code originally supplied to startActivityForResult(),
         *                          allowing you to identify who this result came from.
         * @param resultCode        The integer result code returned by the child activity through its setResult().
         * @param data              An Intent, which can return result data to the caller (various data can be attached to Intent "extras").
         */
        protected void onActivityResult(int requestCode, int resultCode, Intent intent) {
            super.onActivityResult(requestCode, resultCode, intent);
            CordovaPlugin callback = this.activityResultCallback;
            if (callback != null) {
                callback.onActivityResult(requestCode, resultCode, intent);
            }
        }

1. 最后，添加线程池，除非插件没有线程来运行:

        @Override
        public ExecutorService getThreadPool() {
            return threadPool;
        }

1. 拷贝应用的 HTML 和 JavaScript 文件到 Android 项目的 `/assets/www` 目录

1. 从 `/framework/res/xml` 拷贝 `config.xml` 文件到 `/res/xml` 目录
