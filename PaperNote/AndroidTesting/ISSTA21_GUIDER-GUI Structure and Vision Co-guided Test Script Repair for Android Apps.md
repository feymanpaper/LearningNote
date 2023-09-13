### Summary
该论文与安卓UI自动化测试修复相关。App开发者会写测试脚本，但是当App版本更新，UI组件的位置发生变化，测试脚本也需要更新，十分耗费时间，因此作者提出自动化的方法。该方法仅依赖App visual or structural information of app GUIs，配合CV。

### Observation
标识组件的方法: Identity Properties of Widgets. Three properties common to all widgets are especially important for deciding whether two widgets are matching in Guider, namely property resource-id, property content-desc, and property text, since the Android documentation recommends that different widgets should have distinct values for these properties3 4 5. We refer to these properties as identity properties. 