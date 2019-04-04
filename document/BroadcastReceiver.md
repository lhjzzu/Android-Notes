## 广播接收者

### 一 基本概念

1. android系统内部定义了一系列的广播事件，例如:外拨电话，短信到来,sd卡状态，电池电量
2. 我们使用broadcastReceiver去接收系统已经定义好的事件



### 二 ip拨号器

> A向B打电话
>
> 正常流程: A --> 运营商---> B
>
> ip拨号:A-> 第三方-> 运营商->B
>
> 通过在B的电话号前加前缀从而通过第三方去连接运营商，然后再和B连接。
>
> 因为第三方跟运营商有合作，所以通话便宜。

1. 创建广播接收者

   ```java
   public class OutGoingCallReceiver extends BroadcastReceiver{
   }
   ```

2. 在AndroidMainfest.xml进行声明

   ```xml
   
   <!--配置广播接收者   -->
   <receiver android:name="com.itheima.ipdail.OutGoingCallReceiver">
         <intent-filter >
           <action android:name="android.intent.action.NEW_OUTGOING_CALL"/>
         </intent-filter>
   </receiver>
   
   1. 通过action配置该类的广播接收者能够接收"外拨电话"的广播
   
   ```

3. 添加`处理外拨电话`的权限

   ```java
   <uses-permission android:name="android.permission.PROCESS_OUTGOING_CALLS"/>
   ```

4. 在`OutGoingCallReceiver`的`onReceive`方法中对接收到的广播进行处理

   ```java
   //当进行外拨电话的时候 调用
   	@Override
   	public void onReceive(Context context, Intent intent) {
   		String ipNumber = "17951";
   		//[1]获取当前拨打的电话号码 
   		String currentNumber = getResultData();
   		//[2]在当前的号码前面加上 17951  
   		//[2.1]判断当前拨打的号码是否是 长途
   		if (currentNumber.startsWith("0")) {
   			//[2.2]修改拨打电话的号码
   			setResultData(ipNumber+currentNumber);
   		}				
   	}
   
   1. 通过getResultData获取当前拨打的号码
   2. 给拨打的号码加上前缀后，通过setResultData修改当前拨打的号码，从而达到ip拨号的目的
   ```

### 三 sd卡状态监听

1. 创建广播接收者

   ```java
   public class SdcardStateReceiver extends BroadcastReceiver {}
   ```

2. 在AndroidMainfest.xml进行声明

   ```xml
    
   <receiver android:name="com.itheima.sdcardstate.SdcardStateReceiver">
               <intent-filter >
                   <action android:name="android.intent.action.MEDIA_MOUNTED"/>
                   <action android:name="android.intent.action.MEDIA_UNMOUNTED"/>
                   
                   <!--小细节  这里需要配置一个data  约束类型叫file 因为sd里面存的数据类型是file  -->
                    <data android:scheme="file"/>
               </intent-filter>
           </receiver>
           
   1. action中指定之接收"sdk挂载/卸载"的广播
   2. 这里需要配置一个data  约束类型叫file 因为sd里面存的数据类型是file
   ```

3. 在`SdcardStateReceiver`的`onReceive`方法中对接收到的广播进行处理

   ```java
   //当sd状态发生改变的时候执行
   	@Override
   	public void onReceive(Context context, Intent intent) {
   
   		//获取到当前广播的事件类型
   		String action = intent.getAction();
   		if ("android.intent.action.MEDIA_MOUNTED".equals(action)) {
   			System.out.println("说明sd卡挂载了 ....");
   		}else if ("android.intent.action.MEDIA_UNMOUNTED".equals(action)) {
   			System.out.println("说明sd卡卸载了 ");
   		}
   }
   ```

### 四 短信监听

1. 创建广播接收者

   ```java
   public class SmsListenerReceiver extends BroadcastReceiver {}
   ```

2. 在AndroidMainfest.xml进行声明

   ```xml
    <receiver android:name="com.itheima.smslistener.SmsListenerReceiver">
               <intent-filter >
                   <action android:name="android.provider.Telephony.SMS_RECEIVED" />
                   
               </intent-filter>
   </receiver>
   ```

3. 添加`接收短信`的权限

   ```xml
    <uses-permission android:name="android.permission.RECEIVE_SMS"/>
   ```

4. 在`SmsListenerReceiver`的`onReceive`方法中对接收到的广播进行处理

   ```java
   //当短信到来的时候执行 
   	@Override
   	public void onReceive(Context context, Intent intent) {
   		//获取发送者的号码 和发送内容 
   		Object []objects = (Object[]) intent.getExtras().get("pdus");
   		
   		for (Object obj : objects) {
   			//[1]获取smsmessage实例 
   			SmsMessage smsMessage = SmsMessage.createFromPdu((byte[]) obj);
   			//[2]获取发送短信的内容 
   			String messageBody = smsMessage.getMessageBody();
   			String address = smsMessage.getOriginatingAddress();			
   			System.out.println("body:"+messageBody+"-----"+address);			
   		}
   	}	
   1. pdu 协议数据单元（Protocol Data Unit）
   ```

### 五  安装/卸载应用

1. 创建广播接收者

   ```java
   public class AppStateReceiver extends BroadcastReceiver
   {}
   ```

2. 在AndroidMainfest.xml进行声明

   ```xml
    <receiver android:name="com.itheima.appstate.AppStateReceiver" >
               <intent-filter>
                   <action android:name="android.intent.action.PACKAGE_INSTALL" />
                   <action android:name="android.intent.action.PACKAGE_ADDED" />
                   <action android:name="android.intent.action.PACKAGE_REMOVED" />
                   
                   <!-- 想让action事件生效 还需要 配置一个data -->
                   <data android:scheme="package" />
               </intent-filter>
           </receiver>
   
   1. "android.intent.action.PACKAGE_ADDED"存在应用被安装
   2. "android.intent.action.PACKAGE_REMOVED"存在应用被卸载
   3. "android.intent.action.PACKAGE_INSTALL"仅仅作为预留字段
   4. 想让action事件生效 还需要 配置一个data
   ```

3. 在`AppStateReceiver`的`onReceive`方法中对接收到的广播进行处理

   ```java
   //当有新的应用被安装  了  或者有应用被卸载 了  这个方法调用 
   	@Override
   	public void onReceive(Context context, Intent intent) {
   
   		//获取当前广播事件类型 
   		String action = intent.getAction();
   
   		
   		if ("android.intent.action.PACKAGE_INSTALL".equals(action)) {
   			System.out.println("应用安装了11111");
   		}else if ("android.intent.action.PACKAGE_ADDED".equals(action)) {
   			System.out.println("应用安装了22222");
   		}else if("android.intent.action.PACKAGE_REMOVED".equals(action)){
   			System.out.println("应用卸载了"+intent.getData());
   		}
   		
   1. intent.getData()获取应用的包名
   ```

### 六 开机启动

1. 创建广播接收者

   ```java
   public class BootReceiver extends BroadcastReceiver
   {}
   ```

2. 在AndroidMainfest.xml进行声明

   ```xml
    <receiver android:name="com.itheima.bootreceiver.BootReceiver">
               <intent-filter >
                   <action android:name="android.intent.action.BOOT_COMPLETED"/>
               </intent-filter>
   </receiver>
   ```

3. 添加"接收开机"的权限

   ```xml
   <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
   ```

4. 在`BootReceiver`的`onReceive`方法中对接收到的广播进行处理

   ```java
   //当手机重新启动的时候调用 
   	@Override
   	public void onReceive(Context context, Intent intent) {
   		//在这个方法里面开启activity
   		Intent intent2 = new Intent(context,MainActivity.class);
   		//☆☆☆注意 不能在广播接收者里面直接开启activity  需要添加一个标记 添加一个任务栈的标记 
   		intent2.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
   		//开启activity
   		context.startActivity(intent2);
   	}
   1. 不能在广播接收者里面直接开启activity, 需要添加一个任务栈的标记 
   2. 这样一开机就出现一个界面，后面再把各种返回禁用，则变成流氓软件
   ```

5. 在activity中禁用返回

   ````java
   @Override
   public void onBackPressed() {
   //	super.onBackPressed();
   }
   
   1. 禁用返回功能
   ````


### 七 有序广播和无序广播

+ 有序广播: 类似中央发红头文件，按照一定的优先级进行接收

  1. 在A应用中发送自定义的有序广播

     ```java
     //点击按钮 发送有序广播  发大米
     	public void click(View v) {
     		Intent intent = new Intent();
     		intent.setAction("com.itheima.sendrice");
     		/**
     		 * intent  意图 '
     		 * 
     		 * receiverPermission   接收的权限 
     		 * 
     		 * resultReceiver 最终的广播接收者
     		 * 
     		 * scheduler  handler 
     		 * 
     		 * initialCode  初始码 
     		 * initialData  初始化数据 ["习大大给每个村民发了1000斤大米"]
     		 */
     		sendOrderedBroadcast(intent, null, new FinalReceiver(), null, 1, "习大大给每个村民发了1000斤大米", null);
     	}
     
     1. 最终的广播接收者不必在配置文件中进行声明
     2. 将数据放在initialData位置，传递给广播接收者。可以在广播接受者中通过getResultData()方法取出来
     3. 即使是通过abortBroadcast()中断了有序广播的传递，最终的广播接收者也能接收到广播
     ```

  2. 在A应用中或B应用中定义对应的有序广播接收者

     ```xml
     //在xml配置
     
      <!-- 配置省长receiver  优先级最高  -->
             <receiver android:name="com.itheima.receiverice.ProvienceReceiver">
                 <intent-filter android:priority="1000">
                     <action android:name="com.itheima.sendrice"/>
                 </intent-filter>
             </receiver>
             
              <receiver android:name="com.itheima.receiverice.CityReceiver">
                 <intent-filter android:priority="500">
                     <action android:name="com.itheima.sendrice"/>
                 </intent-filter>
             </receiver>
             
               <receiver android:name="com.itheima.receiverice.CountryReceiver">
                 <intent-filter android:priority="100">
                     <action android:name="com.itheima.sendrice"/>
                 </intent-filter>
             </receiver>
             
              <receiver android:name="com.itheima.receiverice.NongMinReceiver">
                 <intent-filter android:priority="-100">
                     <action android:name="com.itheima.sendrice"/>
                 </intent-filter>
             </receiver>
     
     1. 各个有序的广播接收者，根据优先级的大小决定接收广播的顺序
     ```

  3. 各个有序的广播接收者的实现

     ```java
     
     public class ProvienceReceiver extends BroadcastReceiver {
     	@Override
     	public void onReceive(Context context, Intent intent) {
     		//[1]获取发送广播携带的数据
     		String content= getResultData();
     		//[2]显示结果
     		Toast.makeText(context, "省:"+content, 1).show();
     		//[2.1]终止广播 
     		//abortBroadcast(); //直接将后面的广播中止
     		//[3]修改数据 
     		setResultData("习大大给每个村民发了500斤大");
     	}
     }
     
     public class CityReceiver extends BroadcastReceiver {
     	@Override
     	public void onReceive(Context context, Intent intent) {
     		//[1]获取发送广播携带的数据
     		String content= getResultData();
     		//[2]显示结果
     		Toast.makeText(context, "市:"+content, 1).show();
     		//[3]修改数据 
     		setResultData("习大大给每个村民发了200斤大");
     	}
     }
     
     
     public class CountryReceiver extends BroadcastReceiver {
     	@Override
     	public void onReceive(Context context, Intent intent) {
     		//[1]获取发送广播携带的数据
     		String content= getResultData();
     		//[2]显示结果
     		Toast.makeText(context, "乡:"+content, 1).show();
     		//[3]修改数据 
     		setResultData("习大大给每个村民发了10斤大");
     	}
     }
     
     public class NongMinReceiver extends BroadcastReceiver {
     
     	@Override
     	public void onReceive(Context context, Intent intent) {
     		//[1]获取发送广播携带的数据
     		String content= getResultData();
     		//[2]显示结果
     		Toast.makeText(context, "农民:"+content, 1).show();
     	}
     }
     
     
     public class FinalReceiver extends BroadcastReceiver {
     	@Override
     	public void onReceive(Context context, Intent intent) {
     		//[1]获取发送广播携带的数据
     		String content= getResultData();
     		//[2]显示结果
     		Toast.makeText(context, "报告习大大:"+content, 1).show();
     	}
     }
     
     1. 通过abortBroadcast()可以中断广播的传递
     2. 可以通过setResultData()修改传递给下一个广播接收者的数据
     3. 中止后如果修改了数据，那么最终的接收者收到的数据是修改后的数据
     
     ```

  4. 在A中发送广播，此时A或B中定义的广播接受者有序接收到消息

+ 无序广播: 比如新闻联播，在每天7点准时开播

  1. 在A应用中发送自定义的无序广播

     ````java
     //点击按钮 发送一条无序广播 
     	public void click(View v) {
     		Intent intent = new  Intent();
     		intent.setAction("com.itheima.custom");
     		intent.putExtra("name", "新闻联播每天晚上 7点准时开整!!");
     		sendBroadcast(intent);//发送无序广播
     	}
     ````

  2. 在A应用中或B应用中定义对应的广播接收者

     ```xml
     //在xml配置
     <receiver android:name="com.itheima.receivewuxubroadcast.ReceiveCustomReceiver">
                 <intent-filter >
                     <action android:name="com.itheima.custom"/>
                 </intent-filter>
     </receiver>
     ```

     ```java
     //定义广播接收者
     public class ReceiveCustomReceiver extends BroadcastReceiver {
     	//当接收到我们发送的自定义广播
     	@Override
     	public void onReceive(Context context, Intent intent) {
     		//[0]终止广播 无效
     		//abortBroadcast();
     		//[1]获取发送广播携带的数据 
     		String content = intent.getStringExtra("name");
     		Toast.makeText(context, content, 1).show();
     	}
     }
     1. abortBroadcast不能中止无序广播
     ```

  3. 在A中发送广播，此时A或B中定义的广播接受者接收到消息

+ 有序广播和无序广播的区别

  1. 有序广播可以被中止，有序广播的数据可以被更改
  2. 无序广播不可以被中止，数据不可以被修改


### 八 特殊的广播接收者

操作特别频繁的广播事件，如屏幕的锁屏和解锁，电量的变化等。这种事件在清单文件中注册无效，需要通过代码进行动态注册

注册广播接收者的两种方式

1. 动态注册:通过代码的方式进行注册
2. 在清单文件中通过receiver节点进行注册



动态注册屏幕锁屏和解锁接收者

1. 在activity中，注册屏幕锁屏和解锁接收者

   ```java
   //动态的去注册广播接收者 
   screenReceiver = new ScreenReceiver();		
   //创建IntentFilter 对象
   IntentFilter filter = new IntentFilter();
   //添加要注册的action
   filter.addAction("android.intent.action.SCREEN_OFF");
   filter.addAction("android.intent.action.SCREEN_ON");
   //动态注册广播接收者
   registerReceiver(screenReceiver, filter);
   
   1. 通过IntentFilter添加action
   2. 通过registerReceiver方法进行注册
   ```

2. 实现屏幕锁屏和解锁广播接收者的业务处理

   ````java
   public class ScreenReceiver extends BroadcastReceiver {
   	//当我们进行屏幕锁屏和解锁 这个方法执行 
   	@Override
   	public void onReceive(Context context, Intent intent) {
   		//获取当前广播的事件类型 
   		String action = intent.getAction();
   		if("android.intent.action.SCREEN_OFF".equals(action)){
   			System.out.println("屏幕锁屏了 ");
   		}else if ("android.intent.action.SCREEN_ON".equals(action)) {
   			System.out.println("屏幕解锁了");
   		}		
   	}
   }
   ````

3. activity销毁的时候，取消注册

   ````java
   //当对象销毁的时候要取消注册广播接收者 
   unregisterReceiver(screenReceiver);
   ````



### 注意

1. 在4.0及以上，谷歌工程师要求，第一次安装应用必须有界面。这样广播接收者才能生效。(可以第一次安装后，删除清单文件中的Main/Launch后，再次编译安装，达到没有界面的效果)
2. 在设置页面有一个强行停止(force stop)的按钮， 如果用户点击了，那么广播接收者也不生效了。
3. 在2.3没有这样的设计













