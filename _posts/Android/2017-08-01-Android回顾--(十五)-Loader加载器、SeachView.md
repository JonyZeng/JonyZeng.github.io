### Loader:加载器
&emsp;&emsp;Loader实际上是官方封装的一个异步任务，这个异步任务能够很好的处理CPU和资源之间的关系，以及最大的优点是能够动态的刷新内容。

Loader：
- AsyncTaskLoader(主要用于加载耗时的操作)
- CursorLoader(主要用作加载系统内部的数据)

### Loader的用法：
1. 通过上下文获取LoaderManger对象
```
LoaderManager loadManager = MainActivity.this.getLoaderManager();  //用来管理Loader的
```
2. 通过LoaderManager对象初始化Loader对象
```
  /**
     *第一个参数 id:Loader的唯一标识，自定义的，如果一个Activity里面包含有多个Loader的话那么这个Id是不能重复的
     *第二个参数 bundle：初始化Loader的时候需要传递的参数
     *第三个参数 ：Loader的监听回调函数，自定义一个类实现LoaderCallBack这个回调的接口，并且实现里面的方法。
     */
   loadManager.initLoader(2, null, new LoaderManager.LoaderCallbacks<Cursor>() {
            /**
             *当试图操作一个加载器是，会检查是否制定ID的装载器已经存在，如果不存在，就会调用这个方法
             * @param i 就是之前我们自定义的id值
             * @param bundle    也是之前初始化的时候给的值
             * @return
             */
            @Override
            public Loader<Cursor> onCreateLoader(int i, Bundle bundle) {
                //这个方法里面返回一个Loader对象，1、使用CursorLoader时，直接new一个CursorLoader对象。2、使用AsyncTask时，这个Loader对象是通过一个类继承于AsyncTaskLoader这个类的子类
                 return new MyAsyncTaskLoader(MainActivity.this);
            }

            /**
             * 类加载完成的时候进行调用
             * @param loader loader 对象
             * @param cursor 运行结果返回值cursor
             */
            @Override
            public void onLoadFinished(Loader<Cursor> loader, Cursor cursor) {
                while(cursor.moveToNext()){
                    String name = cursor.getString(cursor.getColumnIndex("name"));
                }
                mSimpleCursorAdapter.swapCursor(cursor);
            }

            @Override
            public void onLoaderReset(Loader<Cursor> loader) {
                mSimpleCursorAdapter.swapCursor(null);
            }
        });
```
3. 获取一个SimpleCursorAdapter对象,在onCreate的时候  
```
   //SimpleCursorAdapter.FLAG_REGISTER_CONTENT_OBSERVER:表示的是给当前的这个适配器设置一个观察者这个观察者允许那个数据的改变
        mSimpleCursorAdapter=new SimpleCursorAdapter(MainActivity.this,android.R.layout.simple_list_item_1,cursor,new String[]{"display_name"},new int[]{android.R.id.text1},SimpleCursorAdapter.FLAG_REGISTER_CONTENT_OBSERVER);
        mListView.setAdapter(mSimpleCursorAdapter);
```  
4. 自定义AsyncTaskLoader  
```
/**
 * 自定义AsyncTaskLoader
 */
class MyAsyncTaskLoader extends AsyncTaskLoader{
    Context context;
    public MyAsyncTaskLoader(Context context) {
        super(context);
        this.context=context;
    }

    /**
     * 这个方法是干嘛的?
     * 这个就是AsyncTakLoader的异步的逻辑的处理函数
     * @return
     */
    @Override
    public Cursor loadInBackground() {
        //假设咋们这里需要去查询本地的通讯录  那么咋们需要返回的是一个游标
        ContentResolver contentResolver = context.getApplicationContext().getContentResolver();
        Cursor cursor = contentResolver.query(ContactsContract.Contacts.CONTENT_URI, null, null, null, null);
        return cursor;
    }
    /**
     * 重写这个方法的目的就是让数据都能够强制的进行加载
     */
    @Override
    protected void onStartLoading() {
        super.onStartLoading();
        forceLoad();    //这一个表示的是强制的进行数据的加载
    }
}
```

### Loader中的使用的注意事项
1. 在onLoadFinished方法中，我们需要使用当前适配器对象的swapCursor来设置这个返回的数据，这个适配器只能是SimpleCursorAdapter
2. 在内容需要重置的时候，需要使用adapter.swapCursor(null)来复位数据。
3. 如果我们需要重新启动Loader的时候，我们需要调用getLoaderManager().restartLoader(2,mBundle,new MyLoaderCallBack());来重启那个Loader.

### SeachView的使用
1. 首先需要声明这个控件
2. 找到控件并且给当前的控件设置内容改变的监听。
```
     mSearchView.setOnQueryTextListener(new MyOnQueryListener());
```
这个对象的接口里面包含有两个方法
```
  onQueryTextSubmit(String searchText)   :这个方法表示的意思是当咋们按下了那个搜索按钮或者按下了那个enter键的时候调用
  onQueryTextChange(String newText)      :只要那个EditText里面的内容发生了改变那么咋们的这个方法都会被调用(数据增加/数据减少)
```

### 用EditText也可实现搜索功能：
1. 声明控件EditText
2. 给EditText添加文本改变的事件监听
mEditText.addTextChangedListener(TextWatcher watcher);
括号内的参数是一个TextWatcher的接口
3. 自定义一个类实现TextWatcher，并重写里面的方法
```
   /**
     * 这个就是那个EditText的事件的监听以及处理
     */
    private class MyOnChangeListener implements TextWatcher{
        /**
         * 在文本的改变之前来进行触发
         * @param s
         * @param start
         * @param count
         * @param after
         */
        @Override
        public void beforeTextChanged(CharSequence s, int start, int count, int after) {
          Log.e("-------beforeTextChanged---",s.toString());
        }

        /**
         * 在文本改变的时候来进行触发
         * @param s:改变后的文本
         * @param start
         * @param before
         * @param count
         */
        @Override
        public void onTextChanged(CharSequence s, int start, int before, int count) {
            Log.e("-----onTextChanged----",s.toString());
        }

        /**
         * 在文本的改变之后来进行触发
         * //监听文本的变化的话那么咋们一般情况下就重写这个方法
         * @param s
         */
        @Override
        public void afterTextChanged(Editable s) {
           Log.e("------afterTextChanged------",s.toString());
        }
    }
```
