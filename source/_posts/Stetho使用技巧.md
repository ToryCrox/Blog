---
title: Stetho使用技巧
date: 2018-03-04 23:12:08
tags: android
---

> Stetho是Facebook开源的一个Android应用的调试工具
> 使用很简单而且也有很多资源介绍

## 参考资源
[Android开发调试神器Stetho介绍-只有你想不到没有你看不到](https://www.jianshu.com/p/a7fdcb2641e8)

## 使用步骤

### 1. 项目添加依赖。

```gradle
compile  "com.facebook.stetho:stetho:1.3.1"
compile  "com.facebook.stetho:stetho-okhttp3:1.3.1"
```
如果使用了okhttp，则需要添加第二个依赖

### 2. 初始化 Stetho

```java
public class MyApplication extends Application { 
      public void onCreate() { 
        super.onCreate(); 
        Stetho.initializeWithDefaults(this); 
        }
}
```

### 3. 修改网络请求（可选）

```java
new OkHttpClient.Builder() .
addNetworkInterceptor(new StethoInterceptor()) .build()
```

### 4. 运行你的项目
  
```txt
在chrome中访问 chrome://inspect
找到你的项目 点击 inspect
```
如果发现一直在转圈，需要先翻墙

## 使用技巧

### 1. 动态加载开启Stetho
在一般开发中我们通常是在debug版本下想入Stetho，而在release版本上去除，网资料大都是使用debugCompile的方式

```gradle
debugCompile 'com.facebook.stetho:stetho:1.3.1' 
```
这样存在一个弊端是需要再在debug目录再添加一个Application

受同事启发，使用`DexClassLoader`动态加载的方式可以再方便的引入Stetho，并且不会影响apk的大小

- 首先新建一个app工程，引入Stetho依赖后，里面只需要添加一个类

```java
package com.aleaf.debug;

import android.content.Context;
import android.util.Log;

import com.facebook.stetho.Stetho;

public class StethoReflection {
    private static final String TAG = "StethoReflection";

    public void initStetho(Context context){
        Log.d(TAG,"initStetho context="+context);
        //chrome://inspect
        Stetho.initializeWithDefaults(context);
    }

}
```
编译一个debug版的apk出来，并安装到手机上

- 在需要使用Stetho的app的Application里面使用`DexClassLoader`引入

```
public class MApplication extends Application {
    public void onCreate() {
        //chrome://inspect
        if(BuildConfig.DEBUG){//debug版才开启
            ReflectDebugUtil.reflectInitStetho(this);
        }
    }
}
```
ReflectDebugUtil.java
```
public class ReflectDebugUtil {
    public static final String DEBUG_PACKGE = "com.aleaf.debug";
    public static final String DEBUG_STETHO_CLASS_NAME = "com.aleaf.debug.StethoReflection";

    private void reflectInitStetho(Context context){
        try {
            Context stethoContext = context.createPackageContext(
                    DEBUG_PACKGE, Context.CONTEXT_INCLUDE_CODE
                            | Context.CONTEXT_IGNORE_SECURITY);

            String outDir = context.getFilesDir() + File.separator + "debug";
            if(!new File(outDir).exists()){
                new File(outDir).mkdirs();
            }
            DexClassLoader dexLoader = new DexClassLoader(
                    stethoContext.getApplicationInfo().sourceDir,//dst apk surce path
                    outDir,//
                    context.getApplicationInfo().nativeLibraryDir,//.so
                    context.getClassLoader());
            Class<?> clazz = dexLoader.loadClass(DEBUG_STETHO_CLASS_NAME);
            Object  ste = clazz.newInstance();
            Method m = clazz.getMethod("initStetho",Context.class);
            m.invoke(ste,context);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

这样做的好处时应用apk完全不需要引入Stetho的sdk，打开关闭调试也很方便，只需要安装卸载debug的apk即可