### Service服务
&emsp;&emsp;Android四大金刚之一，不依赖与UI和组件，可以独立的运行在后台。
#### Service的使用：
##### 一、startService
1. 创建一个类继承于Service
2. 重写Service里面的onStart或者onStartCommand，已经onCreate、onBind、onDestory方法
```
public class MyService extends Service {
    public MyService() {
    }

    @Override
    public void onCreate() {
        super.onCreate();
    }

    @Override
    public IBinder onBind(Intent intent) {
        // TODO: Return the communication channel to the service.
        throw new UnsupportedOperationException("Not yet implemented");
    }

    /**
     * 处理业务逻辑的地方
     * @param intent
     * @param flags
     * @param startId
     * @return
     */
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        return super.onStartCommand(intent, flags, startId);
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
    }
}
```
3. 使用startService打开服务
```
        Intent intent = new Intent(MainActivity.this, MyService.class);
        startService(intent);
```
4. 需要在mainifest中声明
```
        <service android:name=".MyService"/>
```
5. 关闭Service
```
stopService(intent)
```
Service的优先级：Service的优先级比不可见的Activity的优先级高，比可见的Activity优先级底。

##### 二、BindService
&emsp;&emsp;绑定模式下的Service会依赖于组件，非绑定模式下的Service不依赖于组件。
1. 编写一个类继承于Service
```
public class MyBinderStartService extends Service{
    Binder binder=new MyBinder();
    public int number=0;
    public boolean isLooping=true;
```
2. 在Service里面编写一个类继承于Binder
3. 在Binder类里面编写一个方法返回当前这个Service的实例:Service.this
```
public class MyBinder extends Binder{
    public MyBinderStartService getService(){
        return MyBinderStartService.this;
    }
}
```
4. 在Service的onBind方法里面返回自定义的这个类的实例
```
@Override
public IBinder onBind(Intent intent) {
    Log.e("---------------------","onBind");
    return binder;
}
```
5. 定义业务逻辑的方法
```
@Override
public int onStartCommand(Intent intent, int flags, int startId) {
    Log.e("---------------------","onStartCommand");
    new Thread(new Runnable() {
        @Override
        public void run() {
            while (isLooping){
                try {
                    Thread.sleep(500);
                    Log.e("----------------------",(number++)+"");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }).start();
    return 0;
}
```
6. 在需要使用这个Service的组件中来进行调用
    - 首先需要准备Intent
``` Intent mIntent=new Intent(MainActivity.this, MyBinderStartService.class);```
    - 编写一个类实现于ServiceConnection这个类并且实现里面的onServiceConnected和onDisServiceConnected
    - 调用BindService(intent,MyServiceConn,flag)
```
/**
 * 连接那个Service
 */
private class MyServiceConn implements ServiceConnection{
    @Override
    public void onServiceConnected(ComponentName name, IBinder binder) {
        MyBinderStartService.MyBinder myBinder=(MyBinderStartService.MyBinder)binder;
        myBinderStartService = myBinder.getService();
        Log.e("---------------","绑定成功........");
    }

    /**
     * 绑定零食出现问题或者  被系统杀死才会调用这个方法
     * @param name
     */
    @Override
    public void onServiceDisconnected(ComponentName name) {
       Log.e("----------------","我被杀死...............");
    }
}
```
绑定Service
```
bindService(mIntent,myServiceConn,BIND_AUTO_CREATE);
```
7. 在需要调用事件的地方调用Service方法。

***注意：***BindService这个方法是异步的，如果在绑定的同时立马执行调用方法，不会成功，BindService的stop方法是没用的，当解除绑定时会调用onDestroy方法，Service就死了。
##### 三、IntentService
&emsp;&emsp;也是一个Service，在IntentService里面封装了异步任务(不需要new线程)，在运行完所有任务之后会自动终止Service的运行。
#### IntentService的使用：
1. 自定义一个类继承于IntentService，并重写构造函数
```
public class MyIntentSevice extends IntentService {

    public MyIntentSevice() {
        super("小子");
    }
```
2. 重写里面的onHandleIntent方法
onHandleIntent：系统new好的异步任务的回调，可以完成耗时操作。
```
 /**
     * 必须重写这个方法:系统给你new好的异步任务的回调
     * 这个里面可以完成那个耗时的操作
     * @param intent
     */
    @Override
    protected void onHandleIntent(Intent intent) {
        try {
            Thread.sleep(5000);  //用睡眠来模拟那个耗时的操作
            Log.e("------------------",Thread.currentThread().getName());
            Log.e("---------------","我执行完成了.....");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
```
3. 在mainifest中声明
4. 在Activity中打开服务
```
        Intent intent = new Intent(MainActivity.this, MyBindService.class);
        startService(intent);
```
### Notification通知
1. 通过NotificationManger获取管理者对象
```
        NotificationManager mNotificationManager=null;
        //第一步:通过NotificationManager获取那个管理者对象
        mNotificationManager= (android.app.NotificationManager) this.getSystemService(Context.NOTIFICATION_SERVICE);
```
2. 使用Notification的Builder获取一个builder对象
```
Notification.Builder builder = new Notification.Builder(IntentServiceActivity.this);
```
3. 通过Builder对象设置相应的值
```
        //通过builder对象设置相应的值
        mBuilder.setContentTitle("天气预报");   //设置标题
        mBuilder.setContentText("城市的天气预报正在掌握中...");  //设置那个提示的内容
        mBuilder.setSmallIcon(R.mipmap.ic_launcher);    //设置前面的那个图标
        mBuilder.setAutoCancel(true);   //点击了那个通知之后那个通知是要消失的
```
4. 显示通知
```

        Notification mNotification=mBuilder.build();
        mNotificationManager.notify(34,mNotification);
```
5. 可以通过id取消单个通知，也可以通过cancelAll取消同一个通知管理者下的所有通知。
```
                mNotificationManager.cancel(34);
                mNotificationManager.cancelAll();
```
