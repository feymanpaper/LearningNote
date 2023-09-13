## 项目遇到的问题
1.umap的uid并不唯一
2.点击了一个三个小点的图标，产生了个不跳转的小页面，不算跳转，但是按back会回退小页面；fragment的问题
3.假设一下场景，页面A跳转到了页面B，页面B有许多组件，当页面B点击某个组件回到了页面A后就无法再执行页面B的其他组件
4.需要设置点击前检测策略，比如当前准备点击的按钮是不是和返回相关。根据standard模式，当点击返回后当前的activity会被销毁，无法按back返回

1.目前获取当前页面的所有文本，是只获取可点击元素的所有文本，因为如果获取当前界面的所有文本，有些元素会变化。我观察到如果获取当前页面的所有文本作为辅助唯一标识一个界面的话，他的一些元素会产生微小的变化。比如一个页面的轮播图，还有飞书的聊天消息所记录的时间。 或者暂时不考虑一个页面动态的情况

2.对于dialog，打开dialog之前的文本和组件，打开dialog之后仍然会存在，然后增加了部分文本和可点击组件
menu和dialog类似

对于fragment，大概率打开之后界面元素和文本都会发生非常大的变化

有些dialog点开了之后，只增加了新的组件，背景的全部没了，这个问题很麻烦

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
dfs细化一些内容
```
def dfs(last_screen):
	cur_screen = 当前的screen
	
	#screen没有变化说明该组件不会造成页面跳转
	if cur_screen == last_screen: 
		return
	# 回边只针对activity,不针对menu, fragment, dialog
	if 产生回边:
		记录信息
		back
	# 否则screen产生了变化，造成了页面跳转(activity, fragment, dialog, menu...)
	if cur_screen没被记录过:
		将cur_screen加入到screen跳转图中
		在last_screen记录哪个组件能到达cur_screen
		在cur_screen记录相关信息 
	else cur_screen之前记录过了:
		#do nothing 
	
	clickable_elements = 当前页面所有可点击组件
	for ele in clickable_elements:
		if 该组件没被点击过 
			点击该组件
			记录该组件被点过
			dfs(cur_screen)
		else 该组件被点击过:
			if该组件不会产生回边:
				if该组件会跳转新的screen且新的screen有没点击完的组件(递归检测):
					点击该组件
					dfs(cur_screen)
```

## 元素定位
直接定位
```python
device = u2.connect()
device(text = "...").click()
```
![[u2元素定位方式.png]]

根据层级关系定位
```python
# 父级元素
my_item = device(resourceId = "...").child(resourceId = "...")
my_item.click()
# 同级元素
my_item2 = my_item.sibling(className = "...")
my_item2.click()
```

根据页面相对位置定位（相对慢）
```python
# left, right, up, down
my_item.left().click()
```

用户输入与清空 
```python
my_item.send_keys("1234")
time.sleep(3)
my_item.clear_text()
```

## 智能等待

wait存在超时时间，不会无休止地等待，可以自行设置超时时间，默认20s
```python
d = u2.connect("...")
d.wait_timeout = 30

d(resourceId = "...").click(timeout = 50)
```


