### 手势
是手指在屏幕上的一切操作，包括单击、双击、长按、滑动、滚动等。
#### 使用方式
1. 声明一个GestureDetector对象
> GestureDetector mGestureDetector=null;//声明成全局变量
2. 实例化手势对象，并且实现手势的监听OnGestureListener
> mGestureDetector=new GestureDetector(MainActivity.this,new MyOnGestureListener());
3. 创建一个类实现GestureDetector.OnGestureListener，并重写里面的6个方法：
> private class MyOnGestureListener implements GestureDetector.OnGestureListener
- onDown：手指按下的监听
```
    /**
     * 手指按下的时候的监听
     */
    @Override
    public boolean onDown(MotionEvent e) {
        Log.e("----------------","onDown");
        return false;
    }
```
- onShowPress：手指短按的监听
```
/**
 * 手指按住不放那么都会触发这个函数
 */
@Override
public void onShowPress(MotionEvent e) {
    Log.e("----------------","onShowPress");
}
```
- onSingleTapUp：单击
```
/**
 * 单击
 */
@Override
public boolean onSingleTapUp(MotionEvent e) {
    Log.e("----------------","onSingleTapUp");
    return false;
}
```
- onScroll：滚动
```
        /**
         * 滚动
         * @param e1:表示的是上一次的开始的时候的事件
         * @param e2 :这一次的时候的事件
         * @param distanceX:在x轴上滚动的距离  从左边往右边滚动那么这个值是负值 / 在x轴上从右边往左边滚动的话那么这个值是正值
         * @param distanceY:在y轴上滚动的距离 从上--->下  负值  从下--->上 正值
         * @return
         */
        @Override
        public boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX, float distanceY) {
            Log.e("----------------","onScroll");
//            Log.e("------distansX----",distanceX+"-----");
//            Log.e("------distansY----",distanceY+"-----");
            int startX= (int) e1.getX();
            int endX= (int) e2.getX();
            int startY= (int) e1.getY();
            int endY= (int) e2.getY();
            if(Math.abs(startX-endX)>Math.abs(startY-endY)){    //默认就是左右的滑动
                if(startX>endX){   //从右边往左边来进行滑动
                    Log.e("-----------","从右边往左边滑动");
                }else{   //从左边往右边进行滑动
                    Log.e("-----------","从左边往右边滑动");
                }
            }else{
                if(endY>startY){   //从上边往下边来进行滑动
                    Log.e ("-----------","从上边往下边滑动");
                }else{   //从左边往右边进行滑动
                    Log.e ("-----------","从下边往上边滑动");
                }
            }
            return false;
        }
```
- onLongPress：长按,和onSowPress是时间上的区别。
```
/**
 * 长按    和上面那个onShowPress的区别是时间上的区别，也就是说如果触发了 onLongPress就一定要触发onShowPress
 */
@Override
public void onLongPress(MotionEvent e) {
    Log.e("----------------","onLongPress");
}
```
- onFlinf：滑动，滑动和滚动的区别：滑动是在屏幕上面快速的移动一段距离，有滑动一定有滚动，有滚动不一样有滑动。
```
/**
 * 滑动      滑动和滚动的区别
 *   滑动是在屏幕上快速的移动一段距离
 *   滚动：是在屏幕上慢慢的移动一段距离
 *   有滑动就一定有滚动
 *   有滚动不一定有滑动
 * @param e1:上一次的那个开始的事件
 * @param e2:上一次结束的事件
 * @param velocityX:表示的是单位时间内在x轴上移动的distance
 * @param velocityY:表示单位事件内在Y轴上移动的距离   这个主要用来判定那个到底是往哪里滑动
 * @return
 */
@Override
public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY) {
    Log.e("----------------","onFling");
    return false;
}
```
4. 将当前触摸区域的事件传递给手势来进行处理。
> 例如：重写onToucheEvent方法，将触摸事件返回给手势去处理
```
@Override
public boolean onTouchEvent(MotionEvent event) {
    return mGestureDetector.onTouchEvent(event);
}
```
### ViewGroup的用法：
ViewGroup：容器，在ViewGroup里面转的是View,或者新的ViewGroup。ViewGroup可以看做是所有容器的父类。

ViewGroup和View的关系和区别？
&emsp;&emsp;View是所有控件和容器的最大的父类，ViewGroup是继承于View的，View也是ViewGroup的父类，在开发中如果要将View和容器区分开的话，可以认为ViewGroup就是所有容器的父类。

##### 学习ViewGroup的作用：
1. 可以自定义布局-->流式布局
2. 可以自定义容器-->自己决定容器能够装多少数据，容器中的控件位置也是自己决定

### 自定义ViewGroup的步骤：
1. 编写一个类继承于ViewGroup，并且重写有两个参数的构造函数
2. 重写ViewGroup里面的onLayout方法。
```

/**
 * 确定那个儿子的位置
 * @param changed
 * @param l:儿子的左边顶部的x的坐标
 * @param t:儿子左边顶部的y坐标
 * @param r:儿子右边底部的x的坐标
 * @param b:儿子的右边的底部的y坐标
 */
@Override
protected void onLayout(boolean changed, int l, int t, int r, int b) {
  //在ViewGroup中获取那个儿子的数量
    int childCount = getChildCount();
  //给孩子设置那个显示的位置
    //得到这个孩子
    View childView=getChildAt(0);
    //给儿子设置个位置
 /*   Log.e("---------",childView.getMeasuredWidth()+"");
    Log.e("---------",childView.getMeasuredHeight()+"");*/
    childView.layout(100,100,childView.getMeasuredWidth()+100,childView.getMeasuredHeight()+100);
}
```
3. 重写ViewGroup中的onMeasure方法
```
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    //测量儿子的第一种模式:交给他爹来测咋们不用管，一般情况下都用这种方法
    //怎么交给他爹测
    //这句话一旦调用的话那么他的孩子就有那个宽高了...
    //优点：方便而且不用谢很多的代码  孩子的高度就有了....
    measureChild(getChildAt(0),widthMeasureSpec,heightMeasureSpec);
    //还有第二种测量模式:这种测量模式就是:自己编写代码来进行测量，较复杂
    //自己测  获取测量模式
    int count=getChildCount();
    Log.e("------孩子的个数---",count+"");
   if(tag){
        tag=false;
        int widthMode=MeasureSpec.getMode(widthMeasureSpec);
        int heightMode=MeasureSpec.getMode(heightMeasureSpec);
        //获取那个建议的宽和高
        int widthSize=MeasureSpec.getSize(widthMeasureSpec);
        int heightSize=MeasureSpec.getSize(heightMeasureSpec);
        //第三步就来确定那个是使用的具体的值还是那个wrap_content
        //表示的是最终算出来的那个宽和高
        int describleWidth=200;
        int describleHeight=200;
        int width=0;
        int height=0;
        if(widthMode==MeasureSpec.AT_MOST){ //这个情况表示的是那个wrap_content
            width=Math.min(widthSize,describleWidth);
        }else if(widthMode==MeasureSpec.EXACTLY){  //这个表示的是精确的值或者是match_parent
            width=widthSize;
        }
        //玩的是那个高的测量模式
        if(heightMode==MeasureSpec.AT_MOST){   //这个表示的是wrap_content
            height=Math.min(describleHeight,heightSize);
        }else if(heightMode==MeasureSpec.EXACTLY){  //表示的是那个实际的值或者是那个充满容器
            height=heightSize;
        }
        Log.e("----------------",width+" 之中");
        Log.e("------------hei----",height+" 之中");
        //下面就将咋们的测量的结果传递给那个给儿子测量的方法     getChildAt(0).measure(MeasureSpec.makeMeasureSpec(width,widthMode),MeasureSpec.makeMeasureSpec(height,heightMode));
    }
```
> 注意：在ViewGroup中只能使用getMeasuredWidth和getMeasuredHeight来测量宽和高。
4. 在需要使用该ViewGroup的布局文件声明控件
```
<com.fox.ViewGroupDemo.MyViewGroup
    android:layout_width="400dp"
    android:layout_height="300dp"
    android:background="#f07">
    <Button
        android:layout_width="100dp"
        android:layout_height="50dp"
        android:text="按钮"/>
</com.fox.ViewGroupDemo.MyViewGroup>
```
