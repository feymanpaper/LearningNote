### Motivation
Callback-driven testing eliminates the need for event generation by directly invoking app callbacks. However, existing callback-driven testing techniques assume prior knowledge of Android callbacks, and they rely on a human expert, who is familiar with the Android API, to write stub code that prepares callback arguments before invocation. Since the Android API is very large and keeps evolving, prior techniques could only support a small fraction of callbacks present in the Android framework.
Many prior techniques rely on UI testing frameworks to exercise the app by generating appropriate events. However, a large class of events is widget-specific, and requires multiple user actions to be taken in a specific order at specific UI coordinates. As we explain in Section III, the onDateChanged event of the DatePickerDialog widget is one such example. Generating such events deterministically is challenging for a UI-based testing tool, unless it has been equipped with the knowledge of how to generate all the correct events. Given the variety of the Android widgets, and the different types of events they support, this is non-trivial.

### Summary
作者的工作主要是和安卓UI自动化测试相关。作者提出了基于callback驱动的测试框架COLUMBUS，该方法首先需要静态分析App源代码找到相关的callback函数；然后需要分析和生成调用callback函数的参数(Primitive type arguments, Reference type arguments)，对于Primitive type arguments采用under-constrained symbolic execution的方式推导possible value， 对于Reference type arguments采用Frida对App Heap进行搜索，并且利用反射相关机制来生成相关实例；然后构建Inter-callback dependency来推到share variables之间的数据依赖关系；最后进行Callback-guided exploration，并且提出了two novel feedback mechanisms，第一个是利用callback dependency生成相关的callback sequences来提高触发bug的概率，第二个是降低crash-including path的优先级而把探索优先级放在unexplored code来最大化代码覆盖率和触发crash bug。

### Limitations:
静态分析存在假阴假阳
可能会推导出错误的callback函数参数

### Evaluation
对比工具: STOAT  and APE , checkpoint-based technique TIMEMACHINE  and callback-driven technique EHBDROID 

### Code
https://github.com/ucsb-seclab/columbus