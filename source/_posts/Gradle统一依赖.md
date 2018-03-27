---
title: Gradle统一依赖
date: 2018-03-11 16:21:32
tags: Gradle
---

## 在工程根目录下创建config.gradle文件

```gradle
//定义support包的版本号
def supportVersion = "27.1.0"

ext{
    //定义各版本号
    configs = [
            compileSdkVersion: 27,
            buildToolsVersion: "25.0.2",
            miniSdkVersion: 17,
            targetSdkVersion: 27
    ]

    libraries = [
            appcompat           : "com.android.support:appcompat-v7:${supportVersion}",
            cardview            : "com.android.support:cardview-v7:${supportVersion}",
            recyclerview        : "com.android.support:recyclerview-v7:${supportVersion}",
            preference          : "com.android.support:preference-v7:${supportVersion}",
            supportv4           : "com.android.support:support-v4:${supportVersion}",
            design              : "com.android.support:design:${supportVersion}",
            palette             : "com.android.support:palette-v7:${supportVersion}",
            constraintLayout    : "com.android.support.constraint:constraint-layout:1.0.2",

            okhttp3             : "com.squareup.okhttp3:okhttp:3.10.0",
            okhttp3Logging      : "com.squareup.okhttp3:logging-interceptor:3.10.0",
            gson                : "com.google.code.gson:gson:2.8.2",
            retrofit            : "com.squareup.retrofit2:retrofit:2.4.0",
            converterGson       : "com.squareup.retrofit2:converter-gson:2.4.0",
            adapterRxjava       : "com.squareup.retrofit2:adapter-rxjava2:2.4.0",

            glide               : "com.github.bumptech.glide:glide:3.7.0",
            rxjava              : "io.reactivex.rxjava2:rxjava:2.1.9",
            rxandroid           : "io.reactivex.rxjava2:rxandroid:2.0.2",
            fastjson            : "com.alibaba:fastjson:1.2.17",
            stetho              : "com.facebook.stetho:stetho:1.3.1",
            stethoOkhttp3       : "com.facebook.stetho:stetho-okhttp3:1.3.1",

            rxlifecycleComponents:"com.trello.rxlifecycle2:rxlifecycle-components:2.2.1",

            butterknife         : "com.jakewharton:butterknife:8.6.0",
            butterknifeCompiler : "com.jakewharton:butterknife-compiler:8.6.0",

            slidr               : "com.r0adkll:slidableactivity:2.0.6",
            eventbus            : "org.greenrobot:eventbus:3.1.1",
            eventbusCompiler    : "org.greenrobot:eventbus-annotation-processor:3.1.1",

            junit               : "junit:junit:4.12"
    ]
}
```
<!--more-->
## 在根目录下的`build.gradle`文件顶部添加:

```gradle
apply from: "config.gradle"
```

## 在各模块的`build.gradle`添加

```gradle

android {
    compileSdkVersion configs.compileSdkVersion
    buildToolsVersion configs.buildToolsVersion

    defaultConfig {
		//省略其它配置
        minSdkVersion configs.miniSdkVersion
        targetSdkVersion configs.targetSdkVersion
    }
}
```

依赖添加:

```gradle
dependencies {
    //support相关包
    compile libraries.appcompat
    compile libraries.design
    compile libraries.cardview
    compile libraries.recyclerview
    compile libraries.preference
    compile libraries.constraintLayout
    compile libraries.palette

    //rx
    compile libraries.rxjava
    compile libraries.rxandroid

    //okhttp
    compile libraries.okhttp3

    //retrofit
    compile libraries.retrofit
    compile libraries.retrofitRxjava

    //gson
    compile libraries.gson
    compile libraries.converterGson
    //glide
    compile libraries.glide

    compile libraries.fastjson

    //stetho调试时使用
    compile libraries.stetho
    compile libraries.stethoOkhttp3


    //butterknife引用
    compile libraries.butterknife
    annotationProcessor libraries.butterknifeCompiler
    
    //eventBus
    implementation libraries.eventbus
    annotationProcessor           libraries.eventbusCompiler

}
```