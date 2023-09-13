Frida：一个hook的框架
Python+Js接口，容易学习
既可以Hook Java，也可以Hook so，xposed只可以hook Java
安卓全平台，不受安卓版本限制

hook普通方法，构造方法，重载方法，构造对象参数，修改对象属性（通过反射，反射也可以修改私有变量的值）

1. attach模式
```
attach到已经存在的进程，核心原理是ptrace修改进程内存。如果此时进程已经处于调试状态(比如做了反调试)，则会attach失败。
```
2. spawn模式
```
启动一个新的进程并挂起，在启动的同时注入frida代码，适用于在进程启动前的一些hook，比如hook RegisterNative函数，注入完成后再调用resume恢复进程。
```



