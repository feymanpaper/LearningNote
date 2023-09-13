### Motivation
However, existing GUI testing tools such as probability-based or model-based ones suffer from low testing coverage when testing practical commercial apps, meaning that they may miss important bugs and issues.
To address these limitations, there has been a growing interest in using deep learning (DL) and reinforcement learning (RL) methods for automated mobile GUI testing. By learning from human testers’ behavior, these methods (limitations: low testing coverage, weak generalization, and heavy reliance on training data) aim to generate human-like actions and interactions that can be used to test the app’s GUI more thoroughly and effectively. These approaches are based on the idea that the more closely the actions performed by the testing algorithm mimic those of a human user, the more comprehensive and effective the testing will be.

### Summary
该工作将安卓UI自动化测试任务建模成Q&A任务，收集静态信息(包括APK中的manifest，UI Tree信息)和动态信息 (i.e., whether and how many times a GUI page has been explored or a widget has been operated)，然后训练编码器和解码器，编码器负责将上述信息编码成Chatgpt能接受的Prompt, 解码器解析Chatgpt的输出，Chatgpt会指示下一步操作 (如点击哪个组件)，点击之后继续收集当前信息，不断重复循环。

### Limitations:
1.只记录了Activity的访问次数 (The visit number of the GUI page is updated through finding the same activity in the memorizer with the “ActivityName” field of the page.)，以Activity为粒度可能没有考虑Fragment。
2.We further analyze the potential reasons for the uncovered cases. First, some widgets or inputs do not have meaningful “text” or “resource-id”, which hinders the approach of effectively understanding the GUI page.
3.Second, some app requires specific operations, e.g., database connection, long press and drag widgets to a fixed location, which is difficult if not impossible to be automatically achieved.

### Evaluation
比较的工具: 
Random-/rule based: Monkey Droidbot 
Model-based: Stoat Ape Fastbot ComboDroid TimeMachine
Learning-based: Humanoid Q-testing GPTDroid

### Observation
1.该工作使用all the widgets represented by the “text” field or “resource-id” field (the first non-empty one in order), and the widget position of the page来唯一标识一个Screen。此外还收集了父节点和兄弟节点的信息We also extract the information from nearby widgets to provide a more thorough perspective, which includes the “text” of parent node widgets and sibling node widgets
2.该工具可能有以下能力Valid text inputs，Compound actions，Test case prioritization，Long meaningful test trace.
