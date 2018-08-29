###Bitmap的二次采样
#### 一、二次采样
##### (一)、意义和目的
1. 用BitmapFactory解码有一张图片时，有时会遇到错误，这往往是由于图片过大造成的。要想正常使用需要分配更少的内存空间来存储。
```BitmapFactory.decodeFile(imageFile);```
2. BitmapFactory.options.inSampleSize:这是恰当的inSampleSize可以使BitmapFactory分配更少的空间以消除该错误。
3. BitmapFactory.Options提供了另一个成员inJustDecodeBounds。设置inJustDecodeBounds为true后，decodeFile并不分配空间，但可计算出原始图片的长度和宽度，即opts.width和opts.height。有了这两个参数，再通过一定的算法，即可得到一个恰当的inSampleSize。
```
BitmapFactory.Options opts = new BitmapFactory.Options();
opts.inSampleSize = 4;
Bitmap bitmap = BitmapFactory.decodeFile(imageFile, opts);
```

##### (二)、获取inSampleSize
设置恰当的inSampleSize是解决该问题的关键之一。查看Android源码，我们得知，为了得到恰当的inSampleSize，Android提供了一种动态计算的方法。
```
private Bitmap createImageThumbnail(String filePath, int newHeight,
int newWidth) {
BitmapFactory.Options options = new BitmapFactory.Options();
options.inJustDecodeBounds = true;
BitmapFactory.decodeFile(filePath, options);
int oldHeight = options.outHeight;
int oldWidth = options.outWidth;
// Log.i(TAG, "高度是：" + oldHeight + "，宽度是：" + oldWidth);
int ratioHeight = oldHeight / newHeight;
int ratioWidth = oldWidth / newWidth;
options.inSampleSize = ratioHeight > ratioWidth ? ratioWidth
: ratioHeight;
options.inPreferredConfig = Config.RGB_565;
 
options.inJustDecodeBounds = false;
Bitmap bm = BitmapFactory.decodeFile(filePath, options);
// Log.i(TAG, "高度是：" + options.outHeight + "，宽度是：" + options.outWidth);
return bm;
}
```
#### 二、Bitmap占用内存的计算
##### (一)、概述：
&emsp;&emsp;Android中一张图片（Bitmap）占用的内存主要和以下几个因数有关：图片长度，图片宽度，单位像素占用的字节数。一张图片（Bitmap）占用的内存=图片长度*图片宽度*单位像素占用的字节数。注：图片长度和图片宽度的单位是像素。图片（Bitmap）占用的内存应该和屏幕密度(Density)无关。创建一个Bitmap时，其单位像素占用的字节数由其参数BitmapFactory.Options的inPreferredConfig变量决定。
##### (二)、inPreferredConfig
inPreferredConfig为Bitmap.Config类型，Bitmap.Config类是个枚举类型，它可以为以下值 Enum Values:
- Bitmap.Config  ALPHA_8
- Bitmap.Config ARGB_4444
- Bitmap.Config ARGB_8888
- Bitmap.Config RGB_565

##### (三）、图片格式占用内存的计算方法：以一张100*100px的图片占用内存为例
- ALPHA_8：图片长度*图片宽度 100*100=10000字节
- ARGB_4444：图片长度*图片宽度*2 100*100*2=20000字节
- ARGB_8888：图片长度*图片宽度*4 100*100*4=40000字节
- RGB_565：图片长度*图片宽度*2 100*100*2=20000字节

相关封装的代码
```
public class BitmapUtil {
    public static Bitmap  getBitmapTwo(int newHeight,int newWidth,String imgPath){
        BitmapFactory.Options options=new BitmapFactory.Options();
        options.inJustDecodeBounds=true;   //这句话的目的的是不让他生成对像
        Bitmap bitmap = BitmapFactory.decodeFile(imgPath, options);
        if(bitmap==null){
            Log.e("---------------","上面没有生成对象....");
        }
        int oldWidth=options.outWidth;
        int oldHeight=options.outHeight;
        Log.e("oldHeight:"+oldHeight,"oldWidth:"+oldWidth);
        int radioHeight=oldHeight/newHeight;   //得到那个缩放的倍数
        int radioWidth=oldWidth/newWidth;
        //设置图片的缩放的倍数
        options.inJustDecodeBounds=false;
        options.inSampleSize=radioHeight>radioWidth?radioWidth:radioHeight;
        Bitmap bitmap1= BitmapFactory.decodeFile(imgPath,options);
        return bitmap1;
    }
}

public class Main2Activity extends AppCompatActivity {
    ImageView mImageView;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main2);
        mImageView= (ImageView) findViewById(R.id.mImageView);
    }
    /**
     * 点击事件的处理
     * @param view
     */
    public void myClick(View view){
       switch (view.getId()){
           case R.id.mBtnNoObject:
               Bitmap bitmap=BitmapUtil.getBitmapTwo(120,120, Environment.getExternalStorageDirectory()+ File.separator+"bobo"+File.separator+"20140320111230_QdnTy.jpeg");
               try {
                   FileOutputStream out=new FileOutputStream(Environment.getExternalStorageDirectory()+File.separator+"bobo.jpg");
                   bitmap.compress(Bitmap.CompressFormat.PNG,100,out);
                   out.close();
               } catch (FileNotFoundException e) {
                   e.printStackTrace();
               } catch (IOException e) {
                   e.printStackTrace();
               }
               if(bitmap!=null){
                   mImageView.setImageBitmap(bitmap);
               }
               break;
       }
    }
}
```
