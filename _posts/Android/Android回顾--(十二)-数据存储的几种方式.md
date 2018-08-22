### ProgressDialog：
一个含有进度条以及消息提示的弹窗
使用方式：
```
// 1、创建对象
ProgressDialog dialog = new ProgressDialog(context);

// 2、通过对象调用相应的方法来完成信息的设置
dialog.setTitle("软件更新");
dialog.setMessage("软件正在更新，请勿关闭");
dialog.setProgressStyle(ProgressDialog.STYLE_HORIZONTAL);

// 3、设置事件的监听
 dialog.setButton(DialogInterface.BUTTON_NEGATIVE, "取消", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                dialog.cancel();
                dialog.dismiss();
            }
        });
// 4、show
dialog.show()
```
### 数据的存储
- sharePrefrence
- 本地的存储(内部存储/外部存储)
- 数据库的存储
- ContentProvider的存储
- 网络存储
#### 内部存储
&emsp;&emsp;内部存储实际上是手机中一块特定的存储区域，在这块区域中主要存放的是手机中安装程序和系统文件。
#### 内部存储的访问
1. 文件的创建
```
 private void createFile() throws IOException{
        File file = getFilesDir();     //通过方法获取文件类的对象
        File file1 = new File(file,"hello.txt");
        if (!file1.exists()){
            file1.createNewFile();
        }
    }
```
2. 向内部存储写入文件内容
```
   private void writeContentToFile(){
       FileOutputStream out = null;
       try {
           File file = new File(getFilesDir(),"hello.txt");
           out = new FileOutputStream(file);
           out.write("yeah!this is JonyZ ".getBytes());
       }catch (FileNotFoundException e) {
           e.printStackTrace();
       } catch (IOException e) {
           e.printStackTrace();
       }finally {
           try {
               out.close();
           } catch (IOException e) {
               e.printStackTrace();
           }
       }
   }

```
3 . 读取文件里面的内容
```
   private void readFileContent(){
        File file = new File(getFilesDir(),"hello.txt");
        try {
            FileInputStream in = new FileInputStream(file);
            byte[] buf = new byte[(int) file.length()]; //这样写是一直文件非常小的时候，直接使用文件的长度。但是，一般在工作中，用于读写的文件都非常的大，不能这样写。后续我会详细的讲解IO流
            in.read(buf);
            in.close();
            String result = new String(buf);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

```
4、创建目录
file.mkdirs():创建级联目录
file.mkdir():创建单个目录
```
File file = getDir("jony",Context.MODE_PRIVATE);  //第一个参数，表示当前应用下的根目录创建目录。第二个参数，私有模式。只有当前应用可以访问
String path = getActivity().getFilesDir().getAbsolutePath()+"jony";
new File(path).mkdir();

```
5、删除目录或者文件
```
    public void deleteFileOrDir(File file){
        if (file.isFile()){
            file.delete();
        }else if (file.isDirectory()){  //如果当前要删除的文件是目录，需要先删除里面的，再删除当前文件
            File[] files = file.listFiles();
            if (files!=null){
                for (int i = 0; i < files.length; i++) {
                    deleteFileOrDir(files[i]);
                }
            }

        }
    }

```
### SharePrefences：轻量级存储
1. 用于何处：
    - 放置用户的登陆信息
    - 放置软件的配置信息
    - 放置一些临时数据

2. 特点：
    - 数据的存储采用的是键值对的形式来进行存储的
    - 里面的键是不能重复的
    - 里面的值是可以重复的 (map)
#### 使用步骤：
1. 添加数据到sp
    - 获取sp对象
SharePrefences sp = getSharePreferences("login",Context.MODE_PRIVATE);
第一个参数：表示sp里面的xml文件的名称，不能添加xml
第二个参数：表示创建的文件的访问权限，一般都是写private表示只能够自己访问这个文件，其他的应用不能访问。
    - 获取Edit对象
Editor edit = sp.edit();
    - 通过Edit对象的put方法来向里面添加数据
edit.putBoolean("key1",true);
edit.putString("userName","jony");
edit.putInt("key2",0825);
    - 添加完数据之后，必须提交这个事物，不提交就没有办法将数据添加进去
edit.commit();
2. 访问sp,从sp中获取数据
    - 获取sp对象
SharePrefences sp = getSharePreferences("login",Context.MODE_PRIVATE);
    - 通过sp的get方法获取数据
sp.getString("userName","");   // 获取数据
第一个参数：获取数据的键
第二个参数：如果没有通过键获取到值，默认的值
boolean b = sp.getBoolean("key1",false);
String userName = sp.getString("userName","");
int key2 = sp.getInt("key2",-1);
3. 清空sp
    - 获取sp对象
    - 获取Edit对象
    - 通过edit.clear()或者edit.remove()来选择删除数据
    - 提交这个事件的请求

### SDcard文件的相关操作
具体的我已经封装好了一个帮助类，如下
```
import android.os.Environment;
import android.os.StatFs;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;

public class SDUtils {
    /**
     * 检测SdCard是否已经挂载
     *
     * @return
     */
    public static boolean checkSdCardExitOrNot() {
        String state = Environment.getExternalStorageState();   //获取SdCard状态
        if (Environment.MEDIA_MOUNTED.equals(state)) {   //挂载状态，可读可写
            return true;
        } else if (Environment.MEDIA_MOUNTED_READ_ONLY.equals(state)) {   //已挂载但只可读
            return false;
        } else {
            return false;
        }
    }

    /**
     * 获取当前SdCard的剩余空间大小
     *
     * @return
     */
    public static int getFreeSpaceSize() {
        //第一步:获取SdCard的根目录
        File file = Environment.getExternalStorageDirectory();
        //第二步：获取StaFs-->这个类是专门用来获取系统消息的
        StatFs fs = new StatFs(file.getAbsolutePath());
        //第三步 获取每一个块的大小
        int blockSize = fs.getBlockSize();//注意，这个方法在api 18的时候已经过时了，目前版本使用的是         fs.getBlockCountLong();
//获取空闲块的数量
        int availableBlocks = fs.getAvailableBlocks();
        return availableBlocks * blockSize / 1024 / 1024;
    }

    /**
     * 判断文件是否存在
     *
     * @param path
     * @return
     */
    public static boolean checkFileExistOrNot(String path) {
        File file = new File(path);
        if (!file.exists()) {
            return false;
        }
        return true;
    }

    /**
     * 判断文件是否存在，不存在就创建
     *
     * @param path
     */
    public static void createFile(String path) {
        File file = new File(path);
        if (!file.exists()) {
            try {
                file.createNewFile();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    /**
     * 创建目录
     *
     * @param path
     */
    public static void createDir(String path) {
        File file = new File(path);
        if (!file.exists()) {
            file.mkdirs();
        }
    }

    /**
     * 删除文件
     *
     * @param file
     */
    public static void deleteFile(File file) {
        if (file.isFile()) { //如果是单文件，那么直接删除
            file.delete();
        } else if (file.isDirectory()) {  //当前文件是目录，先删除下面的文件，再删除当前文件
            File[] files = file.listFiles();
            for (int i = 0; i < files.length; i++) {
                deleteFile(files[i]);
            }
        }
        file.delete();
    }

    public static void copyFile(File fileSource, File fileDesternation) {
        FileInputStream in = null;
        FileOutputStream out = null;
        try {
            in = new FileInputStream(fileSource);
            out = new FileOutputStream(fileDesternation);
            byte[] buf = new byte[1024];
            int length;
            while ((length = in.read(buf)) != -1) {
                out.write(buf, 0, length);
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                in.close();
                out.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

}

```
