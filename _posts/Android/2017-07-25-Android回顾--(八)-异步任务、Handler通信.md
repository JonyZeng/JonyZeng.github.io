### 异步任务：
什么是异步任务？一切使用多线程完成的任务均可以称为异步任务-->使用线程来完成任务。
#### 异步任务的优缺点
优点：1、提高CPU的利用率。 2、在I/O受限等情况下，异步能提升软件的速度。3、能够增强系统的健壮性。4、异步能改善用户体验  
缺点：1、加重了CPU的负担。2、容易出现线程安全问题。3、容易造成死锁。  
#### 使用异步任务的必要性
Android虚拟机是如何运行程序的？   
&emsp;&emsp;当Android的虚拟机在调用Activity对象完成压栈的时候，将当前的对象运行在了一个叫MainThread的主线程中，这个线程又叫UI线程。  
&emsp;&emsp;UI线程这个名字是如何得来的呢？因为主线程只要一个特定的工作就是刷新UI，这个线程的主要工作就是管理UI。  
#### 在主线程中如何刷新UI
&emsp;&emsp;主线程刷新UI实际上是通过子线程向主线程通信而已。线程之间使用的是Handler来进行通信。  
```
//1、在需要发送数据的地方，通过handler建立一个数据发送点,一般都是在子线程中。
handler.sendEmptyMessage(100);
Message msg = new Message();
msg.what=222;
msg.arg1=203040;
msg.arg2=1101010;
msg.obj=new String("hell Word");
Bundle bundle = new Bundle();
bundle.putBoolean("key1",true);
msg.setData(bundle);
handler.sendMessage(msg);

//2、在需要接收数据的地方建立一个数据接收点。
Handler handler= new Handler(){
    @Override
    public void handleMessage(Message msg) {
        super.handleMessage(msg);
            switch (msg.what){
                case 100:
                    Log.i(TAG,"收到了消息");
                    break;
                case 222:
                    int k=msg.arg1;
                    int k1=msg.arg2;
                    String k2= (String) msg.obj;
                    Bundle bundle = msg.getData();
                    boolean b = bundle.getBoolean("key1");
                    break;
            }
    }
};
```
#### Android的单线程模型指的是什么?
指的是UI线程，也就是刷新UI只能在主线程来完成。耗时操作(网络访问，IO流操作，数据库访问)也必须在这个线程完成。这就是Android的单线程模型，只有一个主要的线程一直在执行。
#### 异步任务的用法，采用线程池，调用线程池中的线程来完成任务的执行。
#### AsyncTask中范型参数说明
1、Params: 执行任务的输入参数，也就是我们要执行这个任务的时候输入的参数。假设这个任务是下载图片，那么这个参数就是下载的URL地址，假设不需要参数的话，直接填写void就OK  
2、Progress：任务执行的进度，一般是int类型的，但在泛型中只能是integer，如果不需要直接给一个void。  
3、Result：执行完任务之后的回调数据的数据类型，如果不需要的话 那么直接给个void。  
#### AsyncTask中方法的说明
1、onPreExecute();通过源码分析知道这个家伙是运行在了主线程中，作用是干嘛？主要是用在下载之前初始化UI的，比如说提示用户的信息，或者初始化进度条。  
2、doInBackgrond(Void... params)：用在了线程中，通过实际的方法调用了抽象的方法来实现了回调。  
注意：一般情况下在doInBackground中执行的任务，不管在任务执行中抛出什么异常，我们都选择将其捕获，这样无论什么时候后面的onPostExcute方法总会被执行。  
3、onProgressUpdate(void..  values)：这个方法是用来更新进度条的，他的使用不会自动来进行调用，是通过另外的方法触发的。  
4、onPostExecute(Result result )：这个方法是异步任务最后执行的方法， 在这个方法中有一个返回值，这个值来源于doInbackground的返回值。  
注意：如果DoInBackground中出现了异常没有捕获的话，那么onPostExcute()方法不会被执行。在使用AsyncTask进行任务的时候，并不是我们需要覆盖全部的方法，是根据业务逻辑来确定的。但是DoInBackground这个方法是必须实现的，因为这是抽象类的抽象方法。  
![AsyncTask.png](https://upload-images.jianshu.io/upload_images/7156039-430f2b1b9c5ea661.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 异步任务与线程池
对于Android系统而言4.4以前的版本，允许一次性最多允许5个线程，内部定义了最多在线程池中存在的线程数是128。对于4.4以后的版本，线程最多运行的线程数是根据当前CPU的性能来动态决定的。 当前线程池的最大线程数也是动态决定的。
```
ThreadPoolExcute excutor = new ThreadPoolExecutor(20, 100,1,TimeUnit.SECONDS,new LinkedBlockingQueue<Runnable>());
```
#### 异步任务的取消
myAsyncTask.cancel(true)  这个方法和线程中的interrupt方法是一样的能够终止正处于阻塞状态的线程  
一般情况下取消那个异步任务的话那么需要优雅的去结束   跟线程一样
