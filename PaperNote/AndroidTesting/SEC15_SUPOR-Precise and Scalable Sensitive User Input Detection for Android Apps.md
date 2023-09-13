### Motivation
Previous mobile privacy related research efforts have largely focused on predefined known sources managed by smartphones.
Sensitive user inputs through UI (User Interface), another information source that may contain a lot of sensitive information, have been mostly neglected.

### Summary
作者的工作与隐私相关，主要针对UI界面上的敏感数据，如user input。作者的方法是静态分析XML文件和源代码，配合污点分析找privacy disclosures

### Limitations
原文1: It is very difficult to generate test inputs to trigger all the UI screens in an app in a scalable way.
原文2: Though the developers sometimes dynamically generate UI elements in the code other than defining the UI elements via layout files, we focus on identifying sensitive user inputs statically defined in layout files in this work.
作者的工作并没有覆盖这种动态生成的敏感UI Input

### Observation
1. A well-designed app has to allow the user to easily identify the relevant texts for a particular input field and provide appropriate inputs based on his understanding of the meaning of texts. 
2. First, certain attributes of the widgets can be a good indicator about whether the input is sensitive. Using the inputType attribute with a value “textPassword”, we can directly identify password fields. However, not all sensitive input fields use this attribute value. The hint attributes also may contain useful descriptive texts that may indicate the sensitiveness of the input fields. 
	Besides attributes of UI widgets, we observe that nearby text labels rendered in the UI also provide indication about the sensitiveness of the widgets. ---物理位置上邻近的UI元素也有作用
