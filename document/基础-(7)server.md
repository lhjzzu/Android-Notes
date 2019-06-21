## 服务

### 一 进程

当某个应用组件启动且该应用没有运行其他任何组件时，Android 系统会使用单个执行线程为应用启动新的 Linux 进程。默认情况下，同一应用的所有组件在相同的进程和线程（称为“主”线程）中运行。 如果某个应用组件启动且该应用已存在进程（因为存在该应用的其他组件），则该组件会在此进程内启动并使用相同的执行线程。 但是，您可以安排应用中的其他组件在单独的进程中运行，并为任何进程创建额外的线程。



+ 进程的生命周期

  ```
  Android 系统将尽量长时间地保持应用进程，但为了新建进程或运行更重要的进程，最终需要移除旧进程来回收内存。 为了确定保留或终止哪些进程，系统会根据进程中正在运行的组件以及这些组件的状态，将每个进程放入“重要性层次结构”中。 必要时，系统会首先消除重要性最低的进程，然后是重要性略逊的进程，依此类推，以回收系统资源。
  ```

+ 进程的优先级

  1. 前台进程

     用户当前操作所必需的进程。如果一个进程满足以下任一条件，即视为前台进程：

     + 托管(持有)用户正在交互的 `Activity`（已调用 `Activity` 的 `onResume()` 方法）

     + 托管(持有)某个 `Service`，后者绑定到用户正在交互的 Activity

     + 托管(持有)正在“前台”运行的 `Service`（服务已调用 `startForeground()`）

     + 托管(持有)正执行一个生命周期回调的 `Service`（`onCreate()`、`onStart()` 或 `onDestroy()`）

     + 托管(持有)正执行其 `onReceive()` 方法的 `BroadcastReceiver

       ```
       通常，在任意给定时间前台进程都为数不多。只有在内存不足以支持它们同时继续运行这一万不得已的情况下，系统才会终止它们。 此时，设备往往已达到内存分页状态，因此需要终止一些前台进程来确保用户界面正常响应。
       ```

  2. 可见进程
     没有任何前台组件、但仍会影响用户在屏幕上所见内容的进程。 如果一个进程满足以下任一条件，即视为可见进程：

     + 托管不在前台、但仍对用户可见的 `Activity`（已调用其 `onPause()` 方法）。例如，如果前台 Activity 启动了一个对话框，允许在其后显示上一 Activity，则有可能会发生这种情况。

     + 托管绑定到可见（或前台）Activity 的 `Service`。

       ```
       可见进程被视为是极其重要的进程，除非为了维持所有前台进程同时运行而必须终止，否则系统不会终止这些进程。
       ```

  3. 服务进程

     ```
     正在运行已使用 startService() 方法启动的服务且不属于上述两个更高类别进程的进程。尽管服务进程与用户所见内容没有直接关联，但是它们通常在执行一些用户关心的操作（例如，在后台播放音乐或从网络下载数据）。因此，除非内存不足以维持所有前台进程和可见进程同时运行，否则系统会让服务进程保持运行状态。
     ```

  4. 后台进程

     ```
     包含目前对用户不可见的 Activity 的进程（已调用 Activity 的 onStop() 方法）。这些进程对用户体验没有直接影响，系统可能随时终止它们，以回收内存供前台进程、可见进程或服务进程使用。 通常会有很多后台进程在运行，因此它们会保存在 LRU （最近最少使用）列表中，以确保包含用户最近查看的 Activity 的进程最后一个被终止。如果某个 Activity 正确实现了生命周期方法，并保存了其当前状态，则终止其进程不会对用户体验产生明显影响，因为当用户导航回该 Activity 时，Activity 会恢复其所有可见状态。 有关保存和恢复状态的信息，请参阅 Activity文档。
     ```

  5. 空进程

     ```
     不含任何活动应用组件的进程。保留这种进程的的唯一目的是用作缓存，以缩短下次在其中运行组件所需的启动时间。 为使总体系统资源在进程缓存和底层内核缓存之间保持平衡，系统往往会终止这些进程。
     ```

​         

+ 进程优先级的简单理解
  1. 前台进程:优先级最高，相当于activity执行了onResume方法，用户正在交互
  2. 可视进程:一直影响用户看的见 ， 相当于activity执行了onPause方法
  3. 服务进程: 通过startService方法开启了一个服务
  4. 后台进程: 相当于activity执行了onStop方法，界面不可见，但是activity并没有被销毁
  5. 空进程: 不会维持任何组件运行

### 二 start开启服务

+ 简单示例

  1. 创建service的子类,并在清单文件中声明

     ```java
     public class DemoService extends Service {
     	@Override
     	public IBinder onBind(Intent intent) {
     		
     		System.out.println("onBind");
     		return null;
     	}
     	//当服务第一次创建的时候调用
     	@Override
     	public void onCreate() {
     		System.out.println("onCreate");
     		super.onCreate();
     	}
     
     	@Override
     	public int onStartCommand(Intent intent, int flags, int startId) {
     		System.out.println("onStartCommand");
     		return super.onStartCommand(intent, flags, startId);
     	}
     	//当服务销毁的是调用
     	@Override
     	public void onDestroy() {
     		System.out.println("onDestroy");
     		super.onDestroy();
     	}
     }
     
     ```

     ```xml
      <!--清单文件中配置服务  -->
      <service android:name="com.itheima.servicedemo.DemoService">
      </service>
     ```

  2. 开启服务

     ```java
     //点击按钮 开启服务  通过startservice 
     	public void click1(View v) {
     		Intent intent = new Intent(this,DemoService.class);
     		startService(intent); //开启服务 
     	}
     
     1. 通过startService方法开启服务
     2. DemoService中的onCreate， onStartCommand先后调用
     3. 多次点击时，onCreate方法只调用1次，onStartCommand每次都调用
     4. 服务一旦开启，服务就会在后台长期运行，直到用户手工停止
     ```

  3. 停止服务

     ```java
     public void click2(View v) {
     		Intent intent = new Intent(this,DemoService.class);
     		stopService(intent);
     }
     1. 通过stopService方法停止服务
     2. DemoService中的onDestroy方法被调用
     ```



特点

1. 定义四大组件的方式是一样的
2. 定义一个类集成Service
3. 第一次点击按钮开启服务，服务之星onCreate和onStart方法
4. 第二次点击按钮，再次开启服务，服务执行onStart方法
5. **服务一旦被开启，服务就会后台长期运行，直到用户手动停止**

### 三 电话窃听器

1. 在开机广播里开启服务

   ```xml
    添加权限
   <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
   
    <!--配置广播接收者  -->
    <receiver android:name="com.itheima.phonelistener.BootReceiver">
          <intent-filter >
              <action android:name="android.intent.action.BOOT_COMPLETED"/>
          </intent-filter>
   </receiver>
          
   ```

   ```java
   //定义开机启动广播接收者
   public class BootReceiver extends BroadcastReceiver {
   	@Override
   	public void onReceive(Context context, Intent intent) {
   		//把服务开启 
   		Intent intent1 = new Intent(context,PhoneService.class);
   		context.startService(intent1);
   	}
   
   }
   ```

2. 在清单文件中配置电话状态监听的服务，并实现电话录音的逻辑

   ```xml
    <uses-permission android:name="android.permission.READ_PHONE_STATE"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.RECORD_AUDIO" />
    
    <service android:name="com.itheima.phonelistener.PhoneService"></service>
   
   ```

   ```java
   public class PhoneService extends Service {	
   	private MediaRecorder recorder; //录音机实例
   	@Override
   	public IBinder onBind(Intent intent) {
   		// TODO Auto-generated method stub
   		return null;
   	}
   	@Override
   	public void onCreate() {
   		//[1]获取telephonemanager的实例 
   		TelephonyManager tm =   (TelephonyManager) getSystemService(TELEPHONY_SERVICE);
   		//[2]注册电话的监听 
   		tm.listen(new MyPhoneStateListener(), PhoneStateListener.LISTEN_CALL_STATE);
   		super.onCreate();
   	}
   	
   	@Override
   	public void onDestroy() {
   		super.onDestroy();
   	}
   	
   	//定义一个类用来监听电话的状态
   	private class MyPhoneStateListener extends PhoneStateListener{
   		//当电话设置状态发生改变的时候调用
   		@Override
   		public void onCallStateChanged(int state, String incomingNumber) {
   			
   			//[3]具体判断一下电话的状态 
   			switch (state) {
   			case TelephonyManager.CALL_STATE_IDLE: //空闲状态
   				if (recorder!=null) {
   					 recorder.stop(); //停止录音
   	            	 recorder.reset();   // You can reuse the object by going back to setAudioSource() step
   	            	 recorder.release(); // Now the object cannot be reused
   				}				
   				break;
               case TelephonyManager.CALL_STATE_OFFHOOK: //接听状态
               	System.out.println("开始录");
               	recorder.start();   // Recording is now started
   				break;
               case TelephonyManager.CALL_STATE_RINGING:   //电话响铃状态
               	System.out.println("准备一个录音机");
               	//[1]创建MediaRecorder 实例 
               	 recorder = new MediaRecorder();            	 
               	 //[2]设置音频的来源 
               	 recorder.setAudioSource(MediaRecorder.AudioSource.MIC); //zet             	 
               	 //[3]设置输出的格式 
               	 recorder.setOutputFormat(MediaRecorder.OutputFormat.THREE_GPP);
               	 //[4]设置音频的编码方式
               	 recorder.setAudioEncoder(MediaRecorder.AudioEncoder.AMR_NB);
               	 //[5]设置存放的路径
               	 recorder.setOutputFile("/mnt/sdcard/luyin.3gp");
               	 //[6]准备录
               	 try {
   					recorder.prepare();
   				} catch (IllegalStateException e) {
   					e.printStackTrace();
   				} catch (IOException e) {
   					e.printStackTrace();
   				}				
   				break;
   			}			
   			super.onCallStateChanged(state, incomingNumber);
   		}
   	}	
   }
   1. MediaRecorder.AudioSource.MIC 录制单方通话
   2. MediaRecorder.AudioSource.VOICE_CALL 录制双方通话
   ```

+ 广播接收者在清单文件中声明以后，那么在应用程序启动后就**自动创建**了对应的实例

+ 服务在 清单文件中声明以后，并没有生成对应的实例，需要**手动开启**一个服务

### 四 使用服务注册特殊的广播接收者

1. 创建锁屏/解锁的广播接收者,并实现对应的逻辑

   ```java
   public class ScreenReceiver extends BroadcastReceiver {
   	@Override
   	public void onReceive(Context context, Intent intent) {
   		//获取当前事件类型 
   		String action = intent.getAction();
   		if ("android.intent.action.SCREEN_OFF".equals(action)) {
   			System.out.println("说明屏幕锁屏了");
   		}else if("android.intent.action.SCREEN_ON".equals(action)){
   			System.out.println("~~~~~~~~~~");
   		}		
   	}
   }
   
   1. 特殊的广播接收者,声明在清单文件中无效
   ```

2. 声明，创建服务，并在服务中用代码动态注册广播接收者

   ```xml
   <service android:name="com.itheima.regitstbraodcast.ScreenService"></service>
   ```

   ```java
   
   public class ScreenService extends Service {
   	private ScreenReceiver receiver;
   	@Override
   	public IBinder onBind(Intent intent) {
   		return null;
   	}
   
   	
   	@Override
   	public void onCreate() {
   		//[1]创建ScreenReceiver 实例
   		receiver = new ScreenReceiver();
   		
   		//[2]获取IntentFilter 实例 目的是添加 action
   		IntentFilter filter = new IntentFilter();
   		//[3]添加action
   		filter.addAction("android.intent.action.SCREEN_OFF");
   		filter.addAction("android.intent.action.SCREEN_ON");
   		//[4]动态注册广播接收者
   		registerReceiver(receiver, filter);
   		
   		super.onCreate();
   	}
   	@Override
   	public void onDestroy() {
   		//当服务销毁的时候 取消注册广播接收者
   		unregisterReceiver(receiver);
   		super.onDestroy();
   	}
   }
   ```

3. 开启服务

   ```java
   //开启服务 
   Intent intent = new Intent(this, ScreenService.class);
   startService(intent);
   ```

### 五 bindService开启服务

#### 5.1 bindService的特点

 1. 创建service的子类,并在清单文件中声明

     ```java
     public class DemoService extends Service {
     	@Override
     	public IBinder onBind(Intent intent) {
     		
     		System.out.println("onBind");
     		return null;
     	}
     	//当服务第一次创建的时候调用
     	@Override
     	public void onCreate() {
     		System.out.println("onCreate");
     		super.onCreate();
     	}
     
     	@Override
     	public int onStartCommand(Intent intent, int flags, int startId) {
     		System.out.println("onStartCommand");
     		return super.onStartCommand(intent, flags, startId);
     	}
     	//当服务销毁的是调用
     	@Override
     	public void onDestroy() {
     		System.out.println("onDestroy");
     		super.onDestroy();
     	}
     }
     
     ```

     ```xml
      <!--清单文件中配置服务  -->
      <service android:name="com.itheima.servicedemo.DemoService">
      </service>
     ```

  2. 开启服务

     ```java
     //点击按钮 绑定服务 开启服务的第二种方式  
     	public void click3(View v){
     		Intent intent = new Intent(this,DemoService.class);
     		//连接到DemoService 这个服务 
     		conn = new MyConn();
     		bindService(intent,conn , BIND_AUTO_CREATE);
         }
     
     
     //定义一个类 用来监视服务的状态 
     	private class MyConn implements ServiceConnection{
     
     		//当服务连接成功调用
     		@Override
     		public void onServiceConnected(ComponentName name, IBinder service) {
     			System.out.println("onServiceConnected");
     			
     		}
     		//失去连接调用
     		@Override
     		public void onServiceDisconnected(ComponentName name) {
     			
     		}
     		
     	}
     	
     ```

  3. 停止服务

     ```java
     //点击按钮手动解绑服务
     	public void click4(View v){
     		unbindService(conn);
     	}
     
     ```

4. 调用者销毁时，要解绑服务

   ```java
   @Override
   	protected void onDestroy() {
   		//当Activity销毁的时候 要解绑服务 
   		unbindService(conn);
   		
   		super.onDestroy();
   	}
   ```

####  5.2 为什么要引入bindService

为了调用服务中的方法

#### 5.3 外界调用服务里的方法

1. 在服务内部定义一个方法，并通过中间人对象来调用

   ```java
   public class BanZhengService extends Service {
   
   	//把我定义的中间人对象返回 
   	@Override
   	public IBinder onBind(Intent intent) {
   		return new MyBinder();
   	}
   	//办证的方法
   	public void banZheng(int money){
   		if (money>1000) {
   			Toast.makeText(getApplicationContext(), "我是领导 把证给你办了", 1).show();
   		}else {
   			Toast.makeText(getApplicationContext(), "这点钱 还想办事....", 1).show();
   		}
   	}
   	//[1]定义中间人对象(IBinder)
   	public class MyBinder extends Binder{
   		
   		public void callBanZheng(int money){
   			//调用办证的方法
   			banZheng(money);
   		}
   	}	
   
   }
   
   ```

2. 在外部获取中间人对象，间接调用服务定义的方法

   ```java
   public class MainActivity extends Activity {
   	private MyConn conn;
   	private MyBinder myBinder;//我定义的中间人对象
   	@Override
   	protected void onCreate(Bundle savedInstanceState) {
   		super.onCreate(savedInstanceState);
   		setContentView(R.layout.activity_main);
   		Intent intent = new Intent(this,BanZhengService.class);
   		//连接服务 
   		conn = new MyConn();
   		bindService(intent, conn, BIND_AUTO_CREATE);
   	}
   
   	//点击按钮调用服务里面办证的方法
   	public void click(View v) {
   		myBinder.callBanZheng(10000000);
   	}
   	//监视服务的状态
   	private class MyConn implements ServiceConnection{		
   		//当服务连接成功调用
   		@Override
   		public void onServiceConnected(ComponentName name, IBinder service) {
   			//获取中间人对象
   			myBinder = (MyBinder) service;
   		}
   		//失去连接
   		@Override
   		public void onServiceDisconnected(ComponentName name) {
   			
   		}
   	}
   	
   	@Override
   	protected void onDestroy() {
   		//当activity 销毁的时候 解绑服务 
   		unbindService(conn);
   		super.onDestroy();
   	}
   }
   
   1. 当activity 销毁的时候 解绑服务 
   ```


特点

+ 第一次调用bindService，会执行服务的onCreate方法和onBind方法
+ 当onBind方法返回为null时， onServiceConnected方法是不执行的
+ 第二次调用bindService，服务中的方法都不会执行。
+ **不求同生，但求同死，指的是调用者(activity)和服务之间**
+ 服务不可以多次解绑，多次解绑会报异常
+ 通过bind方式开启服务，服务不能在设置页面找到，相当于一个隐形服务

#### 5.4 通过接口的方式开启服务

接口可以隐藏代码内部的细节，让程序员只暴露自己想暴露的方法

1. 定义一个接口，把想暴露的方法都定义在接口里面

   ```java
   public interface Iservice {
   	//把领导想暴露的方法都定义在接口里
   	public void callBanZheng(int money);	
   }
   //1 只暴露办证的方法
   ```

2. 我们定义的中间人对象实现我们自己定义的接口

   ```java
   public class DemoService extends Service {
   
   	//把我定义的中间人对象返回 
   	@Override
   	public IBinder onBind(Intent intent) {
   		return new MyBinder();
   	}
   	
   	//办证的方法
   	public void banZheng(int money){
   		if (money>1000) {
   			Toast.makeText(getApplicationContext(), "我是领导 把证给你办了", 1).show();
   		}else {
   			Toast.makeText(getApplicationContext(), "这点钱 还想办事....", 1).show();
   		}
   	}
   	//打麻将的方法
   	public void playMaJiang(){
   		System.out.println("陪领导打麻将");
   	}
   	
   	//洗桑拿的方法
   	public void 洗桑拿(){
   		System.out.println("陪领导洗桑拿");
   		
   	}
   
       //[1]定义中间人对象(IBinder)
   	private class MyBinder extends Binder implements Iservice{
   		
   		public void callBanZheng(int money){
   			//调用办证的方法
   			banZheng(money);
   		}
   		public void callPlayMaJiang(){
   			//调用playMaJiang 的方法
   			playMaJiang();
   			
   		}
   		public void callXiSangNa(){
   			//调用洗桑拿的方法
   			洗桑拿();
   		}		
   	}	
   }
   
   1. 中间人对象实现了MyBinder实现了Iservice接口
   ```

3. 获取中间人对象

   ```java
   //监视服务的状态
   	private class MyConn implements ServiceConnection{		
   		//当服务连接成功调用
   		@Override
   		public void onServiceConnected(ComponentName name, IBinder service) {
   			//获取中间人对象
   			myBinder = (Iservice) service;
   		}
   		//失去连接
   		@Override
   		public void onServiceDisconnected(ComponentName name) {
   		}
   	}
   	
   1. 在服务连接成功的时候，获取中间人对象，强制转换为接口类型。此时中间人对象只有办证的方法
   ```

### 六 混合方式开启服务

目的:让服务在后台长期运行，又想调用服务中的方法

1. 先调用startService开启服务，能够保证服务在后台长期运行

   ```java
   Intent intent = new Intent(this,MusicService.class);
   startService(intent);
   ```

2. 调用bindService方法，去获取中间人对象

   ```java
   conn = new MyConn();
   bindService(intent, conn, BIND_AUTO_CREATE);
   
   //监听服务的状态
   private class MyConn implements ServiceConnection{
   		//当服务连接成功 
   		@Override
   		public void onServiceConnected(ComponentName name, IBinder service) {
   			//获取我们定义的中间人对象
   			iservice = (Iservice) service;
   		}
   		@Override
   		public void onServiceDisconnected(ComponentName name) {
   		}
   		
   	}
   ```

3. 调用unbindService解绑服务，此时服务未销毁

   ```java
   @Override
   	protected void onDestroy() {
   		//当Activity销毁的时候 解绑服务   目的是为了不报红色日志 
   		unbindService(conn);
   		super.onDestroy();
   	}
   ```

4. 需要时可以调用stopService手动去停止服务



### 七 aidl介绍

本地服务: 运行在自己应用里面的服务

远程服务: 运行在其他应用里面的服务

实现进程间通信IPC

aidl:专门用来解决进程间通信



总结: 使用aidl的步骤

1. 把Iservice.java接口文件变成一个aidl文件

2. aidl这个语言不认识public, 把public去掉

   ```java
   interface Iservice {
   	 void callMethodService();
   }
   ```

3. 自动生成一个Iservice.java文件，系统自动帮我们创建一个实现了Iservice接口的stub类

   ```java
   public static abstract class Stub extends android.os.Binder implements com.itheima.remoteservice.Iservice
   {
    .........
   }
   ```

4. 定义服务，并声明在清单文件，并实现中间人对象直接继承Stub

   ```java
   
   public class RemoteService extends Service {
   
   	//把我们定义的中间人对象返回
   	@Override
   	public IBinder onBind(Intent intent) {
   		return new MyBinder();
   	}
   	
   	//定义一个方法
   	public void methodService(){
   		System.out.println("我是远程服务里面的方法");
   		Log.i("xxxxxxxxx", "我是远程服务里面的方法");
   
   	}	
   	//[1]定义一个中间人对象(IBinder) 
   	private class MyBinder extends Stub{
   
   		@Override
   		public void callMethodService() {
   			//调用方法
   			methodService();
   		}
   	}	
   }
   
   1. MyBinder继承Stub
   ```

   ```xml
    <!--配置service  -->
    <service android:name="com.itheima.remoteservice.RemoteService">
       <intent-filter >
           <action android:name="com.itheima.remoteservice"/>
       </intent-filter>
    </service>
    
    1. 配置service的intent-filter的action是为了让其他服务隐式调用该服务
   ```

5. 保证`本地服务的应用`aidl文件是同一个，并且保证aidl文件所在的包名相同

6. `本地服务应用`调用远程服务，并获取中间人的对象，去调用远程服务中的方法

   ```java
   public class MainActivity extends Activity {
   
   	private MyConn conn;
   	
   	private Iservice iservice;//中间人对象
   
   	@Override
   	protected void onCreate(Bundle savedInstanceState) {
   		super.onCreate(savedInstanceState);
   		setContentView(R.layout.activity_main);
   		//[1]调用bindservcie 获取中间人对象 
   		Intent intent = new Intent();
   		intent.setAction("com.itheima.remoteservice");
   		conn = new MyConn();
   		//[2]连接服务 目的为了获取我们定义的中间人对象
   		bindService(intent, conn, BIND_AUTO_CREATE);
   		
   	}
   
   	@Override
   	protected void onDestroy() {
   		unbindService(conn);
   		super.onDestroy();
   	}
   	
   	//点击按钮调用第九个应用 服务里面的方法 
   	public void click(View v) {
   		
   		try {
   			iservice.callMethodService();
   			Log.i("xxxxxxxxx", "xxxx");
   		} catch (RemoteException e) {
   			e.printStackTrace();
   		}
   	}
   	
   	//监视服务的状态
   	private class MyConn implements ServiceConnection{
   		//连接成功 
   		@Override
   		public void onServiceConnected(ComponentName name, IBinder service) {
   			//获取中间人对象  注意这里面在获取 中间人对象的方式变了 和之前不一样 
   			iservice = Stub.asInterface(service);
   		}
   
   		@Override
   		public void onServiceDisconnected(ComponentName name) {
   			
   		}
   	}
   }
   
   1. 通过action"com.itheima.remoteservice"创建Intent绑定服务
   2. 通过Stub获取中间人对象
   ```




### 资料

[Android系统中的进程管理：进程的优先级](https://blog.csdn.net/omnispace/article/details/73320950)







