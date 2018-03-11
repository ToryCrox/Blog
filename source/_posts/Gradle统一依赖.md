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
    android = [
            compileSdkVersion: 27,
            buildToolsVersion: "25.0.2",
            miniSdkVersion: 17,
            targetSdkVersion: 27
    ]

    dependences = [
            appcompat           : "com.android.support:appcompat-v7:$supportVersion",
            cardview            : "com.android.support:cardview-v7:$supportVersion",
            recyclerview        : "com.android.support:recyclerview-v7:$supportVersion",
            preference          : "com.android.support:preference-v7:$supportVersion",
            supportv4           : "com.android.support:support-v4:$supportVersion",
            design              : "com.android.support:design:$supportVersion",
            palette             : "com.android.support:palette-v7:$supportVersion",
            constraintLayout    : "com.android.support.constraint:constraint-layout:1.0.2",

            okhttp3             : "com.squareup.okhttp3:okhttp:3.2.0",
            retrofit            : "com.squareup.retrofit2:retrofit:2.1.0",
            retrofitRxjava      : "com.squareup.retrofit2:adapter-rxjava:2.1.0",
            gson                : "com.google.code.gson:gson:2.8.1",
            converterGson       : "com.squareup.retrofit2:converter-gson:2.1.0",

            glide               : "com.github.bumptech.glide:glide:3.7.0",
            rxjava              : "io.reactivex:rxjava:1.2.0",
            rxandroid           : "io.reactivex:rxandroid:1.2.1",
            fastjson            : "com.alibaba:fastjson:1.2.17",
            stetho              : "com.facebook.stetho:stetho:1.3.1",
            stethoOkhttp3       : "com.facebook.stetho:stetho-okhttp3:1.3.1",
            butterknife         : "com.jakewharton:butterknife:8.6.0",
            butterknifeCompiler : "com.jakewharton:butterknife-compiler:8.6.0",

            junit               : "junit:junit:4.12"
    ]
}
```

## 在根目录下的`build.gradle`文件顶部添加:

```gradle
apply from: "config.gradle"
```

## 在各模块的`build.gradle`添加

```gradle
def configs = rootProject.ext.android
def librarys = rootProject.ext.dependences

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
    compile librarys.appcompat
    compile librarys.design
    compile librarys.cardview
    compile librarys.recyclerview
    compile librarys.preference
    compile librarys.constraintLayout
    compile librarys.palette

    //rx
    compile librarys.rxjava
    compile librarys.rxandroid

    //okhttp
    compile librarys.okhttp3

    //retrofit
    compile librarys.retrofit
    compile librarys.retrofitRxjava

    //gson
    compile librarys.gson
    compile librarys.converterGson
    //glide
    compile librarys.glide

    compile librarys.fastjson

    //stetho调试时使用
    compile librarys.stetho
    compile librarys.stethoOkhttp3


    //butterknife引用
    compile librarys.butterknife
    annotationProcessor librarys.butterknifeCompiler

}
```