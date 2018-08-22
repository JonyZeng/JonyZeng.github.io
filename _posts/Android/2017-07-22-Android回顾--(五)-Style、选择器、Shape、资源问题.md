### Style的用法
&emsp;&emsp;style的意思就是样式，当我们在项目中需要定义一个基础共用的风格的时候，我们就把大量的共享元素抽离出来定义在style中。
```
<!--Style的基本用法-->
       <style name="textStyle">
        <!-- 名字不是随便整的  里面有才能用-->
        <item name="android:alpha">1</item>   <!-- 0-1 -->  <!--值越大越不透明-->
        <item name="android:background">#f07</item>
        <item name="android:padding">10dp</item>
        <item name="android:paddingLeft">20dp</item>
        <item name="android:baseline">@id/mTextView</item>
        <item name="android:drawableTop">@drawable/ic_launcher</item>  
    </style>

    <!--Style的继承的用法-->    <!--他会继承他爹的所有的属性的-->
    <style name="textStyleChild" parent="textStyle">
        <item name="android:visibility">visible</item>
        <item name="android:weightSum">	1</item>
        <item name="android:textStyle">italic</item>
        <item name="android:textSize">30sp</item>
        <item name="android:textColor">#ccc</item>
        <item name="android:text">Hello word</item>
        <item name="android:spinnerMode">dialog</item>
        <item name="android:focusable">true</item>
        <item name="android:scrollbarStyle">insideOverlay</item>
    </style>
```
注意: 不是所有的属性都对所有的控件有效，只有控件对应的属性才会有效。
#### 用途:
&emsp;如果多个控件包含有相同的属性，就可以定义一个style提供给其他控件引用
### 选择器
#### 1、选择器是什么？是干嘛用的？
&emsp;&emsp;根据用户的不用的行为可以让View显示出不同状态的配置文件。所有的选择器文件都是放在src/drawable目录下面的
#### 2、选择器的属性状态
```
     <item android:state_checkable=""></item>   是否可以选择 
     <item android:state_checked=""></item>     是否被选中
     <item android:state_selected=""></item>   Spinner的item是否被选中
     <item android:state_pressed=""></item>     是否被按下
     <item android:state_focused="false></item>"        是否获取了焦点
     <item android:state_window_focused=""></item>  窗体是否处于最前端
```
### Shapes: 通过程序配置来完成基本的图形的形状和实现点击的状态
shape的简单模板：
```
<?xml version="1.0" encoding="utf-8"?>  
<shape  //定义形状
xmlns:android="http://schemas.android.com/apk/res/android"  
android:sharp=["rectangle"|"oval"|"line"|"ring"]     //总共有四种形状，矩形(默认)/椭圆形/直线型/环形
    // 以下4个属性只有当类型为环形时才有效  
    android:innerRadius="dimension"     //内环半径  
    android:innerRadiusRatio="float"    //内环半径相对于环的宽度的比例，比如环的宽度为50,比例为2.5,那么内环半径为20  
    android:thickness="dimension"   //环的厚度  
    android:thicknessRatio="float"     //环的厚度相对于环的宽度的比例  
    android:useLevel="boolean">    //如果当做是LevelListDrawable使用时值为true，否则为false.  

   <corners    //定义圆角  
        android:radius="dimension"      //全部的圆角半径  
        android:topLeftRadius="dimension"   //左上角的圆角半径  
        android:topRightRadius="dimension"  //右上角的圆角半径  
        android:bottomLeftRadius="dimension"    //左下角的圆角半径  
        android:bottomRightRadius="dimension" />    //右下角的圆角半径  
  
    <gradient   //定义渐变效果  
        android:type=["linear" | "radial" | "sweep"]    //共有3中渐变类型，线性渐变（默认）/放射渐变/扫描式渐变  
        android:angle="integer"     //渐变角度，必须为45的倍数，0为从左到右，90为从上到下  
        android:centerX="float"     //渐变中心X的相当位置，范围为0～1  
        android:centerY="float"     //渐变中心Y的相当位置，范围为0～1  
        android:startColor="color"      //渐变开始点的颜色  
        android:centerColor="color"     //渐变中间点的颜色，在开始与结束点之间  
        android:endColor="color"    //渐变结束点的颜色  
        android:gradientRadius="float"  //渐变的半径，只有当渐变类型为radial时才能使用  
        android:useLevel=["true" | "false"] />  //使用LevelListDrawable时就要设置为true。设为false时才有渐变效果  

    <padding    //内部边距  
        android:left="dimension"  
        android:top="dimension"  
        android:right="dimension"  
        android:bottom="dimension" />  
  
    <size   //自定义的图形大小  
        android:width="dimension"  
        android:height="dimension" />  

    <solid  //内部填充颜色  
        android:color="color" />  

    <stroke     //描边  
        android:width="dimension"   //描边的宽度  
        android:color="color"   //描边的颜色  
        // 以下两个属性设置虚线  
        android:dashWidth="dimension"   //虚线的宽度 
       android:dashGap="dimension" />      //虚线的间隔，值为0时是实线 
</shape>  
```
### Android中资源的问题
1、字符串资源：   string.xml文件中的     
注意:就是这个资源文件是万能的可以放颜色、字符串、数组、样式等都能放到这个文件里面  
2、媒体资源： raw和assets文件中都可以 .mp3 .mp4 .avi  
&emsp;&emsp; raw文件里面放置的媒体资源在apk打包的时候不会进行压缩处理。Assets目录放置的媒体资源在apk打包的时候会进行压缩处理。  
注意：一般情况下除了软件本身的复制音频外其余的视屏，都不会放到这两个目录下面来。因为这些资源即使压缩了也还是会非常占用资源的。  
3、额外的数据库： 一般挡在Assets目录里面  
4、颜色资源： 放置在是string.xml或者colors.xml文件中  
5、尺寸资源： 放置到values目录下的dimens.xml 或者string.xml  
6、风格和主题资源： style.xml文件或者string.xml文件中  
7、数组资源： 可以定义到arrays.xml或者string.xml文件中 一般情况下定义在string.xml文件中  
