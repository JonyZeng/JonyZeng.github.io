### 选择菜单(OptionsMenu)
1. 第一种使用方式：
    - 在res目录下面建立一个名称是menu的文件夹
    - 在menu下面建立一个xml文件(默认就是menu的类型)
    - 在建立的这个xml文件夹中添加菜单的选项，xml文件中有很多属性
android:orderInCategory = "2"  //表示当前的item在整个item中所占的位置，数字越小越靠前
android:title="选项二"  //确定的是那个选项的内容
android:id="@+id/select_02"   //可以跟当前的选项整一个id  方便区分点击的是谁
android:icon="@drawable/ic_launcher"   //给这个选项整了一个图标
android:showAsAction = "always"  //表示总是将图标显示在标题栏上的右边，不管大小都要显示
    - 在Activity里面重写onCreateOptionsMenu方法
    - 通过getMenuInflater()的布局加载器来加载文件
  getMenuInflater().inflate(R.menu.option_menu_01, menu)   //后面的第二个menu参数表示的意思是将前面的menu布局加载到后面的menu对象中去

2. 第二种方式
    - 在Activity默认重写的onCreateOptionsMenu方法中通过menu对象的add方法来添加item的值。
menu.add(" ");  //添加选项的内容
menu.add(groupId,ItemId,orderInCategory,title);   //第一个参数：表示的是组的id 第二个表示的是 item的Id 第三个参数：Item的排列顺序 第四个参数：当前Item的内容
   - 在item里面添加子菜单
   menu.addSubMenu(groupId,ItemId,orderInCateGory,title);//注意，这个方法只是获取添加子菜单的对象，需要拿到对象之后进行添加。
```
SubMenu subMenu = menu.add("设置");
subMenu.add(1,1,1,"身高设置");
subMenu.add(1,2,1,"体重设置");
SubMenu subMenu1 = menu.add("中国");
subMenu1.add(2,1,1,"四川");
subMenu1.add(2,2,1,"成都");
```
### 菜单事件的监听
1. 第一种玩法：直接重写onOptionsItemSelected(MenuItem item) 
2. 第二种玩法：必须要通过Java代码创建menu才可以
menu.add();  返回的是menuItem对象
menuItem.setOnMenuItemClickListener(MenuItem item) 

选项菜单依赖的对象是Activity，不能依赖View
### 上下文菜单
上下文菜单依赖的对象就是View.也就是说我们可以触发某一个控件从而来显示一个菜单选项。
1. 使用方式：
    - 和选项菜单一样在menu文件夹下建立一个Android.xml file来进行配置就可以了
    - 重写Activity里面的onCreateContextMenu方法
    - 注册在控件上面弹出一个上下文菜单registerForContextMenu(button) 后面的参数表示的是绑定的View
注意：上下文菜单的事件触发是一个长按事件
2. 上下文菜单的事件处理 和菜单事件的监听一样
### PopupMenu的使用
```
PopuMenu popuMenu = new PopuMenu(PopuViewActivity.this,v);    //初始化PopuMenu对象，第二个参数表示将PopuMenu绑定在这个控件上面
popuMenu.getMenuInflater().inflate(R.menu.option,popuMenu.getMenu());    // 绑定布局的对象
popuMenu.setOnMenuItemClickListener(new MyOnMenuListener());  //设置点击事件
popuMenu.setOnDismissListener(new MyOnDissLisener());  //设置消失的时候的监听器
popuMenu.show();    //显示PopuMenu
popuMenu.dismiss();    // 取消显示popuMenu
```
### ContentMenu和PopuMenu的区别：
  - 上下文菜单只能绑定一个View来进行显示
  - PopuMenu能够同时绑定多个View来进行显示
### PopuWindow的使用
```
// 1、初始化PopuWindow对象
  PopuWindow popuWindow = new PopuWindow(PopuWindowActivity.this);
// 2、通过PopuWindow对象来对PopuWindow对象添加相应的设置
  popupWindow.setHeight(LayoutParams.WRAP_CONTENT);  
  popupWindow.setWidth(LayoutParams.WRAP_CONTENT);
  popupWindow.setBackgroundDrawable(new ColorDrawable(0x00000000));    // 用来解决弹出框不消失的问题，一般不需要设置就会消失
  popupWindow.setFocusable(true);      // 获取焦点
  popupWindow.setOutsideTouchable(true);    //  表示点击弹出框以外的区域是否消失，如果为true就消失，false就不消失
  popupWindow.setTouchable(true);    //一般不设置，采用默认的就OK了
// 3、通过布局加载器的对象将xml文件转换成View对象
  LayoutInflater inflater = getLayoutInflater();
  inflater.inflate(R.layout.popuwindow_list,null);
// 4、将获取到的View添加到PopuWindow中
  popupWindow.setContentView(view);
// 5、确认这个PopuWindow显示的位置
  popupWindow.showAsDropDown(v);  // 这个方法在使用的时候，需要多加注意。popupwindow会在button的下面贴button的底部展示出来，但是当button底部到屏幕底部的高度小于popupwindow的高度时，popupwindow就会找button的父view作为参考点，如果也没有符合的parent view ,popupwindow就会使自己的底部贴button的顶部展示
   popupWindow.showAtLocation(getWindow().getDecorView(),Gravity.NO_GRAVITY,50,0);    
  popuWindow.dismiss();

```
### 对话框的写法
```
// 1、初始化对话框的Builder
  AlertDialog.Builder builder=new Builder(DialogActivity.this);
// 2、通过Builder来设置相应的信息
     builder.setTitle("喜欢与否");    //设置的是那个标题
     builder.setIcon(R.drawable.ic_launcher);   //设置的是那个图标
     builder.setMessage("比是否喜欢我?");    //设置的是那个信息
// 3、显示出来
builder.show();
```
