## Eclipse 项目转移到Android Studio遇到的问题
### 1、Android Studio直接导入项目是copy原项目的，无法纳入代码管控
解决方案：

英文地址：http://developer.android.com/sdk/installing/migrate.html
翻译：Android Studio 中文组（大锤译）

如果你之前有用Eclipse做过安卓开发，现在想要把Eclipse中的项目导入到Android Studio的环境中，那么首先要做的是生成Build Gradle的文件。因为Android Studio 是用Gradle来管理项目的，具体操作步骤如下：
#### 从Eclipse中导出
1. 将你的ADT插件版本升级到22.0以上。
2. 在Eclipse中，选择File-->Export。
3. 在弹出的导出窗口中，打开Android的文件夹，选择“Generate Gradle Build Files”。
4. 选中你想要导入到Android Studio中的项目，Finish。

**PS**:导出的项目将会和原来的项目在同一目录，覆盖原来的同时，会新增一个叫build.gradle的文件，导入Android Studio时将首先读取这个文件。
#### 导入到Android Studio
1. 在Android Studio 中，首先关掉你当前的打开的项目。
2. 在欢迎界面，点击Import Project（注：也是可以直接在菜单选择Import project的）
3. 选中你在Eclipse中导出的项目，展开目录，点击build.gradle文件，然后OK
4. 在之后的弹出对话框中，会要求你选择Gradle的配置，选中Use gradle wrapper.(注：也可以自定义你本机装的Gradle)

**PS**：如果没有Grade build文件，也是可以将普通的安卓项目导入到Android Studio中，它会用现有的Ant build.但为了更好地使用之后的功能和充分使用构建变量，还是强烈地建议先从ADT插件中生成Gradle文件再导入Android Studio

### 2、编译错误1
```
Error:duplicate files during packaging of APK D:\work\xxx\build\outputs\apk\xxx-debug-unaligned.apk
Path in archive: META-INF/LICENSE.txt
Origin 1: D:\work\\libs\httpmime-4.1.3.jar
Origin 2: D:\work\\libs\ant.jar
You can ignore those files in your build.gradle:
android {
 packagingOptions {
   exclude 'META-INF/LICENSE.txt'
 }
}
Error:Execution failed for task '::packageDebug'.
> Duplicate files copied in APK META-INF/LICENSE.txt
File 1: D:\work\libs\httpmime-4.1.3.jar
File 2: D:\work\libs\ant.jar
```
**解决方案**：在`build.gradle中android {}`中添加
```
packagingOptions {
        exclude 'META-INF/DEPENDENCIES.txt'
        exclude 'META-INF/LICENSE.txt'
        exclude 'META-INF/NOTICE.txt'
        exclude 'META-INF/NOTICE'
        exclude 'META-INF/LICENSE'
        exclude 'META-INF/DEPENDENCIES'
        exclude 'META-INF/notice.txt'
        exclude 'META-INF/license.txt'
        exclude 'META-INF/dependencies.txt'
        exclude 'META-INF/LGPL2.1'
    }
```

### 3、编译错误2
```
importing a project creates a full copy
Error:(16, 0) Cause: startup failed:
build file 'D:\mc_works\\build.gradle': 16: unexpected char: '\' @ line 16, column 33.
               java.srcDirs = ['src\main\java']
```
**方案：gradle里 “\” 改为 “/”**

### 4、编译错误3 - 无法编译
在项目主`gradle`添加
```
allprojects {
    repositories {
        jcenter()
    }
}
```

### 5、运行出错-jni lib load失败，jni未成功引入项目导致
解决方案：在`build.gradle`里添加下面配置
```
sourceSets {
    main {
jniLibs.srcDirs = ['libs']
```

### 6、原项目根目录下有jni目录，可能触发自动编译ndk
解决方案：`gradle中`添加下面红色部分即可避免触发ndk编译， jni.srcDirs设为空
```
android {
    compileSdkVersion 19
    buildToolsVersion "23.0.0 rc3"

    defaultConfig {
        applicationId "com.xxx"
        minSdkVersion 15
        targetSdkVersion 19
```
        sourceSets.main {
            jni.srcDirs = []
            jniLibs.srcDir 'src/main/libs'
        }
```
.....


    }
```
快捷键可以设置成eclipse习惯（还是有些快捷键不一样）.
> http://blog.csdn.net/nadiee/article/details/47017137