&emsp;&emsp;咳咳、今天抽空写了一个关于时钟的自定义view，感觉里面还是有一些东西值的学习的。所以啦，我就又来啦。自定义view我个人感觉主要就是onMeasure和onDraw方法需要我们多去研究和学习。
&emsp;&emsp;好啦，进入正题。首先，我们需要创建一个类，继承与View。重写里面的onMeasure()和onDraw()方法。
```
  public class ClockView extends View {

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
    }
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    }
}
```
&emsp;&emsp;OK,我们自定义view的大体框架出来了。后面就是根据需求进行细节代码的实现。首先，第一步，我们需要在主界面的xml文件中，引入我们的自定义View。
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"

    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.example.jonyz.clockview.MainActivity"
    android:gravity="center"
    >
        <com.example.jonyz.clockview.View.ClockView
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_gravity="center"
            />
</LinearLayout>

```
其中，需要我们注意的是在父View中设置了居中的属性，这样我们的自定义view才能在屏幕中心被绘制出来。
&emsp;&emsp;第二步、就是写onMeasure()方法里面的测量代码，测量当前view的宽高。
```
 @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int width = getMeaSize(1000, widthMeasureSpec);
        int heigth = getMeaSize(1000, heightMeasureSpec);
        if (width > heigth) {
            width = heigth;
        } else {
            heigth = width;
        }
        setMeasuredDimension(width, heigth);

    }
```
其中的getMeaSize是封装的一个更具测量模式获取宽高的方法。将获取到的宽、高进行比较，取最小的值。最后通过 setMeasuredDimension(width, heigth);方法来确定当前view 的大小。
```
 public int getMeaSize(int defaultSize, int measureSpec) {

        int size = 0;
        int measuSize = MeasureSpec.getSize(measureSpec);
        int Mode = MeasureSpec.getMode(measureSpec);
        switch (Mode) {
            case MeasureSpec.UNSPECIFIED:
                size = Math.min(measuSize, defaultSize);
                break;
            case MeasureSpec.AT_MOST:
                size = Math.min(measuSize, defaultSize);
                break;
            case MeasureSpec.EXACTLY:
                size = Math.min(measuSize, defaultSize);
                break;
        }
        return size;
    }
```
&emsp;&emsp;这里简单的讲一下三种测量模式：
1. UNSPECIFIED(未指定): 父控件没有对子空间添加任何的约束，子控件可以获得任意需要的大小(这种不受控制的一般使用较少);
2. EXACTLY(完全)：布局的时候，参数一般选择math_parent或者一个具体的宽高值。父控件给子控件指定了一个大小，子控件只能够被限定在给定的大小里面。父控件才不会考虑子控件的实际大小。当然，如果是match_parent，就说明这父控件已经知道子控件想要多大的尺寸了（意思就是剩下的都要了，一点不剩）。来，举个通俗易懂的栗子说明一下：房间里面的最大的床只有那么大，不管有多少人都只能睡到这最大的床上，睡不下的人，我才不管你。
3. AT_MOST(至多)：通常参数是warp_conent,意思就是子控件最多达到指定大小的值。warp_content就是父控件不知道子控件到底需要多大的尺寸，需要子控件自己测量之后，和父控件商量说给他尽可能大的尺寸，已便让内容全部显示单不能超过包裹内容的大小。栗子来了，一个房间里面有有各种大小不一的床，一个人就只能睡单人床。两个人就可以睡双人床，但是不能浪费，一个人睡双人床的情况，要保证床的大小刚好睡那么多人。
      第三步、前面测量的准备工作做完之后，我们就要开始大师之作，绘画了。分析需求可以知道，我们首先需要在屏幕中心画一个时钟的圆圈出来。我所采取是采用同心圆，半径相差的方法。具体的实现代码如下：
```
@Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        canvas.save();
        canvas.translate(getWidth() / 2, getHeight() / 2);
        mPaint = new Paint();
        r = getMeasuredWidth() / 2;
        r2 = getMeasuredWidth() / 2 - 3;
        mPaint.setColor(Color.BLACK);
        canvas.drawCircle(0, 0, r, mPaint);
        mPaint.setColor(Color.WHITE);
        canvas.drawCircle(0, 0, r2, mPaint);
```
我们这里在画布（canvas）上面画的时候，是 把画布平移到了屏幕的中心，这样做的目的就是为了方便我们画圆的时候，圆心就是在坐标的圆点位置。
      第四步、在画完表盘之后，我们就开始画表盘的刻度了。先看代码，看了之后带你领略神奇的原理。
```
  /**
     * 绘制表盘刻度
     *
     * @param canvas
     */
    private void prinCircle(Canvas canvas) {
        canvas.save();
        int lineWidth = 0;  
        for (int i = 0; i < 60; i++) {
            if (i % 5 == 0) { //整点
                mPaint.setStrokeWidth(3);  //设置画笔的宽度
                mPaint.setColor(Color.BLACK);
                mPaint.setTextSize(30);
                lineWidth = 60;  //设置整点刻度的线宽度
            } else { //非整点
                lineWidth = 40;
                mPaint.setColor(Color.BLACK);
                mPaint.setStrokeWidth(1);
            }
            canvas.drawLine(0, -r + 10, 0, -r + 10 + lineWidth, mPaint);
            canvas.rotate(6);  //将画布旋转6度，这样就能保证，每一次都是在Y轴上画线
        }
        canvas.restore();

    }
```
哈哈，看了代码之后是不是感觉逻辑贼简单。就把表盘分为60份，每五份就区别出来为小时。当然，光有刻度未免显得太单调了一点，所以我们将数字画到对应的位置。
```
//表盘上面数字 -mRadius + DptoPx(5) + lineWidth + (textBound.bottom - textBound.top)
                String text = ((i / 5) == 0 ? 12 : (i / 5)) + "";  
                Rect textBound = new Rect();    //将数字装在矩形区域内，对整个矩形区域进行旋转
                mPaint.getTextBounds(text, 0, text.length(), textBound);
                mPaint.setColor(Color.BLACK);
                canvas.save();
                canvas.translate(0, -r + 10 + lineWidth + (textBound.bottom - textBound.top));//将画布平移
                canvas.rotate(-6 * i);
                mPaint.setStyle(Paint.Style.FILL);
                canvas.drawText(text, -(textBound.right - textBound.left) / 2, textBound.bottom, mPaint);
                canvas.restore();
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;第五步、这个光有样式，没有实际功能是不符合处女座程序猿哥哥追求完美的特性的，所以，我们还是要让它真实一点，可以走动，并且我每一次进去就可以显示的是当前时间。咳咳，话痨的程序员直接上代码。
```
 //获取当前小时，分钟，秒钟
        Calendar calendar = Calendar.getInstance();
        int mHour = calendar.get(Calendar.HOUR_OF_DAY);
        int mMinute = calendar.get(Calendar.MINUTE);
        int mSecond = calendar.get(Calendar.SECOND);
        //获取时针canvas选择角度
        float angleHour = (mHour % 12) * 360 / 12 + mMinute * 1 / 2;
        float angleMinu = 6 * mMinute;
        int angleSec = 6 * mSecond;
```
获取到了角度之后，我们就需要根据当前时间的角度，让画布旋转。
```
  //时针
        canvas.save();
        canvas.rotate(angleHour);   //根据获取到的角度进行旋转。
        canvas.restore();
```
最后，通过方法每隔一秒刷新onDraw()。
```
 postInvalidateDelayed(1000);
```
  23333333~这就结束啦。我也不知道为什么要结束，就是结束了。最后，送上项目地址，喜欢的可以去看看。
GitHUb:https://github.com/JonyZeng/ClockView


