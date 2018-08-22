### 一、Handle原理图
![handle流程图.png](https://upload-images.jianshu.io/upload_images/7156039-2b08cbc0bf3fa32b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
通过上面绘制的流程图，我们其实可以知道handle理解起来不是很难。下面具体讲解一下使用流程
### 二、Handler、Message、MessageQueue、Looper方法及属性
1. Handler.java:（3个属性，10个方法）
3个属性:
```
final MessageQueue mQueue；封装好的消息被handler发送出去，其实就是将消息放到消息队列中去
final Looper mLooper；每一个handler都有一个looper为其不断接收消息队列中的消息，并返回给handler。
final Callback mCallback;    被handler接收到的消息要通过Callback接口中的handleMessage()方法来进行处理。

```
10个方法：
```
public boolean handleMessage(Message msg);    Callback接口中的handleMessage()方法，用来处理返回给handler的消息的。
public final Message obtainMessage();    获取Message对象，其本质还是调用Message类的obtain()方法来获取消息对象。
public final boolean sendMessage(Message msg)      发送消息
public final boolean sendEmptyMessage(int what)     发送只有what属性的消息,就是发送空白消息
public final boolean post(Runnable r)                          发送消息
public final boolean postAtTime(Runnable r, long uptimeMillis)         定时发送消息
public final boolean postDelayed(Runnable r, long delayMillis)         延迟发送消息
public void dispatchMessage(Message msg)   分发消息，当Looper循环接收消息队列中的消息时，就在不断调用handler的分发消息方法，从而触发Callback接口中的消息处理方法handleMessage()来对消息进行处理。
public final boolean sendMessageDelayed(Message msg, long delayMillis)     sendMessage()就是在调用延时发送消息的方法。
public boolean sendMessageAtTime(Message msg, long uptimeMillis)     延时发送消息的方法就是在调用定时发送消息的方法。
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis)     将消息压入消息队列中
```
2. MessageQueue.JAVA:（2个方法）
```
final boolean enqueueMessage(Message msg, long when)    handler发送的消息正是通过该方法被加进了消息队列中。因为消息队列中有消息，从而触发了Looper通过loop()方法循环接收所有的消息。
final Message next();
```
3. Looper.JAVA:（1个属性，5个方法）
3个属性
```
 static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();   正是由于该线程本地变量，保证了一个线程中只能保存一个Looper对象，也就是说一个Thread中只能有一个Looper对象。
```
5个方法：
```
public static void prepare()            每个handler所在的线程都必须有一个Looper提前准备好。
public static void prepareMainLooper()         主线程中的的Looper已经被自动准备好。而该方法不需要手工调用。
public static Looper getMainLooper()            获取主线程中的Looper对象
public static void loop()                 Looper的作用就是循环接收消息队列中的消息。该方法中执行了一个无限循环。
public static Looper myLooper()       该方法的作用是从线程本地变量中获取到当前线程的Looper对象。
```
4. Message.java:（10个属性，7个方法）
10个属性：
```
public int what;     该属性一般用来标识消息执行的状态。
public int arg1;     用来存放整型数据的属性。
public int arg2;     用来存放整型数据的属性。
public Object obj;    用来存放复杂消息数据的属性。
Bundle  data;          用来存放复杂消息数据的属性。
Handler target;       标识该消息要被发送给哪个Handler对象。
Message sPool;        消息池中的每个消息对象。
int sPoolSize;         记录消息池中剩余消息的数量。
int MAX_POOL_SIZE=50;     消息池中最大消息数量。
Messenger replyTo;        定义消息的信使对象，将来可以用于跨APP的消息传递。
```
7个方法：
```
public static Message obtain()       从消息池中获取消息对象。
public void recycle()                往消息池中归还消息对象。
public void setTarget(Handler target)     给消息设置接收该消息的目标handler对象。
public Handler getTarget()          获取消息的接收handler对象。
public void sendToTarget()             将消息发送到目标handler对象。其本质是调用handler对象的sendMessage()方法来发送当前消息对象。
public void setData(Bundle data)     将Bundle对象设置到message对象中Bundle属性中。
public Bundle getData()      从消息对象中获取Bundle属性的数据。
```
### 消息传递机制中各方法之间的执行顺序
![消息传递顺序.png](https://upload-images.jianshu.io/upload_images/7156039-3ebc703fbb41dbfc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 结语
&emsp;&emsp;对于handler其实没有大家想象的那么难，在平时的工作中，我们使用的时候非常多，经常使用就会有不一样的理解。最后，欢迎大家点赞和提问哦！
