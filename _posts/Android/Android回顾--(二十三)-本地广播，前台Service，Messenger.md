### 发送本地的广播：
&emsp;&emsp;可以限制广播接受者必须是应用里面的接收器
1. 获取LocalBroadcastManager对象
```
LocalBroadcastManager mLocalBroadcastManager=null;
MyBroadcast myBroadcast=null;
myBroadcast=new MyBroadcast();
//首先第一步应该获取那个LocalBroadcatManager这个对象
mLocalBroadcastManager=LocalBroadcastManager.getInstance(LocalBroadcastActivity.this);
```
2. 注册广播
```
IntentFilter mIntentFilter=new IntentFilter();
mIntentFilter.addAction("hello.china");
mLocalBroadcastManager.registerReceiver(myBroadcast,mIntentFilter);
```
3. 发送广播

```
    Intent intent=new Intent();
    intent.setAction("hello.china");
    mLocalBroadcastManager.sendBroadcast(intent);
```
### 前台的Service
定义一个Service
```
/* *
 * 前台Service
 */
public class MyGroundService extends Service {
    @Override
    public void onCreate() {   //只是会执行一次
        //来完成前台的Servie的初始化工作
        //咋们来创建一个前台的Service
        Notification.Builder mBuilder=new Notification.Builder(this);
        mBuilder.setSmallIcon(R.mipmap.ic_launcher);
        mBuilder.setContentTitle("前台的Service");
        mBuilder.setContentText("我不会被系统给杀死");
        Notification mNotification=mBuilder.build();
        startForeground(12,mNotification);
//        stopForeground();
    }
    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.e("-----------------","我被杀死了....你保重...");
    }
    public MyGroundService() {
    }
    @Override
    public IBinder onBind(Intent intent) {
      return null;
    }
}
```
打开Service
```
Intent intent=new Intent(ServiceGroundActivity.this, MyGroundService.class);
startService(intent);
```
#### Messager实现进程间的通信：
messager:互相通信的一种机制
Handle：消息的传输者
Message：需要传递的消息
Messager进行通信的时候实际上也要依赖于Service
##### Service端
1. 建立一个接收信息的平台
2. 在onBind里面返回Messenger对象的getBinder()方法
```
public class MyServerService extends Service {
    @Override
    public IBinder onBind(Intent intent) {
       return mMessenger.getBinder();
    }
    /**
     * 咋们在这个里面只有一个工作就是搭建一个消息的接收平台
     */
    Messenger mMessenger=new Messenger(new Handler(){
        @Override
        public void handleMessage(Message msg) {
            //msg:就是那个客户端发送过来的消息
            int num=msg.arg1;
            Message msg1=Message.obtain(msg);
            msg1.arg1=num;
            //将这个消息给发送出去
            try {
                Log.e("-------------",msg+"");
                Log.e("-------------",msg1+"");
                msg.replyTo.send(msg1);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
            super.handleMessage(msg);
        }
    });
}
```
##### 客户端
1. 绑定Service
2. 建立消息的接收平台
3. 向服务端发送消息
```
public class MainActivity extends AppCompatActivity {
    Messenger mMessenger;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Intent intent=new Intent("hello");
        intent.setPackage("com.qf.mobilephone.myapplicationserver");  //这句话是为了让那个编译器不报那个显示调用这个问题
        bindService(intent,new MyServiceConn(),BIND_AUTO_CREATE);
    }
    /**
     * 消息的处理
     */
    public void myClick(View view){
        Message mMessage=Message.obtain();
        mMessage.arg1=10000;
        mMessage.replyTo=mmMessenger;//这个是告诉服务器客户端的接收者是谁
        try {
            mMessenger.send(mMessage);
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }
    /**
     * 这个呢就是那个接收的平台
     */
    Messenger mmMessenger=new Messenger(new Handler(){
        @Override
        public void handleMessage(Message msg) {
            Log.e("---------------kkk",msg.arg1+"");
            super.handleMessage(msg);
        }
    });
    /**
     * 这个就是那个连接服务端的方法
     */
    private class MyServiceConn implements ServiceConnection{
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            Log.e("---------------","我连接上了.....");
            mMessenger=new Messenger(service);
        }
        @Override
        public void onServiceDisconnected(ComponentName name) {
            mMessenger=null;
        }
    }
}
```
