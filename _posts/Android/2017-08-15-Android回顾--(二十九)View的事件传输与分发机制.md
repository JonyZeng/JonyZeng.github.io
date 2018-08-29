### 自定义View
&emsp;&emsp;在平时工作中，总会需要一些特别的需求，而这些需求是Android系统自带控件不能实现的，所以我们就需要自定义View来实现业务需求。
1. 继承控件：主要完成的功能是在原来的控件基础上添加一些新的功能
2. 组合控件：将多个的单一的控件组合成一个控件。
3. 完全自定义控件：编写一个类直接继承View,重写里面的onMeasure()方法确定尺寸，重写onLayout()方法确定布局位置，重写onDraw()方法绘画控件的形状，再加上相应的回调函数和自定义属性来完成相应的控件的展示和数据的回调。

> 所有的自定义View都是重写的有两个参数的构造函数，因为系统默认就是调用的有两个参数的构造函数。

#### 完全自定义View(控件)
1. 自定义一个类继承于View,并重写里面包含两个参数的构造函数
2. 重写onMeasure()方法，用于测量自身的尺寸
> protected void onMeasure(int widthMeasureSpec，int heightMeasureSpec)
widthMeasureSpec：
heightMeasureSpec：这两个参数是从父容器得来的，这两个参数分别实际上是两个32位的2进制数来表示的，这两个32位的2进制数的高两位是当前控件在xml中声明的测量模式，后30位是父容器根据当前的测量模式计算出来的建议的宽和高。（这个宽和高只是建议的，不是实际的）
```
    /**
     * 测量自身的宽高尺寸
     * @param widthMeasureSpec
     * @param heightMeasureSpec
     */
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        //下面设置默认的宽和高
        int describWidth = 200;
        int describHeight = 200;
        //获取宽高的测量模式
        int widthModel = MeasureSpec.getMode(widthMeasureSpec);
        int heightModel = MeasureSpec.getMode(heightMeasureSpec);
        //获取父类建议的宽高的值
        int widthSize = MeasureSpec.getSize(widthMeasureSpec);
        int heightSize = MeasureSpec.getSize(heightMeasureSpec);
        //通过测量模式确定宽高的最终值
        int width = 0;
        int height = 0;
        if (widthModel == MeasureSpec.AT_MOST) { //说明使用的是那个warp_content模式
            width = Math.min(describWidth,widthSize);
        }else if (widthModel == MeasureSpec.EXACTLY){ //说明使用的是具体的数值或者match_parent这种测量模式
            width = widthSize;
        }
        if (heightModel == MeasureSpec.AT_MOST){
            height = Math.min(describHeight,heightSize);
        }else if (heightModel == MeasureSpec.EXACTLY){
            height = heightSize;
        }
        setMeasuredDimension(width,height);
    }
```
> 测量模式：
> 1. MeasureSpec.AT_MOST:--->对应咱们在xml布局文件中声明的wrap_content
>2. MeasureSpec.EXACTLY:--->对应布局文件中的match_parent/实际的数值
默认情况下，1和2是相等的，也就是自定义View默认情况下是屏幕大小
>3. MeasureSpec.UNSPECIFIED:--->指的是控件的宽高不受父控件的影响，想有多高就有多高，一般用于系统组件，咱们用不着
3. 重写onDraw()方法绘制控件的形状
protected void onDraw(Canvas canvas)	//用来画控件的长相的
mPaint.setAntiAlias(true);	//设置抗锯齿，为了让边缘平滑
```
    /**
     * 绘制控件的形状
     *
     * @param canvas
     */
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        //获取画笔的对象
        Paint paint = new Paint();
        paint.setColor(getResources().getColor(R.color.colorAccent));
        paint.setStrokeWidth(1);//设置线条(画笔)的高度
        paint.setAntiAlias(true);  //设置抗锯齿，边缘平滑
        //获取控件测量出来的宽高
        //这个宽高指的是控件没有超出屏幕情况下的宽和高 如果超出了屏幕，那么这个值就是屏幕的宽高
        //getWidth：获取的是控件的宽度
        //getHeight：获取的是控件的高度
        //只有超出了屏幕的情况下，下面这个才能获取真正的宽和高。如果没有超出屏幕，那么两种方法获取到的值是相等的。
        getMeasuredHeight();
        getMeasuredWidth();
        //画出自定义View的整体轮廓和形状
        canvas.drawRect(0, 0, getWidth(), getHeight(), paint);

    }
```
4. 如果需要添加事件的话就需要回调

### 自定义属性
1. 在Values目录下面建立一个attr的xml文件
2. 在里面添加属性
```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="diyTextView">
        <attr name="diyText" format="string" />
        <attr name="diyColor" format="string" />
    </declare-styleable>
    <declare-styleable name="diyEditView">
        <attr name="diyEditColor" format="color" />
        <attr name="diyPadding" format="integer" />
    </declare-styleable>
</resources>
```
3. 在需要使用自定义属性的地方引入
> xmlns:diyTextView ="http://schemas.android.com/apk/res-auto"

引入的时候，名字需要统一
>  diyTextView:diyText = "自定义属性"
4. 在自定义View中获取属性
- 第一种方式：
> int count = attrs.getAttributeCount();
    for (int i=0;i<count;i++){
       String attrName=attrs.getAttributeName(i);
       String attrValue=attrs.getAttributeValue(i);
        Log.e("---名称:"+attrName,"---值:"+attrValue);
    }
- 第二个方式：
> TypedArray array = context.obtainStyledAttributes(attrs, R.styleable.bobo);
    String text=array.getString(R.styleable.bobo_myText);
    String color=array.getString(R.styleable.bobo_myColor);
/*    Log.e("------------",text);
    Log.e("------------",color);*/
    array.recycle();  //表示循环的取出消息
尽量使用第二种方式，因为第一种方式在使用了资源引用的时候，获取的值是资源R文件里面的引用而不是值。

##### 这两种获取属性方式的却别是什么？
第一种方式获取属性值的时候，如果存在资源的引用的话没有办法直接获取属性值。第二种方式相当于第一种方式的封装，能够直接通过属性的名称来获取属性的值。

刷新当前控件的方法：
>invalidate();//刷新控件不能控件的宽和高
requestLayout();//刷新控件可以改变控件的宽高
### 继承控件
&emsp;&emsp;一般情况下也即是继承一个已知控件然后完成相应的多功能操作。
实例：仿照笔记本样式
```
/**
 * 仿照笔记本
 */
public class MyEditText extends EditText{
    int color=Color.BLACK;    //这个呢是这个颜色
    int padding=50;    //这个呢是那个边距
    public MyEditText(Context context, AttributeSet attrs) {
        super(context, attrs);
        setGravity(Gravity.LEFT | Gravity.TOP );  //这里的|相当于是与的关系，将EditText设置为从左上角开始
        setPadding(60,0,60,0);//设置左上右下的内边距
        setBackgroundResource(R.drawable.background );
    }
    /**
     * 重新布局自己的EditText
     * @param canvas
     */
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        Paint mPaint=new Paint();
        mPaint.setColor(color);    //设置那个画笔的颜色
        mPaint.setAntiAlias(true); //设置那个抗锯齿
        mPaint.setStrokeWidth(1);  //设置那个线条的宽度
        //开始划线
        //从哪里划线  到哪里结束?
        int viewHeight=getHeight();
        int viewWidth=getWidth();
        //下一步应该获取一共应该画多少线
        int lineCount=viewHeight/getLineHeight(); //计算出了一共需要画出多少线
        for (int i=0;i<lineCount-1;i++){
            canvas.drawLine(60,(i+1)*getLineHeight(),viewWidth-60,(i+1)*getLineHeight(),mPaint);//参数分别是：起点x，起点y，终点x，终点y，画笔
      }
}
```
在需要使用该控件的地方声明
```
<com.qf.mobilephone.android1606_28.view2.MyEditText
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:hint="请输入内容"
    />
```
### 组合控件
&emsp;&emsp;将多个已知的控件组合形成一个新的控件
案例：将ListView的一个item组合成一个
1. 编写一个View继承于一个布局
2. 在初始化这个控件的方法里面获取布局加载器
3. 编写需要组合的控件的模板-->用xml文件描述
4. 通过布局加载器加载布局文件
```
    //首先获取那个布局的加载器
    mLayoutInflater= (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
//将xml文件加载给当前对象
    View view = mLayoutInflater.inflate(R.layout.view3_layout, this);
```
5. 找id，编写方法
6. 在使用了该控件的地方实现该控件中的回调接口，并重写里面点击事件。
