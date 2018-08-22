### Activity的四大金刚（组件）：
&emsp;&emsp; Activity、Service、Broadcast-Receiver、Content-Provider
*** 
前面我们提到了控件的概念，今天就来解开关于Android中的四大组件的面纱。
### 四大组件都是用来干嘛的？
1、Activity：主要是用来完成页面的显示以及页面的业务逻辑的处理  
2、Service：服务，一般运行在后台。用于处理一些逻辑问题。一般音乐播放器的播放基本都放在这里面。  
3、Broadcast：广播，分很多种。一般都是用来传递消息的，可以软件内部传，也可以软件与软件传。  
4、ContenProvider：暴露自身软件的数据给其他软件，可以通过这个访问系统的资源。比如电话本，短信这些东西。  
### Activity的生命周期：
咳咳。敲黑板，划重点了！这里一定要好好的理解一下，工作了之后才真的深刻理解到生命周期的重要性。所有的操作都要在对应的生命周期里面去做，不然就会很容易出错。
#### Activity有哪些生命周期方法呢？
1、onCreate()：这个activity生命周期的第一个方法。用来完成页面的初始化操作的。setContentView()用来获取保存的临时数据（有时候activity不是正常的退出，根据业务需求会恢复数据数据）  
2、onStart()：这个是生命周期的第二个方法，执行完毕的时候，程序已经显示出来，用户可见但是不能点击交互。  
3、onResume()：这个第三个执行的方法，执行完毕的时候，程序已经可见并且获取到了焦点。用户可以进行点击交互了。  
4、onPause():这是第四个执行的方法，也是activity死亡生命周期的第一个方法。和onResume()生命周期方法是成对出现的。这个方法表示程序还依然看见，但是用户已经不能够进行点击交互的操作了。  
5、onStop()：死亡生命周期的第二个方法，和onStart()是对应的。表示程序已经不可见了。  
6、onDestory()：死亡生命周期的最后一个方法。和onCreate()对应。释放当前activity所占用的所有资源。  
#### Activity的几种生命周期执行场景
1、Activity的自生自灭  
```
onCreate()-->onStart()-->onResume()-->onPause()-->onStop()-->onStop()-->onDestory()
```
2、开启A activity，跳转B activity，返回 A activity   
```
A onCreate()-->A onStart()-->A onResume()-->A onPouse()-->B onCreate()-->B onStart()-->B onResume()-->A onStop()-->
返回键-->B onPause()-->A onRestart()-->A onStart()-->A onRsume()-->B onStop()-->B onDestory()-->
返回键-->A onPsuse()-->A onStop()-->A onDestory()
```
#### 限制Activity横竖屏
```
android:screenOrientation="portrait"//限制这个页面竖屏 "landscape"是限制此页面横屏显示
```
#### 应用横竖屏切换的时候Activity的生命周期问题？
```
android:configChanges="orientation" //限制横竖屏切换，这样生命周期就不会变换
```
&emsp;&emsp;如果不声明也不设置的话，那么当我们的应用在进行横竖屏切换的时候就会将我们当前显示的这个Activity出栈，然后再启动这个Activity。这样的话，就会出现问题。不断的调用当前Activity的生命周期，这样在游戏或者视屏播放的时候，就会出现问题。
#### Activity的创建
讲了这么久的Activity，还没有给回顾到怎么创建Activity（手动捂脸！）。创建Activity有两种方法：  
1、编写一个类继承于Activity，然后重写Activity里面的onCreate()方法，注意这里面含有一个bundle参数。
```
  protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
	}
```
bundle是个啥？一个内部维护的HashMap对象的一个集合的引用  简单点说就是:内部有一个双列集合我们可以操作双列集合来达到操作Bundle的效果  Bundle---->集合  
2、直接通过编译器创建一个
#### Activity页面之间的跳转：
前面有提到A activity跳转到B activity。那么activity之间是怎么跳转的呢？这时候就该Intent出场了。Intent是四大组件之间的粘合剂，四大组件之间的一个桥梁，没有intent，四大组件之间就是独立的没有任何关系的。
#### Intent实现页面的跳转
```
//Inten的两种构造方式
//1 、带参构造
Inatent intent=new Intent(是当前的上下文对象,后面这个是跳转的页面的class对象);
//2 、无参构造
Intent intent=new Intent();

intent.setClass(getApplicationContext(),SecondActivity.class);   //包的上下文指的是应用的上下文  要跳转页面的class对象 
intent.setClassName(getApplicationContext(),"com.example.activitylife.SecondActivity") // 后面的这个类名必须是包名+类名
intent.setClassName("com.example.activitylife","com.example.activitylife.SecondActivity");  //这种方式可以打开另外一个应用里面的页面
intent.setComponent(new ComponentName("com.example.activitylife","com.example.activitylife.SecondActivity"));  //称为显示的打开一个Acticity 通过组件的形式来进行打开
intent.setComponent(new ComponentName("com.example.androidlife1","com.example.androidlife1.MainActivity"));    
intent.setAction("xiaobobo");       //给当前的Activity设置一个Action  我们可以通过Action来找到当前的Activity
```
注意:如果给应用设置了Action的属性，那么在Activity的申明中必须要加上<category android:name="android.intent.category.LAUNCHER" /> 属性
#### Activity之间的传值问题：
1、从A activity传值给B activity，使用intent传值。   
&emsp;&emsp; a、直接通过intent.putExtra("key",value)的形式进行传值  
&emsp;&emsp; b、在B页面通过getIntent()拿到intent对象，通过intent.getExtra("key",default)获取值（）  
2、从A Activity跳转到B Activity 再跳转到C Activity  
&emsp;&emsp; a、打开Activity的时候只能使用startActivityForResult(intent,requestCode)来打开  第二个值是请求码，可以自定义，后面会根据请求码获取回归数据  
&emsp;&emsp; b、重写onActivityResult()方法 来接收死亡的Activity传回来的数据  
&emsp;&emsp; c、在需要传回数据的地方，使用intent来对数据进行封装 然后再使用setResult(intent)来将数据回传回来。  
&emsp;&emsp; d、在onActivityResult里面解析数据  
*** 
注意：四大组件使用必须要在AndroidManifest.xml文件中进行声明。四大组件都是属于应用，所以我们需要在Application的内部进行声明。而所有的权限是正对应整个工程的，所以需要在Application之外来进行声明。
为啥要声明？ 告诉Android系统这个应用软件里面包含了哪些组件。
#### 声明Activity的格式
```
  <activity
            android:name="com.example.activitylife.MainActivity"    //Activity的名字
            android:label="@string/app_name" >                      //Activity的标签(可以有也可以木有)
     <intent-filter> //过滤器(把含有下面配置的Action给过滤出来)
           <action android:name="android.intent.action.MAIN" /> //动作
           <category android:name="android.intent.category.LAUNCHER" /> 类别  //表示的是建立那个Lancher的图片在Lancher里面是可以启动的
     </intent-filter>
 </activity>
```
一般来说配置四大组件不需要配置这么多，只需要配置名称和启动模式就OK了，除非有特使的业务需求。
