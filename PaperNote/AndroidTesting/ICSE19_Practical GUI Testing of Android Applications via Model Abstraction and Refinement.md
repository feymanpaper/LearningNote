### Motivation
First, if the model is overly fine-grained, the testing tool cannot systematically explore the model due to state explosion. Second, if the model is overly coarse-grained, the testing tool cannot gather sufficiently accurate knowledge on model actions to realize effective guidance (i.e., difficult to replay model action sequences on the tested app)

### Summary
该工作和安卓UI自动化测试相关
该方法采用了dynamic model abstraction来自适应调整App变化的情况
This paper proposes APE, a new, practical model-based automated GUI testing technique for Android apps via effective dynamic model abstraction. This initial abstraction may be ineffective. Based on the runtime information observed during testing, APE gradually refines the model by searching for a more suitable abstraction, an abstraction that effectively balances the size and precision of the model.
Specifically, APE represents the dynamic abstraction with a decision tree, and tunes it on-the-fly with the feedback obtained during testing.

Initially, the testing tool starts with an empty state machine as the model. During each iteration, the testing tool (1) obtains the current GUI tree of the app, (2) identifies an existing, corresponding state, or creates a new state for this GUI tree, and (3) picks a model action and determines a concrete GUI action to interact with the app

### Observation
1.该方法进行了剪枝: Through abstraction, many GUI actions with the same behavior can be mapped to the same model action. Since these GUI actions behave the same, the testing tool does not need to exercise each of them and can instead select a representative GUI action among them when executing the model action.
2.该方法唯一标识组件的策略: An absolute node path uniquely identifies a widget in the tree.
A widget can be uniquely identified (in the tree) by its full attribute path because the order of the widgets on the full attribute path resembles the hierarchy of widgets in the tree, and the index attribute of each widget determines the unique location of the widget among its siblings.Therefore, a GUI tree T can essentially be represented with, and is equivalent to, the set of full attribute paths of all its widgets.
3.剪枝策略：In general, state abstraction is realized based on the similarity of full attribute paths of GUI actions. Specifically, state abstraction determines two GUI actions as equivalent if (1) they have the same action type and (2) their full attribute paths can be reduced to the same attribute path π by removing irrelevant attributes under certain reduction rules. 
Similarly, state abstraction determines two GUI trees as equivalent if all of their GUI actions can be reduced to the same set of attribute paths

### Evaluation
对比工具: Monkey, SAPIENZ, and STOAT