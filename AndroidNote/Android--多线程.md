#### 知识来源
https://www.bilibili.com/video/BV19U4y1G7mk/?spm_id_from=333.999.0.0&vd_source=108d23f95683578313bdaf5d938b5b3d

#### 基本概念
同步任务：在执行程序时，如果没有收到执行结果，就一直等，直到收到执行结果再接着往下执行

异步任务：在执行程序时，如果遇到需要等待的任务，就另外开辟一个子线程去执行，自己继续往下执行其他程序，子线程有结果时，再把结果发往主线程

App一启动，本身也是一个线程，也叫主线程main Thread，负责显示界面和跟用户交互。另外，界面通常被称为UI，因此，主线程也被称为UI线程。
- 主线程不能执行网络请求/文件读写等耗时操作（不然无法及时响应UI），主线程会开启一个子线程去做
- 子线程不能执行UI刷新，不然会报错Only the origin thread that created a view hierarchy can touch its views. 

主动开启一个子线程
```java
new Thread(new Runnable(){
	public void run(){
		//耗时操作 
	}
}).start();
```

#### 多线程通信-Handler机制

AsynTask已经淘汰了

主线程从一开始就建立了这么一套系统
Handler, Message, Looper, Message Queue
![[android_handler.png]]
![[Handler机制.png]]
主线程处理消息的逻辑
```java
private Handler mHandler = new Handler(Looper.myLooper()){
	public void HandleMessage(Message msg){
		super.handlerMessage(msg);
		if(msg.what == 0){
			String strData = (String) msg.obj;
			textView.setText(strData);
		}
	}
}
```
子线程发送消息的逻辑
```java
new Thread(new Runnable(){
	public void run(){
		//耗时操作 
		String res = "...";
		Message message = new Message();
		message.what = 0;
		message.obj = res;
		mHandler.sendMessage(message);
	}
}).start();
```
