### ContentPrivider
#### 简介
&emsp;&emsp;作为四大金刚之一，ContentProvider其实是比较低调的。在我平时工作和业务需求中，很少使用到。它的诞生就是为了给不同的应用提供内让访问。  
&emsp;&emsp;ContentProvider封装了数据的跨进程传输，我们可以世界使用getContentResolver()获取到对象，进行数据的增删查改。内容提供者是以一个或者多个表(和关系型数据库中的表类似)的形式将数据呈现给外部应用。行表示提供程序收集的某种数据类型的实例，行中的每个列表示为实例收集的每条数据。  
&emsp;&emsp;实现一个 ContentProvider 时需要实现以下几个方法：  
- onCreate():初始化provider
- query():查询数据
- insert:插入数据到provider
- update():更新provider的数据
- delete():删除provider的数据
- getType():返回provider中的数据的MIME类型

注意：
1. onCreate默认执行在主线程，不能做耗时的操作。query也最好进行异步处理
2. 上面的4个增删查改操作都可能被多个线程并发访问，因此要注意线程安全

#### ContentProvider与URI
&emsp;&emsp;ContentProvider使用URI表示要操作的数据，这里的URI包含以下两个部分：
- authority：整个提供程序的符号名称
- path：指向表的名称/路径

内容URI统一的形式：content://authority/path
```
content://user_dictionary/words
```
当你调用 ContentResolver 方法来访问 ContentProvider 中的表时，需要传递要操作表的 URI。  

在通过 ContentResolver 进行数据请求时（比如 contentResolver.insert(uri, contentValues);）， 系统会检查指定 URI 的 authority 信息，然后将请求传递给注册监听这个 authority 的ContentProvider 。这个 ContentProvider 可以监听 URI 想要操作的内容，Android 中为我们提供了 UriMatcher 来解析 URI  
#### 权限问题
由于内容提供者要被不同应用访问，因此权限必不可少。我们可以给内容提供者设置 “读/写”权限。  
设置自定义权限分三步:
1. 向系统声明一个权限  

```
<permission
    android:name="top.shixinzhang.permission.READ_CONTENT"    //指定权限的名称
    android:label="Permission for read content provider"
    android:protectionLevel="normal"    
    />
```  
其中 android:protectionLevel可选的值主要如下：  
normal：低风险，任何应用都可以申请，在安装应用时，不会直接提示给用户  
dangerous：高风险，系统可能要求用户输入相关信息才授予权限，任何应用都可以申请，在安装应用时，会直接提示给用户  
signature：只有和定义了这个权限的 apk 用相同的私钥签名的应用才可以申请该权限  
signatureOrSystem：有两种应用可以申请该权限   
和定义了这个权限的 apk 用相同的私钥签名的应用  
在 /system/app 目录下的应用  
这里我们设置的值为 normal。  

2. 给要设置权限的组件设置需要这个权限  

```
<provider
    android:name=".provider.IPCPersonProvider"
    android:authorities="net.sxkeji.shixinandroiddemo2.provider.IPCPersonProvider"
    android:exported="true"    
    android:grantUriPermissions="true"
    android:process=":provider"
    android:readPermission="top.shixinzhang.permission.READ_CONTENT">
```  
这个权限无法在运行时请求，必须在清单文件中使用 <uses-permission> 元素和内容提供者定义的准确权限名称指明你的权限。  

3. 在想要使用上述组件的应用中注册这个权限  

```
<uses-permission android:name="top.shixinzhang.permission.READ_CONTENT"/>
```
在您的清单文件中指定此元素后，您将有效地为应用“请求”此权限。 用户安装您的应用时，会隐式授予允许此请求  

#### 支持的数据类型
Android 本身包括的内容提供程序可管理音频、视频、图像和个人联系信息等数据。  
内容提供者可以提供多种不同的数据类型:
- int
- long
- double
- float
- BLOB //作为64KB字节的数组的二进制大型对象

ContentProvider 还会维护其定义的每个内容 URI 的 MIME 数据类型信息。  

你可以使用 MIME 类型信息确定应用是否可以处理 ContentProvider 提供的数据，或根据 MIME 类型选择处理类型。  

在使用包含复杂数据结构或文件的提供程序时，通常需要 MIME 类型。  
### ContentProvider 的使用
ContentProvider 的使用分为以下 4 步：
1. 设计数据存储
      - 选择文件还是数据库
      - 如果您想提供 Bitmap 或其他庞大的文件导向型数据，请将数据存储在一个文件中，然后间接提供这些数据，而不是直接将其存储在表中
      - 使用二进制大型对象 (BLOB) 数据类型存储大小或结构会发生变化的数据。 例如使用 BLOB 列来存储 JSON
2. 创建 ContentProvider 子类，实现关键方法 
    - ContentProvider 实例通过处理来自其他应用的请求来管理对结构化数据集的访问
    - 所有形式的访问最终都会调用 ContentResolver，后者接着调用 ContentProvider 的具体方法来获取访问权限
    - 注意文章开头提到的避免耗时操作和线程安全
    - 尽管必须实现这些方法，它们的返回值并不重要，只要返回符合要求的数据类型即可，即使不执行任何其他操作
3. 定义提供程序的授权字符串（authority）、内容 URI 以及列名称 
    - 对应前面设计的数据库表名和字段名
    - 如果想让内容提供者应用处理 Intent，则还要定义 Intent 操作、Extra 数据以及标志
    - 还要定义想要访问该数据的应用必须具备的权限
4. 通过 ContentResolver 和 URI 进行增删改查

&emsp;&emsp;写了这么多，其实我对于ContentProvider的使用是真的少得可怜，所以大部分知识点都是查找资料之后，总结出来的。请大家多多包涵。


