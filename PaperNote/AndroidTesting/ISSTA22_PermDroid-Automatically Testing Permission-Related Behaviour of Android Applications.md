### Motivation
先前的方法由于不是permission-aware的，对于Permission-related API Coverage的覆盖率较低，很难检测到Permisson-related Bug，且由于没有目标导向Goal-directed exploration，容易陷入tarpits，比如陷入到某些界面之后很难跳出来。
The Android runtime permission model allows users to grant and revoke permissions at runtime. To verify the robustness of apps, developers have to test the apps repeatedly under a wide range of permission combinations, which is time-consuming and unsuited for regression testing. Existing app testing techniques are of limited help in this context, as they seldom consider different permission combinations explicitly.

A great deal of work has been focused on Android GUI testing. Unfortunately, these general GUI testing tools are not permission-aware, as they do not consider the runtime permission model.

### Summary
先静态分析，定位到权限相关API在App源代码的位置，然后静态分析构建STG (State Transition Graph)，接着把权限相关API信息关联到STG图的节点上，比如标识出STG图中某一个节点会触发权限相关的API。同时拥有触发权限API组件的那个界面，PermDroid会重点测试。对于其他的普通节点的组件，PermDroid对每个组件只会点击一次。动态分析时，会把动态信息补全到STG中，如果动态界面和STG中的某个节点的IOU大于50%，则匹配上。如果遇到了全新的界面，即STG中没见过的界面，动态界面也会更新到STG中。

### Limitations
1.自己定义了一个指标SCR与其他工具比较，可能不够fine-grained
2.测试的App数据集感觉比较不知名，high-profile
3.无法测试一些知名的，商业化的app。作者在论文中也承认了: App use encryption, obfuscation and signature to prevent the apps from being decompiled. The static analyzer of PermDroid may fail to repackage the instrumented apps and to make them runnable.
4.静态分析的技术普遍存在incomplete的问题 

### Observation
1.该方法区分了Activity, Menu, Dialog, Drawer, Fragment

### Evaluation
Monkey, APE, and GESDA

### Code
https://github.com/wsong-nj/PermDroid