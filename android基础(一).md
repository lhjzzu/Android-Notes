### 1 样式和主题

1. style和Theme定义方式是一致的

2. 样式:一般作用于控件上(button, textView等) ， 样式作用范围比较窄
3. 主题:一般作用于activity或者application节点下，作用范围比较大

4. 创建路径: `主题/样式values->styles.xml->Resources->Style/Theme`

5. 示例

   ```xml
   1. 在styles.xml中定义
   <style name="my_style">
      <item name="android:layout_width">wrap_content</item>
      <item name="android:layout_height">wrap_content</item>
      <item name="android:textSize">40sp</item>
      <item name="android:textColor">#000000</item>
   </style>
   <style name="my_theme">
      <item name="android:background">#ff0000</item>
   </style>
   2. 在布局文件中使用样式
     <TextView
         style="@style/my_style"
           android:text="哈哈哈哈" />
       
        <TextView
         style="@style/my_style"
           android:text="呵呵呵呵" />
       
         <TextView
         style="@style/my_style"
           android:text="嘿嘿嘿" />
   3. 在清单文件中使用主题
       <application
            ...
           android:theme="@style/my_theme" >
            ...
       </application>
   
   ```



###2 国际化

1. 在res下定义values-en, value-zh文件夹，并创建对应的strings.xml文件，在各个strings.xml文件中，实现对应的国际化 （中英文为例）

   ```xml
   1. values-en->strings.xml
   <?xml version="1.0" encoding="utf-8"?>
   <resources>
       <string name="app_name">demo</string>
       <string name="action_settings">Settings</string>
       <string name="hello_world">Hello world!</string>
   </resources>
   
   2. values-en->strings.xml
   <?xml version="1.0" encoding="utf-8"?>
   <resources>
       <string name="app_name">示例</string>
       <string name="action_settings">设置</string>
       <string name="hello_world">你好世界</string>
   </resources>
   ```

2. 不同的语言，创建不同的values-xx

### 3 两种context区别

getApplicaitonContext()和Activity的this的区别在于，getApplicaitonContext()不能用于对话框

