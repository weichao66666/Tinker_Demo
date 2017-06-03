
[toc]

---

#<center>**Tinker 热修复/热补丁**</center>

---

#*TODO*

- [] `tinkerId is not set` 逐步产生原因
- [] `tinkerId` 未由 `aaaaaa` 改为 `bbbbbb` 是因为什么
- [] 原理

---

#**Reference**

* [Tencent/tinker](https://github.com/Tencent/tinker "https://github.com/Tencent/tinker")
* [Tinker 热修复 demo 使用指南](http://blog.csdn.net/wuqilianga/article/details/52757149 "http://blog.csdn.net/wuqilianga/article/details/52757149")
* [tinker demo 实现，注意点。](http://blog.csdn.net/pigwithbadguy/article/details/53377105 "http://blog.csdn.net/pigwithbadguy/article/details/53377105")

---

#**运行官方例程**

##**下载例程**

>https://github.com/Tencent/tinker/

##**导入 AS**

例程为：..\tinker-master\tinker-sample-android

##**生成基准 apk**

###**配置 tinkerId**

未设置 tinkerId，编译时报错：

	Error:Execution failed for task ':app:tinkerProcessDebugManifest'.
	tinkerId is not set!!!

![](https://github.com/weichao66666/Tinker_Demo/raw/master/README.md-assets/1496457558569_2.png)

这是因为默认设置 tinkerId 为获取到的 git 最近一次 commit 的版本号，所以如果当前 Project 没有配置 git，或者当前的 Project 还没有 commit 过，或者 git 没有加入到环境变量中，均获取不到该值，tinkerId 为 null。

编译补丁包时，会自动读取基准包 AndroidManifest 的 tinkerId 作为 package_meta.txt 中的 TINKER_ID。将本次编译传入的 tinkerId, 作为 package_meta.txt 中的 NEW_TINKER_ID。如果我们使用 git rev 作为 tinkerId, 这时只要使用 git diff TINKER_ID NEW_TINKER_ID 即可获得所有的代码差异。**如果升级了客户端版本，但 tinkerId 与旧版本相同，会导致加载旧版本的补丁。所以升级客户端版本，需要更新 tinkerId!**

    buildConfigField "String", "TINKER_ID", "\"${getTinkerIdValue()}\""

    def getTinkerIdValue() {
        return hasProperty("TINKER_ID") ? TINKER_ID : gitSha()
    }
    
    def gitSha() {
        try {
            String gitRev = 'git rev-parse --short HEAD'.execute(null, project.rootDir).text.trim()
            if (gitRev == null) {
                throw new GradleException("can't get git rev, you should add git to system path or just input test value, such as 'testTinkerId'")
            }
            return gitRev
        } catch (Exception e) {
            throw new GradleException("can't get git rev, you should add git to system path or just input test value, such as 'testTinkerId'")
        }
    }

如果获取不到该值，可以主动设置。

修改 ..\tinker-master\tinker-sample-android\app\build.gradle 的 tinkerId 为固定字符串：

    //            tinkerId = getTinkerIdValue()
                tinkerId = "aaaaaa"

###**配置 ignoreWarning**

修改 ..\tinker-master\tinker-sample-android\app\build.gradle 的 ignoreWarning 为 true：

    //        ignoreWarning = false
            ignoreWarning = true

###**配置 keystore，与实际使用保持一致**

修改 ..\tinker-master\tinker-sample-android\app\build.gradle：

    signingConfigs {
        release {
            try {
                storeFile file("D:\\jsxf_workspace\\bsszjc_android_keystore.jks")
                storePassword "aaaaaa"
                keyAlias "aaaaaa"
                keyPassword "aaaaaa"
            } catch (ex) {
                throw new InvalidUserDataException(ex.toString())
            }
        }

        debug {
            storeFile file("D:\\jsxf_workspace\\bsszjc_android_keystore.jks")
        }
    }

###**以打包方式生成基准 apk 并安装运行**

![](https://github.com/weichao66666/Tinker_Demo/raw/master/README.md-assets/59325e3bb3b0bb706c000004.png)![](https://github.com/weichao66666/Tinker_Demo/raw/master/README.md-assets/59325e44b3b0bb706c000005.png)

Tinker 会在工程的 app\build\bakApk 目录下保存打包好的 apk 文件。

![](https://github.com/weichao66666/Tinker_Demo/raw/master/README.md-assets/1496470408618_2.png)

##**生成热修复/热补丁**

###**配置 tinkerOldApkPath**

修改 ..\tinker-master\tinker-sample-android\app\build.gradle 的 tinkerOldApkPath：

    //    tinkerOldApkPath = "${bakPath}/app-debug-1018-17-32-47.apk"
        tinkerOldApkPath = "${bakPath}/app-release-0603-14-12-17.apk"

###**配置 tinkerApplyResourcePath**

修改 ..\tinker-master\tinker-sample-android\app\build.gradle 的 tinkerApplyResourcePath：

    //    tinkerApplyResourcePath = "${bakPath}/app-debug-1018-17-32-47-R.txt"
        tinkerApplyResourcePath = "${bakPath}/app-release-0603-14-12-17-R.txt"

###**配置 tinkerApplyMappingPath**

修改 ..\tinker-master\tinker-sample-android\app\build.gradle 的 tinkerApplyMappingPath：

    //    tinkerApplyMappingPath = "${bakPath}/app-debug-1018-17-32-47-mapping.txt"
        tinkerApplyMappingPath = "${bakPath}/app-release-0603-14-12-17-mapping.txt"

###**配置 tinkerId**

    //            tinkerId = "aaaaaa"
                tinkerId = "bbbbbb"

###**修改一些内容**

比如在布局文件 ..\tinker-master\tinker-sample-android\app\src\main\res\layout\activity_main.xml 最下面加一个文本框：

    <?xml version="1.0" encoding="utf-8"?>
    <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:paddingBottom="@dimen/activity_vertical_margin"
        android:paddingLeft="@dimen/activity_horizontal_margin"
        android:paddingRight="@dimen/activity_horizontal_margin"
        android:paddingTop="@dimen/activity_vertical_margin"
        tools:context=".app.MainActivity">
    
        <TextView
            android:id="@+id/textView"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Sample patch!" />
    
        <Button
            android:id="@+id/loadPatch"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_alignParentLeft="true"
            android:layout_alignParentStart="true"
            android:layout_below="@+id/textView"
            android:text="load patch" />
    
        <Button
            android:id="@+id/loadLibrary"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_alignParentLeft="true"
            android:layout_alignParentStart="true"
            android:layout_below="@+id/loadPatch"
            android:text="load library" />
    
        <Button
            android:id="@+id/cleanPatch"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_alignParentLeft="true"
            android:layout_alignParentStart="true"
            android:layout_below="@+id/loadLibrary"
            android:text="clean patch" />
    
        <Button
            android:id="@+id/killSelf"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_alignParentLeft="true"
            android:layout_alignParentStart="true"
            android:layout_below="@+id/cleanPatch"
            android:text="kill self" />
    
        <Button
            android:id="@+id/showInfo"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_alignParentLeft="true"
            android:layout_alignParentStart="true"
            android:layout_below="@+id/killSelf"
            android:text="show info" />
    
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_alignParentLeft="true"
            android:layout_alignParentStart="true"
            android:layout_below="@+id/showInfo"
            android:text="Hello, Tinker!" />
    </RelativeLayout>

比如修改 ..\tinker-master\tinker-sample-android\app\build.gradle 的 MESSAGE：

    //        buildConfigField "String", "MESSAGE", "\"I am the base apk\""
            buildConfigField "String", "MESSAGE", "\"I am the patch apk\""

###**生成差异文件**

debug 执行 tinkerPatchDebug，release 执行 tinkerPatchRelease：

![](https://github.com/weichao66666/Tinker_Demo/raw/master/README.md-assets/593257acb3b0bb706c000003.png)

Tinker 会在工程的 app\build\bakApk 目录下保存新打包好的 apk 文件。

同时会在 ..\tinker-master\tinker-sample-android\app\build\outputs\tinkerPatch 目录下保存生成的基准 apk 和新 apk 的差异文件 patch_signed_7zip.apk：

![](https://github.com/weichao66666/Tinker_Demo/raw/master/README.md-assets/593260a3b3b0bb706c000006.png)

##**使用热修复/热补丁**

###**复制差异文件到指定位置**

将 patch_signed_7zip.apk 复制到根目录（MainActivity 中定义了名称和位置）。

        loadPatchButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                TinkerInstaller.onReceiveUpgradePatch(getApplicationContext(), Environment.getExternalStorageDirectory().getAbsolutePath() + "/patch_signed_7zip.apk");
            }
        });

###**加载差异文件并重启应用才能生效**

点击界面中的第一个按钮 LOAD PATCH：

    06-03 16:41:37.650 32023-32023/? I/Tinker.SamplePatchListener: receive a patch file: /storage/emulated/0/patch_signed_7zip.apk, file size:6521
    06-03 16:41:37.662 32023-32023/? W/Tinker.UpgradePatchRetry: onPatchListenerCheck retry file is not exist, just return
    06-03 16:41:37.665 32023-32023/? I/Tinker.SamplePatchListener: get platform:all
    06-03 16:41:37.685 4253-5458/? I/ActivityManager: Start proc 32597:tinker.sample.android:patch/u0a308 for service tinker.sample.android/com.tencent.tinker.lib.service.TinkerPatchService
    06-03 16:41:37.773 32597-32597/? W/Tinker.TinkerLoader: tryLoadPatchFiles: we don't load patch with :patch process itself, just return
    06-03 16:41:37.774 32597-32597/? D/Tinker.DefaultAppLike: onBaseContextAttached:
    06-03 16:41:37.777 32597-32597/? I/Tinker.SamplePatchListener: application maxMemory:192
    06-03 16:41:37.778 32597-32597/? W/Tinker.Tinker: tinker patch directory: /data/user/0/tinker.sample.android/tinker
    06-03 16:41:37.779 32597-32597/? I/Tinker.Tinker: try to install tinker, isEnable: true, version: 1.7.11
    06-03 16:41:37.779 32597-32597/? I/Tinker.TinkerLoadResult: parseTinkerResult loadCode:-1, process name:tinker.sample.android:patch, main process:false, systemOTA:false, oatDir:null, useInterpretMode:false
    06-03 16:41:37.779 32597-32597/? W/Tinker.TinkerLoadResult: tinker is disable, just return
    06-03 16:41:37.780 32597-32597/? I/Tinker.DefaultLoadReporter: patch loadReporter onLoadResult: patch load result, path:/data/user/0/tinker.sample.android/tinker, code: -1, cost: 3ms
    06-03 16:41:37.780 32597-32597/? W/Tinker.Tinker: tinker load fail!
    06-03 16:41:37.783 32597-32597/? D/Tinker.DefaultAppLike: onCreate
    06-03 16:41:37.786 32597-32597/? D/Tinker.DefaultAppLike: onTrimMemory level:80
    06-03 16:41:37.786 32597-32611/? I/Tinker.DefaultPatchReporter: patchReporter onPatchServiceStart: patch service start
    06-03 16:41:37.787 32597-32597/? W/Tinker.UpgradePatchRetry: onPatchRetryLoad retry is not main process, just return
    06-03 16:41:37.790 32597-32611/? W/Tinker.UpgradePatchRetry: try copy file: /storage/emulated/0/patch_signed_7zip.apk to /data/user/0/tinker.sample.android/tinker_temp/temp.apk
    06-03 16:41:37.793 32597-32611/? I/Tinker.TinkerPatchService: try to increase patch process priority
    06-03 16:41:37.794 4253-4266/? V/ActivityManager: Attempted to start a foreground service (ComponentInfo{tinker.sample.android/com.tencent.tinker.lib.service.TinkerPatchService}) with a broken notification (no icon: Notification(pri=0 contentView=null vibrate=null sound=null defaults=0x0 flags=0x40 color=0x00000000 vis=PRIVATE))
    06-03 16:41:37.804 4253-4266/? V/ActivityManager: Attempted to start a foreground service (ComponentInfo{tinker.sample.android/com.tencent.tinker.lib.service.TinkerPatchService$InnerService}) with a broken notification (no icon: Notification(pri=0 contentView=null vibrate=null sound=null defaults=0x0 flags=0x40 color=0x00000000 vis=PRIVATE))
    06-03 16:41:37.843 32597-32611/? I/Tinker.UpgradePatch: UpgradePatch tryPatch:patchMd5:68ff10e1ccbf624dc89a3dbb8aef1045
    06-03 16:41:37.844 32597-32611/? W/Tinker.PatchInfo: read property failed, e:java.io.FileNotFoundException: /data/user/0/tinker.sample.android/tinker/patch.info: open failed: ENOENT (No such file or directory)
    06-03 16:41:37.844 32597-32611/? W/Tinker.PatchInfo: read property failed, e:java.io.FileNotFoundException: /data/user/0/tinker.sample.android/tinker/patch.info: open failed: ENOENT (No such file or directory)
    06-03 16:41:37.844 32597-32611/? I/Tinker.UpgradePatch: UpgradePatch tryPatch:patchVersionDirectory:/data/user/0/tinker.sample.android/tinker/patch-68ff10e1
    06-03 16:41:37.845 32597-32611/? W/Tinker.UpgradePatch: UpgradePatch copy patch file, src file: /storage/emulated/0/patch_signed_7zip.apk size: 6521, dest file: /data/user/0/tinker.sample.android/tinker/patch-68ff10e1/patch-68ff10e1.apk size:6521
    06-03 16:41:38.519 32597-32611/? W/Tinker.DexDiffPatchInternal: success recover dex file: /data/user/0/tinker.sample.android/tinker/patch-68ff10e1/dex/classes.dex.jar, size: 421728, use time: 667
    06-03 16:41:38.519 32597-32611/? I/Tinker.DexDiffPatchInternal: try Extracting /data/user/0/tinker.sample.android/tinker/patch-68ff10e1/dex/test.dex.jar
    06-03 16:41:38.521 32597-32611/? I/Tinker.DexDiffPatchInternal: isExtractionSuccessful: true
    06-03 16:41:38.521 32597-32611/? I/Tinker.DexDiffPatchInternal: patch recover, try to optimize dex file count:2, optimizeDexDirectory:/data/user/0/tinker.sample.android/tinker/patch-68ff10e1/odex/
    06-03 16:41:38.525 32597-32613/? I/Tinker.DexDiffPatchInternal: start to parallel optimize dex /data/user/0/tinker.sample.android/tinker/patch-68ff10e1/dex/classes.dex.jar, size: 421728
    06-03 16:41:38.526 32597-32614/? I/Tinker.DexDiffPatchInternal: start to parallel optimize dex /data/user/0/tinker.sample.android/tinker/patch-68ff10e1/dex/test.dex.jar, size: 470
    06-03 16:41:38.692 32597-32614/? I/Tinker.DexDiffPatchInternal: success to parallel optimize dex /data/user/0/tinker.sample.android/tinker/patch-68ff10e1/dex/test.dex.jar, opt file size: 12720, use time 166
    06-03 16:41:40.353 32597-32613/? I/Tinker.DexDiffPatchInternal: success to parallel optimize dex /data/user/0/tinker.sample.android/tinker/patch-68ff10e1/dex/classes.dex.jar, opt file size: 2617776, use time 1827
    06-03 16:41:40.355 32597-32611/? I/Tinker.ParallelDex: All dexes are optimized successfully, cost: 1831 ms.
    06-03 16:41:40.356 32597-32611/? I/Tinker.DexDiffPatchInternal: recover dex result:true, cost:2508
    06-03 16:41:40.357 32597-32611/? W/Tinker.BsDiffPatchInternal: patch recover, library is not contained
    06-03 16:41:40.362 32597-32611/? I/Tinker.ResDiffPatchInternal: res dir: /data/user/0/tinker.sample.android/tinker/patch-68ff10e1/res/, meta: resArscMd5:3c153ddbeec07517b2de63472ecea8e8
                                                                    arscBaseCrc:1559301911
                                                                    pattern:res/.*
                                                                    pattern:resources\.arsc
                                                                    pattern:assets/.*
                                                                    addedSet:assets/only_use_to_test_tinker_resource.txt
                                                                    modifiedSet:res/layout/activity_main.xml
                                                                    modifiedSet:res/layout-v17/activity_main.xml
    06-03 16:41:40.369 32597-32611/? I/Tinker.ResDiffPatchInternal: no large modify resources, just return
    06-03 16:41:40.516 32597-32611/? I/Tinker.ResDiffPatchInternal: final new resource file:/data/user/0/tinker.sample.android/tinker/patch-68ff10e1/res/resources.apk, entry count:427, size:524389
    06-03 16:41:40.516 32597-32611/? I/Tinker.ResDiffPatchInternal: recover resource result:true, cost:158
    06-03 16:41:40.516 32597-32611/? I/Tinker.DexDiffPatchInternal: dex count: 2, final wait time: 12
    06-03 16:41:40.520 32597-32611/? I/Tinker.DexDiffPatchInternal: check dex optimizer file exist: classes.dex.dex, size 2617776
    06-03 16:41:40.520 32597-32611/? I/Tinker.DexDiffPatchInternal: check dex optimizer file exist: test.dex.dex, size 12720
    06-03 16:41:40.521 32597-32611/? I/Tinker.DexDiffPatchInternal: check dex optimizer file format: classes.dex.dex, size 2617776
    06-03 16:41:40.524 32597-32611/? I/Tinker.DexDiffPatchInternal: check dex optimizer file format: test.dex.dex, size 12720
    06-03 16:41:40.525 32597-32611/? I/Tinker.PatchInfo: rewritePatchInfoFile file path:/data/user/0/tinker.sample.android/tinker/patch.info , oldVer:, newVer:68ff10e1ccbf624dc89a3dbb8aef1045, fingerprint:Huawei/MT7-UL00/hwmt7:6.0/HuaweiMT7-UL00/C17B571:user/release-keys, oatDir:odex
    06-03 16:41:40.527 32597-32611/? W/Tinker.UpgradePatch: UpgradePatch tryPatch: done, it is ok
    06-03 16:41:40.527 32597-32611/? I/Tinker.DefaultPatchReporter: patchReporter onPatchResult: patch all result path: /storage/emulated/0/patch_signed_7zip.apk, success: true, cost: 2734
    06-03 16:41:40.528 32597-32611/? I/Tinker.PatchFileUtil: safeDeleteFile, try to delete path: /data/user/0/tinker.sample.android/tinker_temp/temp.apk
    06-03 16:41:40.553 32023-32636/? I/Tinker.SampleResultService: SampleResultService receive result: 
                                                                   PatchResult: 
                                                                   isSuccess:true
                                                                   rawPatchFilePath:/storage/emulated/0/patch_signed_7zip.apk
                                                                   costTime:2734
    06-03 16:41:40.554 32023-32636/? W/Tinker.DefaultTinkerResultService: deleteRawPatchFile rawFile path: /storage/emulated/0/patch_signed_7zip.apk
    06-03 16:41:40.555 32023-32636/? I/Tinker.PatchFileUtil: safeDeleteFile, try to delete path: /storage/emulated/0/patch_signed_7zip.apk
    06-03 16:41:40.555 32023-32636/? I/Tinker.SampleResultService: tinker wait screen to restart process

点击按钮 KILL SELF，再打开应用，会发现修改的内容已经更新了，并且差异文件被删除了。

![](https://github.com/weichao66666/Tinker_Demo/raw/master/README.md-assets/5932781db3b0bb706c000008.png)![](https://github.com/weichao66666/Tinker_Demo/raw/master/README.md-assets/59327821b3b0bb706c000009.png)

#**可能遇到的问题**

##**签名异常**

    06-03 15:25:54.920 6888-6904/? E/ShareSecurityCheck: /storage/emulated/0/patch_signed_7zip.apk
        java.security.SignatureException
            at com.android.org.conscrypt.OpenSSLX509Certificate.verifyOpenSSL(OpenSSLX509Certificate.java:353)
            at com.android.org.conscrypt.OpenSSLX509Certificate.verify(OpenSSLX509Certificate.java:384)
            at org.apache.harmony.security.utils.WrappedX509Certificate.verify(WrappedX509Certificate.java:156)
            at com.tencent.tinker.loader.shareutil.ShareSecurityCheck.a(ShareSecurityCheck.java:158)
            at com.tencent.tinker.loader.shareutil.ShareSecurityCheck.a(ShareSecurityCheck.java:133)
            at com.tencent.tinker.loader.shareutil.ShareTinkerInternals.a(ShareTinkerInternals.java:118)
            at com.tencent.tinker.loader.shareutil.ShareTinkerInternals.a(ShareTinkerInternals.java:102)
            at com.tencent.tinker.lib.c.f.a(UpgradePatch.java:60)
            at com.tencent.tinker.lib.service.TinkerPatchService.onHandleIntent(TinkerPatchService.java:129)
            at android.app.IntentService$ServiceHandler.handleMessage(IntentService.java:66)
            at android.os.Handler.dispatchMessage(Handler.java:102)
            at android.os.Looper.loop(Looper.java:150)
            at android.os.HandlerThread.run(HandlerThread.java:61)
    06-03 15:25:54.921 6888-6904/? E/Tinker.UpgradePatch: UpgradePatch tryPatch:onPatchPackageCheckFail

..\tinker-master\tinker-sample-android\app\build.gradle 中声明的签名文件和实际签名文件不一致，需要保持一致。

##**加载差异文件失败**

    06-03 16:17:40.254 20935-20935/? I/Tinker.SamplePatchListener: receive a patch file: /storage/emulated/0/patch_signed_7zip.apk, file size:6512
    06-03 16:17:40.258 20935-20935/? I/Tinker.DefaultLoadReporter: patch loadReporter onLoadPatchListenerReceiveFail: patch receive fail: /storage/emulated/0/patch_signed_7zip.apk, code: -2

Android 6.0+ 需要动态获取 SD 读取权限。

修改 ..\tinker-master\tinker-sample-android\app\src\main\java\tinker\sample\android\app\MainActivity.java：

    public class MainActivity extends AppCompatActivity {
        private static final String TAG = "Tinker.MainActivity";
    
        private static final String[] PERMISSIONS = new String[]{
                Manifest.permission.READ_EXTERNAL_STORAGE  // 读取权限
        };
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
    
            getPermissions();
        }
    
        private void getPermissions() {
            if (ContextCompat.checkSelfPermission(this, Manifest.permission.READ_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED) {
                // Android 6.0 申请权限
                requestPermissions(PERMISSIONS, 1);
            } else {
                init();
            }
        }
    
        @Override
        public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
            if (requestCode == 1) {
                if (grantResults[0] != PackageManager.PERMISSION_GRANTED) {
                    Log.d(TAG, "未授权不可使用");
                } else {
                    init();
                }
            }
        }
    
        private void init() {
            Log.e(TAG, "i am on onCreate classloader:" + MainActivity.class.getClassLoader().toString());
            //test resource change
            Log.e(TAG, "i am on onCreate string:" + getResources().getString(R.string.test_resource));
    //        Log.e(TAG, "i am on patch onCreate");
    
            Button loadPatchButton = (Button) findViewById(R.id.loadPatch);
    
            loadPatchButton.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    TinkerInstaller.onReceiveUpgradePatch(getApplicationContext(), Environment.getExternalStorageDirectory().getAbsolutePath() + "/patch_signed_7zip.apk");
                }
            });
    
            Button loadLibraryButton = (Button) findViewById(R.id.loadLibrary);
    
            loadLibraryButton.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    // #method 1, hack classloader library path
                    TinkerLoadLibrary.installNavitveLibraryABI(getApplicationContext(), "armeabi");
                    System.loadLibrary("stlport_shared");
    
                    // #method 2, for lib/armeabi, just use TinkerInstaller.loadLibrary
    //                TinkerLoadLibrary.loadArmLibrary(getApplicationContext(), "stlport_shared");
    
                    // #method 3, load tinker patch library directly
    //                TinkerInstaller.loadLibraryFromTinker(getApplicationContext(), "assets/x86", "stlport_shared");
    
                }
            });
    
            Button cleanPatchButton = (Button) findViewById(R.id.cleanPatch);
    
            cleanPatchButton.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    Tinker.with(getApplicationContext()).cleanPatch();
                }
            });
    
            Button killSelfButton = (Button) findViewById(R.id.killSelf);
    
            killSelfButton.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    ShareTinkerInternals.killAllOtherProcess(getApplicationContext());
                    android.os.Process.killProcess(android.os.Process.myPid());
                }
            });
    
            Button buildInfoButton = (Button) findViewById(R.id.showInfo);
    
            buildInfoButton.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    showInfo(MainActivity.this);
                }
            });
    
        }
    
        public boolean showInfo(Context context) {
            // add more Build Info
            final StringBuilder sb = new StringBuilder();
            Tinker tinker = Tinker.with(getApplicationContext());
            if (tinker.isTinkerLoaded()) {
                sb.append(String.format("[patch is loaded] \n"));
                sb.append(String.format("[buildConfig TINKER_ID] %s \n", BuildInfo.TINKER_ID));
                sb.append(String.format("[buildConfig BASE_TINKER_ID] %s \n", BaseBuildInfo.BASE_TINKER_ID));
    
                sb.append(String.format("[buildConfig MESSSAGE] %s \n", BuildInfo.MESSAGE));
                sb.append(String.format("[TINKER_ID] %s \n", tinker.getTinkerLoadResultIfPresent().getPackageConfigByName(ShareConstants.TINKER_ID)));
                sb.append(String.format("[packageConfig patchMessage] %s \n", tinker.getTinkerLoadResultIfPresent().getPackageConfigByName("patchMessage")));
                sb.append(String.format("[TINKER_ID Rom Space] %d k \n", tinker.getTinkerRomSpace()));
    
            } else {
                sb.append(String.format("[patch is not loaded] \n"));
                sb.append(String.format("[buildConfig TINKER_ID] %s \n", BuildInfo.TINKER_ID));
                sb.append(String.format("[buildConfig BASE_TINKER_ID] %s \n", BaseBuildInfo.BASE_TINKER_ID));
    
                sb.append(String.format("[buildConfig MESSSAGE] %s \n", BuildInfo.MESSAGE));
                sb.append(String.format("[TINKER_ID] %s \n", ShareTinkerInternals.getManifestTinkerID(getApplicationContext())));
            }
            sb.append(String.format("[BaseBuildInfo Message] %s \n", BaseBuildInfo.TEST_MESSAGE));
    
            final TextView v = new TextView(context);
            v.setText(sb);
            v.setGravity(Gravity.LEFT | Gravity.CENTER_VERTICAL);
            v.setTextSize(TypedValue.COMPLEX_UNIT_DIP, 10);
            v.setLayoutParams(new ViewGroup.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.WRAP_CONTENT));
            v.setTextColor(0xFF000000);
            v.setTypeface(Typeface.MONOSPACE);
            final int padding = 16;
            v.setPadding(padding, padding, padding, padding);
    
            final AlertDialog.Builder builder = new AlertDialog.Builder(context);
            builder.setCancelable(true);
            builder.setView(v);
            final AlertDialog alert = builder.create();
            alert.show();
            return true;
        }
    
        @Override
        protected void onResume() {
            super.onResume();
            Utils.setBackground(false);
        }
    
        @Override
        protected void onPause() {
            super.onPause();
            Utils.setBackground(true);
        }
    }

---















