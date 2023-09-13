## APK描述
APKs are fairly simple: they're a ZIP file with a bunch of metadata, all the application's assets & config files, and one or more binary `.dex` files, which contain the compiled application.

XAPKs are more complicated: they're a zip file that contains multiple APKs. In practice, they'll contain one large primary APK, with the main application code & resources, and then various small APKs which include the config or resources only relevant to certain types of devices. There might be separate config APKs for devices with larger screens, or different CPU architectures. For reverse engineering you usually just need the main APK, and you can ignore the rest.

Dex: These DEX files contain all the JVM classes of the application, in the compiled bytecode format used by Android's Runtime engine (ART, which replaced Dalvik a few years back).

Hybrid-App:
https://mp.weixin.qq.com/s/x-mmH0g3Y0AaFDqmIDzdhQ

## 安卓四大组件
https://notes.sunofbeach.net/categories/?category=Android%E5%BC%80%E5%8F%91%E5%9F%BA%E7%A1%80
https://www.bilibili.com/video/BV1it411j7yN/?p=3&spm_id_from=pageDriver&vd_source=108d23f95683578313bdaf5d938b5b3d

### 1. Activity
#### 显式意图
显式意图，也就是可以看到我们的目标跳转组件，比如说上节课我们跳转到SecondActivity，我们startActivity(this,SecondActivity.class);

这里面的话，我们可以直接看到SecondActivity，也就是说可以看到具体的类名。这就是显式意图

intent的发起端 
```Java
Intent intent = new Intent(this, targetActivity.class);
intent.putExtra(key, value);
startActivity(intent);
```
intent的接收端
```Java
Intent intent = getIntent();
String account = intent.getStringExtra("account");
String password = intent.getStringExtra("password");
```

#### 隐式意图
隐式意图是相对于显式意图来说的，显式意图可以看得到对应的类，而隐式意图看不到。
步骤：
1.创建Intent对象
2.在AndroidManifest.xml里配置目标跳转Activity的意图过滤器
3.给这个intent对象设置Action，设置它的category值，如果5.1系统需要设置包名package name
4.startActivity 
```Java
Intent intent = new Intent();
intent.setAction("com.southbeaches.LOGIN_INFO");  //这句是在manifests.xml中接收端的Activity里面写的
intent.addCategory(Intent.CATEGORY.DEFAULT);
intent.putExtra(key, value);
startActivity(intent);
```
为什么会有两种意图来实现跳转？
显式意图一般用于应用内的组件跳转
隐式意图一般用于应用间的组件跳转，跳转第三方应用  

#### 传递对象 
Parcelable：可序列化，序列化到内存，效率高
Serializable: 序列化到SD卡，效率低

步骤：
先实现界面的跳转，创建对象（实现Parcelable接口)，把这个对象用putExtra的方式扔进去并且需要一个key，在目标界面，用key获取传递的对象
```Java
public class User implements Parcelable {
}
```
---
```Java
User user = new User("TrillGates", 25);
Intent intent = new Intent(this, SecondActivity.class);
        intent.putExtra("user", user);
startActivity(intent);
```
---
```Java
User user = intent.getParcelableExtra("user");
Log.d(TAG, "usr Name == " + user.getName());
Log.d(TAG, "usr age == " + user.getAge());
```

#### 协议的形式传数据
实现拨打电话功能，setData传递
下面这一句代码行指定了data的scheme
```xml
 <intent-filter>
    <action android:name="android.intent.action.CALL" />
    <category android:name="android.intent.category.DEFAULT" />
    <data android:scheme="tel" />
</intent-filter>

```

因此需要 
```Java
Intent intent = new Intent():
intent.addAction(“android.intent.action.CALL”);
intent.setCategory(“android.intent.category.DEFAULT”);
Uri uri = Uri.parse("tel:10086");
//intent.setData(Uri.parse(“tel://10086”));
intent.setData(uri);
```

#### 数据回传
场景：点击一个充值按钮，然后跳转到第二个界面，第二个界面进行充值，充值完成之后，告诉第一个界面结果，包括充值成功，充值失败
步骤：
1.要注意过程，不是用startActivity，而是用startActivityForResult；第一个参数intent。 这个方法有两个重载的方法，一个是两个参数，一个是三个参数。 第二个参数是requestCode，其实就是暗号！ 第三个参数是设置Activity是如何启动的，在ActivityOptions这个类里面，就是封装Activity的启动参数的。这个Bundle可以理解为一个可以封装多种集合
2.要复写onActivityResult方法，这里会有返回值
3.在第二个Activity，setResult设置resultCode, 在intent中putExtra，然后调用finish方法
4.第一个Activity不可finished，否则就接收不到结果了。
![[request_code.png]]
![[result_code.png]]

#### Activity生命周期
https://developer.android.com/guide/components/activities/activity-lifecycle
onCreate: 
You must implement this callback, which fires when the system first creates the activity. On activity creation, the activity enters the _Created_ state. In the `onCreate()` method, you perform basic application startup logic that should happen only once for the entire life of the activity. 
是在Activity被创建的时候调用的.在这个方法里头，我们一般做一些初始化的动作，比如说，设置和获取到UI的控件，设置对应的监听事件等。

onStart:
When the activity enters the Started state, the system invokes this callback. The `onStart()` call makes the activity visible to the user, as the app prepares for the activity to enter the foreground and become interactive. For example, this method is where the app initializes the code that maintains the UI.
已经可见了，但是没有获取到焦点，不可以进行操作

onResume: 
When the activity enters the Resumed state, it comes to the foreground, and then the system invokes the `[onResume()]` callback. This is the state in which the app interacts with the user. The app stays in this state until something happens to take focus away from the app. Such an event might be, for instance, receiving a phone call, the user’s navigating to another activity, or the device screen’s turning off.
可见并且获取到了焦点，可以进行交互

onPause，暂停，失去焦点，不可操作
onStop，已经不可见
onDestroy

**完整的声明周期：**
也就是说，从onCreate开始，一直到onDestroy方法执行完成。这就是一个完整的声明周期。 一般来说，完整的声明周期走完所有的声明周期方法。在onCreate方法的时候初始化资源，在onDestroy方法释放资源。
**Activity可见的声明周期：**
可见的声明周期就是调用onStart到调用onStop这段声明周期。在这其间，用户可以看到UI,也可以进行交互。在这两个方法之间，你可以让要显示的数据显示给用户。栗子我就不说了。
**前台生命周期：**
什么是前台呢？也就是正在操作的，相对于后台运行来说的。前台生命周期是从onResume到onPause这期间。在这期间的话,Activity会跑在前台。Activity可能会频繁地切换于onResume和onPause这两个方法间。前面我们讲到了，onResume是获取到了焦点了，onPause就失去焦点了。

![[activity_lifecycle.png]]

**横竖屏切换生命周期变化**
可以看到的是，Activity执行了onPause，再执行onStop和onDestroy方法。 也就是说，它先是走完了自己的生命周期，再重新开始。
对于横竖屏生命周期的总结是：先销毁掉原来的生命周期，然后再重新跑一次。
但是，这样子是不是会有问题呢？有些场景下： 比如说，做游戏开发 。横竖屏的切换，生命周期重新加载，那么当前页面的数据也会重新开始了。

第一种方法：指定该Activity是横屏或者是竖屏，在配置文件里修改
```xml
    <activity android:name=".LandscapeActivity"
            android:screenOrientation="landscape">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
```
第二种方法：能让屏幕随着屏幕的旋转而旋转，并不硬性生命周期的变化
```xml
  <activity android:name=".LandscapeActivity"
            android:configChanges="orientation|screenSize|keyboardHidden">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>

                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
```

#### Activity启动模式
**任务栈(task stack)**

**standard模式（默认）**
创建新的任务，并且置于当前栈顶，当我们点击返回的时候，销毁当前任务，出栈
```java
        <activity
            android:name=".NewActivity"
            android:launchMode="standard"
            android:theme="@style/AppTheme"/>
```

**singleTop模式**
如果栈顶已经是当前的任务了，那么就不会创建新的任务
使用场景：一般来说，为了保证只有一个任务，而不被创建多个，就需要这种模式。比如说，浏览器的收藏夹，可以被javaScript的代码控制，比如说通知，可以被拉起来的这些，比较被动的任务，则使用SingleTop模式，防止被多次创建。如果已经在顶部了，或者我们可以理解为已经聚焦了，就没必要再创建了。

**singleTask模式**
如果要创建的任务不存在，就会创建任务并放到栈顶；如果要创建的任务已经存在，就会把这个任务以上的任务全部出栈，使当前任务成为栈顶任务
使用场景：假设这个任务，要占比较多的内存开销，就会使用SingleTask的模式来保证它在栈里只有一个

**singleInstance模式**
前面三种启动模式都是在同一个任务栈，而singleInstance是独立一个任务栈，是单一的对象，独占一个栈，不会再创建，只会把它提前
应用场景：在整个系统中只有唯一一个实例，比如Launcher；比如有道词典的取词，因为它在每个界面都可以取词
### 2. BroadcastReceiver
#### 动态注册
**监听电量变化例子**
第一步：需要注册电量权限
```xml
<uses-permission android:name="android.permission.BATTERY_STATS"/>
```
定义广播接收器BroadcastReceiver类，继承自BroadcastReceiver
```java
public class BatteryStatusReceiver extends BroadcastReceiver {
    private static final String TAG = "BatteryStatusReceiver";
    @Override
    public void onReceive(Context context, Intent intent) {
        String action = intent.getAction();
        Log.d(TAG, "action is == " + action);
        if (action.equals(Intent.ACTION_BATTERY_CHANGED)) {
            Log.d(TAG, "电量改变了的广播...");
        } else if (action.equals(Intent.ACTION_BATTERY_LOW)) {
            Log.d(TAG, "电池低电量广播...");
        } else if (action.equals(Intent.ACTION_BATTERY_OKAY)) {
            Log.d(TAG, "电池充电完成。...");
        }
    }
}
```
第二步：设置频道和注册广播(动态注册)
```Java
IntentFilter intentFilter = new IntentFilter();    
//设置频道，也就是设置要监听的广播action.
intentFilter.addAction(Intent.ACTION_BATTERY_CHANGED);        intentFilter.addAction(Intent.ACTION_BATTERY_OKAY);
intentFilter.addAction(Intent.ACTION_BATTERY_LOW);
mReceiver = new BatteryStatusReceiver();
//注册广播
registerReceiver(mReceiver, intentFilter);
```
第三步：动态注册时，需要在onDestroy中释放资源
```java
 protected void onDestroy() {
        super.onDestroy();
        //取消广播注册，释放资源
        if (mReceiver != null) {
            unregisterReceiver(mReceiver);
            mReceiver = null;
            Log.d(TAG, "unregister receive..");
        }
    }
```
#### 静态注册
在AndroidManifest里进行注册。首先在Application节点里头，添加一个receiver节点，name则是我们的广播接收者，并且添加意图过滤的action
```xml
 <receiver android:name=".BootCompletedReceiver">
    <intent-filter>
	    <action android:name="android.intent.action.BOOT_COMPLETED"/>
    </intent-filter>
</receiver>
```
#### 静态和动态注册方式的区别
静态注册可以一直监听着，即使应用没有起来，也可以监听着，但是耗资源，长期监听着，静态注册的广播优先级高于动态注册的广播；动态注册的优点就是省资源，需要的时候才监听，不需要的时候需要取消注册。

重要应用场景：监听应用的下载和安装，收集用户习惯
额外的知识：我们学习android的广播机制有什么用呢，其实就是用于通知。如果在应用内，我们常用的通知方式是回调和广播。这两者之前，回调的速度快，保障性高，而广播则简单，但是速度没有回调高。什么情况下使用广播呢？当有多个地方使用等待通知的时候，可以使用广播。原理上广播和回调差不多的，广播的原理就是使用Binder机制，把action注册到ActivityManagerService里头，然后广播的时候，就去里面寻找符合规则的，再调用onReceive这个方法。
另外一种情况就是跨进程通讯，后面我们会学习到AIDL，这里的话是广播。广播是可以跨应用通知的，比如我们接收到了系统的广播对吧！也可以进行权限的控制，谁可以接收到这样的广播。

#### 自定义广播
发送广播
```java
public void sendBroadcast(View view) {
    Intent intent = new Intent();
        //action只能有一个,所以叫setAction而不是addActon。
        //而广播接收者可以监听多个广播,所以是addAction
        //action的命名一般是报名+动作名,这样子比较唯一
        intent.setAction("com.sunofbeaches.broadcastdemo.SEND_BROADCAST_CLICK");
        //也可以携带数据
        intent.putExtra("Content", "这是我点击按钮发送的广播!");
        sendBroadcast(intent);
    }
```
接收广播（要注意的是，内部广播接收者类，需要是静态的，Public的，注册的时候，是外部类名$内部类名）
```java
  @Override
        public void onReceive(Context context, Intent intent) {
            String action = intent.getAction();
            Log.d(TAG, "Inner receiver 接收到的actions... " + action);
            if ("com.sunofbeaches.broadcastdemo.SEND_BROADCAST_CLICK".equals(action)) {
                String content = intent.getStringExtra("Content");
                Log.d(TAG, "content is == " + content);
            }
        }
```
静态注册
```xml
<receiver android:name=".SendBroadcastActivity$InnerReceiver">
    <intent-filter>
        <action android:name="com.sunofbeaches.broadcastdemo.SEND_BROADCAST_CLICK"/>
    </intent-filter>
</receiver>
```

注意：`<permisson>`是声明权限, `<uses-permission>`是使用有什么权限 

#### 有序广播
1、有序，一级一级往下传
2、可以终止往下传达
3、可以修改广播的内容

广播发送者
```java
  public void sendDonation(View view) {
        Intent intent = new Intent();
        intent.setAction("com.sunofbeaches.broadcastdemo.DONATION");
        Bundle bundle = new Bundle();
        bundle.putInt("money", 1000 * 500);
        sendOrderedBroadcast(intent, null, null, null, 1, "给每个贫困的学生资助1000元", bundle);
    }
```
其中sendOrderedBroadcast方法
```java
  public void sendOrderedBroadcast(Intent intent, String receiverPermission, BroadcastReceiver resultReceiver, Handler scheduler, int initialCode, String initialData, Bundle initialExtras)
```
-   第一个参数，是意图对象，用于封装数据和设置过滤。
-   第二个参数是权限
-   第三个参数是广播接收者，这个广播接收者是最终接收的广播接收者，用于检查数据是否有传达或者数据被修改。
-   第四个参数是一个自定义的Hanlder，用于处理结果接收者，也就是上面那个接收者的回调。
-   第五个参数是初始码，这个会作为结果码，通常是Activity.RESULT_OK，也就是-1。
-   第六个参数是用于传递数据的，这个数据在各个Receiver里获取到，通过getResultData方法获取。这个其实通常为null
-   第七个参数也是用于封装数据的，不同的是，这个用于封装数据集合，从上面的代码可以知道 ，用来封装了一个钱的数据。

定义顺序的优先级，priority字段从-1000到1000，为优先级
```xml
<receiver android:name=".PoorStudentReceiver">
    <intent-filter android:priority="-1000">
        <action android:name="com.sunofbeaches.broadcastdemo.DONATION"/>
    </intent-filter>
</receiver>
```
终止广播的传达：
```java
    @Override
    public void onReceive(Context context, Intent intent) {
        //终止广播往下传
        abortBroadcast();
    }
```

#### 我发的广播谁可以收到
发送广播之前在manifest定义权限`<permission>`
```xml
<permission android:name="com.sunofbeaches.permission.DONATION"/>
```
发送广播时需要在`sendOrderedBroadcas`方法设置权限
```java
    public void sendDonation(View view) {
        Intent intent = new Intent();
        intent.setAction("com.sunofbeaches.broadcastdemo.DONATION");
        Bundle bundle = new Bundle();
        bundle.putInt("money", 1000 * 500);
        sendOrderedBroadcast(intent, Manifest.permission.DONATION, null, null, 1, "给每个贫困的学生资助1000元", bundle);
    }
```
接收广播需要在manifest.xml定义权限`<uses-permission>`
```xml
<uses-permission android:name="com.sunofbeaches.permission.DONATION"/>
```

#### 谁有权限给我发广播
接收广播，receiver的节点里定义`<permission>`，Permission声明就是用于控制谁有权限给我发广播
发送广播，需要在manifest.xml设置`<uses-permission>`.

### 3.Service

用俗话话应该是长期于后台运行的程序，如果是官方一点，首先它是一个组件，用于执行长期运行的任务，并且与用户没有交互。
每一个服务都需要在配置文件AndroidManifest.xml文件里进行生命，怎么生命呢？
使用`<service>`标签，其实跟前面的activity，广播接收者receiver一样生命。
通过Context.startService()来开启服务，通过Context.stop()来停止服务。当然啦，还有一种启动形式就是通过Context.bindService()的方法

#### 服务的用处
服务是用于执行长期后台运行的操作。有些时候，我们没有界面，但是程序仍然需要工作。比如说，在后台播放音乐，下载任务。

1、前台进程：可以理解为是最顶部的，直接跟用户交互的。比如说我们操作的Activity界面.
2、可见进程：可以见的，但是不操作的，比如说我们在一个Activity的顶部弹出一个Dialog，这个Dialog就是前台进程，但是这个Activity则是可见进程。
3、服务进程：服务可以理解为是忙碌的后台进程，虽然是在后台，但是它很忙碌。
4、后台进程：后台进程就是退隐到后台，不做事的进程。
5、空进程：空进程是不做事的，没有任何东西在上面跑着，仅作缓存作用。
假设，内存不够用了，会先杀谁呢？
首先杀的是空进程，要是还不够就杀后台进程，要是还不够，那么就杀服务，但是服务被杀死以后，等内存够用了，服务又会跑起来了。
所以：如果我们需要长期后台操作的任务，使用Service就对了！其实Framework里多数是服务。音乐播放，即使退到了后台，也可以播放；下载东西，退到后台也能下载；记录日志。

#### 服务生命周期

注册服务，四大组件都需要注册。在配置文件里配置如下：
```xml
<service android:name=".FirstService"/>
```
创建一个类，继承Service
```java
public class FirstService extends Service{
    private static final String TAG = "FirstService";
    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }
    @Override
    public void onCreate() {
        Log.d(TAG, "onCreate...");
        super.onCreate();
    }
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Log.d(TAG, "onStartCommand...");
        return super.onStartCommand(intent, flags, startId);
    }
    @Override
    public void onDestroy() {
        Log.d(TAG, "onDestroy...");
        super.onDestroy();
    }
}
```
接着，我们写一个Activity去控制服务
```java
public class MainActivity extends AppCompatActivity {
    private static final String TAG = "MainActivity";；
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
    public void startService(View view) {
        Log.d(TAG, "start service ... ");
        startService(new Intent(this, FirstService.class));
    }
    public void stopService(View view) {
        Log.d(TAG, "stop service....");
        stopService(new Intent(this, FirstService.class));
    }
}
```

![[service_lifecycle.png]]
startService服务的生命周期为
onCreate
onStartCommand
onDestroy

bindService服务的生命周期为
onCreate
onBind
onUnbind
onDestr

#### 绑定启动服务
前面的开启服务方式，有一个弊端。就是没法进行通讯。
另外一种启动服务的方式–通过绑定服务的形式来启动服务。对应的停止服务则是解绑服务。

创建一个类，继承Service
```java
public class SecondService extends Service {
    private static final String TAG = "SecondService";
    @Override
    public IBinder onBind(Intent intent) {
        Log.d(TAG, "onBind");
        return null;
    }
    @Override
    public boolean onUnbind(Intent intent) {
        Log.d(TAG, "onUnbind");
        return super.onUnbind(intent);
    }
    @Override
    public void onCreate() {
        Log.d(TAG, "onCreate");
        super.onCreate();
    }
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Log.d(TAG, "onStartCommand");
        return super.onStartCommand(intent, flags, startId);
    }
    @Override
    public void onDestroy() {
        Log.d(TAG, "onDestroy");
        super.onDestroy();
    }
}
```
编写BindServiceActivity
```java
public class BindServiceActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_bind_service);
    }
    public void bindServiceClick(View view) {
        //创建意图对象
        Intent intent = new Intent(this, SecondService.class);
        //第一个是参数是意图对象,第二个参数是回调,第三个参数是标记,这个是自动创建的意,如果服务没有start,那么会自己创建。
        //automatically create the service as long as the binding exists
        bindService(intent, mServiceConnection, BIND_AUTO_CREATE);
    }
    private ServiceConnection mServiceConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {

        }
        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };
    public void unBindServiceClick(View view) {
        //解绑服务
        if (mServiceConnection != null) {
            unbindService(mServiceConnection);
        }
    }
}
```
此时生命周期变成了：
onCreate
onBind
unBind
onDestroy

`onBind`方法里需要返回一个东西，也就是IBinder，假设需要调用服务里的一个方法，我们可以在里面声明 一个方法叫服务的内部方法！
```java
public class SecondService extends Service {
//服务内部的类CommunicateBinder，继承了binder类，binder类实现了IBinder接口：
    public class CommunicateBinder extends Binder{
        void callInnerMethod(){
            innerMethod();
        }
    }
    private static final String TAG = "SecondService";
    private void innerMethod() {
        Log.d(TAG, "innerMethod was called...");
    }
    @Override
    public IBinder onBind(Intent intent) {
        Log.d(TAG, "onBind");
        return new CommunicateBinder();
    }
    ...
    //其余和上面一样 
}
```
所以我们在绑上的时候，就返回这个类给绑定服务的地方，我们看看绑定服务的地方是怎么获取到这个类的：
```java
public class BindServiceActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_bind_service);
    }
    public void bindServiceClick(View view) {
        //创建意图对象
        Intent intent = new Intent(this, SecondService.class);
        //第一个是参数是意图对象,第二个参数是回调,第三个参数是标记,这个是自动创建的意,如果服务没有start,那么会自己创建。
        //automatically create the service as long as the binding exists
        bindService(intent, mServiceConnection, BIND_AUTO_CREATE);
    }
    private SecondService.CommunicateBinder mCommunicateBinder;
    private ServiceConnection mServiceConnection = new ServiceConnection() {


        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            if (service instanceof SecondService.CommunicateBinder) {
                mCommunicateBinder = (SecondService.CommunicateBinder) service;
            }
        }
        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };
    public void unBindServiceClick(View view) {
        //解绑服务
        if (mServiceConnection != null) {
            unbindService(mServiceConnection);
        }
    }
    public void callServiceMethod(View view) {
        if (mCommunicateBinder != null) {
            //调用服务内部的方法
            mCommunicateBinder.callInnerMethod();
        }
    }
}
```
这里面有一段代码是回调函数 ：
```java
public void onServiceConnected(ComponentName name, IBinder service) {
            if (service instanceof SecondService.CommunicateBinder) {
                mCommunicateBinder = (SecondService.CommunicateBinder) service;
            }
        }
```
拿到了绑定上以后传回来的那个类，这样子我们就可以调用服务里的方法了：
```java
public void callServiceMethod(View view) {
        if (mCommunicateBinder != null) {
            //调用服务内部的方法
            mCommunicateBinder.callInnerMethod();
        }
    }
```

**接口进行隐藏方法，完善代码**
这样的代码还不够完美，对于服务内部的方法，应该隐藏起来，而公共的东西进行抽取，所以，我们应该定义一个接口，把服务里的`CommunicateBinder`隐藏起来，那隐藏起来以后，外部怎么能调用呢？当然是通过接口的形式来实现啦！
```java
   public class CommunicateBinder extends Binder{
        void callInnerMethod(){
            innerMethod();
        }
    }
```
我们创建一个接口：
```java
public interface IServiceControl {
    void callServiceInnerMethod();
}
```
接着，我们私有服务里这个类，并且实现这个接口：
```java
 private class CommunicateBinder extends Binder implements IServiceControl{
        @Override
        public void callServiceInnerMethod() {
            innerMethod();
        }
    }
```
修改`BindServiceActivity`的代码 
```java
public class BindServiceActivity extends Activity {
    public void bindServiceClick(View view) {
        //创建意图对象
        Intent intent = new Intent(this, SecondService.class);
        //第一个是参数是意图对象,第二个参数是回调,第三个参数是标记,这个是自动创建的意,如果服务没有start,那么会自己创建。
        //automatically create the service as long as the binding exists
        bindService(intent, mServiceConnection, BIND_AUTO_CREATE);
    }
    private IServiceControl mCommunicateBinder;
    private ServiceConnection mServiceConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            if (service instanceof IServiceControl) {
                mCommunicateBinder = (IServiceControl) service;
            }
        }
        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };
    public void unBindServiceClick(View view) {
        //解绑服务
        if (mServiceConnection != null) {
            unbindService(mServiceConnection);
        }
    }
    public void callServiceMethod(View view) {
        if (mCommunicateBinder != null) {
            //调用服务内部的方法
            mCommunicateBinder.callServiceInnerMethod();
        }
    }
}
```

总结一下绑定服务的特点：
1、绑定服务在系统设置里是没有显进服务正在跑着的；
2、如果onBind方法返回的是null,那么onServiceConnected方法不会被调用;
3、绑定服务的生命周期跟Activity是不求同时生，但求同时死，Activity没了，服务也要解绑；
4、服务在解除绑定以后会停止运行，执行unBind方法—>onDestroy方法；
5、绑定服务开启的服务，只可以解绑一次，多次解绑会抛异常；
6、绑定的connection要跟解绑的connection要对应着，否则没法解绑。

稍微总结一下，startService和bindService的区别，优点和缺点：
1、startService这个方法来启动服务的话，是长期运行的，context销毁，服务仍然存在，只有stopService才会停止服务。而bindService来启动服务，不用的时候，需要调用unBindService，否则会导致context泄漏，所以bindService不是长期运行的。当context销毁的时候，则会停止服务运行。
2、startService来启动服务可以长期运行，但是不可以通讯，而bindService的方式来启动服务则可以通讯，两者都有优缺点，所以我们就有了混合起来使用的方法。

#### 例子：银行服务

普通用户行为的接口
```java
public interface INormalUserAction {
    void savemoney(float money);
    float getmoney();
    float loanmoney();
}
```
普通用户行为接口实现，需要继承Binder
```java
public class NormalUserAction extends Binder implements INormalUserAction  {
    //这里要继承binder
    private static final String TAG = "NormalUserAction";
    @Override
    public void savemoney(float money) {
        Log.d(TAG, "savemoney is ---" + money);
    }
    @Override
    public float getmoney() {
        Log.d(TAG, "getmoney is ---100.00");
        return 100.00f;
    }
    @Override
    public float loanmoney() {
        Log.d(TAG, "loanmoney is 100.00");
        return 100.00f;
    }
}
```
普通用户Activity，需要`bindService`指定意图和connection , `NormalUserConnection`为内部类，里面有onServiceConnected和onServiceDisconnected两个回调方法，其中涉及到了向下转型
```java
public class NormalUserActivity extends Activity {

    private static final String TAG = "NormalUserActivity";
    private NormalUserConnection mnormalUserConnection;
    private boolean mIsBind;
    private INormalUserAction mnormalUserAction;
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_user);
        doBindService();//写一个绑定服务的方法
    }
    private void doBindService() {
        Log.d(TAG,"执行doBindService");
        Intent intent =new Intent();
        intent.setAction("com.example.ACTION_NORMAL_USER");
        intent.addCategory(Intent.CATEGORY_DEFAULT);//理解这步
        intent.setPackage(this.getPackageName());//必须声明包名,否则会报错
        mnormalUserConnection = new NormalUserConnection();
        mIsBind = bindService(intent, mnormalUserConnection, BIND_AUTO_CREATE);
    }
    private class NormalUserConnection implements ServiceConnection {
        private INormalUserAction mnormalUserAction;
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
           //通过接口来转换,等于这个方法里面的IBinder 的service;这样写就能获取对方的控制权
            Log.d(TAG,"Service connected"+name);
            mnormalUserAction = (INormalUserAction) service;
        }
        @Override
        public void onServiceDisconnected(ComponentName name) {
            Log.d(TAG,"Service disconnected"+name);
        }
    }
    public  void  saveClick(View view){
        Log.d(TAG,"savemoney");
        mnormalUserAction.savemoney(100000);//走到这一步开始出错,反射异常和空指针异常
    }
    public  void  getClick(View view){
Log.d(TAG,"get money");
mnormalUserAction.getmoney();
    }
    public  void  loanClick(View view){
     Log.d(TAG,"loan money");
     mnormalUserAction.loanmoney();
    }
    @Override
    protected void onDestroy() {
        super.onDestroy();
        if (mIsBind&&mnormalUserConnection!=null){
            unbindService(mnormalUserConnection);
            mnormalUserConnection = null;
            mIsBind = false;
        }
    }
}
```
`onBind`方法需要返回一个继承了IBinder的对象 
```java
public class BankServices extends Service {


    private static final String TAG = "BankServices";

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {

        String action = intent.getAction();
        if (!TextUtils.isEmpty(action)){
            Log.d(TAG,"进行回调");
            if ("com.example.ACTION_NORMAL_USER".equals(action)) {
                Log.d(TAG,"符合用户Action");
                     return  new NormalUserAction();
            }else if ("com.example.ACTION_BANK_WORKER".equals(action)){
                     return new BankWorkerAction();
            }else if ("com.example.ACTION_BANK_BOSS".equals(action)){
                     return new BankBossAction();
            }
        }
        Log.d(TAG, "走到这里");
        return new NormalUserAction();
    }
}
```

#### 例子：模拟支付宝
看视频理解

#### 零散知识点
Activity和Service均是继承自Context，因此可以用this
***
onStart也是服务的生命周期，只是这个方法已经过时了。
***
以绑定的方式启动的服务，在context销毁的时候，必须解绑，否则会泄漏；startService方式开启，即使context销毁，服务仍然存在
***
bindService开启的服务，在系统里是看不到服务在运行的;如果是通过startService的方式启动的服务，则会在应用里看到。
***
向上转型：
好处：隐藏了子类型，提高了代码的扩展性。
坏处：只能使用父类的功能，不能使用子类特有功能，功能被限定。
使用场景：不需要面对子类型，通过提高扩展性，或者使用父类的功能即可完成操作，就是使用向上转型。

向下转型：
好处:可以使用子类型的特有功能
坏处:面对具体的子类型，向下转型具有风险。即容易发生ClassCastException，只要转换类型和对象不匹配就会发生。解决方法：使用关键字instanceof。
***
end

### 4.AccessibilityService
[[Android--AccessibilityService]]
