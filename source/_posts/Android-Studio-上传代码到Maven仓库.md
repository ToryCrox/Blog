---
title: Android Studio 上传代码到Maven仓库
date: 2018-07-04 22:47:24
tags: [Gradle, Android]
---

将自己的代码库上传到Maven一般有几个选择:
1. 本地仓库
2. 自己搭建的maven私有仓库, 如: Nexus
3. 上传到Maven
4. 上传到jcenter
5. 上传到jitpack，这个上传很方便，比较推荐


## 参考

[Android Studio上传项目到Maven仓库](https://www.jianshu.com/p/57f8af15ef9c/)

## 几点注意

1. 上传library不能引用aar
2. 配置了`publishNonDefault true`会引用上传时将Release和Debug的aar都上传到Maven，所以要么去除要么按照下面的方法配置
3. 注意如果上传到snapshots测试仓库中，version必须以`-SNAPSHOT`结尾

## Gradle上传配置

```java
apply plugin: "maven"

def compileMode = 1

//分别为正式仓库，测试仓库和本地仓库，
def releaseUrl = "http://192.168.1.78:8090/nexus/content/repositories/releases"
def snapshotsUrl = "http://192.168.1.78:8090/nexus/content/repositories/snapshots/"
def localUrl = 'file://' + new File(System.getProperty('user.home'), '.m2/repository').absolutePath
def uploadUrl = compileMode == 1 ? snapshotsUrl : compileMode == 2 ? localUrl : releaseUrl

uploadArchives {
    repositories {
        mavenDeployer {
            repository(url: uploadUrl) {
                authentication(userName: NAME, password: PASSWORD) // maven授权信息
            }

            pom.version = VERSION
            pom.artifactId = ARTIFACT_ID
            pom.groupId = GROUP_ID   
        }
    }
}
```
> 注意如果上传到snapshots测试仓库中，version必须以`-SNAPSHOT`结尾

## 多flavor或多buildType配置

maven上传是默认不支持多flavor的，如果library配置了`publishNonDefault true`, **在执行`gradlew upload`时会将releas和debug的aar包都上传上去，导致在引用时无法找到aar**, 这点千万要注意, 需要修改gradle配置

```java
uploadArchives {
    repositories {
        mavenDeployer {
            repository(url: uri(uploadUrl)){
                //配置账号密码
                authentication(userName: "userName", password: "password")
            }
            android.libraryVariants.all { variant ->
                def isFlavor = !variant.flavorName.isEmpty()
                def isRelease = variant.buildType.name == "release"

                //只上传release的，如果没有多个flavor的去掉isFlavor的判断
                if(isRelease && isFlavor){
                    def _flavorName = "${variant.flavorName}"

                    //此处需要再进行一次过滤，否则debug的会被上传上去
                    addFilter(_flavorName) { artifact, file ->
                        (artifact.attributes.classifier == "${variant.flavorName}Release")
                    }
                    pom(_flavorName).version = android.defaultConfig.versionName
                    pom(_flavorName).groupId = "ARTIFACT_ID"
                    pom(_flavorName).artifactId = "GROUP_ID-${_flavorName}"
                }
            }
        }
    }
}
```


## 将源码，java doc等一并上传

```java
// 指定编码
tasks.withType(JavaCompile) {
    options.encoding = "UTF-8"
}
  
// 打包源码
task sourcesJar(type: Jar) {
    from android.sourceSets.main.java.srcDirs
    classifier = 'sources'
}
  
task javadoc(type: Javadoc) {
    failOnError  false
    source = android.sourceSets.main.java.sourceFiles
    classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
    classpath += configurations.compile
}
  
// 制作文档(Javadoc)
task javadocJar(type: Jar, dependsOn: javadoc) {
    classifier = 'javadoc'
    from javadoc.destinationDir
}
  
artifacts {
    archives sourcesJar
    archives javadocJar
}
```

## 上传jitpack

jitpack网址:https://jitpack.io/

添加jitpack的仓库引用:

```java
maven { url 'https://jitpack.io' }
```