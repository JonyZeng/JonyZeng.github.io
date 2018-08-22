### AIDL：
&emsp;&emsp;一种接口定义语言来公开服务的接口，跨进程访问的服务
### 建立AIDL服务的步骤：
##### 服务端的实现
1. 在Java源文件目录下建立一个aidl文件，直接通过点击new创建aidl文件，Android Studio会自动帮你新建aidl包。具体位置如图：
![aidl文件位置.png](https://upload-images.jianshu.io/upload_images/7156039-6ce4ad7f5f81fbe2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
// UserInfo.aidl
package com.ting.android.aidldemo;

// Declare any non-default types here with import statements

interface UserInfo {
    String getInfo();
}

```
aidl文件的内容与Java代码非常相似，但是要注意，不能加任何权限修饰符，AIDL不支持的数据类型。
2. 如果UserInfo.aidl文件中的内容输入正确，我们rebuild项目，ADT会自动生成一个UserInfo.java文件，我们开发中不需要关注这个文件的内容。
3. 编写一个Service类，MyAidlServletService类继承于Service，在MyAidlServletService类中定义了一个内部类(MyBinder),该类继承自UserInfo.Stub。
```
public class MyAidlServletService extends Service {

    public class MyBinder extends UserInfo.Stub{

        @Override
        public String getInfo() throws RemoteException {
            return "这里是服务端返回的信息";
        }
    }
    public MyAidlServletService() {
    }

    @Override
    public IBinder onBind(Intent intent) {
       return new MyBinder();  //这里一定要返回内部类的对象
    }
}
```
UserInfo.Stub是自动生成的，不需要关注。只需要继承之后，实现要调用的方法即可。在onBind方法中必须返回内部类的对象，否则客户端无法获取服务对象
4. 在AndroidMainifest.xml文件中配置MyAidlServletService类。
```
        <service
            android:name=".MyAidlServletService"
            android:enabled="true"
            android:exported="true">
            <intent-filter>
                <action android:name="com.ting.android.aidldemo.UserInfo" />
            </intent-filter>
        </service>
```
其中，"com.ting.android.aidldemo.UserInfo"是客户端要访问AIDL服务的ID。客户端要保持一致。
##### 客户端的实现
1. 将服务端的aidl文件包直接拷贝到客户端同样的目录下面。
![客户端aidl.png](https://upload-images.jianshu.io/upload_images/7156039-bd61d318c430d7ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2. 新建一个ServiceConnection，在里面获取到UserInfo的对象
```
    /**
     * 需要绑定远程的Service
     */
    private class MyServletConn implements ServiceConnection {

        @Override
        public void onServiceConnected(ComponentName componentName, IBinder iBinder) {
            userInfo = UserInfo.Stub.asInterface(iBinder);
            Log.i("AIDL: ","绑定成功");
        }

        @Override
        public void onServiceDisconnected(ComponentName componentName) {
            userInfo = null;
        }
    }
```
3. 通过intent绑定Service
```
        Intent mIntent = new Intent("com.ting.android.aidldemo.UserInfo"); //服务器配置的 action name 要保持一致。
        mIntent.setPackage("com.ting.android.aidldemo");    //这里设置服务器的包名，这样避免要求显示调用
        bindService(mIntent, new MyServletConn(), BIND_AUTO_CREATE);
```
### 传递复杂数据的AIDL服务
&emsp;&emsp;AIDL服务只支持有限的数据类型，因此，如果用AIDL服务传递一些复杂的数据就需要做进一步处理，AIDL服务支持的数据类型如下：
- Java的简单类型(int、char、boolean)，不需要导入(import)
- String和CharSequence。不需要导入(import)
- List和Map。但是要注意，List和Map对象的元素类型必须是AIDL服务支持的数据类型。不需要导入(import)
- AIDL自动生成的接口。需要导入(import)
- 实现android.os.Parcelable接口的类，需要导入(import)
对于传递需要导入的复杂数据类型，具体的操作步骤如下
##### 服务端实现
1. 创建一个IMyAidlInterface.aidl文件
```
// IMyAidlInterface.aidl
package com.ting.android.aidldemo;

//注意：这里有一个巨坑，这包名不能加aidl。不然会报错
import com.ting.android.aidldemo.Product;

interface IMyAidlInterface {
    Map getMap(in String country, in Product product);
    Product getProduct();
}
```
&emsp;&emsp;Product是一个实现android.os.Parcelable接口的类，需要使用import导入这个类。如果方法的类型是非简单类型，例如，String、List或者自定义的类，需要使用in、out或inout进行修饰，其中in表示这个值可以被客户端设置，out表示这个值可以被服务端设置，inout表示这个值既可以被客户端设置，又可以被服务端设置。
2. 编写Product类，该类是用于传递的数据类型
```
public class Product implements Parcelable {
    private int id;
    private String name;
    private float price;

    public static final Parcelable.Creator<Product> CREATOR = new Parcelable.Creator<Product>() {

        @Override
        public Product createFromParcel(Parcel parcel) {
            return new Product(parcel);
        }

        @Override
        public Product[] newArray(int i) {
            return new Product[i];
        }
    };

    public Product() {
    }

    Product(Parcel in) {
        readFromParcel(in);
    }

    @Override
    public int describeContents() {
        return 0;
    }

    public void readFromParcel(Parcel parcel) {
        id = parcel.readInt();
        name = parcel.readString();
        price = parcel.readFloat();
    }


    @Override
    public void writeToParcel(Parcel parcel, int i) {
        parcel.writeInt(id);
        parcel.writeString(name);
        parcel.writeFloat(price);
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public float getPrice() {
        return price;
    }

    public void setPrice(float price) {
        this.price = price;
    }

    @Override
    public String toString() {
        return "Product{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", price=" + price +
                '}';
    }
}
```
&emsp;&emsp;Product类必须实现android.os.Parcelable接口，该接口用于序列化。在Product类中必须有一个静态常量，常量名必须是CREATOR，而且CREATOR常量的数据类型必须是Parcelable.Creator。通过实现newArray和createFromParcel方法可以将序列化的数据转换为Product对象和Product数组。在readFromParcel方法中需要从序列化数据读取相应的值。在writeToParcel方法中需要将要序列化数据写入Parcel对象。从这一点，对象序列化和反序列化的过程是通过手工完成的。
3. 编写一个Product.aidl文件
```
package com.ting.android.aidldemo;

parcelable Product;
```
4. 编写一个IMyAidlInterface类
```
// IMyAidlInterface.aidl
package com.ting.android.aidldemo;

//注意：这里有一个巨坑，这包名不能加aidl。不然会报错
import com.ting.android.aidldemo.Product;

interface IMyAidlInterface {
    Map getMap(in String country, in Product product);
    Product getProduct();
}
```
5. 在Androidmainifest.xml文件中配置IMyAidlInterface
```
        <service
            android:name=".MyService"
            android:enabled="true"
            android:exported="true">
            <intent-filter>
                <action android:name="com.ting.android.aidldemo.MyService"/>
            </intent-filter>
        </service>
```
##### 客户端实现
1. 将服务端的aidl文件包直接拷贝到客户端同样的目录下面。
![服务端aidl.png](https://upload-images.jianshu.io/upload_images/7156039-8b847ff3ddf20511.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2. 新建一个ServiceConnection，在里面获取到UserInfo的对象
```
    private class MyServiceConnection implements ServiceConnection {

        @Override
        public void onServiceConnected(ComponentName componentName, IBinder iBinder) {
            iMyAidlInterface = IMyAidlInterface.Stub.asInterface(iBinder);
            Log.i("AIDL: ", "绑定成功");
        }

        @Override
        public void onServiceDisconnected(ComponentName componentName) {
            iMyAidlInterface = null;
        }
    }
```
3. 通过intent绑定Service
```
        Intent mIntent2 = new Intent("com.ting.android.aidldemo.MyService");
        mIntent2.setPackage("com.ting.android.aidldemo");    //这里设置服务器的包名，这样避免要求显示调用
        bindService(mIntent2, new MyServiceConnection(), BIND_AUTO_CREATE);
```
###注意：
在开发过程中，对于复杂类型需要导入，容易出现找不到导入的包。需要在gradle.build中添加如下代码。
```
    sourceSets {
        main {
            java.srcDirs = ['src/main/aidl']
        }
    }

    sourceSets {
        main {
            manifest.srcFile 'src/main/AndroidManifest.xml'
            java.srcDirs = ['src/main/java', 'src/main/aidl']
            resources.srcDirs = ['src/main/java', 'src/main/aidl']
            aidl.srcDirs = ['src/main/aidl']
            res.srcDirs = ['src/main/res']
            assets.srcDirs = ['src/main/assets']
        }
    }
```
Github地址：https://github.com/JonyZeng/AIDLDemo
