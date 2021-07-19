## Build.Gradle文件配置
build.gradle文件分为两个，一个全局的，一个是对应module的。

<img src="https://github.com/jniosdeveloper/CloudImageBed/blob/master/location.jpg?raw=true" alt="module位置" style="zoom:50%;" />

下面是外层build.config文件的配置
```
// buildscript中的声明是gradle脚本自身需要使用的资源
// 可以声明的资源包括依赖项、第三方插件、maven仓库地址等
buildscript {
    apply from: "${rootDir.absolutePath}/versions.gradle"
    ext.kotlin_version = "1.4.10"
    // 仓库配置
    repositories {
        google()
        // 代码托管库：设置之后可以在项目中轻松引用jcenter上的开源项目
        jcenter()
    }
    dependencies {
        // 声明gradle插件
        classpath "com.android.tools.build:gradle:4.1.0"
        // 引入jetbrains关于kotlin的相关插件
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}
// 指定本地仓库 / 远程仓库
allprojects {
    repositories {
        google()
        jcenter()
        maven { url 'https://jitpack.io' }
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```

module下对应的build.gradle文件配置
```
static def realTime() {
    return new Date().format("yyMMdd-HHmmss")
}

static def changeApkName(applicationVariants) {
    // 重命名打包后的apk名称
    applicationVariants.all { variant ->
        variant.outputs.all { output ->
            output.outputFileName = "cdmetro_${variant.productFlavors[0].name}_v${variant.versionName}_${realTime()}_${variant.versionCode}.apk"
        }
    }
}
// 声明是 Android 应用程序还是库模块
plugins {
    // 表示这是一个应用程序模块,可直接运行
    id 'com.android.application'
    // 添加相关插件
    id 'kotlin-android'
    id 'kotlin-android-extensions'
    id 'kotlin-kapt'
}
// 配置项目构建的各种属性
android {
    // 编译sdk的版本
    compileSdkVersion versions.compileSdkVersion
    // 默认配置、应用程序包名、最小sdk版本、目标sdk版本、版本号、版本名称
    defaultConfig {
        // 应用程序的包名
        applicationId versions.applicationId
        // 最小sdk版本，如果设备小于这个版本或者大于maxSdkVersion将无法安装这个应用
        minSdkVersion versions.minSdkVersion
        // 目标sdk版本，充分测试过的版本（建议版本）
        targetSdkVersion versions.targetSdkVersion
        // 内部版本号
        versionCode versions.versionCode
        // 版本名，显示给用户看到的版本号(外部版本号)
        versionName versions.versionName
        // Instrumentation单元测试
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
        // 设置包含进APK里处理器架构文件
        ndk {
            abiFilters 'armeabi', 'armeabi-v7a', 'arm64-v8a'
        }
    }
    // 签名信息配置；
    signingConfigs {
        release {
            storeFile rootProject.file('hello.jks')
            storePassword '123456'
            keyAlias 'jackie'
            keyPassword '123456'
        }
    }
    // 指定生成安装文件的配置，是否对代码进行混淆；
    buildTypes {
        debug {
            signingConfig signingConfigs.release
        }
        release {
            signingConfig signingConfigs.release
            // 是否对代码进行混淆，true表示混淆
            minifyEnabled true
            // 移除无用的resource文件
            shrinkResources true
            // release的Proguard默认为Module下的proguard-rules.pro文件
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            changeApkName(applicationVariants)
        }
    }

    // 定义的产品特性 配置多种环境
    flavorDimensions "aaaa"
    productFlavors {
        cdTest {
            dimension "aaaa"
            buildConfigField("String", "HOST", "\"${HOST_DEBUG}\"")
        }
        cdProduct {
            dimension "aaaa"
            buildConfigField("String", "HOST", "\"${HOST_RELEASE}\"")
        }
    }

    // 进行 Java 的版本配置
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }

    kotlinOptions {
        jvmTarget = '1.8'
    }
}
// 指定当前项目的所有依赖关系，本地以来，库依赖以及远程依赖
dependencies {
    testImplementation 'junit:junit:4.+'
    androidTestImplementation 'androidx.test.ext:junit:1.1.1'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.2.0'

    // androidx
    implementation kotlin_jdk
    implementation deps.androidx.appcompat
    implementation deps.androidx.material
    implementation deps.androidx.core_ktx
    implementation deps.androidx.constraintlayout
    implementation deps.androidx.room
    implementation deps.androidx.room_rxjava2
    kapt deps.androidx.room_compiler

    // activity
    def activity_version = "1.2.3"
    // Kotlin
    implementation "androidx.activity:activity-ktx:$activity_version"

    // lifecycle
//    implementation deps.androidx.lifecycle
    def lifecycle_version = "2.3.1"
    // ViewModel
    implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:$lifecycle_version"
    // LiveData
    implementation "androidx.lifecycle:lifecycle-livedata-ktx:$lifecycle_version"
    // Lifecycles only (without ViewModel or LiveData)
    implementation "androidx.lifecycle:lifecycle-runtime-ktx:$lifecycle_version"
    // Saved state module for ViewModel
    implementation "androidx.lifecycle:lifecycle-viewmodel-savedstate:$lifecycle_version"
    // Jetpack Compose Integration for ViewModel
    implementation "androidx.lifecycle:lifecycle-viewmodel-compose:1.0.0-alpha04"
    // Annotation processor
    kapt "androidx.lifecycle:lifecycle-compiler:$lifecycle_version"

    // rxjava2
    implementation deps.rx.adapter_rxjava2
    implementation deps.rx.rxjava2
    implementation deps.rx.rxandroid
    implementation deps.rx.permisson

    // epoxy
    implementation deps.epoxy.lib
    kapt deps.epoxy.processor

    // glide
    implementation deps.image.glide
    implementation deps.image.transformations
    kapt deps.image.gilde_complier

    // smartrefresh
    implementation deps.smartrefresh

    // Retrofit
    implementation deps.http.retrofit
    implementation deps.http.converter_gson

    // okhttp
    implementation deps.http.okhttp_interceptor
    implementation deps.http.okhttp
}
```

## 关于Activity的相关基类

#### MvvmActivity
```
/**
 * 使用了lifecycle的activity
 *
 */
// 使用了lifecycle的activity:使用生命周期感知型组件处理VM生命周期
abstract class MvvmActivity<V: BaseViewModel>: AppCompatActivity() {
    /*
    * 类后面跟泛型列表参数的作用,表示类中的属性或者方法中需要用到泛型参数去声明
    * */
    protected var mViewModel: V? = null
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // 设置内容为全屏展示
        if (isSetInnerLayoutFullScreen())
            setInnerLayoutFullScreen()
        // 根据布局ID设置内容View
        setContentView(providerLayoutId())
        // 获取bundle传入数据
        getBundleData()
        // 初始化toolBar
        setToolbar()
        // 初始化ViewModel
        initVm()
        // 初始化布局控件
        initWidget()
        // 设置观察者
        setObserver()
        // 设置对属性的观察
        subscribeUi()
    }
    /**
     * 布局文件
     */
    abstract fun providerLayoutId(): Int
    /**
     * 获取bundle传入的数据
     */
    open fun getBundleData() {}
    private fun setToolbar() {
        // apply函数返回传入的对象的本身
        providerToolbar()?.apply {
            setSupportActionBar(this)
            supportActionBar?.let {
                it.setDisplayHomeAsUpEnabled(setHomeAsUpEnabled())
                it.setDisplayShowHomeEnabled(setHomeAsUpEnabled())
                if (providerTitle() > 0) top_title.setText(providerTitle())
            }
        }
        title = null
    }
    /**
     * 是否显示toolbar上的返回箭头
     */
    open fun setHomeAsUpEnabled(): Boolean = false
    /**
     * 设置toolbar navigation title
     */
    open fun providerTitle(): Int {
        return -1
    }
    /**
     * 是否有toolbar
     */

    open fun providerToolbar(): Toolbar? = null

    private fun initVm() {
        providerViewModel()?.let {
            val factory = providerFactory()
            mViewModel = if (factory != null) {
                ViewModelProvider(this, factory).get(it)
            } else {
                ViewModelProvider(this, ViewModelProvider.NewInstanceFactory()).get(it)
            }
            mViewModel?.bind2Life(lifecycle)
        }
    }
    /**
     * 绑定viewmodel
     */
    open fun providerViewModel(): Class<V>? = null
    /**
     * 提供viewmodel 的factory
     */
    open fun providerFactory(): ViewModelProvider.NewInstanceFactory? = null
    /**
     * 初始化一些控件
     */
    open fun initWidget() {}
    private fun setObserver() {
        // 判断object为null的操作
        provideObserver()?.let {
            lifecycle.addObserver(it)
        }
    }

    /**
     * 设置视图监听器
     * kotlin的可变参数
     */
    fun setOnClickListener(vararg views: View, listener: View.OnClickListener) {
        for (v in views) {
            v.setOnClickListener(listener)
        }
    }

    /**
     * 提供依赖activity生命周期的Oberver
     */
    open fun provideObserver(): LifecycleObserver? = null
    /**
     *设置对属性的观察
     */
    open fun subscribeUi() {}

    private fun hideSystemUI() {
        // Enables regular immersive mode.
        // For "lean back" mode, remove SYSTEM_UI_FLAG_IMMERSIVE.
        // Or for "sticky immersive," replace it with SYSTEM_UI_FLAG_IMMERSIVE_STICKY
        window.decorView.systemUiVisibility = (View.SYSTEM_UI_FLAG_IMMERSIVE
                // Set the content to appear under the system bars so that the
                // content doesn't resize when the system bars hide and show.
                or View.SYSTEM_UI_FLAG_LAYOUT_STABLE
                or View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
                or View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
                // Hide the nav bar and status bar
                or View.SYSTEM_UI_FLAG_HIDE_NAVIGATION
                or View.SYSTEM_UI_FLAG_FULLSCREEN)
    }
    open fun isSetFullScreen() = false
    /**
     * 是否设置Activity是否全屏，内容穿过Status bar
     */
    open fun isSetInnerLayoutFullScreen() = false
    /**
     * 设置activity为沉浸模式
     */
    private fun setInnerLayoutFullScreen() {

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M){
            /*
            * 每一个AC都对应一个window,而decorView则是window对应的布局
            * systemUiVisibility:改变状态栏或者其他系统UI的可见性
            * SYSTEM_UI_FLAG_LIGHT_STATUS_BAR:设置状态栏的颜色
            * SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN:设置全屏布局
            * */
            window.decorView.systemUiVisibility=View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN or View.SYSTEM_UI_FLAG_LIGHT_STATUS_BAR
//            window.decorView.systemUiVisibility = View.SYSTEM_UI_FLAG_FULLSCREEN or View.SYSTEM_UI_FLAG_LAYOUT_STABLE
        }else {

            window.apply {
                decorView.systemUiVisibility = View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN or View.SYSTEM_UI_FLAG_LAYOUT_STABLE
                //改为透明黑，避免纯白色标题栏看不到时间等图标
                statusBarColor = Color.parseColor("#20000000")
            }
            supportActionBar?.hide()
        }
    }

    /**
     * 状态栏高度
     */
    fun getStatusBarHeight(): Int {
        var statusBarHeight = 0
        val resourceId = resources.getIdentifier("status_bar_height", "dimen", "android")
        if (resourceId > 0) {
            statusBarHeight = resources.getDimensionPixelSize(resourceId)
        }
        return statusBarHeight
    }
}
```

#### BaseActivity

```
abstract class BaseActivity<V : BaseViewModel> : MvvmActivity<V>() {
    // 设置默认字体缩放大小
    private val mDefaultFontScale = 1f
    // toast
    private var mToast: Toast? = null
    /*
    * 当前Activity由被覆盖状态回到前台或解锁屏
    * */
    override fun onResume() {
        super.onResume()

    }
    /*
    * pause表示暂停，当Activity要跳到另一个Activity或应用正常退出时都会执行这个方法
    * */
    override fun onPause() {
        super.onPause()

    }
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O_MR1) {
            window.decorView.importantForAutofill = View.IMPORTANT_FOR_AUTOFILL_NO_EXCLUDE_DESCENDANTS
        }
        // 设置顶部中间的文字
        setCenterTitle()
        // 注册事件
        registerEvent()
        if (BuildConfig.DEBUG) {
            /*
            * StrictMode是一个开发者工具，常用于捕获在应用主线程中发生的磁盘I/O、网络访问违例等问题。
            * 这里是在检测VM策略
            * 检测结果会在日志中体现
            * */
            StrictMode.setVmPolicy(
                    StrictMode
                            .VmPolicy.Builder()
                            .detectActivityLeaks()
                            .penaltyLog()
                            .build()
            )
        }
    }
    /**
     * 数据加载时的dialog
     */
    private lateinit var mRequestDialog: LoadingDialog
    /**
     * dialog默认显示的文字
     */
    // StringRes:告诉编译器改方法必须要求返回String类型的res资源
    @StringRes
    open fun providerLoadingText(): Int = R.string.loading

    // SuppressLint:告诉编辑器不用检查
    @SuppressLint("ResourceType")
    private fun showMRequestDialog(@StringRes msg: Int = 0) {
        /*
        * 这里的::,因为后面的isInitialized不属于mRequestDialog实例的对象方法
        * 如果不加,则.语法是没有用的
        * */
        (!::mRequestDialog.isInitialized) thenNoResult {
            mRequestDialog = LoadingDialog(this).apply { setText(providerLoadingText()) }
        }
        (msg > 0) thenNoResult { mRequestDialog.showWithText(msg) } elseNoResult { mRequestDialog.show() }
    }
    private fun dismissMRequestDialog() {
        (::mRequestDialog.isInitialized) thenNoResult { mRequestDialog.dismiss() }
    }

    override fun onConfigurationChanged(newConfig: Configuration) {
        if (newConfig.fontScale != mDefaultFontScale) {
            resources
        }
        super.onConfigurationChanged(newConfig)
    }
    // 设置顶部中间的文字
    open fun setCenterTitle() {

    }
    private fun registerEvent() {

    }
    override fun onDestroy() {
        // 在调用父类的destroy方法前,先调用当前AC中的VM和dialog的销毁方法
        mViewModel?.onDestroy()
        dismissMRequestDialog()
        super.onDestroy()
    }
    protected fun showMToast(msg: String, duration: Int = Toast.LENGTH_SHORT) {
        if (mToast == null) {
            // also函数：返回值 = 传入的对象的本身
            mToast = Toast.makeText(applicationContext, msg, duration).also {
                it.setGravity(TOAST_GRAVITY, 0, 0)
                it.show()
            }
        } else {
            mToast!!.setText(msg)
            if (!mToast!!.view.isShown)
                mToast?.show()
        }
    }

    protected fun showMToast(@StringRes msg: Int, duration: Int = Toast.LENGTH_SHORT) {
        if (mToast == null) {
            /*
            * also 和 apply的差别主要存在于，lambda表达式内context表示方式，
            * also是通过传入 的参数（it）来表示的，而apply是通过this来表示的。
            * */
            mToast = Toast.makeText(this, msg, duration).also {
                it.setGravity(TOAST_GRAVITY, 0, 0)
                it.show()
            }
        } else {
            mToast!!.setText(msg)
            if (!mToast!!.view.isShown)
                mToast?.show()
        }
    }

    fun goPermissionSetting() {
        startActivity(Intent().apply {
            action = Settings.ACTION_APPLICATION_DETAILS_SETTINGS
            data = Uri.fromParts("package", packageName, null)
        })
    }

    @CallSuper
    override fun subscribeUi() {
        // 最常用的场景就是使用let函数处理需要针对一个可null的对象统一做判空处理
        mViewModel?.let {
            it.toastMsg.observe(this, Observer<String> { msgId ->
                showMToast(msgId)
            })
            it.toastResId.observe(this, Observer { resId ->
                showMToast(resId)
            })
            it.showLoadingDialog.observe(this, Observer { isShow ->
                if (isShow!!) showMRequestDialog() else dismissMRequestDialog()
            })
            it.showLoadingDialogWithMsg.observe(this, Observer { isShowInfo ->
                isShowInfo?.let { it ->
                    if (it.first)
                        showMRequestDialog(it.second)
                    else
                        dismissMRequestDialog()
                }
            })

            it.loginOverTime.observe(this, Observer { goLogin ->

            })
            it.getToastInfo().observe(this, Observer { info ->
                val msg = when (info.second) {
                    is Int -> getString(info.second as Int)
                    is String -> info.second as String
                    else -> "error"
                }
                showMToast(msg)
            })
        }
    }

    /**
     * 重启应用
     */
    fun restartApplication() {
        startActivity(App.instance.applicationContext.packageManager.getLaunchIntentForPackage(packageName)?.apply {
            flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TASK
        })
    }

}
```











