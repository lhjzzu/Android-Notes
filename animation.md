## 动画

帧动画

View动画(补间动画)

属性动画

### 一 帧动画

由图片组成的动画

1. 定义动画

   ```xml
   1. 在res下创建drawable文件夹, 存放要进行动画的图片
   2. 创建定义动画的xml文件
   
   <?xml version="1.0" encoding="utf-8"?>
   <animation-list xmlns:android="http://schemas.android.com/apk/res/android"
       android:oneshot="false" >
       <item
           android:drawable="@drawable/girl_1"
           android:duration="200"/>
       <item
           android:drawable="@drawable/girl_2"
           android:duration="200"/>
       <item
           android:drawable="@drawable/girl_4"
           android:duration="200"/> 
   </animation-list>
   
   ```

2. 使用动画

   ````java
   @Override
   	protected void onCreate(Bundle savedInstanceState) {
   		super.onCreate(savedInstanceState);
   		setContentView(R.layout.activity_main);
   		// [1]找到控件 显示动画效果
   		final ImageView imageView = (ImageView) findViewById(R.id.iv);
   		// [2]设置背景资源
   		imageView.setBackgroundResource(R.drawable.my_anim);
   		//[2.1]兼容低版本的写法  
   		new Thread(){
               public void run() {
   			SystemClock.sleep(200); 
   			// [3] 获取AnimationDrawable 类型
   			AnimationDrawable rocketAnimation = (AnimationDrawable) imageView
   					.getBackground();
   			//[4]开启动画 
   			imageView.start();			
   		};}.start();
   	}
   
   1. 兼容低版本:延迟一会儿执行，等待资源准备好
   ````

### 二 View动画



### 三 属性动画