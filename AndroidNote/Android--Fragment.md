![[Fragment总结.png]]
## Fragment介绍
Fragment:是一部分内容构成的片段，体现在屏幕上是一块内容区域

- 在Fragment之前，我们通常把一个Activity作为一个页面。
-   ﻿但随着页面元素的增加以及场景的复杂，单不页面己经不能满足需要，在屏幕上通常要同时展示多个区域、多个页面内容，这些内容的切换通常是整体的。
-   ﻿因此，为了让屏幕展示更多内容，以及对这些内容统一管理，引入了Fragment。

-   ﻿﻿Fragment，就是将一块内容区域封装在一起，统一管理，构成一个Fragment
-   ﻿﻿Fragment是依附在Activity上而存在的。一个Activity中可以有多个Fragment, 各Fragment之间可以传递数据、互相切换。
-   ﻿﻿Fragment与Activity很相似，也有生命周期函数(onGreate、onPause、onDestroy等）
-   ﻿如下是一个Fragment从开始到结束的生命周期流程：  
    onAttach->onCreate->**onCreateView->onViewCreated**->onActivityCreated->onStart->onResume-> onPause->onStop->onDestroyView->onDestroy->onDetach

![[Fragment直观理解.png]]

## Fragment的静态创建
![[Fragment静态创建.png]]![[Fragment静态创建注意点.png]]
## Fragment的动态创建
![[fragment动态创建.png]]
![[fragment动态创建2.png]]
![[fragment动态创建注意.png]]
## Fragment的生命周期
![[fragment生命周期.png]]
Fragment静态创建的生命周期
Fragment先创建，Activity再创建
最后Fragment先销毁，Activity再销毁
![[静态创建生命周期.png]]
Fragment静态创建销毁的生命周期
![[静态销毁生命周期.png]]
Fragment动态创建生命周期
Activity先创建，Fragment再创建
最后Fragment先销毁，Activity再销毁
![[fragment动态创建生命周期.png]]
fragment动态创建销毁生命周期
![[fragment动态创建销毁生命周期.png]]

## Fragment增删改查
### 添加-add
![[fragment添加的方式1.png]]
![[fragment添加2.png]]
Fragment添加机制，最好不要用add方式
![[fragment添加机制.png]]
命令行查看当前存在的Fragment
```
adb shell dumpsys activity fragment
```
![[命令行查看当前存在的fragment.png]]
addToBackStack("...")这里输入的参数表示的是添加记录的名字而不是栈的名字
add会在界面中一层层添加上去
开发者可以自行决定让不让fragment入BackStack，如果当前activity全部fragment都入栈，那么back会一次弹出栈，最后栈中没有元素了就退出当前activity
如果当前activity有多个fragment，但是都没有入栈，那么此时back就会直接退出当前activity
![[Fragment_addToBackStack.png]]
### 查找-find和移除-remove
查找用id或者tag可以找到
![[fragment查找.png]]
没有BackStack的情况，依次移除当前容器最上面的一个Fragment
![[fragment移除.png]]
存在BackStack的情况，会从容器中移除，但是在BackStack仍然存在 ，需要按返回键才能彻底移除
![[fragment移除2.png]]
### 替换-replace
![[fragment的移除.png]]
![[fragment替换2.png]]

## Fragment传递数据
### Activity向Fragment传递数据
1.通过方法传递，构造方法，普通public方法（编程语言的基本性质，没有涉及到安卓层面）
2.通过Argument传递。Argument 本意是 “论点，论据”，这里就是一种引申意，好像是一种传递给Fragment的依据，数据。这是Android本身为我们提供的向Fragment传递数据的方式，可用来一次性传递复杂、大量数据，如setArguments
public setArguments(Bundle args) // 传递数据
public Bundle getArguments( // 接收数据
3.通过接口传递(被观察者是Activity, 观察者是Fragment)
![[fragment通过接口传递数据.png]]
### Fragment向Activity传递数据
1.通过getActivity
由于Activity是Fragment的承载者，每个Fragment都可以通过getActivity(方法获得承载它的Activity对象，从而调用这个Activity的方法向Activity传递数据
2.通过接口传递（被观察者是Fragment, 观察者是Activity）

### Fragment之间传递数据
![[Fragment之间传递数据.png]]
![[Fragment之间通过接口传递数据.png]]

## Fragment实现底部导航
### Fragment+普通的Button布局实现底部导航
### Fragment+ViewPager实现底部导航(可以滑动)
**ViewPager的用法**
```
数据+适配器+ViewPager
数据: List<Fragment>, List<View>
适配器: FragmentPagerAdapter
```
FragmentPagerAdapter用法要重写两个方法
![[FragmentPagerAdapter用法.png]]

#### ViewPager+Fragment+普通BottomView实现底部导航页
	ViewPager的用法
	﻿﻿FragmentPagerAdapter的使用
	﻿﻿ViewPager切换页面与底部导航按钮的联动

最方便！
#### ViewPager+ Fragmentt BottomNavigationView 实现底部导航页
	ViewPager的用法
	﻿﻿FragmentPagerAdapter的使用
	﻿﻿﻿BottomNavigationView的使用
	﻿﻿﻿ViewPager切换页面与BottomNavigatiofiview的联动
![[bottomNavigatonView2.png]]
![[ViewPager与BottomNavigationView联动.png|600]]
![[ViewPagerFragme效果预览.png|250]]

ViewPager+Fragment+TabLayout+BottomNavigationView 实现底部导航页
	ViewPager的用法
	﻿﻿FragmentPagerAdapter的使用
	﻿﻿﻿﻿BottomNavigationView的使用
	﻿﻿﻿﻿ViewPager切换页面与BottomNavigationView的联动
	﻿﻿﻿﻿TabLayout的用法
	﻿﻿﻿﻿﻿嵌套Fragment (ChildFragment）的用法

![[ViewPager+Fragment+TabLayout+BottomNavigationView效果.png|250]]
#### TabLayout
![[TabLayout.png]]

#### Fragment+DrawerLayout+NavigationView实现侧滑菜单页面结构
	 ﻿Fragment的添加替换
	 ﻿﻿ ﻿﻿DrawerLayout的用法
	 ﻿﻿ ﻿﻿﻿﻿Navigation View
	 ﻿﻿ ﻿﻿﻿﻿两种菜单形式
		 ﻿﻿ ﻿﻿﻿﻿侧滑莱单在上
		 ﻿﻿ ﻿﻿﻿﻿侧滑菜单在下

![[DrawerLayout用法.png]]
![[NavigationView用法.png]]
