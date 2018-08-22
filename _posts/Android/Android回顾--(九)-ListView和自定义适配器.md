### Listview：列表控件
ListView的特有属性
```
android：divider="@null"  //去掉listView的分割线。"@drawable/ic_launcher"  可以通过使用图片进行自定义分割线
android：dividerHeight="50dp"     //设置分割线的高度
android：listSelector="@android:color/transparent"    //取消item的点击效果
android：background="#f07"    //设置listview的背景
android:cacheColorHint="#0000000"  //滑动ListView的时候那个默认的前景色  这个属性一般情况下和Background联合使用   一般情况下把所有的值都设置成0  就表示前景色是透明的
android:fastScrollEnabled="true"    // 当我们快速滚动的时候出现一个快速的滚动条
android:smoothScrollbar="false"   // 指的是滑动块是否使能
android:scrollbarStyle="insideOverlay"  // 这个就是那个Scroll的风格  细看才知道
android:scrollbars="none"  //取消滚动条的显示
android:fadeScrollbars="true"  //表示滚动条逐渐消失
android:scrollbarSize=""       // 设置滚动条的大小

```
#### 通过系统提供的适配器来完成ListView的适配工作
适配器的主要功能：将数据源和模板以及控件之间建立一个诶关系，这个关系呢就是能够达到数据适配的工作。
#### ListView的两个点击事件
1、listView.setOnitemClickListener();    //item的点击事件
![单击事件.png](https://upload-images.jianshu.io/upload_images/7156039-983263f4a3667ea7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2、listView.setOnItemLongClickListener();    //长按点击事件
![长按点击事件.png](https://upload-images.jianshu.io/upload_images/7156039-4fd49fe17589da07.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
注意：如果在item里面有可以获取焦点的的控件的话，一般情况下都会将该控件的focusable设置成false，防止该控件获取了焦点而item没有获取到焦点。  有三种方法可以设置：
1、在该控件的属性下设置
```
android:focusable="false"
```
2、在java文件中获取该控件的对象，动态设置不获取焦点
```
Button btn = new convertView.findviewById(R.id.btn);
btn.setFocusable(false);
```
3、在模板的根布局上面设置
```
android:descendantFocusabiliy="blocksDescendants"
```
在simpleAdapter中含有一个方法叫 setViewBinder 这个方法的调用是依赖于绑定的控件。也就是说给多少控件赋值，那么每个控件在绑定值的时候都会将该方法进行回调，我们可以利用这个回调的函数给里面绑定值的控件添加相应的事件
![控件绑定事件.png](https://upload-images.jianshu.io/upload_images/7156039-b5d27d2d90632744.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 自定义适配器
1. 编写一个类继承于BaseAdapter
![自定义适配器的构建.png](https://upload-images.jianshu.io/upload_images/7156039-3927d700a1055312.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2. 重写里面的方法
    - getCount():返回item的数量
![getCount.png](https://upload-images.jianshu.io/upload_images/7156039-85faba681bcb58b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  - getItem(): 返回数据源中的item对象
![getItem.png](https://upload-images.jianshu.io/upload_images/7156039-17dda17ee6b09a14.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  - getItemId(): 返回每个条目的位置
![getItemId.png](https://upload-images.jianshu.io/upload_images/7156039-38e5235217b0d0c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  - getView(): 返回每个条目的view
![getView.png](https://upload-images.jianshu.io/upload_images/7156039-586c8e3c47897b20.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)







