---
title: Gradle/Groovy一览
date: 2018-03-10 16:20:03
tags: Gradle
---

## 参考资料

- [Gradle Plugin 用户使用指南](https://juejin.im/entry/592f93ada0bb9f0058a22641)
- [Android Studio 自定义Gradle Plugin][https://www.jianshu.com/p/af2b0a43133f]
- [Gradle 入门--只此一篇](https://www.jianshu.com/p/001abe1d8e95)
- [给 ANDROID 初学者的 GRADLE 知识普及](http://stormzhang.com/android/2016/07/02/gradle-for-android-beginners/)
- [深入理解Android之Gradle](http://blog.csdn.net/innost/article/details/48228651)
- [重新认识AndroidStudio&Gradle](http://blog.csdn.net/JF_1994/article/details/52781751)
- [Gradle入门系列](http://blog.jobbole.com/71999/)
- 官方文档 https://docs.gradle.org/current/dsl/
- api文档： http://www.groovy-lang.org/api.html

## 简介
`Groovy`是一种动态语言。它和`Java`一样，也运行于Java虚拟机中。几乎完全兼容Java
`Gradle`提供了构建项目的一个框架，可以单独安装使用，安装地址https://gradle.org/install/

`gradle-wrapper`顾名思义，这表示是包装过的`Gradle`，`Android`工程中一般就的就是它，所以命令行执行的时候就会变成`gradlew [task名称]`，这里的gradlew，其实提指的就是工程根目录下的`gradlew.bat`和`gradlew`文件
> 如果工程里面没有`gradle-wrapper`，可以通过`gradle wrapper`生成

以下几个常用命令（注意Win系统下`gradlew`而linux系统下需要用`./gradlew`）
* `./gradlew -v` 版本号
* `./gradlew clean` 清除9GAG/app目录下的build文件夹
* `./gradlew build` 检查依赖并编译打包
这里注意的是 `./gradlew build` 命令把 `debug`、`release` 环境的包都打出来，如果正式发布只需要打 Release 的包，该怎么办呢，下面介绍一个很有用的命令 assemble , 如

* `./gradlew assembleDebug` 编译并打Debug包
* `./gradlew assembleRelease` 编译并打Release的包
* `./gradlew :app:dependencies --configuration compile` 查看依赖关系

完成工程构建示例
```cmake
├── app #Android App目录，可改名，但需要在settings.gradle里面配置
│   ├── app.iml
│   ├── build #构建输出目录
│   ├── build.gradle #构建脚本
│   ├── libs #so相关库
│   ├── proguard-rules.pro #proguard混淆配置
│   └── src #源代码，资源等
├── build
│   └── intermediates
├── build.gradle #工程构建文件
├── gradle
│   └── wrapper
|         ├── gradle-wrapper.jar
|         └── gradle-wrapper.properties #gradle的版本配置
├── gradle.properties #gradle的配置
├── gradlew #gradle wrapper linux shell脚本
├── gradlew.bat #gradle wrapper window下的shell脚本
├── local.properties #配置Androod SDK位置文件
└── settings.gradle #工程配置
```

## Gradle脚本写法
> 都以`gradlew`的写法为例

### 几个重要对象Project，Task ，Action

**Project:**是Gradle最重要的一个领域对象，我们写的build.gradle脚本的全部作用，其实就是配置一个Project实例。它里面有几个重要的成员变量和方法，例如: 

```gradle
rootProject  //整个工程实例
project      //模块工程实例
//两个用法一致，下面以project为例
project.name   //工程名称
project.afterEvaluate{ } //整个工程构建完成后执行，注意是构建，不是执行完成
project.file("") //工程路径，里面接相对路程以获得文件对象
```
**Task:**被组织成了一个有向无环图（DAG）。Gradle中的Task要么是由不同的Plugin引入的，要么是我们自己在build.gradle文件中直接创建的
可以通过`gradlew tasks`来查看有哪些任务task


### 定义`task`

```gradle
task myTask {
    doFirst {
        println 'hello'
    }
    doLast {
        println 'world'
    }
}
```
用以下命令执行

```
gradlew myTask
```

> 这段代码的含义：给Project添加一个名为“myTask”的任务
> 用一个闭包来配置这个任务,Task提供了doFirst和doLast方法来给自己添加Action。
> `注意:` 要执行的代码一定要放到doLast或者doFirst中，不会它会在任务构建完成前执行

```gradle
//Test文件夹下建一个src目录，建一个dst目录，src目录下建立一个文件，命名为test.txt
task copyFile(type: Copy){
    from "src"
    into "dst"
}
```
这是一个“显式地声明Task的类型“的方式

### task的依赖关系

```gradle
task taskA {
    doLast {
        println 'this is taskA from project 1'
    }
}

task taskB {
    doLast{
        println 'this is taskB from project 1'
    }
}

taskB.dependsOn taskA
```

然后我们在命令行运行：

```
gradle taskA
```

运行结果会先执行taskB的打印，然后执行taskA的打印

> 通过`dependsOn` 的依赖方式，可让我们已有的task后面添加任务，例如Maven的打包上传`gradlew upload`(不太确定是不是这个命令)和`AndResGuard`资源压缩就是通过依赖实现的

### 通过project.afterEvaluate{ }添加任务

例如我们编译完成apk后，需要把apk，mapping等文件拷出来并重命名，但是又不想改变原有的命令
例如新建文件`copyApk.gradle`

```gradle
def outputPath = "./outApk/"
def outApkDir = file(outputPath)
if(!outApkDir.exists()) outApkDir.mkdir()

import java.text.SimpleDateFormat
/**
 * 获取Git 分支名
 */
def getGitBranch() {
    try {
        return 'git symbolic-ref --short -q HEAD'.execute().text.trim()
    } catch (Exception e) {
        return ''
    }
}

/**
 *组合最终需要的apk名称
 *若要定制最终输出的文件名，请修改该方法
 */
def getTargetApkName(){
    SimpleDateFormat format = new SimpleDateFormat("yyyyMMddHHmmss")
    def prefx = rootProject.name
    def today = format.format(new Date())
    def versionName = android.defaultConfig.versionName
    return "${prefx}_${getGitBranch()}_v${versionName}_${today}"
}
project.afterEvaluate {
    def relaseTask = "assembleRelease"
    //assembleReleass任务后添加拷贝命令
    if(tasks.findByName(relaseTask) != null){
        tasks.getByName(relaseTask) {
            it.doLast {
                println "$project.name: After assembleRelease, copy "
                def apkName = getTargetApkName()
                println "${apkName}"
                copy{
                    from "/build/outputs/apk/release"
                    into outputPath
                    include "app-release-unsigned.apk"
                    rename("app-release-unsigned.apk", "${apkName}.apk")
                }
                copy{
                    from "/build/outputs/mapping/release"
                    into outputPath
                    include "mapping.txt"
                    rename("mapping.txt", "${apkName}_mapping.txt")
                }
            }
        }
    }
}
```

在app模块的`build.gradle`的最下面引用该文件

```
apply from: "copyApk.gradle"
```

### 一些方法的使用

#### 拷贝文件
用copy方法

```gradle
copy{//拷贝maping文件
	  from "/build/outputs/mapping/release"   //从哪个目录
	  into outputPath    //拷贝到哪个目录
	  include "mapping.txt"  //拷贝哪个文件，可以用通配符
	  rename("mapping.txt", "${apkName}_mapping.txt")  //重命名，不能用通配符
}
```
#### 引入layoutlib.jar包
该包下面有一些系统使用的类，但不建议引入（可以用反射）

```gradle
dependencies {
    provided files(getLayoutLibPath())
}

def getLayoutLibPath() {
    def rootDir = project.rootDir
    def localProperties = new File(rootDir, "local.properties")
    def sdkDir = null
    if(localProperties.exists()){
        Properties properties = new Properties()
        localProperties.withInputStream {
            instr -> properties.load(instr)
        }
        sdkDir = properties.getProperty('sdk.dir')
    }
    if(!sdkDir){//linux环境下的获取sdk的路径
        sdkDir = System.getenv("ANDROID_HOME")
    }
    if(sdkDir){
        def compileSdkVersion = android.compileSdkVersion
        Console.println("app compileSdkVersion : " + compileSdkVersion)
        def androidJarPath = sdkDir + "/platforms/" + compileSdkVersion + "/data/layoutlib.jar"
        return androidJarPath
    }
    return rootDir
}
```

#### 从AndroidManifest下面获取VersionName

```gradle
//获取apk版本号
def getVersionNameAdvanced(flavor){
    flavor = flavor ? flavor : "main"
    def xmlFile = project.file("./src/$flavor/AndroidManifest.xml")
    def rootManifest = new XmlSlurper().parse(xmlFile)
    return rootManifest['@android:versionName']
}
```

#### 获取当前git的分支名

```gradle
/**
 * 获取Git 分支名
 */
def getGitBranch() {
    try {
        return 'git symbolic-ref --short -q HEAD'.execute().text.trim()
    } catch (Exception e) {
        return ''
    }
}
```



## `Groovy`基础
