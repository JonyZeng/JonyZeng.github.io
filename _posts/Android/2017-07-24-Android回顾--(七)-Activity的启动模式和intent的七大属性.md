### Task以及backTask
1、Task：任务-->Android程序里面的理解 一组Activity的对象的集合就称为Task  
2、BackTask：一组处于任务栈中的Activity的集合  
&emsp;&emsp;对于Android软件而言，Android软件的使用是依赖于Java来进行编程的，我们每次打开一个页面，实际上是这个Android虚拟机帮助我们创建一个Activity的类的对象，然后将咱们创建的这个对象按照一定的规律放入任务栈中。  
任务栈：也是栈，按照栈的存储特点：现进后出。  
&emsp;&emsp;一个Activity在创建对象放入到任务栈中的时候，首先会获取Activity的Affinity属性，如果当前的Activity拥有自身的Affinity属性的话，那么就按照我们Affinity属性所标记的任务栈，找到该任务栈将我们的Activity对象放到任务栈中，如果我们的Activity没有Affinity这个属性的话，那么就会去找当前Application的Affinity属性。如果Application也没有Affinity属性的话，那么就会将当前工程的包名作为Affinity属性来进行使用。如果Application拥有Affinity属性的话，那么就会将Application的Affinity属性当做任务栈的一个标记。  
注意：Affinity属性在配置文件中使用的时候，实际上使用的是taskAffinity这个属性来替代的。  
#### 关于任务栈，里面有三个属性需要关注一下
```
android:allowTaskReparenting="true"  允许对象进行移栈    当我们创建这个对象的时候首先会去栈里面找名字和当前这个Affnity属性相同的栈，然后将当前的对象放到栈里面
android:clearTaskOnLaunch="true"    当我们按下那个Home键的时候会将当前这个对象上面的所有对象全部出栈只是留下当前对象和对象以下的对象
android:taskAffinity="com.foxocnn." 这个表示的是将当前的对象放到指定标记的任务栈中
```
### Activity的启动模式：
&emsp;&emsp; 启动模式是在面试中经常问到的，在平时开发中，关于优化也会涉及到启动模式的选择。启动模式是用来描述Activity创建对象的时候，在任务栈里面的存在形式的。  
Android有四种启动模式：  
1、 android:launchMode="standard"   //标准的启动模式，每打开一个Activity就会在当前的任务栈里面创建一个对象，如果同一个页面打开多次的话，就会出现体验上满的问题。  
2、 android:launchMode="singleTop"  //在打开页面之前都会检查在当前任务栈的栈顶是否存在当前页面的对象 如果已经有这个对象了，就不会重复创建 ，直接拿出来使用。  
3、android:launchMode="singleTask"  //在打开Activity之前首先会检测当前的栈里面是否存在该类的对象，如果存在，那么就将当前这个存在的对象的上面的所有的对象全部出栈，显示当前对象。  
4、android:launchMode="singleInstance"  //如果一个Activity的属性使用了singleInstance的话，那么在创建这个类的对象的时候就会去任务栈里面找具有相同的打开模式并且存在相同Affinity属性的话，那么就减这个对象当知道该任务栈中。如果上面的两个条件有一个不满足的话，那么都会重新的打开一个任务栈来存储Activity的对象。  
常见的就是浏览器的应用、当我们一个应用打开浏览器之后如果浏览器没有关闭的话 此时如果另外一个浏览器也需要打开这个浏览器的话就会重用刚刚打开的对象。  
#### singleInstance容易出现的两个坑：
1、定义三个Activity，分别为A Activity、B Activity、C Activity，除了B Activity的启动模式为singleInstance，其他都是默认的standard。应用通过startActivity开启了一个A Activity，在A Activity里面startActivity了一个B Activity，在B Activity里面startActivity了一个C Activity。正常情况下，当前的任务栈中的顺序应该是：A Activity-->B Activity-->C Activity。那么在当前C Activity页面按返回键，finish当前页面之后应该回到了B Activity的界面。但是，生活总是不会让你如意的。页面直接回到了A Activity。是不是感到很纳闷？其实这才是正常的情况，singleInstance模式是存在于另一个任务栈中的，也就是说A Activity和C Activity是处于同一个任务栈中，B Activity则是存在另一个栈中。所以当关闭C Activity的时候，他自然就会去找当前任务栈中存在的Activity。当前的任务栈里面的所有Activity都关闭之后，才会去找另一个任务栈中的Activity，也即是说只有当A Activity也被finish之后才会回到B Activity界面。如果还想回到B Activity页面怎么办？在B Activity里面定义一个全部变量，public static boolean returnActivityB; 界面需要跳转的时候将returnActivityB=true;然后在A Activity界面onStart方法里面判断returnActivityB是都为true，是的话就跳转B Activity，同时将returnActivityB=fasle;这样就能解决跳转的问题。  
2、定义两个Activity，A Activity和B Activity，A Activity的启动模式为默认的，B Activity的启动模式为singleInstance。挡在A Activity通过startActivity启动了B Activity。按下home键，应用退到后台。此时再点击图标进入APP，正常情况因该是回到的B Activity，但是，还是那句话，生活总是不会让你如意的。当前界面显示的是A Activity。这是因为当重新启动的时候，系统会先去找Launcher这个任务栈里面去寻找Activity，也查看是否有存在的Activity。没有的话则会重新启动Launcher。解决方法和前面的一样。  
### Intent的七大属性
![Intent的七大属性.png](https://upload-images.jianshu.io/upload_images/7156039-11c77aa78d37d2db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
1、ComponentName(组件名称)
![Componentname.png](https://upload-images.jianshu.io/upload_images/7156039-03043ad92fd379cc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2、Action(动作)
![Action.png](https://upload-images.jianshu.io/upload_images/7156039-e4ff76da5682cd48.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
3、Category(类别)
![Category.png](https://upload-images.jianshu.io/upload_images/7156039-c98963d95868d9c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
4、Data(数据)，Type(MIME类型)
![Data.png](https://upload-images.jianshu.io/upload_images/7156039-c8c50b3784e4c3f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
5、Extras(额外)
![Extras.png](https://upload-images.jianshu.io/upload_images/7156039-e44479933baafa99.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
6、Flags(标记)
![Flags.png](https://upload-images.jianshu.io/upload_images/7156039-84ea96df2057c9b1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### Intent里面包含有一个Flags
```
 Intent.addFlags()     .....一般用在打开一个Activity的时候  表示的是Actiivy打开之后以怎样的形式放到这个任务栈中

intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);    //如果一个Activity打开的时候使用了这种方式的话那么他的作用===singleTask
intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);     //意思是重新整一个任务栈===singleInstance
intent.addFlags(Intent.FLAG_ACTIVITY_NO_HISTORY);   //  no-history
intent.addFlags(Intent.FLAG_ACTIVITY_SINGLE_TOP);   //  singleTop
intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TASK);   //  退出整个任务
```
