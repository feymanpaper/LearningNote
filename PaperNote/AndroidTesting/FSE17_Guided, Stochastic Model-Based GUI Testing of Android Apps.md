### Motivation
Model-based testing (MBT)--path-explosion problem
Random Model-based testing--However, without a strong guidance, such tests are often redundant and ineffective to detect bugs. In addition, the previous research on model-based GUI testing only considers UI-level events (e.g., click, edit), and disregards system-level events (e.g., screen rotation, incoming calls) during testing. Without combining both types of events, the effectiveness of MBT may be further limited due to inadequate testing.

### Summary
这篇工作与安卓UI自动化测试相关，这篇工作的早期版本是ICSE16(Companion) FSMDroid。
方法如下：
该工作通过UI automator拿到GUI信息，通过静态分析识别源代码中的event进行建模，采用GUIDED STOCHASTIC MODEL MUTATION不断变异优化。
Stoat operates in two phases: (1) Given an app as input, it uses dynamic analysis enhanced by a weighted UI exploration strategy and static analysis to reverse engineer a stochastic model of the app’s GUI interactions; and (2) it adapts Gibbs sampling to iteratively mutate/refine the stochastic model and guides test generation from the mutated models toward achieving high code and model coverage and exhibiting diverse sequences. During testing, system-level events are randomly injected to further enhance the testing effectiveness.

### Evaluation
比较的工具: 
RQ1: 基于FSM Model: MobiGUITAR and PUMA
RQ2: (1) Monkey (random fuzzing), (2) A3E (systematic UI exploration), and (3) Sapienz (genetic algorithm)

### Code
https://tingsu.github.io/files/stoat.html

### Observation
1.会随机触发系统事件:Moreover, Stoat adopts a simple yet effective strategy to enhance MBT: randomly inject various system-level events(e.g. SMS and Browser) into UI tests during MCMC sampling.
2.具备重启和回退能力: During the UI exploration, if the app enters into an unknown state (e.g., the app crashes, exits, becomes non-responding, or navigates to an irrelevant app) after some event is executed, restoreApp will be executed to restart the app or navigate the app back to the previous page.
3.支持的事件: click, touch, edit (generate random texts of numbers or letters), navigation (e.g., back, scroll, menu). 
4.唯一标识一个界面的方法: an app state s is abstracted as an app page (represented as a widget hierarchy tree, where non-leaf nodes denote layout widgets (e.g., LinearLayout) and leaf nodes executable widgets (e.g., Button)); when a page’s structure (and properties) changes, a new state is created. If the app exits/crashes, the ending state is treated as a final state.
5.对界面去重: In detail, (1) the hierarchy tree of an state is encoded into a string, and converted into a hash value for efficiently detecting duplicate states; (2) minor UI information, e.g., text changes (e.g.,the contents of TextViews/EditTexts) and UI property changes (e.g., the checked property of RadioButtons/CheckBoxs), is omitted without creating new states; (3) ListViews are only differentiated as empty and non-empty.