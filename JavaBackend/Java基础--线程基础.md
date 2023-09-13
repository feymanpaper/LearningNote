### 基本概念
#### 程序
为完成特定任务，用某种语言编写的一组指令集合。简单地说就是我们写的代码

#### 进程
1.进程是运行中的程序，比如QQ，启动了一个进程，操作系统就会为该进程分配内存空间；正在运行的程序实例
2.进程是程序的一次执行过程，或者是正在运行的一个程序。是动态过程，有它自身的产生，存在和消亡的过程

#### 线程
1.线程是由进程创建的，是进程的一个实体
2.一个进程可以拥有多个线程
单线程，同一个时刻，只允许执行一个线程
多线程，同一个时刻，可以执行多个线程，比如：一个qq进程，可以同时打开多个聊天窗口，一个迅雷进程，可以同时下载多个文件 

#### 并发
同一个时刻，多个任务交替执行，造成一种貌似同时的错觉，简单的说，单核cpu实现的多任务是并发

#### 并行
同一个时刻，多个任务同时执行，多核cpu可以实现并行。并发和并行可以在一个电脑同时存在 

### 线程基本使用
#### 创建线程的两种方式

1.继承Thread类，重写run方法
2.实现Runnable接口，重写run方法
![[Thread和Runnable的类图关系.png]]

#### 1.继承Thread类，重写run方法
Practise1:开启一个线程，每隔1秒，输出"喵喵"
```java
public class Thread01{
	public static void main(String args[]){
		Cat cat = new Cat();
		cat.start();   //启动线程
		//说明：当main线程启动一个子线程Thread0，主线程不会阻塞，会继续执行
		//这时，主线程和子线程交替执行
		System.out.println("主线程继续执行");
	}
}

class Cat extends Thread{
	public void run(){
		//重写run方法，写上自己的业务逻辑
		while(true){
			System.out.println("喵喵");
			//打印线程名字
			System.out.println(Thread.currentThread().getName());
			try{
				Thread.sleep(1000);
			}catch.....
		}
	}
}
``` 
当我们打开IDE的run程序，此时会开启一个进程，该进程会先启动main线程，当我们在main线程start方法时，Thread0线程启动。若Thread0线程打印80次循环，main线程打印60次循环，那么main线程会先退出，Thread0执行完再退出，然后进程退出。**所以main线程未必是最后一个退出的**，所有线程都结束后，进程才退出。 
![[案例的进程分配进程过程.png]]
说明：
1.当一个类继承了Thread类，该类就可以当成线程使用
2.重写run方法，写自己的业务代码
3.run Thread类实现了Runnable接口的run方法

为什么上面是调用start方法来开启线程而不是run方法？
run方法就是一个普通的方法，没有真正的启动一个线程，main线程就会把run方法执行完毕才向下执行，串行执行。此时Thread.currentThread().getName()得到的是main而不是Thread0

读Thread类源码 
```
	1）
	public synchronized void start(){
		start0();
	} 
	2)
	//start0是本地方法，是JVM调用，底层是c/c++实现
	//真正实现多线程的效果是start0而不是run
	private native void start0();
```
start方法调用start0方法后，该线程并不一定立马执行，只是将线程变成了可运行状态，具体什么时候执行，取决于cpu，由cpu统一调度

#### 2.实现Runnable接口，重写run方法
practise2：该程序每隔一秒输出hi，当输出10次后，自动退出
```java
public class Thread02{
	public static void main(String[] args){
		Dog dog = new dog();
		//这里不能调用start
		//创建了Thread对象，把dog对象放入了Thread
		Thread thread = new Thread(dog);
		thread.start();
	}
}

class Dog implements Runnable{
	int count = 0;
	while(true){
		System.out.println("hi" + (++count) + Thread.currentThread.getName());
		try{
			Thread.sleep(1000);
		}catch...
		if(count = 10) break;
	}
}
```

为什么可以把dog放到Thread里？
底层使用了设计模式--代理模式
下列例子是最简单的一个Thread类模拟
```java
class Thread02{
	public static void main(String[] args){
		Tiger tiger = new Tiger();
		ThreadProxy threadProxy = new ThreadProxy(tiger);
		threadProxy.start();
	}
}
class Animal{}
class Tiger extends Animal implements Runnable{
	public void run(){
		System.out.println("老虎叫");
	}
}
//线程代理类吗，模拟了一个极简的Thread类 
class ThreadProxy implements Runnable{
	private Runnable target = null; //类型是Runnable
	public void run(){
		//动态绑定，运行类型为Tiger
		if(target != null) target.run();
	}
	public ThreadProxy(Runnable target){
		this.target = target;
	}
	public void start(){
		start0();  //这个方法可以真正实现多线程
	}
	public void start0(){
		run();
	}
}
```

practise3:创建两个线程吗，一个线程每隔一秒输出“hello world”，输出10次退出，一个线程每隔一秒输出“hi”，输出5次退出
```java
public class Thread03{
	public static void main(String[] args){
		new Thread(new T1()).start();
		new Thread(new T2()).start();
	}
}
class T1 implements Runnable{
	public void run(){
		int count = 0;
		while(true){
			System.out.println("hello world" + Thread.currentThread().getName());
			count++;
			if(count == 10) break;
		}
	}
}
class T2 implements Runnable{
	public void run(){
		...
	}
}
```

### 继承Thread和Runnable的区别
1.从java设计来看，通过继承Thread和实现Runnable接口来创建线程本质上没有区别，从jdk帮助文档我们可以看到Thread类本身就实现了Runnable接口
2.实现Runnable接口方式更加适合多个线程共享一个资源的情况，并且避免了单继承的限制
```java
T3 t3 = new T3("hello");
new Thread(t3).start();
new Thread(t3).start();
System.out.println("主线程完毕");
```

practise4:编程模拟三个售票窗口售票100，分别实现Thread和Runnable实现，并分析有什么问题
```java
public class SellTicket{
	public static void main(String[] args){
		//Thread方式 
		SellTicket01 sellTicket01 = new SellTicket01();
		SellTicket01 sellTicket02 = new SellTicket01();
		SellTicket01 sellTicket03 = new SellTicket01();
		//这里会出现超卖现象
		sellTicket01.start();
		sellTicket02.start();
		sellTicket03.start();

		//Runnable方式 
		SellTicket02 sellTicket02 = new SellTicket02();
		new Thread(sellticket02).start();
		new Thread(sellticket02).start();
		new Thread(sellticket02).start();
	}
}

//Thread方式
public SellTicket01 extends Thread{
	private static int ticketNum = 100; //让多个线程共享
	public void run(){
		while(true){
			if(ticketNum <= 0){
				System.out.println("售票结束...");
				break;
			}
			//模拟
			try{
				Thread.sleep(50);
			}catch....
			System.out.println("窗口" + Thread.currentThread().getName() + "售出一张" + " 剩余" + --ticketNum);
		}
	}
}

//Runnable方式 
class SellTicket02 implements Runnable{
	private static int ticketNum = 100; //让多个线程共享
	public void run(){
		while(true){
			if(ticketNum <= 0){
				System.out.println("售票结束...");
				break;
			}
			//模拟
			try{
				Thread.sleep(50);
			}catch....
			System.out.println("窗口" + Thread.currentThread().getName() + "售出一张" + " 剩余" + --ticketNum);
		}
	}
}
```
上面的两种方式都会产生超卖和重卖票的问题

### 线程终止
1.当线程完成任务后，会自动退出
2.可以使用变量来控制run方法退出的方式停止线程，即通知方式
```java
public class ThreadExit{
	public static void main(String[] args){
		T t1 = new T();
		t1.start();
		//让t1退出run方法，从而终止t1线程--->通知方式
		Thread.sleep(1000);
		t1.setLoop(false);
	}
}
class T extends Thread{
	private boolean loop = true;
	while(loop){
		System.out.println("run");
	}
}
```

### 线程常用方法
1.start底层会创建新的线程，调用run，run是一个简单的方法调用，不会启动新的线程
2.线程优先级的范围 MIN：1，NORM：5，MAX：10
3.interrupt，中断线程，但并没有真正地结束线程，一般用于中断正在休眠线程
4.sleep：线程静态方法，使当前线程休眠
```java
//interrupt会触发InterruptedException
try{
	Thread.sleep(200000);
}catch(InterruptedException e){
	System.out.println("被interrupt了");
}
```

yield：线程的礼让，让出cpu，让其他线程执行，但礼让的时间不确定，所以不一定礼让成功
join：线程的插队。插队的线程一旦插入成功，则肯定先执行完插入的线程所有任务

### 工作线程和守护线程
1.用户线程，也叫工作线程，当线程的任务执行完或通知方式结束
2.守护线程：一般为工作线程服务，当所有用户线程结束，守护线程自动结束。常见的例子：垃圾回收机制
```java
//如果希望当main线程结束后，子线程自动结束，可以将子线程设置成守护线程
public class ThreadMethod3{
	public static void main(String[] args){
		MyDaemonThread thread = new MyDaemonThread();
		thread.setDaemon(true);
		thread.sleep();
	}
	class MyDaemonThread{
		...
	}
}
```

### 线程的生命周期
JDK中用 Thread.State枚举了线程的六种状态
![[线程生命周期.png]]

### 线程同步
#### Synchronized
线程同步机制：
1.在多线程编程，一些敏感数据不允许被多个线程同时访问，此时就使用同步访问技术，保证数据在任何同一时刻，最多有一个线程访问，以保证数据的完整性
2.也可以这么理解，即当有一个线程在对内存进行操作时，其他线程都不可以对这个内存地址进行操作，直到该线程完成操作，其他线程才能对该内存地址操作

synchronized同步代码块
synchronized也可以放在方法声明中，表示整个方法为同步方法

```java
public class SellTicket{
	public static void main(String[] args){
		//Runnable方式 
		SellTicket02 sellTicket02 = new SellTicket02();
		new Thread(sellticket02).start();
		new Thread(sellticket02).start();
		new Thread(sellticket02).start();
	}
}

//Runnable方式 
class SellTicket02 implements Runnable{
	private static int ticketNum = 100; //让多个线程共享
	public void run(){
		while(true){
			//设置一个值让while循环退出
			sell();
		}
	}
	public synchronized void sell(){
			if(ticketNum <= 0){
				System.out.println("售票结束...");
				break;
			}
			//模拟
			try{
				Thread.sleep(50);
			}catch....
			System.out.println("窗口" + Thread.currentThread().getName() + "售出一张" + " 剩余" + --ticketNum);
		}
	}
}
```

#### 互斥锁
1.Java语言中，引入了对象互斥锁的概念，来保证共享数据操作的完整性
2.每个对象都对于一个可称为互斥锁的标记，这个标记来保证在任一时刻，只能有一个线程访问该对象
3.关键字synchronized来与对象互斥锁联系，当某个对象用synchronized修饰时，表明该对象在任一时刻只能由一个线程访问
4.同步局限性：导致程序的执行效率降低
5.同步方法（非静态）的锁可以是this（默认），也可以是其他对象
6.同步方法（静态）的锁为当前类本身（默认当前类.class）

实现步骤：先分析上锁的代码，选择同步代码块或者同步方法，**要求多个线程的锁对象为同一个**

#### 死锁
多个线程都占用了对方的锁资源，但不肯相让，导致了死锁，在编程是一定要避免死锁的发生
```java
public class DeadLock{
	public static void main(String[] args){
		DeadLockDemo A = new DeadLockDemo(true);
		DeadLockDemo B = new DeadLockDemo(false);
		A.start();
		B.start();
	}
}
class DeadLockDemo extends Thread{
	static Object o1 = new Object();
	static Object o2 = new Object();
	boolean flag;
	public DeadLockDemo(boolean flag) this.flag = flag;
	public void run(){
		if(flag){
			synchronized(o1){
				System.out.println(Thread.currentThread().getName())
				synchronized(o2){
					System.out.println(Thread.currnetThread...);
				}
			}
		}else{
			synchronized(o2){
				System.out.println(Thread.currentThread...);
				synchronized(o1){
					System.out.println(Thread.currentThread...);
				}
			}
		}
	}
}
```

#### 释放锁
1.当前线程的同步方法，同步代码块执行结束
2.当前线程在同步代码块，同步方法中遇到break，return
3.当前线程在同步代码块，同步方法中出现了未处理的Error或者Exception，导致异常结束
4.当前线程在同步代码块，同步方法中执行了线程对象的wait方法，当前线程暂停，并且**释放锁**

下面的操作不会释放锁
1.线程执行同步代码块或者同步方法时，程序调用Thread.sleep()，Thread.yield()方法暂停当前线程的执行，不会释放锁
2.线程执行同步代码块时，其他线程调用了该线程的suspend()方法将该线程挂起，该线程不会释放锁。应该尽量避免使用suspend和resume来控制线程