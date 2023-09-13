Gradle:编译，打包Android工程的一个构建工具

一个project可以用多个module，module可以是app类型的，也可以是library类型的
![[Android_module_project.png]]

build.gradle(Project级别)
gradle版本与gradle plugin插件版本要符合匹配关系（开发者文档可以查）

```
compileSdkVersion, minSdkVersion, targetSdkVersion, buildToolVersion之间的区别
```
compileSdkVersion: 是编译代码所使用的sdk版本，在sdk manager下载了才能用
minSdkVersion：是对app可运行的手机设备的最小版本限制。与sdk manager无关
targetSdkVersion：是对app要运行的手机设备的目标版本的标识，与sdk manager无关，标识该app是为了某个版本的手机设备设计的，在目标版本的设备上做了充分的测试。当你的手机大于这个版本时，该app也能运行。高版本的手机是可以运行低版本的软件的。
因此，minSdkVersion和targetSdkVersion是对我们开发的app所能运行设备的系统版本的范围约束。因此不可以小于minSdkVersion，但也没有最高限制。
**重要原则：minSdkVersion<=targetSdkVersion<=compileSdkVersion**

另外，如果使用了 support library, suport library的版本要和compileSdkVersion一致

buildToolVersion是独立出来的一个东西，和上面三个没关系，是构建代码的工具的版本。
与sdk manager里面的 sdk tools 下载的东西是对应的。要想使用哪个版本，必须下载了对应的sdk Build-tools。
