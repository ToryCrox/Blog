---
title: Fragment笔记
date: 2018-04-15 14:23:18
tags: Android
---

> 两年前的笔记，再拿出来回顾才发现许久不用有些内容都记混淆了

### 基本概念
Fragment是Android3.0(sdk 17)后引入的一个新的API,出现的初衷是为了适应大屏幕的平板电脑,我们可以把他看成一个小型的Activity，又称Activity片段.Fragment并不能单独使用，必须嵌套在Activity中使用。


V4包下提供的也有一个Fragment,使用的时候要注意:
> 1. 如果使用v4包下的`Fragment`,那么所在的那个`Activity`就要继承FragmentActivity `AppCompatActivity`下继承了`FragmentActivity`，可以直接使用
> 2. 需要把`getFragmentManager( )`改成`getSupportFragmentManager()`


几个子类

> 对话框:DialogFragment 
> 列表:ListFragment 
> 选项设置:PreferenceFragment
> WebView界面:WebViewFragment

<!--more-->
生命周期:
![@Fragment的生命周期](/images/Fragment的生命周期.png)

Fragment的生命周期是跟随Activity的

### Fragment基本创建与使用
静态加载
![@Fragement静态加载](/images/Fragement静态加载.png)

注意点:
name属性是全限定类名哦，就是要包含Fragment的包名
layout属性会覆盖fragment布局文件中的layout属性
必须为布局设置id，系统在Activity恢复时用它来存储状态, 否则配置发生变化或内存重启时会报错(如旋转屏幕)

```xml
<fragment
     android:id="@+id/fg_fragment"
     android:name="com.tory.demo.sample.test.LifeCycleFragment"
     android:layout_width="match_parent"
     android:layout_height="100dp"/>
```

动态加载
![@Fragment的动态加载](/images/Fragment的动态加载.png)

注意点:
Fragment的管理是通过事务来管理的，所以可以进行多次add,remove等
注意事务需要commit

```java
LifeCycleFragment fragment = new LifeCycleFragment();
getSupportFragmentManager().beginTransaction()
     .add(R.id.frame_container,fragment,"LifeCycleFragment")
     .addToBackStack("LifeCycleFragment")
     .commit();
```

Fragment的事务操作
![@Fragment的事件操作](/images/Fragment的事件操作.png)

> fragment的每一次改变的提交给activity，都称为一个事务
> `add()`, `remove()`,`replace()` ,`show()`, `hide()`等一系列方法，只有commit了以后才会生效
> Fragment的事务操作是**异步**的，commit了以后相当于进入了一个队列，然后等待合适的时刻执行
>  addToBackStack() 可以把fragment的本次操作放入Activity的返回栈中，通过点击back键可以回到以前状态，可以通过此方法移除或隐藏Fragment，个人**不建议使用**。
>  从FragmentManger.beginTransaction真正返回的确是一个`BackStackRecord`类， 其实现了FragmentTransaction所定义的接口。



### 使用技巧与建议:
1. 对`Fragment`传递数据，建议使用`setArguments(Bundle args)`，而后在`onCreate`中使用`getArguments()`取出，在 “内存重启”前，系统会帮你保存数据，不会造成数据的丢失。和Activity的Intent原理一致。

2. 使用`newInstance(参数)` 创建`Fragment`对象，优点是调用者只需要关系传递的哪些数据，而无需关心传递数据的Key是什么。
可结合第一点使用

3.  Fragment之间的通信，在同一个`Activity`下的两个`Fragment`的通信：`FragmentA`调用其宿主`Activity`的方法，宿主`Activity`再根据`FragmentA`的调用参数去调用`FragmentB`的方法并传递参数给B

在AS里面右键，新建Fragment, AS提供了一个Fragment的模板写法
> 1. 使用newInstance(参数) 创建Fragment对象
> 2. 使用setArguments(Bundle args)，而后在onCreate中使用getArguments()取出 在 “内存重启”前，系统会帮你保存数据，不会造成数据的丢失。和Activity的Intent原理一致。
> 3. 在Fragment中定义一个内部回调接口,在Activity中实现

4. 使用`DialogFragment`代码`Dialog`，便于管理`Dialog`的生命周期和保存状态。
`DialogFragment`在`show`的时候容易出现以下错误`java.lang.IllegalStateException: Fragment already added`，这是重复添加引起的，`fragment.isAdded()`
```java
if(!fragment.isAdded()){
    fragment.show(getFragmentManager(),"fragment");
}
```

5. add(), show(), hide(), replace(),remove()...
`show()`，`hide()`最终是让`Fragment`的`View setVisibility()`，不会调用生命周期;
如:  `hide()`
 - 设置`mHidden` 为`true`
 - `fragment.mView.setVisibility(View.GONE`); 隐藏`fragment`的`view`
 - `fragment.onHiddenChanged(true)`; 调用onHiddenChange
可以发现也可能通过`fragment.getView().setVisibility();`来控制fragment的显示，只是此时没有改变`mHidden`的状态
`replace()`的话会销毁前一个视图，即调用`onDestoryView`、`onCreateView`等一系列生命周期

6. onHiddenChanged的回调时机
当使用`add()+show()`，`hide()`跳转新的Fragment时，旧的Fragment回调`onHiddenChanged()`，不会回调`onStop()`等生命周期方法，而新的Fragment在创建时是不会回调`onHiddenChanged()`，这点要切记。
有类似需求可以自己实现

7. ViewPager中的Fragment生命周期
ViewPager会预先加载预先加载包换当前可见的Fragment和其两侧的三个Fragment
具体参考这篇文章：http://www.jianshu.com/p/660ec2666faa
![Fragment在ViewPager中](/images/Fragment在ViewPager中.png)

8. setUserVisibleHint
使用`ViewPager`时，切换回上一个Fragment页面时（已经初始化完毕），不会回调任何生命周期方法以及`onHiddenChanged()`，只有`setUserVisibleHint(boolean isVisibleToUser)`会被回调，所以如果你想进行一些懒加载，可以重载该方法实现(需要配合fragment生命周期使用)。
具体实现可以参考:http://www.jianshu.com/p/c5d29a0c3f4c
`setUserVisibleHint`的回调实际上是由`ViewPager`触发的，与fragment的show，hide状态没有太大关系

9. `commit()`, `commitNow()` 和`executePendingTransactions()`;

commit并不是立即执行的, 它会被发送到主线程的任务队列当中去, 当主线程准备好执行它的时候执行; 所以在使用commit的后要注意通过isAdded()或isShow()等方法是并不可靠的，因为很有可能因为Fragment还没到添加上去而引起一些错误。

想要同步执行可以使用`commitNow`或者`executePendingTransactions`; commitNow是立刻执行**当前事务**, 而`executePendingTransactions`立即执行在队列中的所有事务！


### 使用中出现的问题:

#### Activity重新创建
通过事务动态添加Fragment的时候我们可能很自然的这样写

```java
@Override
onCreate(Bundle savedInstance) {
     getSupportFragmentManager().beginTransaction()
         .add(R.id.frame_container,fragment)
         .commit();
}
```
表面上看起来没有什么情况，看起来好像也没有什么不对，但在实际项目中可以看到很多这样的写法

```java
@Override
onCreate(Bundle savedInstance) {
   if (savedIntance == null) {
      // create fragment and add it to Activity.
   }
}
```
因为系统配置信息发生变化或内存重启的时候，系统会把Activity杀掉，然后再重新创建它。此时系统会尽可能的恢复以前的状态，所以以前添加的Fragment也会重新添加进去，也就没有必要重新创建一个Fragment;
这种情况下会引发几个问题
1. Fragment对象的获取,这个可以在需要和时候通过`findFragmentByTag`或`onAttachFragment`获取，也可以通过自己写回调接口来实现。
2. Fragment的状态会丢失，丢失的原因是onCreate重新创建时，会调用Framgment的默认无参构造来创建Fragment对象。所以这也是为什么文档中说Fragment一定要有一个默认的构造函数，而且最好不要有带参数的构造函数，传参数要用setArguments。
所以需要在`onSaveInstanceState()`时，把一些变量保存，然后在onCreate时恢复
3. hide、show问题，主要也是Fragment的状态会丢失问题引起的，因为恢复的所有的Fragment的都是以show状态恢复的，需要自己手动恢复其状态。这里是一个示例:http://www.jianshu.com/p/d9143a92ad94

```java
public class TestActivity extends Activity{
    private static final String STATE_SAVE_IS_HIDDEN = "STATE_SAVE_IS_HIDDEN";

    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
    ...
    if (savedInstanceState != null) {
        boolean isSupportHidden = savedInstanceState.getBoolean(STATE_SAVE_IS_HIDDEN);

        FragmentTransaction ft = getFragmentManager().beginTransaction();
        if (isSupportHidden) {
            ft.hide(this);
        } else {
            ft.show(this);
        }
        ft.commit();
    }

    @Override
    public void onSaveInstanceState(Bundle outState) {
        ...
        outState.putBoolean(STATE_SAVE_IS_HIDDEN, isHidden());
    }
}
```

#### state loss异常
`FragmentTransaction`是异步的，commit()仅是相当于把操作加入到`FragmentManager`的队列，然后`FragmentManager`会在某一个时刻来执行，并不是立即执行。
所以在实际项目中可能会偶尔出现这种报错:`Can not perform this action after onSaveInstanceState`。从这段报错信息可以猜测出我们`commit()`是`onSaveInstanceState()`之后了。
出现这种情况有两种解决方案:
1. 尽量要在`onSaveInstanceState`之前进行`Fragment`进行操作, 可以添加标志符解决
参考Dialer
2. 使用`commitAllowStateLoss`，它可以保证在Activity onStop以后仍然顺利执行commit操作，但是会造成状态丢失。

> 报错的原因见`BackStackRecord.commitInternal -> FragmentManager.enqueueAction` ，即在`commit`的时候，`FragmentManager`通过`checkStateLoss`进行了一次状态检测
> 在使用`DilaogFragment`的`show`和`dissmiss`方法时也要注意发生该异常的情况，建议使用`dismissAllowingStateLoss`方法

commitAllowingStateLoss会造成状态丢失，具体情况如下:
1. 当前您的 `Activity` 在显示 `FragmentA`
2. 您的 Activity 被切换到后台了（(`onStop()` 和 `onSaveInstanceState()` 函数被调用了）
3. 这个时候您的 `Activity` 的逻辑发生了一些变化，您使用 `FragmentB` 替换了 `FragmentA` 并调用了 `commitAllowingStateLoss()` 函数来提交这个变化。

这个时候，当用户再次切回您的 Activity 的时候可能出现如下两种状态：
1. 如果系统内存不足并且杀死了您的应用，当用户重新打开您的 应用的时候，系统将会恢复您的应用到上面第二步的状态，而 `FragmentB` 是不会显示的。
2. 如果系统没有杀死您的应用，用户则可以看到 `FragmentB`。当 Activity 再次回到后台的时候（`onStop`）， `FragmentB` 的状态才会被保存起来。


### 文章

[我为什么主张反对使用Android Fragment](https://asce1885.gitbooks.io/android-rd-senior-advanced/content/wo_wei_shi_yao_zhu_zhang_fan_dui_shi_yong_android_fragment.html)