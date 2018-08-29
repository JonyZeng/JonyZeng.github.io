### LruCache
#### 缓存介绍
##### 一、Android中缓存的必须要性
&emsp;&emsp;在平时业务中，对于缓存特别的重视，是优化的重要部分，也是提升用户体验的有效手段之一。
1. 没有缓存的弊端：
- 流量开销：对于C/B模式，从远程获取图片算是经常要用的一个功能，而图片资源往往会消耗比较大的流量
- 加载速度：如果应用中图片加载速度很慢的话，用户就感觉体验非常差

解决办法：异步下载+本地缓存
2. 缓存带来的好处
- 服务器的压力减小
- 客户端的响应速度变快
- 客户端的数据加载出错情况减小，提高了应用的稳定性
- 一定程度上可以支持离线浏览
3. 缓存管理的应用场景
- 提供网络服务的场景
- 数据更新不需要实时更新
- 缓存的过期时间是可以接受的
4. 大位图导致内存开销大的原因是什么？
- 下载或加载的过程中容易导致阻塞(大位图Bitmap对象是png格式的30到100倍)
- 大位图在加载到ImageView控件前的解码过程：BitmapFactory.decodeFile()会有内存消耗。（decodeByteArray()
5. 缓存设计的要点
- 命中率
- 合理分配占用的控件
- 合理的缓存层级
##### 二、加载图片的正确流程
&emsp;&emsp;内存--文件--网络
1. 先从内存缓存中获取，取到则返回，取不到则进行下一步
2. 从文件缓存中获取，取到则返回数据，没取到就进行下面的网络请求
3. 从网络下载图片，更新到内存缓存和文件缓存
##### 三、内存缓存分类
&emsp;&emsp;在JDK1.2以前的版本中，当一个对象不被任何变量引用，那么程序就无法再使用这个对象。也就是说，只有对象处于可触及状态，程序才能使用它。这 就像在日常生活中，从商店购买了某样物品后，如果有用，就一直保留它，否则就把它扔到垃圾箱，由清洁工人收走。一般说来，如果物品已经被扔到垃圾箱，想再 把它捡回来使用就不可能了。但有时候情况并不这么简单，你可能会遇到类似鸡肋一样的物品，食之无味，弃之可惜。这种物品现在已经无用了，保留它会占空间，但是立刻扔掉它也不划算，因为也许将来还会派用场。对于这样的可有可无的物品，一种折衷的处理办法是：如果家里空间足够，就先把它保留在家里，如果家里空间不够，即使把家里所有的垃圾清除，还是无法容纳那些必不可少的生活用品，那么再扔掉这些可有可无的物品。
&emsp;&emsp;从JDK1.2版本开始，把对象的引用分为四种级别，从而使程序能更加灵活的控制对象的生命周期。
这四种级别由高到低依次为：强引用、软引用、弱引用和虚引用。
1. 强引用：(在Android中LruCache就是强引用缓存)
&emsp;&emsp;平时我们编程的时候例如：Object object=new Object（）；那object就是一个强引用了。如果一个对象具有强引用，那就类似于必不可少的生活用品，垃圾回收器绝不会回收它。当内存空间不足，Java虚拟机宁愿抛出OOM异常，使程序异常终止，也不会回收具有强引用的对象来解决内存不足问题。
2. 软引用（SoftReference）：
&emsp;&emsp;软引用类似于可有可无的生活用品。如果内存空间足够，垃圾回收器就不会回收它，如果内存空间不足了，就会回收这些对象的内存。只要垃圾回收器没有回收它，该对象就可以被程序使用。软引用可用来实现内存敏感的高速缓存。 软引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾回收，Java虚拟机就会把这个软引用加入到与之关联的引用队列中。
使用软引用能防止内存泄露，增强程序的健壮性。 
3. 弱引用（WeakReference）：   
&emsp;&emsp;弱引用与软引用的区别在于：只具有弱引用的对象拥有更短暂的生命周期。在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。不过，由于垃圾回收器是一个优先级很低的线程， 因此不一定会很快发现那些只具有弱引用的对象。  弱引用可以和一个引用队列（ReferenceQueue）联合使用，如果弱引用所引用的对象被垃圾回收，Java虚拟机就会把这个弱引用加入到与之关联的引用队列中。 
4. 虚引用（PhantomReference）   
&emsp;&emsp;"虚引用"顾名思义，就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收。 虚引用主要用来跟踪对象被垃圾回收的活动
虚引用与软引用和弱引用的一个区别在于：虚引用必须和引用队列(ReferenceQueue)联合使用。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中。程序可以通过判断引用队列中是否已经加入了虚引用，来了解被引用的对象是否将要被垃圾回收。程序如果发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象的内存被回收之前采取必要的行动。 
##### Lru：Least Recently Used
&emsp;&emsp;近期最少使用算法，是一种页面置换算法，其思想是在缓存的页面数目固定的情况下，那些最近使用次数最少的页面将被移除。对于内存缓存来说，强引用缓存大小固定为4M,如果当缓存的图片大于4M的时候，近期使用次数最少的图片就会被从强引用缓存中删除
##### 四、内存保存
&emsp;&emsp;在内存中保存的话，只能保存一定的量，而不能一直往里面存储，需要设置数据的过期时间、Lru算法等。工作中采用的解决方案是：将常用的数据放到一个缓存(A)中，不常用的数据放到另外一个缓存(B)中。当需要获取数据时，先从A中获取，如果A中没有数据，再去B中获取。B中的数据主要是A中Lru算法出来的数据。
这里的内存回收主要是针对B内存，从而保持A中的数据可以有效的被命中。
![方案解决.png](https://upload-images.jianshu.io/upload_images/7156039-76e464d12c9c7a82.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
Lru存放于获取数据的流程
![Lru数据存放流程图.png](https://upload-images.jianshu.io/upload_images/7156039-8af350207d4f928a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
代码实现图片的三级缓存，我已经封装了一个帮助类，如下：
```
package com.ting.android.viewpagefgdemo.RadioGroupAndViewPager;

import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.os.AsyncTask;
import android.os.Environment;
import android.util.LruCache;
import android.widget.ImageView;

import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.net.HttpURLConnection;
import java.net.MalformedURLException;
import java.net.URL;

/**
 * Lru缓存图片帮助类
 */
public class LruCacheUtils {

    private String folderPath = Environment.getExternalStorageDirectory() + "/jony";
    private LruCache<String, Bitmap> lruCache = null;
    private static LruCacheUtils lruCacheUtils = null;

    public static LruCacheUtils getInstance() {
        if (lruCacheUtils == null) {
            synchronized (LruCacheUtils.class) {
                lruCacheUtils = new LruCacheUtils();
            }
        }
        return lruCacheUtils;
    }

    private LruCacheUtils() {
        long maxMemory = Runtime.getRuntime().maxMemory() / 8;
        lruCache = new LruCache<String, Bitmap>((int) maxMemory) {
            /**
             * 获取每张图片的大小
             *
             * @param key
             * @param value
             * @return
             */
            @Override
            protected int sizeOf(String key, Bitmap value) {
                return value.getByteCount();
            }

            /**
             * 删除用的比较少的那些对象，不用重新写。但是在业务需求中，要知道有这么一个方法
             *
             * @param evicted
             * @param key
             * @param oldValue
             * @param newValue
             */
            @Override
            protected void entryRemoved(boolean evicted, String key, Bitmap oldValue, Bitmap newValue) {
                super.entryRemoved(evicted, key, oldValue, newValue);
            }
        };
    }

    /**
     * 从服务器获取图片
     *
     * @param imgUrl
     * @param mImageView
     */
    public void getImage(String imgUrl, ImageView mImageView) {
        //根据Lru算法获取图片
        //第一步、从内存中获取图片
        Bitmap bitmap = lruCache.get(imgUrl);
        if (bitmap == null) {//说明内存里面没有图片
            bitmap = getBitMapFromSdCard(imgUrl);   //第二步、从sdCar里面获取图片
            if (bitmap == null) {//说明sdCard里面也没有图片
                //第三步、只能从网站获取图片
                new MyAsyncTask(mImageView).execute(imgUrl);
            } else { //说明sdCard里面有图片
                lruCache.put(imgUrl, bitmap);
                mImageView.setImageBitmap(bitmap);
            }

        } else { //说明内存里面有图片
            mImageView.setImageBitmap(bitmap);
        }
    }

    /**
     * 从sdCard里面获取图片
     *
     * @param imgUrl
     * @return
     */
    private Bitmap getBitMapFromSdCard(String imgUrl) {
        String fileName = new File(imgUrl).getName();
        String newPath = folderPath + File.separator + fileName;
        File file = new File(newPath);
        if (file.exists()) { //判断文件是都存在
            return BitmapFactory.decodeFile(newPath);
        }
        return null;
    }

    /**
     * 异步从网络加载图片
     */
    private class MyAsyncTask extends AsyncTask<String, Void, Bitmap> {
        ImageView mImageView;
        String imgUrl = null;

        MyAsyncTask(ImageView mImageView) {
            this.mImageView = mImageView;
        }

        @Override
        protected Bitmap doInBackground(String... strings) {
            try {
                imgUrl = strings[0];
                URL url = new URL(strings[0]);
                HttpURLConnection httpURLConnection = (HttpURLConnection) url.openConnection();
                return BitmapFactory.decodeStream(httpURLConnection.getInputStream());
            } catch (MalformedURLException e) {
                e.printStackTrace();
            } catch (IOException e) {
                e.printStackTrace();
            }
            return null;
        }

        @Override
        protected void onPostExecute(Bitmap bitmap) {
            super.onPostExecute(bitmap);
            //第一步、将图片保存在内存中
            lruCache.put(imgUrl, bitmap);
            //第二步、将图片保存到sdCard
            saveImageToSdcard(imgUrl, bitmap);
        }

        /**
         * 将图片保存到sdCard中
         *
         * @param imgUrl
         * @param bitmap
         */
        private void saveImageToSdcard(String imgUrl, Bitmap bitmap) {
            try {
                FileOutputStream outputStream = null;
                String fileName = new File(imgUrl).getName();   //获取传入过来的图片名字
                //检测一个给定的sdCard路径下面是否存在文件夹
                File fileFolder = new File(folderPath);
                if (!fileFolder.exists()) { // 判断根目录是都存在
                    fileFolder.mkdirs();
                }
                //保存这个文件到文件夹
                String saveFilePath = folderPath + File.separator + fileName;//保存文件的路径
                File out = new File(saveFilePath);
                outputStream = new FileOutputStream(out);
                //三个参数：第一个、图片的格式 第二个、压缩的比较 100表示没压缩 第三个、输出流的位置，保存的位置
                bitmap.compress(Bitmap.CompressFormat.PNG, 100, outputStream);
            } catch (FileNotFoundException e) {
                e.printStackTrace();
            }
        }
    }

    /**
     * 设置那个Sdcard的存储的路径
     *
     * @param path
     */
    public void setCacheSdcardFolderPath(String path) {
        folderPath = path;
    }
}

```
使用方式：
```
LruCacheUtils.getInstance().getImage(imageUrl,mImageView);
```
### End 
关于图片的三级缓存是app性能优化之一，虽然现在又很多开源的图片加载框架，容易上手使用，但是我们还是要明白其中的原理。欢迎大家点赞关注哦！

