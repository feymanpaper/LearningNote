## 思路
dfs核心思路 
```
dfs:
	记录当前screen的信息
	for当前screen所有能点击的组件:
		click该组件
		dfs
	循环执行完,当前screen没东西可点了，back
```

screen表示一个界面，主要是为了区分同一个activity的fragment, menu, dialog, drawer

如何唯一地标识一个screen？
计划获取当前页面所有**可点击组件**的具体信息:resource_id, text, cnt,  如果该节点可点击但文本为NULL那大概率文本存在其子节点上，递归找子节点文本内容。cnt是我们自己添加的，主要是为了区分多个resoure_id相同, text相同的情况（比如resour_id和text都为NULL), 因此我们添加一个cnt来表示该可点击组件是当前screen的第几个可点击组件。
然后我们会把当前screen对应的package_name + class_name + activity_name + 上述信息进行哈希SHA256，生成唯一标识一个screen的签名

dfs细化一些内容
```
def dfs(last_screen):
	cur_screen = 当前的screen
	
	#screen没有变化说明该组件不会造成页面跳转
	if cur_screen == last_screen: 
		return
	# 回边针对activity,也需要针对menu, fragment, dialog
	if 产生回边:
		记录信息
	# 否则screen产生了变化，造成了页面跳转(activity, fragment, dialog, menu...)
	if cur_screen没被记录过:
		将cur_screen加入到screen跳转图中
		在last_screen记录哪个组件能到达cur_screen
		在cur_screen记录相关信息 
	else cur_screen之前记录过了:
		#do nothing 
	
	clickable_elements = 当前页面所有可点击组件
	for ele in clickable_elements:
		#还需要判断当前界面是否是当前界面？其他界面可能没跳回来
		if 该组件没被点击过 
			点击该组件
			记录该组件被点过
			dfs(cur_screen)
		else 该组件被点击过:
			if该组件不会产生回边:
				if该组件会跳转新的screen且新的screen有没点击完的组件(递归检测):
					点击该组件
					dfs(cur_screen)
	#检查一下栈深度再确定要不要back？
```

## 难点
- [x] 产生activity回边
- [ ] 某些fragment点击后，产生了局部的新内容，但是拿到的可点击组件包括背景的可点击组件，然而这些可点击组件是上一个screen的，产生了重叠
- [ ] Fragment回边问题
- [ ] 有些按钮无法通过back回退
- [x] 不是树是图，要改数据结构
- [x] ScreenA点击一个Menu，打开了ScreenB，点击了ScreenB的一个按钮到了ScreenC,假设ScreenC的组件遍历完了，会back到A，此时程序仍在执行B的for循环，需要检测for循环中当前界面是否为real当前界面，如果不是就return，因此for循环又到了A，此时假如ScreenB没有点完，因为for循环已经过了，会错过一些组件。
- [x] B. 页面变化的问题: 比如A->B, B->C，C回退之后到B，此时的Screen B可能会发生微小的变化。由于我们是基于包名+activity名、、+所有可点击组件的组件内文本来唯一标识一个Screen，因此即使产生了微小的变化，算法会认为是不同的Screen，从而产生BUG
- [ ] 页面渲染问题，难以知道安卓的界面什么时候渲染完成，盲目地提高sleep时间也不是一直高效的办法
- [ ] 必须从某个节点，就递归地知道所有孩子节点是否存在可点
- [ ] 必须从某个节点，就递归地找到他的组件，然后看是否会产生环
- [x] 从右下角往上走会不会效率更高


## V1.0
钉钉未登陆状态
总共点击的activity个数 9
总共点击的Screen个数: 23
总共点击的组件个数: 96

飞书
总共点击的activity个数 10
总共点击的Screen个数: 32
总共点击的组件个数: 94

飞书未登陆
总共点击的activity个数 3
总共点击的Screen个数: 15
总共点击的组件个数: 51

哈喽出行
总共点击的activity个数 6
总共点击的Screen个数: 10
总共点击的组件个数: 33

v.1.1
总共点击的activity个数 8
总共点击的Screen个数: 20
总共点击的组件个数: 110


## V1.2.0
### 解决的问题
1. 基于文本相似度，给Screen发生微小变化的情况容错，使用LCS最长公共子序列，如果为90%相似，则认为是同一个Screen;  但需要注意，发生微小变化仍是一个Screen的情况: 回退的时候界面左上角多了个叉；发生微小变化不是一个Screen的情况: 点击一个组件后产生了一个小的dialog，dialog的文本很少，但是需要视为不同的Screen

2. 修复了之前的唯一标识一个组件的bug，增加了location来唯一标识一个组件

3. merge_same_clickable_elements,优化: 若clickable_eles中存在连续k个相同的ele,合并为1个,不用每个都点击,进行合并,适合选择国家和地区的场景

4. 增加了退出判定策略，for循环执行完毕后，需要增加判断1.当前界面是否为当时遍历的当前界面。2.判定是否为最后一个界面，最后一个界面即进入的第一个界面first_screen_text。3.否则正常回退，press back

5. 给一个点击无效的组件一个容错机会,点过组件无效可能只是因为屏幕挡住了

### 需要解决的问题
A->B->C，C->back->A，此时A会因for循环到下一个组件，但B此时还并没有点完

## V1.2.1
1. for结束追加判断,如果循环结束发现该组件click_map对应的下一个Screen存在没点的组件,就让循环再执行，防止的意外情况A->B->C, C->back->A,此时for循环结束后,即使B还没点完,也不会再继续点B

具体案例:ScreenA->Dialog ScreenB -> ScreenC, ScreenC -> back -> ScreenA控制解决了 A->B->C，C->back->A，此时A会因for循环到下一个组件，但B此时还并没有点完 的问题。

2. 对于那些点过了，但是没有产生页面变化的组件，一个容错机会,点过组件无效可能只是因为屏幕挡住了

测试结果
钉钉未登陆
总共点击的activity个数 8
总共点击的Screen个数: 21
总共点击的组件个数: 136

飞书未登陆
总共点击的activity个数 4
总共点击的Screen个数: 6
总共点击的组件个数: 44

钉钉已登陆
总共点击的activity个数 8
总共点击的Screen个数: 18
总共点击的组件个数: 54


## V1.2.2
1. 修正判断有向图是否存在环的算法

飞书已登陆
总共点击的activity个数 15
总共点击的Screen个数: 33
总共点击的组件个数: 176
(问题出现在了飞书聊天框，点到了进行聊天的界面，聊天可发的内容很多，如表情等等。需要针对聊天这种情况特殊处理，已经针对谷歌自带的输入框和输入法作特殊处理，后面需要用更通用的方法)

钉钉已登陆
总共点击的activity个数 13
总共点击的Screen个数: 35
总共点击的组件个数: 201
(问题出现在了钉钉的拍摄视频, 还有上传头像，这里是钉钉内部自己的activity，需要特殊处理。已经对谷歌自带的拍照等等情况作处理，还需要通用处理)

此外，如果点到某个组件，突然弹出来权限申请(缺少某个权限需要用户授权)，这种情况程序会退出。

TODO:
1.存储已经点完的Screen
2.发现程序很容易因为循环过程中界面不一样。。退出程序，如何解决？
3.分析只预测一层和只检测一层环，如果多几层会什么影响
4.分析如果产生了多余的新界面会有什么影响 
5.分析ViewPager
6.基于ClickMap的去环, 而不是基于图
7.看看能不能根据view pager的特性做特殊处理, 
8.更强力的容错措施，如果发生了异常问题就重启app，然后记录上次遍历过的组件和Screen，不点点过了的出现问题的组件


## V1.3.0
1. FSM

钉钉未登陆
总共点击的activity个数 8
总共点击的Screen个数: 15
总共点击的组件个数: 102

TODO:
- [x] 处理已经点完所有组件的Screen
- [x] 发现程序很容易因为循环过程中界面不一样。。退出程序，如何解决？
用FSM来解决，只要不出现意外界面，不需要控制程序与UI不一致的问题

3.分析只预测一层和只检测一层环，如果多几层会什么影响
4.分析如果产生了多余的新界面会有什么影响 
5.分析ViewPager
6.基于ClickMap的去环, 而不是基于图
7.看看能不能根据view pager的特性做特殊处理, 
8.更强力的容错措施，如果发生了异常问题就重启app，然后记录上次遍历过的组件和Screen，不点点过了的出现问题的组件
9.像界面A点击了一个组件出现了小框(界面B)，但是界面B包含了界面A的组件，然而此时在B的界面点A的组件点了没用，这种情况如何处理。
10.界面存在大量组件，但是很多组件点了没用，而且肉眼不可见，但是在ui automator2的ui tree上能获取到

## V1.3.1
### 更新
1. 重写了check_cycle和is_all_children_finish, 用call_map来处理, 避免死循环问题。注意:即使存在回边，该边也会加入到children_node中，但是不会加入到call_map中.
2. 增加了click_ele的最大点击次数，如果点了大于4次，就不再点该组件
3. 增加了差分来检测框,差分暂时失败，主要是get_screen_info获取的text和差分之后的text不一致
4. 遇到了意外不可回退的框，会触发随机点机制
钉钉未登陆
总共点击的activity个数 8
总共点击的Screen个数: 15
总共点击的组件个数: 102

### TODO:
- [x] 处理已经点完所有组件的Screen
- [x] 增加了click_ele的最大点击次数，如果点了大于4次，就不再点该组件。主要解决的问题是出现意外的环？
- [x] 发现程序很容易因为循环过程中界面不一样。。退出程序，如何解决？用FSM来解决，只要不出现意外界面，不需要控制程序与UI不一致的问题
- [x] call_map和children，对于已经存在的界面也需要增加
- [x] 需要有容错检测机制，如果发现一直处于状态3，此时可能出现了死框，即框的所有组件都已经点击完毕，但是该框不能通过back回退，回退无用，此时会卡住。因此随机点一个遇到了意外不可回退的框，会触发随机点机制。随机点是否会产生问题，比如图的状态不对。这里需要维护图的状态，比如last screen node和cur screen node的关系
- [x] 差分暂时失败，主要是get_screen_info获取的text和差分之后的text不一致
- [x] 基于callmap的去环, 而不是基于child node

0.如何提出更好的差分策略
2.call_map需要有更新机制，比如意外出现了广告或者权限界面，这个时候call_map会把该界面加进来，但是可能并不是call_map的下一个
3.分析只预测一层和只检测一层环，如果多几层会什么影响
4.分析如果产生了多余的新界面会有什么影响 
5.分析ViewPager
7.看看能不能根据view pager的特性做特殊处理, 
8.更强力的容错措施，如果发生了异常问题就重启app，然后记录上次遍历过的组件和Screen，不点点过了的出现问题的组件
9.像界面A点击了一个组件出现了小框(界面B)，但是界面B包含了界面A的组件，然而此时在B的界面点A的组件点了没用，这种情况如何处理。
10.界面存在大量组件，但是很多组件点了没用，而且肉眼不可见，但是在ui automator2的ui tree上能获取到

12.遇到了意外的界面，人为关闭还是调用cv关闭，意外的界面如权限，广告, 意外界面的包名可能是app的，也可能是系统的，也可能是广告？？
13.建立call_map刷新机制？有些意外的界面并不是点了某个组件就会触发，这个时候可能会死循环

### Bug情况:
#### Bug1: 
该界面A所有组件已经点击完成，因此会回退press_back
![[IMG_20230505_124117.jpg|300]]
回退会遇到弹框界面B，此时弹框界面B也已经点击完成所有组件，因此会触发回退press_back。然后又回到界面A，造成死循环。
![[IMG_20230505_124121.jpg|300]]

![[Pasted image 20230504144010.png]]

![[Pasted image 20230504144245.png]]

情况:尝试随机点策略解决

## V1.3.2
### 更新
1. 写了检测算法检测环，尝试解决## V1.3.1中存在的bug1

### Bug情况:
#### Bug1: 
```
'NoneType' object has no attribute 'is_screen_clickable_finished'
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
总共点击的activity个数 23
总共点击的Screen个数: 71
总共点击的组件个数: 306
时间为 2326.8734159469604
end
```

解决情况:怀疑是get_state和do_transition，都会调用get_screen_info(d)，get_screennode_from_screenmap，产生的结果可能不一致。需要用同一个node和screen_text

### Bug2:
```
--------------------------------------------------
状态为3 当前Screen已经存在
****************************************************************************************************
'NoneType' object has no attribute 'all_text'
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
总共点击的activity个数 13
总共点击的Screen个数: 35
总共点击的组件个数: 138
时间为 934.8038568496704
end

```

解决情况:怀疑是get_state和do_transition，都会调用get_screen_info(d)，get_screennode_from_screenmap，产生的结果可能不一致。需要用同一个node和screen_text

### Bug3:
```
状态为4 当前Screen不存在
该screen为新: 编辑好友名片 --总共4, 减少0
正常点击组件&0: com.alibaba.lightapp.runtime.ariver.TheOneActivity1Swipe-com.alibaba.android.rimet-android.widget.TextView-com.alibaba.android.rimet:id/title_bar_name-(209,132)-编辑好友名片

状态为4 当前Screen不存在
'NoneType' object has no attribute 'clickable_elements'
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
总共点击的activity个数 26
总共点击的Screen个数: 92
总共点击的组件个数: 397
时间为 2855.8843920230865
end
```

解决情况:怀疑是get_state和do_transition，都会调用get_screen_info(d)，get_screennode_from_screenmap，产生的结果可能不一致。需要用同一个node和screen_text

## V1.3.2
### 更新
1. 在get_state和do_transition之间传递context(dict)，context记录了cur_screen_info和cur_screen_node;  编写get_screen_info_from_context方法和get_cur_screen_node_from_context方法

### Bug1:

```状态为6 可能出现不可回退的框
状态为3 当前Screen已经存在
****************************************************************************************************
该screen已存在: 搜索 综合联系人群组功能外部联系人已选择： 发送 --总共10, 减少0
****************************************************************************************************
产生回边
已点击过&0: com.alibaba.android.dingtalkim.activities.MsgForwardActivity-com.alibaba.android.rimet-android.widget.ImageButton--(77,132)-
已点击过&1: com.alibaba.android.dingtalkim.activities.MsgForwardActivity-com.alibaba.android.rimet-android.widget.EditText-android:id/search_src_text-(644,132)-搜索
已点击过&2: com.alibaba.android.dingtalkim.activities.MsgForwardActivity-com.alibaba.android.rimet-android.widget.RelativeLayout--(108,258)-综合
已点击过&3: com.alibaba.android.dingtalkim.activities.MsgForwardActivity-com.alibaba.android.rimet-android.widget.RelativeLayout--(324,258)-联系人
已点击过&4: com.alibaba.android.dingtalkim.activities.MsgForwardActivity-com.alibaba.android.rimet-android.widget.RelativeLayout--(540,258)-群组
已点击过&5: com.alibaba.android.dingtalkim.activities.MsgForwardActivity-com.alibaba.android.rimet-android.widget.RelativeLayout--(756,258)-功能
已点击过&6: com.alibaba.android.dingtalkim.activities.MsgForwardActivity-com.alibaba.android.rimet-android.widget.RelativeLayout--(972,258)-外部联系人
已点击过&7: com.alibaba.android.dingtalkim.activities.MsgForwardActivity-com.alibaba.android.rimet-android.widget.TextView-com.alibaba.android.rimet:id/tv_selection_status-(110,1959)-已选择：
已点击过&8: com.alibaba.android.dingtalkim.activities.MsgForwardActivity-com.alibaba.android.rimet-android.widget.Button-com.alibaba.android.rimet:id/btn_finish_select-(931,1959)-发送
正常点击组件&9: com.alibaba.android.dingtalkim.activities.MsgForwardActivity-com.alibaba.android.rimet-android.widget.FrameLayout--(969,901)-1

****************************************************************************************************
该screen已存在: 搜索 综合联系人群组功能外部联系人已选择： 发送 --总共10, 减少0
****************************************************************************************************
可能产生了不可去掉的框
'Device' object is not subscriptable
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
总共点击的activity个数 13
总共点击的Screen个数: 33
总共点击的组件个数: 131
时间为 920.4691770076752
end
ERROR:root:'Device' object is not subscriptable
Traceback (most recent call last):
  File "/Users/feymanpaper/codeSpace/pyWorkSpace/privacy_policy/FSM.py", line 399, in <module>
    FSM()
  File "/Users/feymanpaper/codeSpace/pyWorkSpace/privacy_policy/FSM.py", line 345, in FSM
    do_transition(state, context)
  File "/Users/feymanpaper/codeSpace/pyWorkSpace/privacy_policy/FSM.py", line 272, in do_transition
    handle_special_screen(context)
  File "/Users/feymanpaper/codeSpace/pyWorkSpace/privacy_policy/FSM.py", line 24, in handle_special_screen
    random_click_one_ele(context)
  File "/Users/feymanpaper/codeSpace/pyWorkSpace/privacy_policy/FSM.py", line 98, in random_click_one_ele
    cur_screen_pkg_name, cur_activity, cur_screen_all_text, cur_screen_info = get_screen_info_from_context(d)
  File "/Users/feymanpaper/codeSpace/pyWorkSpace/privacy_policy/core_functions.py", line 153, in get_screen_info_from_context
    cur_screen_pkg_name = context["cur_screen_pkg_name"]
TypeError: 'Device' object is not subscriptable
```

情况：产生的原因可能是
1.函数参数传递错误，已经修复
2.该页面共有10个组件，点到了第10个组件，刚好触发了随机点策略，但并不应该触发。解决思路，将handle_exist_screen判断cur_screen_node.is_screen_clickable_finished()也作为单独一个状态, click_one_ele(context)作为另外一个状态。

## V1.3.2
### 更新
1. 将handle_exist_screen判断cur_screen_node.is_screen_clickable_finished()也作为单独一个状态4, click_one_ele(context)作为另外一个状态3
2. 增加了check_screen_list和check_state_list，防止这种情况: 该页面共有10个组件，点到了第10个组件，刚好触发了随机点策略，但并不应该触发。

```
状态为3 当前Screen已经存在
****************************************************************************************************
该screen已存在: 备注和标签把他推荐给朋友特别关注设置特别关注人的消息提醒，避免错过重要消息 前往设置 添加到桌面 添加好友 举报该用户 --总共10, 减少0
****************************************************************************************************
产生回边
已点击过&0: com.alibaba.android.user.profile.more.UserProfileMoreV2Activity-com.alibaba.android.rimet-android.widget.ImageButton--(77,132)-
clickmap--该界面点击完成&1: com.alibaba.android.user.profile.more.UserProfileMoreV2Activity-com.alibaba.android.rimet-android.view.ViewGroup-com.alibaba.android.rimet:id/profile_cell_alias-(540,297)-备注和标签
clickmap--该界面点击完成&2: com.alibaba.android.user.profile.more.UserProfileMoreV2Activity-com.alibaba.android.rimet-android.view.ViewGroup-com.alibaba.android.rimet:id/profile_cell_send_card-(540,473)-把他推荐给朋友
已点击过&3: com.alibaba.android.user.profile.more.UserProfileMoreV2Activity-com.alibaba.android.rimet-android.view.ViewGroup-com.alibaba.android.rimet:id/profile_cell_concern-(540,649)-特别关注
该组件点击次数过多不点了&4: com.alibaba.android.user.profile.more.UserProfileMoreV2Activity-com.alibaba.android.rimet-android.view.View-com.alibaba.android.rimet:id/sv_switch-(978,649)-
已点击过&5: com.alibaba.android.user.profile.more.UserProfileMoreV2Activity-com.alibaba.android.rimet-android.widget.TextView-com.alibaba.android.rimet:id/tv_concern_desc-(460,770)-设置特别关注人的消息提醒，避免错过重要消息 前往设置
正常点击组件&6: com.alibaba.android.user.profile.more.UserProfileMoreV2Activity-com.alibaba.android.rimet-android.widget.TextView-com.alibaba.android.rimet:id/tv_short_cut-(540,891)-添加到桌面
--------------------------------------------------
```

```
--------------------------------------------------
状态为10 错误
意外情况
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
总共点击的activity个数 24
总共点击的Screen个数: 91
总共点击的组件个数: 408
时间为 2705.0341277122498
end
ERROR:root:意外情况
Traceback (most recent call last):
  File "/Users/feymanpaper/codeSpace/pyWorkSpace/privacy_policy/FSM.py", line 413, in <module>
    FSM()
  File "/Users/feymanpaper/codeSpace/pyWorkSpace/privacy_policy/FSM.py", line 358, in FSM
    do_transition(state, content)
  File "/Users/feymanpaper/codeSpace/pyWorkSpace/privacy_policy/FSM.py", line 282, in do_transition
    raise Exception("意外情况")
Exception: 意外情况
```

## V1.3.3
### 更新
1. 更新了组件顺序，在添加点击组件时，如果检测到该组件含有 "设置", "我的",之类的隐私信息，会从list的头部insert加入，而不是append。以便于后续点击时优先点击权限相关的
2. 对系统权限弹框进行了处理
3. 对一些不必要点击的组件进行了限制

出现问题，比如ScreenA存在 "我的"组件，组件A点了之后。到ScreenB也存在 "我的组件", 但是这个时候ScreenB不会点 "我的"，因为ele_vis_map显示已经点过

### 测试隐私界面情况:
csdn：没有点到，因为有些界面的组件把他覆盖了，A盖在了B上面，然后A点完之后，B的组件在A被点到了

芒果TV闪退

优酷视频:
ui automator2异常

Keep:
两个界面需要下滑

Boss直聘:
界面需要下滑

大麦：可以点到
菜鸟:  可以点到
小红书: 可以点到

12306智行火车票:  没点到，需要修改策略,出现问题，比如ScreenA存在 "我的"组件，组件A点了之后。到ScreenB也存在 "我的组件", 但是这个时候ScreenB不会点 "我的"，因为ele_vis_map显示已经点过

小宇宙: 没点到，点到某个组件之后去了外部页面webview


## V1.3.4
### 更新
1. 增加了状态8，进行double press back
2. 增加了组件差分策略，检测一些带有覆盖情况的框
3. 处理了跳出了app外，不可点框的情况(会重复出现状态1)
4. 重写共享组件策略，从策略: 每个screen有自己的call_map, 多个Screen共享组件,ele_vis_map,  ele_uid_cnt_map，变为策略: 每个Screen有自己的单独的组件。且拥有单独的ele_vis_map,  ele_uid_cnt_map, call_map。
5. 修正了get_two_clickable_eles_diff的错误写法，之前写成了并集union,应该用 交集intersection

测试结果: 如果连续的出现clickable_eles_diff，由于get_two_clickable_eles_diff只会和当前的cur_eles和last_eles进行比较，因此如果last_eles已经是diff过的, 但是cur_eles和已经diff过的比较会产生不对的结果。

计划改正: 需要让cur_eles和last_eles比较，然后last_screen需要有另外一个属性存储单独的diff_list

测试结果:
点到了第三方界面之后，遇到框，无法回退，一直保持在状态1
```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
总共点击的activity个数 3
总共点击的Screen个数: 10
总共点击的组件个数: 28
时间为 173.03048396110535


--------------------------------------------------
状态为5 当前Screen不存在,新建Screen
****************************************************************************************************
该screen为新: 关于Cookies - 华为官网关于Cookies - 华为官网 CNHuawei-v4 搜索 简介 打开菜单 Close华为夏季全场景新品发布会HUAWEI 问界 M5 系列 华为高阶智能驾驶版 HUAWEI P60 系列 万　境　生　辉 HUAWEI nova 11 系列 敢 拍 敢 出 色 HUAWEI 问界 M5 系列 华为高阶智能驾驶版 HUAWEI P60 系列 万　境　生　辉 Huawei-v4  搜索  简介  打开菜单  Close 华为夏季全场景新品发布会预约观看 HUAWEI 问界 M5 系列 华为高阶智能驾驶版 HUAWEI P60 系列 万　境　生　辉 HUAWEI nova 11 系列 敢 拍 敢 出 色 HUAWEI 问界 M5 系列 华为高阶智能驾驶版 HUAWEI P60 系列 万　境　生　辉 HUAWEI 问界 M5 系列 华为高阶智能驾驶版 HUAWEI P60 系列 万　境　生　辉 HUAWEI nova 11 系列 敢 拍 敢 出 色 HUAWEI 问界 M5 系列 华为高阶智能驾驶版 HUAWEI P60 系列 万　境　生　辉 HUAWEI 问界 M5 系列 华为高阶智能驾驶版  HUAWEI P60 系列 万　境　生　辉  HUAWEI nova 11 系列 敢 拍 敢 出 色  HUAWEI 问界 M5 系列 华为高阶智能驾驶版  HUAWEI P60 系列 万　境　生　辉 --总共11, 减少8
****************************************************************************************************
省略组件&0: com.alibaba.lightapp.runtime.activity.CommonWebViewActivitySwipe-com.alibaba.android.rimet-android.widget.RelativeLayout-com.alibaba.android.rimet:id/toolbar-(540,160)-关于Cookies - 华为官网
省略组件&1: com.alibaba.lightapp.runtime.activity.CommonWebViewActivitySwipe-com.alibaba.android.rimet-android.widget.RelativeLayout-com.alibaba.android.rimet:id/back_layout-(77,160)-
已点击过&2: com.alibaba.lightapp.runtime.activity.CommonWebViewActivitySwipe-com.alibaba.android.rimet-android.widget.TextView-com.alibaba.android.rimet:id/title-(540,159)-关于Cookies - 华为官网
该组件点击次数过多不点了&3: com.alibaba.lightapp.runtime.activity.CommonWebViewActivitySwipe-com.alibaba.android.rimet-android.widget.RelativeLayout-com.alibaba.android.rimet:id/more_layout-(1003,160)-
正常点击组件&4: com.alibaba.lightapp.runtime.activity.CommonWebViewActivitySwipe-com.alibaba.android.rimet-android.view.View--(540,1244)-CNHuawei-v4 搜索 简介 打开菜单 Close华为夏季全场景新品发布会HUAWEI 问界 M5 系列 华为高阶智能驾驶版 HUAWEI P60 系列 万　境　生　辉 HUAWEI nova 11 系列 敢 拍 敢 出 色 HUAWEI 问界 M5 系列 华为高阶智能驾驶版 HUAWEI P60 系列 万　境　生　辉 
--------------------------------------------------

--------------------------------------------------
状态为1 不是当前要测试的app,即app跳出了测试的app
进行回退
--------------------------------------------------
```
## V1.3.5
### 更新
1. 增加了策略is_child_clickable，如果一个组件，其子节点组件可点，则选取子节点来点而不选取父节点
2. 给screen增加了diff_clickable_elements属性，如果该screen是被diff过的，该screen的clickable_elements存储所有可点组件，diff_clickable_elements存储diff组件；如果该screen没有被diff过，diff_clickable_elements为None。因此判断last_screen和cur_screen是否diff的时候需要比较clickable_elements，在真正点组件的时候用diff_clickable_elements来点。解决1.3.4存在的问题。
3. 增加了get_merged_clickable_elements的策略。 更改了is_same_two_clickable_eles的判断，增加了对横坐标或者纵坐标相同的判断。如果有连续k个横坐标相同且class, resourceid, package相同就merge；如果有连续k个纵坐标相同且class, resourceid, package相同就merge

测试发现：
芒果TV有些viewpager组件是可点的，但是ui tree上面的clickale=False

测试结果:
```
小宇宙:
KeyError: 100，表示点完了回到home screen
总共点击的activity个数 33
总共点击的Screen个数: 91
总共点击的组件个数: 459
时间为 2906.1110939979553
end
```

TODO:需要自己写一个app来测试，为什么点完了 "个人"，没有点到 “发现”和“订阅”的viewpager。是不是环检测算法出现了问题?

测试结果:
智行12306
```
状态为7 出现了不可回退的框, 需要重启?
重启机制???
总共点击的activity个数 10
总共点击的Screen个数: 87
总共点击的组件个数: 646
时间为 4176.956385135651
end
```

TODO: 原因是当前界面唯一标识符all_text = "", 为空。但是screen_map之前已经存在一个标识符为空的screen，因此拿错了screen。
需要修改screen的唯一标识符

## V1.3.6
### 更新
1. 增加了暂停程序时会打印已遍历的Screen和组件信息
2. 将screen的唯一标识符改成clickable_ele_text + clickable_ele_loc
3. 优化了while循环的起始点，不是从0开始而是根据cur_screen_node.already_clicked_cnt开始
4. 去掉了screen_info
5. 目前设定的阈值是，相似度为90%则认为是同一个界面，相似度为80%则认为出现了框

insight，Screen的元素文本内容可能会改变，但仍然是同一个Screen。Screen的可点击组件可能会增加，但仍然是同一个Screen。
对于框，增加了新的组件文本和组件个数，但是不属于同一个Screen

测试结果，对于12306智行，到了注册页面之后press_back退不出来了，需要有重启机制

测试结果钉钉
```
状态为7 出现了不可回退的框, 需要重启?
重启机制???
总共点击的activity个数 34
总共点击的Screen个数: 201
总共点击的组件个数: 1434
时间为 26745.786102056503
end
```
钉钉点击左上角设置-->点击个人信息查询-->调用press_back导致无法回退。暂时没找到不能回退的原因，自己测试和操作的时候是可以通过press_back回退的。

需要有重启机制

## V1.3.7(stable)
### 更新
1. 增加了重启机制
2. 增加了interrupt和重启时dump_screen_map_to_json保存Json文件的方式

钉钉测试结果 
```
--------------------------------------------------
状态为8 出现了不可回退的框, 启用double_press_back
--------------------------------------------------
------------------------------------------------
状态为4 当前Screen已经点完
进行回退
--------------------------------------------------
--------------------------------------------------
状态为4 当前Screen已经点完
进行回退
--------------------------------------------------
--------------------------------------------------
状态为4 当前Screen已经点完
进行回退
--------------------------------------------------
--------------------------------------------------
状态为8 出现了不可回退的框, 启用double_press_back
--------------------------------------------------
KeyboardInterrupt ...
do something after Interrupt ...
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
总共点击的activity个数 34
总共点击的Screen个数: 236
总共点击的组件个数: 1645
```
发现一直处于上述循环，无法退出，说明没有检测到，没有重启，需要修改检测机制

12306智行测试结果
没有点到隐私相关界面，原因是重启次数过多，导致了没有继续点我的(一个组件最多能点4次)
需要调整策略，或者减少重启次数

界面相似度等算法时间复杂度过高，几乎每次都要遍历screen_map，想想能不能优化？

## V1.3.8
### 更新
1. 对每次get_screennode_from_screenmap_by_similarity都增加了对screen_map的保存
2. 增加了get_max_similarity_screen_node，当相似度达到90%，认为是之前存在的界面；当相似度达到70%，认为是差分新界面。否则为全新界面。
3. 将随机点策略修改为从 cycle_set和call_map里面随机取出来点一个, cycle_set是记录点击后造成回边的组件，不包括自回边
4. 暂时取消了click_cnt > 4次就不点该组件的策略

测试智行12306

- [ ] 测试发现存在环，说明存在bug
- [ ] 遇到聊天/咨询界面会比较难出来(adhoc找Activity关键字匹配: WebView, Chat)，LCS算法计算时间过长，checkcycle运行时间过长，如何优化？
- [ ] 隐患：在normal的界面如果press_back没用的话，会不停随机点

## V1.3.9
### 更新
1. 增加了Memory类(单例设计模式)，用来存储LCS算法产生的界面相似度
2. adhoc地检查是否在Webview相关的Screen，字符串匹配
3. TODO:LCS算法过慢，需要优化

- [ ] 优化LCS算法, CPython?

测试结果: 在WebView相关的Screen中无法通过一次press back退出当前Screen


## V1.3.10
### 更新
1. TODO:LCS算法过慢，需要优化
2. 增加了处理back无法退出Webview界面的情况。
3. 将记录数据的全局变量total_eles_cnt ,stat_screen_set ,self.stat_activity_set ,start_time重构到了StatRecorder(单例模式)
4. 将配置信息的全局变量CLICK_MAX_CNT, sleep_time_sec, target_pkg_name重构到了Config类(单例模式)
5. 将State相关的变量state_map重构到了StateHandler中
6. 将运行时需要保存的上下文全局变量screen_list，state_list， error_screen_list，error_clickable_ele_uid_list，screen_map，ele_uid_map重构到了RuntimeContent(单例模式)中

钉钉测试结果，意外自己关闭了，没测完
```
KeyboardInterrupt ...
do something after Interrupt ...
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
总共点击的activity个数 47
总共点击的Screen个数: 274
总共点击的组件个数: 1024
时间为 6611.01750087738
```

## V1.3.11(Stable)
### 更新
1. 删除first_activity, first_screen_text等全局变量
2. 将last_clickable_ele_uid , last_screen_node全局变量重构到RuntimeContent

```
do something after Interrupt ...
总共点击的activity个数 62
总共点击的Screen个数: 250
总共点击的组件个数: 764
时间为 3276.629905939102
```

```
测试钉钉，
重启过，然后一直位于状态4，进行回退
do something after Interrupt ...
总共点击的activity个数 63
总共点击的Screen个数: 535
总共点击的组件个数: 2022
时间为 40054.40872001648
```

bug: 没有对check_errorscreen一次press_back退不出来的情况进行检测, 需要增加二次press_back和重启机制

- [ ] 隐患TODO: 当前只在random_click_one_ele, click_one_ele会更新last_screen_node, 对于error_screen和webview_screen等会press_back的界面，是不是加到图的边的。而且不知道是否会产生未知的隐患问题
- [ ] 音视频检测

## V1.3.12
### 更新
1. 将utils.py的函数搬移到core_functions.py
2. 编写build_hierarchy， print_ui_root， get_clickable_eles_tree函数。提取当前UI界面的树结构

智行12306测试结果
包名
TODO: 目前发现，智行12306遇到了某个Screen，这个Screen会在短时间内发生较大的界面变化，但从人眼看是同一个界面，然后程序会不停认为是创建了新界面，导致卡住。思考如何解决这种情况。
ctrip.android.view.h5v2.ZTH5Containe
```
该screen为新: &关闭 87 154&分享 992 154--尾号1027累计获得 313 149--尾号6682累计获得 313 210--尾号0407累计获得 313 273--尾号6733累计获得 313 337--尾号9787累计获得 313 402--尾号0187累计获得 313 463--尾号3344累计获得 313 526--尾号9122累计获得 313 590--尾号1127累计获得 313 655--尾号0897累计获得 313 716--尾号3874累计获得 313 779--尾号6912累计获得 313 843--尾号9939累计获得 313 908--尾号3354累计获得 313 969--尾号8074累计获得 313 1032--尾号9127累计获得 313 1096--尾号1343累计获得 313 1161--尾号5476累计获得 313 1222--尾号7315累计获得 313 1285--尾号4022累计获得 313 1349--尾号1027累计获得 313 1414--尾号6682累计获得 313 1475&尾号6682累计获得 313 210&尾号0407累计获得 313 273&尾号6733累计获得 313 337&尾号9787累计获得 313 402&尾号0187累计获得 313 463&尾号3344累计获得 313 526&尾号9122累计获得 313 590&尾号1127累计获得 313 655&尾号0897累计获得 313 716&尾号3874累计获得 313 779&尾号6912累计获得 313 843&尾号9939累计获得 313 908&尾号3354累计获得 313 969&尾号8074累计获得 313 1032&尾号9127累计获得 313 1096&尾号1343累计获得 313 1161&尾号5476累计获得 313 1222&尾号7315累计获得 313 1285&尾号4022累计获得 313 1349&尾号1027累计获得 313 1414--打卡提醒已开启 540 1024--txt2 540 1165--当前参与人数：578 540 1200--次日打卡成功，只赚不 540 1239--当前参与人数：578 540 1277--次日打卡成功，只赚不 540 1316--当前参与人数：578 540 1200--次日打卡成功，只赚不 540 1239--当前参与人数：578 540 1277--次日打卡成功，只赚不 540 1316&次日打卡成功，只赚不 540 1239&当前参与人数：578 540 127--总共7, 减少0

该screen为新: &关闭 87 154&分享 992 154--尾号1808累计获得 313 141--尾号8654累计获得 313 206--尾号2189累计获得 313 270--尾号0414累计获得 313 331--尾号1822累计获得 313 394--尾号9894累计获得 313 459--尾号2461累计获得 313 523--尾号8864累计获得 313 584--尾号2267累计获得 313 647--尾号2624累计获得 313 712--尾号2504累计获得 313 776--尾号2706累计获得 313 837--尾号8833累计获得 313 900--尾号1377累计获得 313 965--尾号6894累计获得 313 1029--尾号8963累计获得 313 1090--尾号3956累计获得 313 1153--尾号8877累计获得 313 1218--尾号2928累计获得 313 1282--尾号8758累计获得 313 1343--尾号1808累计获得 313 1406--尾号8654累计获得 313 1471&尾号8654累计获得 313 206&尾号2189累计获得 313 270&尾号0414累计获得 313 331&尾号1822累计获得 313 394&尾号9894累计获得 313 459&尾号2461累计获得 313 523&尾号8864累计获得 313 584&尾号2267累计获得 313 647&尾号2624累计获得 313 712&尾号2504累计获得 313 776&尾号2706累计获得 313 837&尾号8833累计获得 313 900&尾号1377累计获得 313 965&尾号6894累计获得 313 1029&尾号8963累计获得 313 1090&尾号3956累计获得 313 1153&尾号8877累计获得 313 1218&尾号2928累计获得 313 1282&尾号8758累计获得 313 1343&尾号1808累计获得 313 1406--打卡提醒已开启 540 1024--txt2 540 1165--当前参与人数：576 540 1200--次日打卡成功，只赚不 540 1239--当前参与人数：576 540 1277--次日打卡成功，只赚不 540 1316--当前参与人数：576 540 1200--次日打卡成功，只赚不 540 1239--当前参与人数：576 540 1277--次日打卡成功，只赚不 540 1316&次日打卡成功，只赚不 540 1239&当前参与人数：576 540 127--总共7, 减少0
```

钉钉测试结果：

```
--------------------------------------------------
状态为12 当前Screen为error不该点
进行回退
--------------------------------------------------


--------------------------------------------------
状态为4 当前Screen已经点完
进行回退
--------------------------------------------------


--------------------------------------------------
状态为12 当前Screen为error不该点
进行回退
--------------------------------------------------


--------------------------------------------------
状态为4 当前Screen已经点完
进行回退
--------------------------------------------------
```

钉钉测试结果:
手机关机了
```
总共点击的activity个数 60
总共点击的Screen个数: 599
总共点击的组件个数: 2398
时间为 33469.06203389168
```


TODO：可能要改界面元素的策略，比如一个可点元素肯定是叶子结点。如果父亲节点可点，叶子结点不可点，要传递给叶子结点

Update: 现在感觉不想改了

## V1.3.13
### 更新
1. 将FSM大量函数搬移到StateHandler中
2. TODO发现组件元素可能有重复的，甚至是横坐标纵坐标重复，需要处理
3. insight: UI Tree无法唯一标识，因为UI Tree结构也会变化
4. TODO:更改日志结构
5. TODO:调整get_screen_info，将在一开始就直接处理组件和界面信息
6. insight: 可以搜索ViewTree里面有没有Webview类

```
智行12306 App界面
0--ctrip.android.view.h5v2.ZTH5Container-com.yipiao-android.widget.TextView--(87,154)-关闭
1--ctrip.android.view.h5v2.ZTH5Container-com.yipiao-android.widget.TextView--(992,154)-分享
7--ctrip.android.view.h5v2.ZTH5Container-com.yipiao-android.widget.TextView--(87,154)-关闭
8--ctrip.android.view.h5v2.ZTH5Container-com.yipiao-android.widget.TextView--(992,154)-分享

19--ctrip.android.view.h5v2.ZTH5Container-com.yipiao-android.widget.TextView--(540,1239)-次日打卡成功，只赚不赔
20--ctrip.android.view.h5v2.ZTH5Container-com.yipiao-android.widget.TextView--(540,1277)-当前参与人数：2177

33--ctrip.android.view.h5v2.ZTH5Container-com.yipiao-android.widget.TextView--(540,1200)-次日打卡成功，只赚不赔
34--ctrip.android.view.h5v2.ZTH5Container-com.yipiao-android.widget.TextView--(540,1239)-当前参与人数：2177
```

## V1.3.13
### 更新
1. 将FSM单独成为一个类，run.py为运行开始的文件
2. 增加了README.md

## V1.3.14
### 更新
1. 更改日志结构，输出screen_text(包括可点击组件的文本和不可点击组件的文本)
2. 增加了搜索ViewTree里面有没有Webview类来判断当前界面是不是Webview

## V1.3.15
### 更新
1. 发现组件元素可能有重复的，进行uid去重
2. TODO横坐标纵坐标重复，需要处理
3. 调整get_screen_info，将在一开始就直接处理组件和界面信息

```
&您提供的姓名、手机 540 1250&没有账号？极速注册仅 540 1159& 69 162&反馈建议 953 162&1529575028 540 524&输入12306密码 492 689& 942 689&阅读同意 170 826&账户授权协议 379 827&订单授权协议 631 827&找回密码 940 827&登录12306 540 989& 540 606&二代身份证 540 1294&港澳台居民居住证 540 1459&外国人永久居留身份证 540 1624&港澳居民来往内地通行 540 1789&台湾居民来往大陆通行 540 1954&护照 540 2136& 66 162&注册12306账户 540 162&极速注册推荐 270 291&普通注册 810 291&二代身份证 650 442&证件号码 154 581&请输入身份证号码 673 581& 179 723&请输入您的真实姓名 673 722&手机号码 154 863&(+86) 317 859&请填写您的手机号码 732 863&下一步 540 1064

&您提供的姓名、手机 540 1250&没有账号？极速注册仅 540 1159& 69 162&反馈建议 953 162&1529575028 540 524&输入12306密码 492 689& 942 689&阅读同意 170 826&账户授权协议 379 827&订单授权协议 631 827&找回密码 940 827&登录12306 540 989& 66 162&注册12306账户 540 162&极速注册推荐 270 291&普通注册 810 291&二代身份证 650 442&证件号码 154 581&请输入身份证号码 673 581& 179 723&请输入您的真实姓名 673 722&手机号码 154 863&(+86) 317 859&请填写您的手机号码 732 863&下一步 540 1064
```

## V1.3.16
### 更新
1. 分离了部分get_state代码
2. 处理了ExitApp状态，可能会回退不回来，此时需要进行重启
3. 添加了日志，重写print函数

## V1.3.17
### 更新
1. 暂存，将运行的状态对象RuntimeContent序列化，以便下次运行能接上去
2. 将Json和SavedInstance放到Utils文件夹
3. 将call_map添加到打印dumpjson


测试钉钉，效果如下: 
最终到第一个界面认为点击完成退出了
```
状态为完成
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
总共点击的activity个数 36
总共点击的Screen个数: 444
总共点击的组件个数: 1537
时间为 8007.48105764389
```
4. 修正重启机制检测的bug

钉钉测试结果:
不断重启，因为检测到first_screen为error_screen
```
状态为STATE_Restart
需要重启
总共点击的activity个数 71
总共点击的Screen个数: 303
总共点击的组件个数: 781
时间为 4186.135831832886
```

## V1.4.0
### 更新
1. 修正重启机制检测的bug
2. 将restart分成stuck_restart和home_screen_restart来区分是因为卡住导致的重启还是意外跳到home_screen导致的重启
3. 在程序开始，以及重启的时候将last_screen_node设置为root, 然后当last_screen_node为root时，就设置first_screen_node，来防止first_screen_node产生多变化的情况。
4. 增加了逻辑: press_back之后会把last_screen_node设置为None，防止错位
5. 增加了逻辑: press_back之前把last_screen_ck_ele_uid设置为“”， 防止错位 
6. 增加了逻辑，为handle_stuck_restart， handle_exit_app，handle_WebView_screen增加了add_not_target_pkg_name_screen_call_graph，将其加入到call_graph中，并且让stuck_restart时能够找到产生问题的last_ck_ele_uid

1. TODO: 目前存在问题，如果first_screen_text发生了变化就有问题了。
2. TODO：隐患，last_screen_node, last_clickable_ele可能导致错位
3. TODO:  隐患，如果最开始的界面是广告之类，当最开始的界面点完后，第一次back触发重启，此时会认为是State_Terminate
4. TODO: 智行不断变化的情况
5. TODO: 处理WebView, 目前是只看一层,  后面采用计时或者其他方法

钉钉测试结果:
"我的"按钮对应的screen已经全部点完，但是每次重启之后，在主界面并不知道"我的"按钮对应的screen已经全部点完，因此每一次重启都点"我的"，进入到"我的"界面之后，由于已经全部点完，因此会press_back到home_screen导致继续重启。

产生了不一致的问题，ScreenA的call_map: key为"我的", value为"我的"界面ScreenB，认为是没有点完的。但是跳转到了"我的"界面时，从screen_map找到的是ScreenC，ScreenC有diff_list，且已经点完。
```
状态为STATE_HomeScreenRestart
需要重启
总共点击的activity个数 39
总共点击的Screen个数: 107
总共点击的组件个数: 231
时间为 1187.5243270397186
```

12. 将改变更新call_map的函数从click_one_ele增加到add_exist_screen_call_graph和add_new_screen_call_graph，防止call_map在handle_exist_screen的时候没有更新
13. 隐患TODO: 一些第三方的界面是没有screen_map的 
14. 将LogUtils从单例模式改成静态类

钉钉测试结果: 
```
状态为STATE_Terminate
总共点击的activity个数 82
总共点击的Screen个数: 274
总共点击的组件个数: 748
时间为 3801.660382270813
```

## V1.4.1
### 更新
智行12306测试结果
由于一直卡在权限框
```
KeyboardInterrupt ...
do something after Interrupt ...
总共点击的activity个数 10
总共点击的Screen个数: 355
总共点击的组件个数: 1178
时间为 7029.756446123123
```

1. random_click_one_ele机制分成了random_click_ele和random_click_backpath_ele，前者是任意点击一个组件，后者是任意点击一个有call_map或者cycle_set 的组件.
2. 弃用is_cur_callmap_finish，从循环call_map比较target_screen_text改成直接判断next_screen_node是否is_screen_clickable_finished

智行12306测试结果
ScreenA，状态为STATE_ExistScreen点击某个组件之后到ScreenB，状态为STATE_FinishScreen
一直循环，没有跳出来
ScreenNode的is_cur_callmap_finish可能有问题
将条件改成if next_screen_node.is_screen_clickable_finished():
```
KeyboardInterrupt ...
do something after Interrupt ...
总共点击的activity个数 9
总共点击的Screen个数: 246
总共点击的组件个数: 948
时间为 5067.15108704567
```

智行12306测试结果
耗费了大量时间在买票
```
总共点击的activity个数 20
总共点击的Screen个数: 560
总共点击的组件个数: 1821
时间为 11867.60411310196
```

```
总共点击的activity个数 25
总共点击的Screen个数: 880
总共点击的组件个数: 2709
时间为 20310.44722509384
```

小宇宙测试结果
```
总共点击的activity个数 29
总共点击的Screen个数: 70
总共点击的组件个数: 222
时间为 1079.2367928028107
```

```
总共点击的activity个数 35
总共点击的Screen个数: 110
总共点击的组件个数: 327
时间为 1555.650595664978
```

```
总共点击的activity个数 41
总共点击的Screen个数: 248
总共点击的组件个数: 576
时间为 2844.7585678100586
```

抖音测试结果:
```
总共点击的activity个数 46
总共点击的Screen个数: 197
总共点击的组件个数: 459
时间为 2532.252564907074
```

测试大麦发现clickable的组件和有文本的组件是兄弟关系，而clickable组件本身没有text，因此考虑如果组件本身及组件孩子没有text，可以考虑增加兄弟节点的文本

## V1.4.2
### 更新
1. 测试发现小红书有组件本身及孩子都没有文本，因此增加了clickable_ele.get("content-desc")的文本 
2. 增加了定时任务，增加了计算WebView个数
3. 增加了PrivacyUtils静态计算隐私

TODO测试大麦发现clickable的组件和有文本的组件是兄弟关系，而clickable组件本身没有text，因此考虑如果组件本身及组件孩子没有text，可以考虑增加兄弟节点的文本

TODO小红书有个界面非常多可点组件，横坐标一部分相同，纵坐标一部分相同，矩形
对此类进行合并。而且需要考虑可以让screen_text为合并前的, 但真正点击的时候为合并后的

B站测试结果
```
总共点击的activity个数 13
总共点击的Screen个数: 101
总共点击的组件个数: 157
时间为 692.0576519966125
```
TODO B站和其他的app分享到微博之后，back回来会遇到弹框，因此回退不回来

中国大学MOOC测试结果
```
总共点击的activity个数 16
总共点击的Screen个数: 74
总共点击的组件个数: 162
时间为 1028.769774198532
```

```
总共点击的activity个数 19
总共点击的Screen个数: 91
总共点击的组件个数: 196
时间为 1218.5678391456604
```

后面的实验：
时间统一
WebView的次数
找佳颖的30个app的隐私数据项和一鸣的隐私词典进行匹配

钉钉一小时测试结果:
```
总共点击的activity个数 76
总共点击的Screen个数: 249
总共点击的组件个数: 694
总共触发的WebView个数: 41
时间为 3603.6882750988007

42
'BSSID', '导航', '习惯', '应用列表', '图像', '会话', '名单', '预约', '验证码', '内存', '预约', '声音', '文章', '相机', '图片', '成交', '点赞', '年龄', '版本', '定位', '访问量', '联网', '收款', 'SSID', '热点', '统计数据', '社区', '铃声', 'API', '接收人', '建立', '实名', '反馈', '下载', '城市', '显示', '输入', '文字', '天气', '简介', '备注', '职业'
```


菜鸟测试结果：
发现call_map会存在没更新的可能，比如某个Screen的按钮的call_map到达一个正常的Screen，因此存在call_map，后来这个按钮到达了Webview，就会死循环

4.为add_not_target_pkg_name_screen_call_graph函数也添加上更新call_map的逻辑。并且在click_one_ele添加判断了next_screen_node是否为非本app包名及是否为WebView

菜鸟一小时测试结果
```
总共点击的activity个数 31
总共点击的Screen个数: 270
总共点击的组件个数: 432
总共触发的WebView个数: 47
时间为 3614.3507680892944

70
'请求', '商品', '应用列表', '产品', '分享', '支付', '设置', '地图', '订阅', '问题', '预约', '榜单', '关注', '验证码', '头像', '短信', '预约', '昵称', '通讯录', '登录', '密码', '相机', '查询', '图片', '加购', '变更', '状态', '成交', '版本', '标识', '直播', '容量', '定位', '录制', '会员', '物流', '绑定', '历史', '收藏', '下单', '配送', '库存', '寄件', '收货', '手机号', '相册', '通讯', '学校', '点击', '付款', '专业', '儿童', '账号', '生日', '实名', '视频', '反馈', '浏览页面', '城市', '店铺', '共享', '授权', '认证', '剪切板', '交易', '备注', '搜索', '购物车', '订单', '联系人'
```


## V1.4.3
### 更新
%%1. 将ck_eles_text = to_string_ck_els(cur_ck_eles)放到最开始得到的cur_ck_eles，之后再对cur_ck_eles进行优化，防止优化之后的ck_eles_text变得更少。我们将比较每个Screen的所有ck_eles_text，但是真正点击的时候是采用优化后的%%
%%1. 有时候分享到微博时，跳到微博之后，back会遇到弹框，则无法回到app了。因此增加了STATE_OutsystemSpecialScreen%%
1. 对于跳转到第三方App，且无法通过back回到原来的app的情况，将把第三方app杀死，然后跳回到测试的目标App；因此增加了State_KillOtherApp和 handler_kill_other_app
2. 修复add_not_target_pkg_name_screen_call_graph的bug，否则在支付宝测试的时候WebView会陷入死循环

淘宝一个小时的测试结果:
```
总共点击的activity个数 32
总共点击的Screen个数: 238
总共点击的组件个数: 645
总共触发的WebView个数: 14
时间为 3601.6737501621246

17
['商品评价', '验证码', '名字', '聊天', '包裹', '寄件', '邮政编码', '儿童', '空调', '反馈', '输入', '地区', '兴趣爱好', '关键词', '文件', '联系人', '聊天记录']
```

口碑，由于"我的"界面是WebView，无法测试
飞猪旅行，由于“我的”界面是WebView，无法测试

优酷一个小时测试结果:
```
总共点击的activity个数 23
总共点击的Screen个数: 318
总共点击的组件个数: 475
总共触发的WebView个数: 14
时间为 3612.565263748169

21
['配置', '错误', '名单', '热度', '声音', '发票', '聊天', '会员等级', '钱包', '评分', '历史', '热点', '讨论', '肌肤', '学校', '角色', '音乐', '建立', '反馈', '影评', '联系人']
```

支付宝一个小时的测试结果:
```
总共点击的activity个数 64
总共点击的Screen个数: 314
总共点击的组件个数: 618
总共触发的WebView个数: 86
时间为 3602.6515901088715

82
'分享', '麦克风', '速度', '行程', '会话', '问题', '名单', '预约', 'SSL', '会员名', '榜单', '关注', '电话', '热度', '头像', '预约', '昵称', '剪贴板', '通讯录', '声音', '文章', '链接', '相机', '蓝牙', '查询', '图片', '报错', '变更', '录音', '语音', '车型', '车辆', '评论', '发票', '年龄', '发布', '直播', '定位', '聊天', '会员等级', '会员', '钱包', '评分', '历史', '口令', '体重', '通信', '收藏', '下单', '收入', '身高', '收货', '相册', '通讯', '点击', '音乐', '评价', '空调', '反馈', '点评', '下载', '城市', '地区', '场景', '店铺', '兴趣爱好', '名称', '授权', '认证', '剪切板', '简介', '开票', '语言', '营业执照', '订单', '里程', '健康', '联系人', '聊天记录', '行车', 
```

TODO：
飞猪旅行，跳到权限界面会不断触发随机点没有回来
阿里拍卖，会不断重启.产生UndefineDepth回退重启，然后进入到ExceedDepth，回退到UndefineDepth重启

计算深度时找map是用的`==`符号，而不是相似度比较，可能会有问题

