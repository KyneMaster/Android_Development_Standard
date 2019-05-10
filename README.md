# -Android-
阿里巴巴在阿里云推出Android开发手册，并提供认证考试

####  命名规范
*layout、drawable、anim、color、dimen、style、string*
#### 基本组件
1. activity间通讯数据量大使用**EventBus**
2. 持久化存储应该在activity#onPause()/onStrop()中实行
3. Intent隐式跳转前通过**resolveActivity**检查
4. Activity#onSaveInstanceState()用来保存Activity被意外销毁时保存UI状态，只能保存临时性数据，比如UI控件的属性等。
5. Activity 间通过隐式 Intent 的跳转，在发出 Intent 之前必须通过resolveActivity检查。
4. Service#onStartCommand()/onBind()不可执行耗时操作，改用**IntentService**
5. 避免在 BroadcastReceiver#onReceive()中执行耗时操作，如果有耗时工作， 应该创建 IntentService
6.  如果广播仅限于应用内，则可以使用 LocalBroadcastManager#sendBroadcast()实现，避免敏感信息外泄和 Intent 拦截的风险。
7.  对于只用于应用内的广播，优先使用LocalBroadcastManager来进行注册和发送，LocalBroadcastManager安全性更好，同时拥有更高的运行效率
8. 确保FragmentTransaction#commit()在 Activity#onPostResume()或者 FragmentActivity#onResumeFragments()内调用。
9. 不要在Activity#onDestroy()内执行释放资源的工作
10. 避免使用嵌套的Fragment。
    > 1) onActivityResult()方法的处理错乱，内嵌的  Fragment 可能收不到该方法的回调， 需要由宿主 Fragment 进行转发处理;
    > 2) 突变动画效果; 
    > 3) 被继承的 setRetainInstance()，导致在 Fragment 重建时多次触发不必要的逻辑。
13. 尽量使用显示Intent启动或绑定Service。隐式必须搭配使用setPackage()方法设置Intent的指定包名。
14. onPause方法中不适合做耗时较长的工作
15. Activity或者Fragment中动态注册BroadCastReceiver时，registerReceiver()和unregisterReceiver()要成对出现

#### UI与布局
1. 推荐merge、ViewStub来优化布局
2. 不能在Activity没有完全显示时显示PopupWindow和Dialog
    > 推荐 Activity#onAttachedToWindow()之后。最好是在onWindowFoucsChanged()之后

3. 尽量不用animationDrawable,帧动画，少量可以
4. 不要使用ScrollView嵌套ListView/GridView/RecyclerView
    > 推荐使用NestedScrollView

5. 使用Toast建议定义一个全局对象
6. Adapter item对每个子控件都要设置属性，由于item复用原因
7. 在需要时刻刷新某一区域的组件时，建议通过以下方式避免引发全局 layout 刷新:
    > 1) 设置固定的View大小的宽高，如倒计时组件等;
    > 2) 调用View的layout方法修改位置，如弹幕组件等;
    > 3) 通过修改Canvas位置并且调用invalidate(int l, int t, int r, int b)等方法限定刷新区域
    > 4) 通过设置一个是否允许requestLayout的变量，然后重写控件的requestLayout、onSizeChanged方法， 判断控件的大小没有改变的情况下，当进入requestLayout的时候，直接返回而不调用super的requestLayout方法
8. 不要在Android的Application对象中缓存数据

#### 进程、线程与消息通信
1. 在Application的业务初始化代码中加入进程判断，确保只在自己需要的进程中初始化。
2. 新建线程时，必须通过线程池提供，节约开支
    > *使用线程池的好处*
    > 1) 减少在创建和销毁线程上所花的时间及系统资源 的开销。
    > 2) 解决资源不足问题
    > 3) 如果不使用线程池，有可能造成系统创建大量同类线程而导致消耗完内存或者切换过度的问题
    > 4) 创建匿名线程不便与后续的资源使用分析、性能分析。
3. 线程池不允许使用Executors去创建
4. ThreadPoolExecutor设置线程存活时间（setKeepAliveTime），确保空闲时线程被释放
5. 禁止在多线程之间用SharePreferences共享数据，官方不推荐

#### 文件与数据库
1. 应用间共享文件时，不要通过放宽文件系统权限的方式，而应使用FileProvider
2. 执行SQL语句时，应使用 SQLiteDatabase#insert()、update()、delete()，不要使用 SQLiteDatabase#execSQL()，以免SQL注入风险
3. SharedPreference提交数据时，commit和apply的区别：
    > apply首先写入内存，然后异步写入磁盘,commit 直接写入磁盘
4. 多线程操作写入数据库时，需要使用事务。通过SQLiteOpenHelper

#### Bitmap、Drawable与动画
1. 加载大图片或者一次性加载多张图片，应该在异步线程中进行。图片的加载，涉及到IO操作，以及CPU密集操作，很可能引起卡顿
2. 使用完毕的图片，应该及时回收，释放宝贵的内存
    > 
    ```
        if (Build.VERSION.SDK_INT <= Build.VERSION_CODES.GINGERBREAD_MR1) {
            bitmap.recycle();
    }    
    ```
3. 页面退出，及时清理动画资源  Activity#onPause()或 Activity#onStop()
4. 在ListView、ViewPager、RecyclerView、GridView等组件使用图片时，应做好图片的缓存。推荐使用Fresco、Glide等图片库。
5. png图片使用TinyPNG或类似工具压缩处理，减少包体积
6. 应根据实际展示需求，压缩图片，而不是直接使用原图
7. 在Activity#onPause()或onStop()回调中，关闭当前activity正在执行的动画
8. 使用inBitmap重复利用内存空间，避免重复开辟新内存
9. RGB_565没有透明度。

#### 安全
1. 将 android:allowbackup 属性必须设置为 false，阻止应用数据被导出
2. 在 SDK 支持的情况下，Android 应用必须使用 V2 签名，这将对 APK 文 件的修改做更多的保护。
3. 不应在没有严格权限控制的情况下，将 android:exported 设置为 true。
4. 所以在产品的线上版本中关闭调试接口，不要输出敏感信息。
5. 确保应用发布版本的 android:debuggable 属性设置为 false。
6. Android APP 在 HTTPS 通信中，验证策略需要改成严格模式
7. 在 Android 4.2(API Level 17)及以上，对安全性要求较高的应用可在 Activity中，对 Activity 所关联的 Window 应用 WindowManager.LayoutParams.FLAG_SECURE，防止被截屏、录屏
8. 本地加密密钥不能硬解码在代码中，更不能本地缓存机制缓存
