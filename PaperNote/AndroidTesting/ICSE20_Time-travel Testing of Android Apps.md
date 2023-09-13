### Motivation
Today, Android app testing is like playing a game of Super Metroid, albeit without the ability to save after important milestones and to travel back in time when facing the consequences of a wrong decision.
作者认为直接重启不是很好的办法，因此提出了基于虚拟机保存状态重启的办法
This problem is overcome only partially by restarting the Android app because (i) we must start from the beginning, (ii) there is no clean slate, e.g., database entries remain, and (iii) how to detect when we are stuck is still an open question. For Android testing, the ability to save and travel back to the most interesting states goes a long way towards a more systematic exploration of the state space.

### Summary
该工作与安卓UI自动化测试相关，方法是利用虚拟机来保存状态，当遇到stuck等情况能恢复到之前的状态。遇到界面时采用随机点的机制，并且通过代码覆盖率的变化情况来将状态标记为interesting, lack of progress，来进行相应处理。
In our example, one can think of TimeMachine as an automatic player that explores the map of Super Metroid through very fast random actions, automatically saves after important milestones, and once it gets stuck or dies, it travels back to secret passages and less visited rooms seen before in order to maximize the coverage of the map.

Implementing the following strategies: 
• Specifying criteria which constitute an “interesting” state, e.g., increases code coverage. Only those states will be saved. 
• Specifying criteria which constitute “lack of progress”, e.g., when testing techniques traverse the same sequence of states in a loop. 
• Providing an algorithm to select the most progressive state for time-travelling when a lack of progress is detected.

### Limitations:
1.基于虚拟机重启的方法耗时相对是比较久的;
2.随机点组件具有不确定性

### Observation
1.该方法对Screen的唯一标识符为UI Tree的层次结构，为了防止state space explosion，忽略了text的变化
State identification. In order to identify what constitutes a state, our framework computes an abstraction of the current program state. A program state in Android app is abstracted as an app page which is represented as a widget hierarchy tree (non-leaf nodes indicate layout widgets and leaf nodes denote executable or displaying widgets such as buttons and text-views). A state is uniquely identified by computing a hash over its widget hierarchy tree. In other words, when a page’s structure changes, a new state is generated.
2.利用虚拟机来保存状态和重启
State saving & restoring. We leverage virtualization to save and restore a state. Our framework works on top of a virtual machine where Android apps can be tested. App states can be saved and restored with VM
3.该方法检测Loop和Dead ends
4.该方法采用基于Code coverage的启发式方法，比如对当发现代码覆盖率增加和没变化的情况进行区分，来剪枝
5.该方法会触发系统事件: TimeMachine also includes a system-level event generator taken from Stoat to support system events such as phone calls and SMSs.

### Evaluation
比较的工具: Monkey, Sapienz, Stoat

### Code
https://github.com/DroidTest/TimeMachine