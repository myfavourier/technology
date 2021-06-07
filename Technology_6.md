# Technology_6

## gradle编译流程

### 概述

Gradle是一个基于Apache Ant和Apache Maven概念的项目自动化构建开源工具，它通过组织执行一系列的Task来完成自动化的构建，减少构建项目时的复杂度，而Android Studio也使用Gradle作为基础的构建工具。

Gradle自身是用Groovy和Java进行开发的，同时Gradle脚本使用基于Groovy的特定领域语言(DSL)来声明项目配置，不用像Maven/Ant那样基于XML来进行各种繁琐的配置。目前也增加了基于Kotlin语言的Kotlin DSL，因为是Jetbrains自家的语言，所以IDE支持也会更加完善



### Gradle 项目结构分析

在了解Gradle build的生命周期之前，先简单说一下Gradle的项目层次，新建一个Android工程，可以看到项目的结构大致如下：

![image.png](http://sf1-vcloudcdn.pstatp.com/img/tos-cn-i-0000/c7817734afb64081a63e2c76f15f56d0~tplv-noop.image?width=872&height=810)

#### settings.gradle

settings.gradle是用于配置整个项目结构的脚本，对应Gradle中的`Settings`对象，其中最常用的几个方法：

- include 依赖子工程
- includeFlat 平级依赖工程
- project 获取Project对象

一般都是在这里进行子模块的引用

```
include  ':app' , ':submodule' 
project(':app').projectDir = new File("./...") // 指定子模块的目录位置
```



#### rootProject/build.gradle

位于项目根目录的build.gradle脚本，负责项目整体的配置，对应一个`Project`对象，其中最常用的几个方法

- buildscript 配置构建脚本
- repositories 配置仓库地址，后面查找依赖的时候就按顺序依此查找
- dependencies 配置项目的依赖项
- allprojects 所有项目的通用配置项

```
buildscript { // 配置构建脚本
    repositories { // 构建脚本的仓库
        google()
        jcenter()
        
    }
    dependencies { // 构建脚本的依赖，默认依赖Android Gradle Plugin，也可以添加自定义的Plugin
        classpath 'com.android.tools.build:gradle:3.5.1'
    }
}

allprojects {
    repositories {
        google()
        jcenter()        
    }
}
```

#### mudule/build.gradle

位于子模块的build.gradle脚本，负责这个模块的配置，也对应一个Project对象，类似根目录的build.gradle。

```
// 引入Android Gradle Plugin，如果是library，引入'com.android.library'
apply  plugin :  'com.android.application'
      
android {
    // 配置编译相关的内容
    compileSdkVersion 29
    buildToolsVersion "29.0.2"
    defaultConfig {
        applicationId "me.lyz.gradledemo"
        minSdkVersion 23
        targetSdkVersion 29
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }
    // 根据编辑类型(Debug/Release)来配置一些编译属性
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    // 添加模块相关的依赖
}
```

#### gradle.properties

gradle全局的统一配置，key-value配置，用于配置例如守护进程开关，aapt2开关，jvm参数和一些自定义的通用参数，需要上传到仓库

#### local.properties

和gradle.properties类似，主要是一些本地化的配置，像头条工程里需要源码依赖的模块，或者一些安全性较高的仓库账号密码

### Gradle build的生命周期

Gradle build的生命周期主要分为三大部分：初始化阶段，配置阶段和执行阶段

#### 初始化阶段（Initialization）

Gradle支持单工程或者多工程构建，初始化阶段的任务是确定有多少工程需要构建，创建整个项目的层次结构，并且为每一个项目创建一个`Project`实例对象。

如果是多工程构建，一般都会在根工程目录下声明一个`settings.gradle`脚本，在脚本中include所有需要参与构建的子工程，通过解析`settings.gradle`脚本，读取include信息，确定有多少个Project需要构建。

#### 配置阶段（Configuration）

配置阶段的主要任务是生成整个构建过程的有向无环图

确定了所有需要参与构建的工程后，通过读取解析各个工程对应的`build.gradle`脚本，构造`Task`任务，并根据`Task`的依赖关系，生成一个基于`Task`的有向无环图`TaskExecutionGraph`

#### 执行阶段（Execution）

通过读取配置阶段生成有向无环图`TaskExecutionGraph`，按顺序依此执行各个`Task`，像流水线一样，一步一步构建整个工程，这也是构建过程中最耗时的阶段。

#### 生命周期回调

Gradle build的整个生命周期大致如下，在各个生命周期执行阶段也都提供了回调

![image.png](http://sf1-vcloudcdn.pstatp.com/img/tos-cn-i-0000/f1aee4a872a148928a4a480ce77de3cf~tplv-noop.image?width=586&height=1330)

我们可以通过gradle或者project对象添加一些监听器

```
gradle.addBuildListener(new BuildListener() {
    @Override
    void buildStarted(Gradle gradle) {
				// 开始构建
    }

    @Override
    void settingsEvaluated(Settings settings) {
				// settings.gradle执行解析完毕
    }

    @Override
    void projectsLoaded(Gradle gradle) {
				// Project初始化构造完成
    }

    @Override
    void projectsEvaluated(Gradle gradle) {
				// 所有Project的build.gradle执行解析完毕
    }

    @Override
    void buildFinished(BuildResult result) {
				// 构建结束
    }
})


gradle.taskGraph.whenReady() {
    println "taskGraph build ready"
}

gradle.taskGraph.beforeTask {
    println "$name beforeTask"
}

gradle.taskGraph.afterTask {
    println "$name afterTask"
}
```



通过这些生命周期的回调，我们可以根据实际的需求做一些自定义的操作，例如：

- 为指定的Task添加Action
- 获取各个阶段的耗时情况
- 动态改变Task的依赖关系，插入一些自定义的Task

### Android项目的构建流程

Android项目的构建流程也都会经历上述的三大阶段，这里先看一下官方文档中的构建流程图

![image.png](http://sf1-vcloudcdn.pstatp.com/img/tos-cn-i-0000/f80bca90b19d486fa9e5db59f42f1482~tplv-noop.image?width=950&height=1068)

看起来不是特别复杂，简单来说就是四大步骤：

1. 编译器将源代码转换成 DEX 文件，并将其他所有内容转换成编译的资源。
2. APK 打包器将 DEX 文件和编译的资源组合成单个 APK
3. APK 打包器使用调试或发布密钥库为 APK 签名
4. 使用 ZipAlign 工具对APK进行优化

现在我们知道了，Gradle编译项目时都是通过执行一个个Task的，那么将Android项目的编译流程拆分到Task的维度，又具体是需要执行哪些Task呢？

首先新建一个Android工程，执行Android项目的构建流程，可以发现没有任何修改的情况下就已经有30多个Task需要执行：

```
> Task :app:preBuild
> Task :app:preDebugBuild
> Task :app:checkDebugManifest
> Task :app:generateDebugBuildConfig
> Task :app:compileDebugRenderscript
> Task :app:compileDebugAidl
> Task :app:javaPreCompileDebug
> Task :app:mainApkListPersistenceDebug
> Task :app:generateDebugResValues
> Task :app:generateDebugResources
> Task :app:mergeDebugResources
> Task :app:createDebugCompatibleScreenManifests
> Task :app:processDebugManifest
> Task :app:processDebugResources
> Task :app:compileDebugJavaWithJavac
> Task :app:compileDebugSources
> Task :app:mergeDebugShaders
> Task :app:compileDebugShaders
> Task :app:generateDebugAssets
> Task :app:mergeDebugAssets
> Task :app:processDebugJavaRes
> Task :app:mergeDebugJavaResource
> Task :app:checkDebugDuplicateClasses
> Task :app:mergeLibDexDebug
> Task :app:transformClassesWithDexBuilderForDebug
> Task :app:validateSigningDebug
> Task :app:signingConfigWriterDebug
> Task :app:mergeDebugJniLibFolders
> Task :app:mergeDebugNativeLibs
> Task :app:stripDebugDebugSymbols
> Task :app:mergeProjectDexDebug
> Task :app:mergeExtDexDebug
> Task :app:packageDebug
> Task :app:assembleDebug
```



#### 关键Task的说明

- `mergeDebugResources`，收集所有AAR中和源码中的资源文件，合并到一个目录下
- `processDebugManifest`，收集所有`AndroidManifest.xml`文件，合并为一个Manifest文件
- `processDebugResources`，通过AAPT生成R.java和资源索引文件及符号表
- `compileDebugJavaWithJavac`，javac，将java文件编译成class文件
- `transformClassesWithMultidexlistForDebug`，使用MultiDex才会有这个task
- 分析类之间的依赖，确定哪些类必须放在Maindex中
- 生成混淆配置项
- `transformClassesWithDexBuilderForDebug`，合并class生成dex文件
- `packageDebug`，打包生成APK

#### Android Transform

Android Gradle Plugin在1.5.0-beta1时开始提供Transform API，可以在class转换为dex之前，对class进行一些特殊的处理，直接使用Transform API，可以不用去关注相关task的生成与执行流程，只要专注于对input的文件进行处理。

回到原理上来看，我们可以将其理解为一个Gradle Task，将自定义的Transform注册进去后，Gradle的TaskManager会在javac执行完后，遍历所有的transform，添加到TransformManager中，并且会先添加自定义的transform，再添加Proguard，MultiDex，DexBuilder等transform task

根据添加的顺序，先执行自定义的transform，后面再执行自带的transform，第一个transform会接收javac编译和资源编译的中间产物，每个transform任务对class处理完后再传递给下一个transform任务。

![image.png](http://sf1-vcloudcdn.pstatp.com/img/tos-cn-i-0000/6bc12a8f3f2f4475a92fa86f314c9a77~tplv-noop.image?width=900&height=1090)