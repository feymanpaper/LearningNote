
### Motivation
To automate GUI exploration, existing techniques often try to visit as many GUI pages as possible via specific strategies, e.g., random (like Monkey) or heuristic (like Stoat, A3E). However, their effectiveness is still unclear and much under-explored. (widget-page dependency, widget-widget dependency and system event dependency)

Widget-page dependency, Widget-widget dependency, System-event dependency.
These types of dependencies are hardly identified during the runtime exploration. For an app which has not been instrumented, it is difficult in most cases to determine whether a widget in the current page has an event handler, whether its event callback depends on the state of other widgets, and whether the current page has special lifecycle callbacks that need to be triggered by system events.

### Summary
The process of the approach is two-stage. 
First, a dependency integrated page transition model (dPTM) is statically constructed from an Android apk. The model uses page as the basic unit to describe the transitions between them. It captures the dependencies as the elements denoting the widgets with callbacks in a page (widget-page dependency), the widgets whose states influence a callback (widget-widget dependency), and the lifecycle callbacks of the page (system-event dependency). Second, a depth-first traversal based dynamic exploration is guided utilizing the static model. During a step of page exploration, the widgets with callbacks and the lifecycle callbacks are exercised preferentially, and the widgets whose callback are affected by the states of other widgets are fully exercised based on the combination of the states.

### Evaulation
对比工具: Monkey and Stoat

### Limitations
1. First, GESDA may fall into a “quagmire”. It means to constantly explore the newly appeared widgets whose logics are the same, so that the coverage cannot be increased and GESDA has no chance to leave the page. For example, when exercising MoneyWallet, GESDA fell in a Calendar page where there was new clickable date icons appeared when clicking an existing date icon. GESDA thus has a low coverage on this app
2. Second, GESDA does not have the ability to send broadcasts like Monkey, therefore it cannot cover the code related to broadcast receiver. For example, GESDA performs poor for Atomic because it is a chat client consuming series of broadcasts.
3. In order to compare the exploration effectiveness with Monkey and Stoat, we carefully choose some apps from Stoat benchmark and supplement others from F-Droid. These apps with different design structures and different sizes are able to demonstrate the ability of GESDA. However, we acknowledge these apps may not be broadly representative.
4. 没有重启机制，遇到日历就被卡住

### Observation
1.作者将页面分成了Pages in the app are classified into Activity, Menu and Dialog.
2.作者将组件记录为，wType, wId, wText, wEventHandler 
3.先前的工作没有识别组件之间的依赖关系，如要勾选checkbox才能点击按钮。但是该工作有完成这个任务，In the callback of the OK button, there exists a branch associated with the check state of the checkbox. Neither Monkey nor Stoat checked the checkbox before clicking the OK button during the exploration. Therefore the code surrounded by red dotted framework in pressOk was not covered.
4.先前的工作只能随机触发系统事件但该工作有完成这个任务: Both Monkey and Stoat can simulate system events such as rotation, but in a random manner.
5.The algorithm takes the dPTM of the app as input and a timeout indicating the time the exploration can spend.