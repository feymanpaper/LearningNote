Accessibility services should only be used to assist users with disabilities in using Android devices and apps. They run in the background and receive callbacks by the system when `[AccessibilityEvent]`s are fired. Such events denote some state transition in the user interface, for example, the focus has changed, a button has been clicked, etc. Such a service can optionally request the capability for querying the content of the active window. Development of an accessibility service requires extending this class and implementing its abstract methods.

在 Accessibility Service 中，可以通过模拟点击的方式来点击一个组件，并通过监测 `AccessibilityEvent` 的方式来判断该组件是否跳转到了下一个界面。
具体来说，可以按照以下步骤实现：

1.  使用 `findAccessibilityNodeInfosByViewId()` 方法或其他方法找到要点击的组件。
2.  判断找到的组件是否可以被点击，如果可以则调用 `performAction()` 方法模拟点击，如果不能则进行其他处理。
3.  等待一段时间，让应用程序有足够的时间响应点击事件，并监测是否有 `AccessibilityEvent` 发生，如果发生了则进行处理。
```java
public class MyAccessibilityService extends AccessibilityService {
    private boolean mIsClicked = false; // 是否已经点击了目标组件

    @Override
    public void onAccessibilityEvent(AccessibilityEvent event) {
        if (mIsClicked && event.getEventType() == AccessibilityEvent.TYPE_WINDOW_STATE_CHANGED) {
            // 如果已经点击了目标组件，并且发生了界面跳转，表示已经返回到上一个界面
            mIsClicked = false;
            return;
        }

        AccessibilityNodeInfo rootNodeInfo = getRootInActiveWindow();
        if (rootNodeInfo == null) {
            return;
        }

        // 找到要点击的组件
        AccessibilityNodeInfo targetNode = findTargetNode(rootNodeInfo);
        if (targetNode != null) {
            // 如果目标组件可以被点击，则点击
            if (targetNode.isClickable()) {
                targetNode.performAction(AccessibilityNodeInfo.ACTION_CLICK);
                mIsClicked = true;
            } else {
                // 否则进行其他处理
                // ...
            }
        }
    }

    private AccessibilityNodeInfo findTargetNode(AccessibilityNodeInfo rootNodeInfo) {
        // 使用 findAccessibilityNodeInfosByViewId() 方法或其他方法找到要点击的组件
        // ...

        return targetNode;
    }
}

```


在Accessibility Service中，实现点击一个按钮后返回可以使用performGlobalAction()方法来模拟按下系统的Back键，如下所示：
```java
performGlobalAction(GLOBAL_ACTION_BACK);
```

在 Accessibility Service 中，可以通过监听 `AccessibilityEvent` 的 `TYPE_WINDOW_STATE_CHANGED` 事件类型来实现对所有页面跳转的监听。
需要注意的是，某些应用程序可能会使用非标准的方式进行页面跳转，此时可能无法通过 `TYPE_WINDOW_STATE_CHANGED` 事件类型来进行监听。如果遇到这种情况，可以考虑使用其他方法进行监听，如监测 `View` 的状态变化等。

### 问题

1:进程线程的问题（暂时解决）
DelayRendering会造成当前的线程MyService阻塞，而UI进程仍在渲染，因此等当前的线程执行完回调函数的方法时，继续执行下一个回调函数的方法时会导致问题，比如下一个回调函数执行的方法getRootInActiveWindow得到的是UI进程渲染出的最新的状态，而不是回调函数当时监听的状态
解决方法：将Thread.sleep加到getRootInActiveWindow和performAction之间

2.若全部不加Thread.sleep，会切换太快导致Access没有检测到页面跳转而调用回调函数onAccessibilityEvent（暂时解决）
解决方法：加上Thread.sleep 给UI线程足够的时机渲染

3:无法确定哪些组件点了可以跳转
目前的策略
暂时不处理没有文本的  
后期需要替换成其他的策略

疑点：
为什么print能打印出DDProgressDialog，但是在deal的时候没有检测到？
可能是Access错过了


存在延时问题 ，如果把Thread.sleep去掉又不存在了
```java
AccessibilityNodeInfo nextnodeInfo = getRootInActiveWindow();
AccessibilityNodeInfo currentNode = nodeList.get(0);  
currentNode.performAction(AccessibilityNodeInfo.ACTION_CLICK);  
try {  
    Thread.sleep(3000);  
} catch (InterruptedException e) {  
    throw new RuntimeException(e);  
}  
  
AccessibilityNodeInfo nextnodeInfo = getRootInActiveWindow();
```

![[Pasted image 20230313120851.png]]