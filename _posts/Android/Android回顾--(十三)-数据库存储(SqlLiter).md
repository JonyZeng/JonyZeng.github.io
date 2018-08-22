### SqlLiter：
&emsp;&emsp;小型数据库，占用资源少，操作简单。
使用方法：
#### 第一种使用方式：
1. 通过上下文的openOrCreateDatabase("jony.db",Context.MODE_PRIVATE,null);方法来创建和打开数据库
2. 通过返回的SqliteDataBase对象进行对数据库的CRUD
3. 每一次用完之后，都必须关闭数据库。
具体的使用方式，参照我封装的一个数据库帮助类
```
package com.example.jonyz.demoe;

import android.content.ContentValues;
import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;

public class SQLUtil {
    /**
     * 创建数据库
     */
    private void createDataBase(Context context) {
        SQLiteDatabase database = context.openOrCreateDatabase("jony.db", Context.MODE_PRIVATE, null);
        database.close();
    }

    /**
     * 创建表
     */
    private void createTable(Context context) {
        SQLiteDatabase database = context.openOrCreateDatabase("jony.db", Context.MODE_PRIVATE, null);  // 创建或拿到数据库对象
        String sql = "create table student(num Integer, name varchar(20), gender varchar(5), JAVA Integer, Servlet Integer)";   //编写sql执行语句
        database.execSQL(sql);//执行sql语句
        database.close();   // 关闭数据库
    }

    /**
     * 插入数据
     *
     * @param context
     */
    private void insertData(Context context) {
        SQLiteDatabase database = context.openOrCreateDatabase("jony.db", Context.MODE_PRIVATE, null);
        //第一种方式插入数据，通过写sql语句操作数据库
        String sql = "insert into student values(149,'JonyZ','boy',99,99)";
        database.execSQL(sql);
        //第二种方式插入数据，通过insert方法插入数据
        ContentValues contentValues = new ContentValues();
        contentValues.put("name", "帅哥");
        contentValues.put("gender", "boy");
        contentValues.put("num", 123);
        contentValues.put("JAVA", 99);
        contentValues.put("Servlet", 98);
        database.insert("student", "name", contentValues);

        database.close();
    }

    /**
     * 更新数据
     *
     * @param context
     */
    private void updateData(Context context) {
        SQLiteDatabase database = context.openOrCreateDatabase("jony.db", Context.MODE_PRIVATE, null);
        //第一种方式插入数据，通过写sql语句更新数据库
        String sql = "update student set gender='boy' where gender is null";
        database.execSQL(sql);
        //第二种方式通过insert方法更新数据
        ContentValues contentValues = new ContentValues();
        contentValues.put("name", "狗蛋");
        contentValues.put("gender", "other");
        database.update("student", contentValues, "gender=?", new String[]{"boy"});

        database.close();
    }

    /**
     * 查询数据库
     *
     * @param context
     */
    private void queryData(Context context) {
        SQLiteDatabase database = context.openOrCreateDatabase("jony.db", Context.MODE_PRIVATE, null);
        //第一种方式插入数据，通过写sql语句查询数据库
        String sql = "select name,gender,JAVA+Servlet as '总成绩' from student";
        Cursor cursor = database.rawQuery(sql, null);
        while (cursor.moveToNext()) {    //通过返回的游标查询数据
            Student stu = new Student();  // 可以通过gsonfromat生成实体类
            stu.setName(cursor.getString(cursor.getColumnIndex("name")));
            stu.setGender(cursor.getString(cursor.getColumnIndex("gender")));
            stu.setJava(cursor.getString(cursor.getColumnIndex("总成绩")));
            stus.add(stu);
        }
        database.close();
    }

    private void deleteData(Context context) {
        SQLiteDatabase database = context.openOrCreateDatabase("jony.db", Context.MODE_PRIVATE, null);
        //第一种方式删除数据库
        String sql = "delete from student where num = 123";
        database.execSQL(sql);
        // 第二种方式删除数据库
        database.delete("student", "gender=?", new String[]{"other"});

        database.close();
    }
}
```
#### 第二种使用方式
&emsp;&emsp;编写一个类继承SqlLiteOpenHelper通过重写里面的方法进行数据库的操作。具体的使用方式，参照我封装的帮助类。  
```
package com.example.jonyz.demoe;

import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;

import java.util.List;

public class DBHelper extends SQLiteOpenHelper {


    public DBHelper(Context context, String name, SQLiteDatabase.CursorFactory factory, int version) {
        super(context, name, factory, version);
    }

    //不是初始化这类的对象的时候创建数据库，而是在第一次访问数据库的时候创建数据库并调用里面的onCreate方法
    @Override
    public void onCreate(SQLiteDatabase sqLiteDatabase) {
        // onCreate()方法仅仅执行一次 在创建数据库的时候执行
        String sql = "create table People(_id integer primary key autoincrement, userName varchar(20),gender varchar(5))";
        sqLiteDatabase.execSQL(sql);//注意，创建表的时候是不能够关闭的 一旦关闭了数据库，后面就无法获取数据库
    }

    /**
     * 数据库的版本更新
     *
     * @param sqLiteDatabase
     * @param i
     * @param i1
     */
    @Override
    public void onUpgrade(SQLiteDatabase sqLiteDatabase, int i, int i1) {
        if (i == 1 && i1 == 2) {    //从1升级到2这个版本号
            String sql = "alter table People add address varchar(50)";
            String sql1 = "create table Dog(_id integer primary key autoincrement, dogName varchar(20), dogSex varcahr(10))";
            sqLiteDatabase.execSQL(sql);
            sqLiteDatabase.execSQL(sql1);   //不能关闭数据库连接
        }
    }

    /**
     * 插入数据
     *
     * @param tableName 表名
     * @param proples   插入对象
     */
    public void insertData(String tableName, List<Prople> proples) {
        SQLiteDatabase database = getWritableDatabase();    //获取一个可写的数据库对象
        String sql = "insert into Prople(userName,gender) values('美女','boy')";
        database.execSQL(sql);
        database.close();
    }

    /**
     * 更新数据库
     */
    public void upadteData() {
        SQLiteDatabase database = getWritableDatabase();    //获取一个可写的数据库对象
        String sql = "update People set userName = '张三'";
        database.execSQL(sql);
        database.close();
    }

    /**
     * 查询数据库
     */
    public void queryData(){
        SQLiteDatabase database = getWritableDatabase();    //获取一个可写的数据库对象
        String sql = "select userName,_id from People";
        Cursor cursor = database.rawQuery(sql, null);
        while (cursor.moveToNext()){    //一个一个的往下走，进行查询
            String suerName = cursor.getString(cursor.getColumnIndex("userName"));
            int id = cursor.getInt(cursor.getColumnIndex("_id"));
        }
    }

    public void deleteData(){
        SQLiteDatabase database = getWritableDatabase();    //获取一个可写的数据库对象
        String sql = "delete from People where _id=2";
        database.execSQL(sql);
        database.close();
    }
}
```
对比上面数据库操作方式，一般在工作中常用的是第二种。当然，现在还有非常多的第三方数据库操作库，比如：OrmLite、LitePal、GreenDao3.2、DBFlow、Realm。我这里就不过多的阐述了。
### SimpleCursorAdapter:
```
SimpleCursorAdapter(CursorActivity.this,R.layout.simple_cursor_adapter_item,cursor,new String[]{"userName","gender"},new int[]{R.id.mTextViewName,R.id.mTextViewGender});
     第一个参数:上下文
     第二个参数:模板
     第三个参数:游标
     第四个参数:数据库表结构里面的列（我们需要将列的值赋给模板的控件）
     第五个参数:就是那个模板上面的控件的id的集合
  注意:如果咋们的表没有自增的_id的话那么这个是没法用的
```
