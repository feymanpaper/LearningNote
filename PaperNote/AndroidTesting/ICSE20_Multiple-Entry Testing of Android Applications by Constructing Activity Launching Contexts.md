### Motivation
Existing GUI testing approaches of Android apps usually test apps from a single entry. In this way, the marginal activities far away from the default entry are difficult to be covered. The marginal activities may fail to be launched due to requiring a great number of activity transitions or involving complex user operations, leading to uneven coverage on activity components. Besides, since the test space of GUI programs is infinite, it is difficult to test activities under complete launching contexts using single-entry testing approaches
Because of the infinite test space of GUI programs, it is difficult to launch activities under complete contexts using SET. 

### Summary
作者的工作与安卓UI自动化测试相关。Motivation提到之前的工作大多是基于SMT(Single-Entry Testing)，在某个Activity离Default Entry较远的时候会比较难触发，此外SMT在每个Activity测试的时间分布也不是均匀的。因此作者提出了MET(Multiple-Entry Testing)。该工作主要利用了安卓ICC(Inter-component Communication)的特性, 可以使用Intent来触发Activity的跳转，前提是将manifest.xml中的activity的配置改成android:exported=true或者设置intent  filters。因此，该方法不需要只以MainActivity作为起点，而是可以拥有多个起点来测试各个Activity。该方法使用静态分析收集触发Activity所需要的Intent相关参数，并且提出了adaptive exploration framework来动态测试。

### Limitations
该方法是基于Activity的转换图，可能没有考虑到Fragment等复杂界面；静态分析收集触发Activity所需要的Intent相关参数存在少量假阳假阴，也可能造成触发activity失败

### EVALUATION
和Monkey, APE, IntentFuzzer比较

### Code
Fax. 2019. https://github.com/hanada31/Fax. (2019).