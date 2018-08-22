### Fragment
&emsp;&emsp;碎片，抽象的理解就是一个控件，只是这个控件内部维护了自身的生命周期。Fragment是依赖于Activity，用于显示页面的一部分内容。

这里有的人就会提出疑问了，Android中其他的控件也能在页面显示一部分的内容，为什么还要引入Fragment？
- 首先因为Fragment自身拥有生命周期，那么我们就阔以在Fragment的实例中来完成这一部分内容的显示于隐藏，这样这部分的操作逻辑就和Activity分离了，降低了程序之间的耦合性
- Fragment的出现实际上是为了解决平板电脑屏幕过宽，实现分屏效果。

#### 使用方式
#### 静态的使用方式
像控件一样，在XML文件里面声明控件 然后控件就显示出来了
- 第一步：编写一个类继承于Fragment
- 第二步：在xml文件中，编写Fragment的布局
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <fragment android:name="com.example.news.ArticleListFragment"
            android:id="@+id/list"
            android:layout_weight="1"
            android:layout_width="0dp"
            android:layout_height="match_parent" />
    <fragment android:name="com.example.news.ArticleReaderFragment"
            android:id="@+id/viewer"
            android:layout_weight="2"
            android:layout_width="0dp"
            android:layout_height="match_parent" />
</LinearLayout>
```
- 第三步：重写Fragment里面重写onCreatView()方法，通过inflater来加载布局文件  第一个参数是布局id 第二个参数是container（可以为空）第三个是false
   返回值是view 我们可以通过view来找到Fragment布局文件里面的控件

Fragment中的控件的事件的处理：
- 可以在宿主Activity里面直接通过findviewById来找到这个控件。
- 可以在onCreateView方法里面通过view.findViewById来找到这个控件
- 可以在Fragment里面通过获取上下文在，然后找到id getActivity().findViewById();
- 需要碎片的布局文件中咱们直接声明一个Fragment，然后加id就可以了

#### Fragment生命周期
![官方生命周期图.png](https://upload-images.jianshu.io/upload_images/7156039-6473edcddb5dce9e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
  onAttach(Activity activity)：生命周期调用的第一个方法，主要将我们的Fragment寄生到Activity里面去//在高低版本都可以使用，当API低于23时,下面的
（ onAttach(Context context)这个方法不会调用）
  onCreate()：创建Fragment的方法//这里完了就是执行最后面的两个方法
  onStart()：Fragment可见，但是不能获取焦点
  onResume()：Fragment可以获取焦点，也就是可以进行交互了
  onPause()：不能获取焦点
  onStop()：不可见
  onDestoryView()：销毁Fragment绑定的view
  onDestroy()：回收Fragment占用的资源
  onDetach():清除Activity里面的尸体
  //后面的两个方法很重要，是要重点掌握的
  onActivityCreated()：处理事务，用来完成处理逻辑，Activity创建完成调用
  onCreateView()：绑定Fragment，创建Fragment的显示，绑定view到Fragment

```  

### 动过容器动态加载替换Fragment的使用方式
- 第一步：通过上下文获得Fragment的管理者对象
```
 getFragmentManager();//app包里面的
 getSupportFragmentManager();//支持包里面的*/
FirstFragment fg1 = new FirstFragment ();
SecondFragment() fg2 = new SecondFragment();
final FragmentManager fm = getFragmentManager();
//到底有啥用,我们可以通过id找到Fragment，可以通过管理者来获取Fragment的对象，然后操作这个对象  
Fragment fragment = fm.findFragmentById(R.id.firstFragment);
fm.findFragmentByTag(); //可以找到我们布局里面的tag属性
```
- 第二步：开启事务
```
 FragmentTransaction ft = fm.beginTransaction();
//添加哪个Fragment到这个容器里面
ft.add(R.id.fragment1,fg1);//使用replace()也是一样的
ft.commit();//提交
```
注意：replace()实际是先清除容器里面的Fragment，调用remove方法，然后再执行添加当前的Fragment，add()方法。如果我们每一次都用replace()的话，那么会造成一个问题，就是每一次都会先将前一个先删除，后面一个才能添加上。  造成每一个都需要重新CreatView()那么这么做的话，如果连续不断的去切换的话就会造成卡顿现象，为了解决这个问题就引出了一个概念就是Fragment的优化

#### Fragment的优化使用：
```
pirvate void addContent(Fragment from,Fragment to){
   //获取管理者
   final FragmentManager fm = getFragmentManager();
   //开启事务
   FragmentTransaction ft = fm.beginTransaction();
   //判断下一个Fragment是否存在，不存在就添加，提交
   if(!to.isAdded()){
        ft.add(R.id.fragment1,from);   
    }else{
         //存在我们就隐藏上一个，展示下一个Fragment
         ft.hide(from).show(to);
     }
      ft.commit();
}
```
注意事项：当我们添加进一个Fragment后，然后再向这个container中添加哪个Fragment的话，那么前一个Fragment没有被杀掉，只是没有显示而已
将当前Fragment的界面显示至帧布局中
```
//创建Fragment对象
Fragment fg1=new Fragment01();
// 获取Fragment的管理器打开事务
FragmentManager fm = getFragmentManager();
//通过Fragment的管理器打开事务FragmentTransaction ft = fm.beginTrasaction();
//通过事务把内容显示到帧布局(容器)
ft.replace(R.id.fl,fg1);
//提交
ft.commit();
```
我们一般是在onActivityCreateed()方法中处理一些事务。在使用过程中我们要注意一下几点、：
- Fragment被当做控件来使用的时候，要定义id，否则会报错，因为默认找不到，不能使用
- Fragment是依赖于、寄生于Activity的，所以在宿主Activity中也能找到Fragment的控件
- 在Fragment里面是不包含宿主(activity)的上下文，所以我们如果想要在Fragment里面获取上下文的话，需要使用getActivity();
- Fragment中的事件处理一般在onActivityCreated方法中进行。

### Fragment对象的使用
1. 第一种：在宿主Activity中直接new对象，作为全部变量
2. 第二种：写一个FragmentFactory帮助类，返回所有碎片的对象，在需要使用fragment的地方通过对应的下标获取相应对象
```
  public static class FragmentFactory{
        static List<Fragment> fragmentList = new ArrayList<>();
        static {
            //声明两个Fragment的对象
            FristFragment mFristFragment = new FristFragment();
            SecondFragment mSecondFragment = new SecondFragment();
            fragmentList.add(mFristFragment);
            fragmentList.add(mSecondFragment);
        }
        public static Fragment getFragment(int position){
            return fragmentList.get(position);
        }
    }
```
使用工厂类
```
            FragmentManager fragmentManager = getFragmentManager();
            FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
            fragmentTransaction.add(R.id.contentPanel,FragmentFactory.getFragment(0));
            fragmentTransaction.commit();

```

### 回退栈：
栈的存储模式：先进后出
Activity也有自己的回退栈
```
ft.addToBackStack(null);  // 将当前的事务加入到回退栈中，按返回键之后就可以返回到加入回退栈里面保存的上一个Fragment 
```
事务：指的是一系列指令的合集，这一系列的指令要么全部执行，要么都不执行
```
FragmentManager fm = getFragmentManager(); 
fm.popBackStack();//回滚，将栈顶的Fragment出栈//就是Fragment的弹栈
ft.popBackStack("goushen",0);//前面的参数是栈的名字,后面的参数是0，表示将“goushen”上面所有的Fragment全部出栈（不包括自己），不包含它的下面的对象
ft.popBackStack("goushen",FragmentManager.POP_BACK_STACK_INCLUSIVE);//前面的参数是栈的名字,后面的参数是FragmentManager.POP_BACK_STACK_INCLUSIVE，表示将“goushen”上面所有连同它自己一起出栈（包括自己），不包含它下面的对象
ft.popBackStack(commitId,0);//根据前面提交方法返回的id来进行弹栈//指的是弹出指定id以上或者包含id及以上的所有对象
```
这个地方的id是ft.commit()的返回值，不是Fragment得id
```
ft.popBackStackImmediate();立即执行
ft.popBackStackImmediate("",0);
注意事项:压栈弹栈和Activity差不多

ft.addOnBackStackChanged(){}//栈改变的监听事件

ft.getBackStackEntryCount();获取栈里面的对象的数量
ft.getBackStackEntryAt();根据索引取得栈里面的某一个对象
ft.getBackStackEntryAt().getName();获取对象的名字

ft.removeOnBackStackChangedListener();//删除栈里面的监听事件
```
#### Fragment向Activity数据传递的步骤
1. 定义一个接口回调
2. 在接口回调中声明一个回调的方法
3. 将接口对象置空
4. 让需要接收数据的地方实现定义的接口
5. 在Fragment的onActivityCreate方法里面获取当前接口的对象getActivity();
6. 在我们需要回调的地方进行回调。

#### Fragment向点击之后跳转的Fragment传递数据
```
      if (fragment != null && fragmentManager != null) {
            Bundle args = new Bundle();
            args.putString("title", title);
            args.putInt("tag", tag);
            fragment.setArguments(args);
            FragmentTransaction transaction = fragmentManager.beginTransaction();
            transaction.replace(frameId, fragment, fragmentTag);
            if (addToBackStack) {
                transaction.addToBackStack(fragment.getClass().getSimpleName());
            }
            transaction.commit();
        }
```
