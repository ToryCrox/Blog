---
title: Android单元测试总结
date: 2019-08-03 15:11:20
tags: Android 单元测试
---

> 当前做Launcher单元测试的总结的一些要点，回顾一下


## 简述
参考资料:
> [Android自动化测试--学习浅谈](https://www.jianshu.com/p/cb06c4be07fa)
> [Android单元测试实践](http://smallsoho.com/android/2017/04/19/Android%E5%8D%95%E5%85%83%E6%B5%8B%E8%AF%95%E5%AE%9E%E8%B7%B5/)
> [Android单元测试 - 如何开始？](https://www.jianshu.com/p/bc99678b1d6e)
> [Android单元测试系列](https://blog.csdn.net/qq_17766199/article/category/7226691): java的单元测试比较详细，有MVP和一些流行框架的测试方法

Android有关的单元测试大体分为两类: 
1.  本地单元测试
> - 运行在jvm上的测试框架，不需要Android环境
> - 位于`src/test/java`
> - gradle引入时使用testCompile
> - 有`Junit4`、`Mockito`、`Powermockito`，`Robolectric`

2.  Android Instrumentation测试
> - 运行在Android环境上的测试框架，依赖真机或都模拟器环境
> - 代码位于`src/androidTest/java`
> - gradle引入时使用`androidTest`
> 有`AndroidJUnitRunner`，`Espresso`, `UI Automator`

各种框架简介:
1. `Junit4`: 基础的Java单元测试
2. `Mockito`: 模拟测试的类，是一个工具类的集合，配置其它框架使用
3. `Robolectric`: JVM环境中模拟`Android`的环境，可以在不连接Android设备的情况下进行测试。听起来很美好，但使用起来不是很方便，还一堆坑，介绍文章: https://www.jianshu.com/p/d0bc9ebaaea1
4. `Espresso`: UI测试，适合白盒测试
5. `UI Automator`: UI测试，适合黑盒测试，测试组的自动化脚本应该就是基于这个写的


## 框架结构

使用`Android Instrumentation`测试，测试代码写在`src/androidTest/java`目录下

使用到的框架有`junit4`, `mockito`, `Instrumentation`, `uiautomator`, `espresso`
`gradle`下的依赖方式:

```gradle
defaultConfig {
    testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
}

//dependences
androidTestCompile libraries.mockitoAndroid
androidTestCompile libraries.testRunner
androidTestCompile libraries.uiautomator
androidTestCompile libraries.annotations
androidTestCompile libraries.espresso
```
<!--more-->
### 各框架的基本使用

#### Junit4
主要提供各种注解和断言
最重要的注解`@Test`表明了测试的方式，根据根据方法上的注解，其执行顺序为`@BeforeClass –> @Before –> @Test –> @After –> @AfterClass`

断言一般常用assertNotNull, assertEquals等
详细Api和使用见此博客:https://blog.csdn.net/qq_17766199/article/details/78243176

#### mockito
用来模拟对象，脱离对`Android Api`依赖
详细使用方法见: https://blog.csdn.net/qq_17766199/article/details/78450007

对于一些依赖的流程过多的测试可以用该方法隔离依赖: 例如LauncherAppState的初始化是在Launcher启动时，在样在测试依赖`LauncherAppState`的代码时就可以用mock来模拟对象，如:

```java
Mock LauncherAppState mMockApp;
Mock IconCache mMockIconCache;

public void setUp() throws Exception {
      super.setUp();
      MockitoAnnotations.initMocks(this);

      when(mMockApp.getIconCache()).thenReturn(mMockIconCache);
      when(mMockApp.getContext()).thenReturn(mTargetContext);
}
```


#### uiautomator
主要进行UI测试，能够通过Id, text和description来查找元素，不限进程，使用方便，确点是只能获取到当前显示的最顶层Window上的view，对应着sdk下的`uiautomatorviewer`工具。
> 这个在自动化测试时也是一个重要的工具，可以针对Release版本进行测试，只是使用的jar包和shell脚本的方式做的。

使用时例如:
UI Automator的Api主要包括三个方面 : `UiDevice`, `UiObject/UiObject2`, `UiSelector/BySelector`, `UiObject`和`UiObject2`功能一样，只是查找用的`Selector`不一样，一般都用比较方便的`BySelector`和`UiObject2`，例如: 

```java
UiDevice mDevice = UiDevice.getInstance(InstrumentationRegistry.getInstrumentation());
UiObject2 button =  mDevice.findObject(By.text("text"))
button.click()
```

#### Espresso
`uiautomator`虽然使用很方便，但是还是有很多局限，无法获取到一些View的属性，也只能通过Assert来判断。这时就可以结合`Espresso`使用
基本使用方法: https://www.jianshu.com/p/37f1897df3fd

下面是官方网站给出的一个例子，

```java
onView(withId(R.id.my_view))            
        .perform(click()) 
        .check(matches(isDisplayed())); 
```
可以自定义`Matcher`来检查更为复杂的属性

```java
final InvariantDeviceProfile idp = SettingDeviceProfile.getProfileById(gridSizeIds[index]);

onView(withId(R.id.workspace)).check(matches(new TypeSafeMatcher<View>() {

    Point reallySize = new Point();
    Point targetSize = new Point();

    @Override
    protected boolean matchesSafely(View item) {
        if(!(item instanceof Workspace)){
            return false;
        }
        Workspace workspace = (Workspace) item;
        CellLayout page = workspace.getCurrentDropLayout();
        reallySize.set(page.getCountX(), page.getCountY());
        targetSize.set(idp.numColumns, idp.numRows);
        return page.getCountX() == idp.numColumns && page.getCountY() == idp.numRows;
    }

    @Override
    public void describeTo(Description description) {
        description.appendText(" gridSizeId is ")
                .appendText(String.valueOf(idp.gridSizeId))
                .appendText(", target is")
                .appendText(targetSize.toString())
                .appendText(", really is ")
                .appendText(reallySize.toString());
    }
}));
```



### 开发要点:

1. 测试的包括两方面: 纯功能测试和UI测试。

> 纯功能测试不要涉及Ui，不能启动`Launcher`，也不能使用`uiautomator`和`espresso`，代码统一放在`com.transsion.launcher.fun`包下
> Ui测试，才`uiautomator`, `espresso`为辅，写到`com.transsion.launcher.ui`下

2. 所有测试均继承`BaseContextAndroidTest`，使用这个基类提供的`mTargetContext`和`mTargetPackage`，要注意需要包名时不要使用`BuildConfig.APPLICATION_ID`,要使用基类提供的`mTargetPackage`

3. 所有需要测试的类前面要加上`@RunWith(AndroidJUnit4.class)`的注解，测试方法加上`@Test`的注意，且必需为public的

4. 有异常可在方法上加上`throws Exception`上抛出，尽量不要使用`try catch`

5. 多使用assert进行断言

7. 需要Launcher对象，可以定义新建`LauncherActivityRule`后获取，例如

```java
 @Rule
 public LauncherActivityRule mActivityMonitor = new LauncherActivityRule();
 @Test
 public void findFreezerIcon(){
	mActivityMonitor.getActivity()
}
```
8. 需要等待时使用`waitForIdle`或者`Wait.sleep();`
9. `Wait.atMost()`可以等待某一条件成立
10. `uiautomator`只能检查有限的属性，可检查的属性可从`sdk/tools/uiautomatorviewer.bat`来看
11.  异步Api可通过CountDownLatch来等待

> `CountDownLatch`是一个同步工具类，它允许一个或多个线程一直等待，直到其他线程的操作执行完后 再 执行。
> `CountDownLatch`提供`countDown()` 方法递减锁存器的计数，如果计数到达零，则释放所有等待的线程，例如:

```java
@Test
public void testLoadWallpaper() throws InterruptedException {
    final CountDownLatch latch = new CountDownLatch(1);
    final ArrayList<LocalWallpaperInfo> wallpaperInfos = new ArrayList<>();
    GuideLoadPresenter.GuideUi ui = new GuideLoadPresenter.GuideUi() {
        @Override
        public void loadComplete(ArrayList<LocalWallpaperInfo> infos, Drawable previewIcon) {
            wallpaperInfos.addAll(infos);
            latch.countDown();
        }
    };
    GuideLoadPresenter presenter = new GuideLoadPresenter(ui);
    presenter.startLoad(mTargetContext, false);
    latch.await(20, TimeUnit.SECONDS);
    Assert.assertNotNull(ui);//GuideLoadPresenter内的ui是弱引用
    Assert.assertThat(wallpaperInfos.size(), anyOf(is(1), is(2)));
}
```


## 测试点

纯功能测试:
由于`Launcher`各功能偶合性太强，可以写的测试用例有限，很多功能需要解偶才能写，这里建议多写工具方法的测试，如检测壁纸的util方法：

1. `AppLaunchCountRecorder` 测试记录常用应用功能是否正常
2. `GuideLoadPresenter` 测试是否能够正确获取壁纸信息
3. `LoadCursor` 能否正常解析cursor中的数据
4. `WallpaperUtils` 测试能否正常获取壁纸及检查壁纸深浅


Ui测试，才`uiautomator`, `espresso`为辅

1. `Settings`界面
	- 设置界面桌面提示
	- 修改图标大小
	- 选择桌面网格
	- 图标锁定点击开关，相关设置是否禁用
	- A-Z界面移动方向切换，是否生效
	- 文件颜色切换，是否生效
	- 智能整理
	- 关于XOS桌面几项功能跳转

2. AllApp界面
	- 切换横向，竖向
	- 竖向界面竖向滑动
	- 竖向界面检查常用应用一栏
	- 竖向界面是否有搜索框，点击搜索框能否弹出输入法
	- 图标长按能否弹出菜单，发送到桌面

3. Launcher桌面
    - 页面批量器， 检查是否与桌面页数匹配
    - 拖动图标创建文件夹
    - 拖动图标到文件夹
    - 文件夹内左右滑动
    - 文件夹重命名
    - 点击打开文件夹添加界面
    - 添加界面添加图标
    - 长按空白处进入编辑模式
    - 是否有冷藏室
    - 冷藏室，冷藏解冻
    - 安装，卸载应用(内置一个test的apk测试)


### 附录
Activity基础测试类:
```java
public class BaseUiAndroidTest extends BaseContextAndroidTest {

    public static final long DEFAULT_UI_TIMEOUT = 1000L;
    public static final long DEFAULT_WAITTING_TIMEOUT = 5000L;
    
	protected Context mTargetContext;
    protected String mTargetPackage;
    UiDevice mDevice;
    
    /**
    * 初始化Context， Package等
    */
    @Before
    public void setUp() throws Exception {
        mTargetContext = InstrumentationRegistry.getTargetContext();
        mTargetPackage = mTargetContext.getPackageName();
        mDevice = UiDevice.getInstance(InstrumentationRegistry.getInstrumentation());
    }


    protected String getText(@StringRes int resId){
        return mTargetContext.getResources().getString(resId);
    }

	//启动Activity，且等打开完毕
    public void startActivitySync(@NonNull Intent intent){
        mTargetContext.startActivity(intent);
        waitForIdle();
    }

    public void pressBack(){
        mDevice.pressBack();
    }

    public void waitForIdle(){
        mDevice.waitForIdle();
    }

	//通过id查找
    protected UiObject2 findViewById(@IdRes int id) {
        return mDevice.wait(Until.findObject(getSelectorForId(id)), DEFAULT_UI_TIMEOUT);
    }

    protected UiObject2 findViewByText(@StringRes int resId){
        return findViewBySelector(By.text(getText(resId)));
    }

    protected BySelector getSelectorForId(int id) {
        String name = mTargetContext.getResources().getResourceEntryName(id);
        return By.res(mTargetPackage, name);
    }

    protected UiObject2 findViewBySelector(BySelector condition){
        return mDevice.wait(Until.findObject(condition), DEFAULT_UI_TIMEOUT);
    }

	//滚动查找元素
    protected UiObject2 scrollAndFind(UiObject2 container, BySelector condition) {
        do {
            UiObject2 widget = container.findObject(condition);
            if (widget != null) {
                return widget;
            }
        } while (container.scroll(Direction.DOWN, 1f));
        return container.findObject(condition);
    }

    protected UiObject2 scrollPageAndFind(UiObject2 container, BySelector condition) {
        do {
            UiObject2 widget = container.findObject(condition);
            if (widget != null) {
                return widget;
            }
        } while (container.scroll(Direction.RIGHT, 1f));
        return container.findObject(condition);
    }
}

```
