### BroadCast广播
广播也是Android四大金刚之一，用途就是向所有用户传递信息
1. 系统广播：开关机广播、低电量广播、短信广播...
    - 定义一个广播的接收者，然后让这个接收者的类继承Broadcast
    - 重写这个类里面的onReceive这个方法
    - 按照Android四大组件的定义，需要声明这个广播常驻内存/动态广播
    - 系统广播一般情况下都需要添加权限
2. 自定义广播：

#### 广播的分类：
1. 常驻内存的广播(在mainifest.xml文件中声明的)
只要安装了这个软件，不受限于是否启动，只要发送了action匹配的广播，就会收到。
2. 动态广播：
  在需要动态广播的地方注册步骤如下：
1：new一个广播的对象
2：new一个过滤器IntentFilter
3：在过滤器中传入需要监听的广播类型intentFilter.addAction
4：注册广播接收者registerReceiver将广播对象和过滤器传过去
5：重写onDestroy方法解除绑定：unRegisterReceiver
```
                MyScreenOpenOrCloseBroadcas myScreenOpenOrCloseBroadcas=new MyScreenOpenOrCloseBroadcas();
                IntentFilter mIntentFilter=new IntentFilter();
                mIntentFilter.addAction(Intent.ACTION_SCREEN_ON);
                mIntentFilter.addAction(Intent.ACTION_SCREEN_OFF);
                registerReceiver(myScreenOpenOrCloseBroadcas,mIntentFilter);
```
***广播的注册与取消注册必须是成对出现的，不然业务中会报错。***
### 自定义广播：
1. 首先需要写一个广播接受者
```
public class MyBroadcast extends BroadcastReceiver{

    @Override
    public void onReceive(Context context, Intent intent) {
        String name = intent.getStringExtra("name");
        String age = intent.getStringExtra("age");
    }
}
```
2. 需要确定是定义在常驻内存的广播还是动态广播
常驻内存的广播声明在mainifest.xml中
```
 <receiver android:name="MyBroadcast">
            <intent-filter>
                <action android:name="com.fox.demo.mybroadcast"/>
            </intent-filter>
</receiver>
```
动态广播声明在Activity中
```
IntentFilter intentFilter=new IntentFilter();
intentFilter.addAction("com.fox.demo.broadcast");  //一个接收者可以接收多个action
intentFilter.addAction("com.fox.demo");
 registerReceiver(myBroadcast,intentFilter);//注册广播
```
3. activity中使用Intent来发送广播，接收者中使用intent.getStringExtra获取发送的数据
```
Intent mIntent = new Intent();
mIntent.setActiojn("com.fox.demo);
sendBroadcast(mIntent);
```
4. 关闭Activity的时候必须解绑广播接收者
``` unregistReceiver(myBroadcast);```
***注意：一个广播可以有多个Action，一个Action可以有多个广播。***
#### 广播的分类：有序广播和无序广播
无序广播：广播传输的时候是异步的，可以同时向多个接收器发送数据。
有序广播：广播的数据向下进行传输的时候是按照广播排列的顺序依次向下传输的。
#### 有序广播的优先级：
优先级相同的情况下，动态注册大于静态注册（也就是排列顺序在前面），先安装大于后安装。
eg.C的优先级大于B，在C中setResultData，B中通过getResultData可以接收到。
通过abortBroadast可以终止广播，有序广播就不会再继续往下发送。
#### 粘性广播与非粘性广播
普通广播的onReceive方法里面的传值超过10秒钟就会出现ANR，也就是不能在里面执行耗时的操作。
粘性广播：超出了普通广播的生命周期的范畴，他会在内存中找到一块区域来存储当前的Intent对象，当条件成熟时会自动重新发送。
#### 解决广播出现跨域的问题：
1：自己发出去的广播只允许自己能接收
2：自己的接收器只接收自己的数据
解决方案就是给广播定义权限。
发送的普通广播，所有应用都能收到。
发送的权限广播，表示发送给拥有指定权限的广播，没有该权限就收不到。
