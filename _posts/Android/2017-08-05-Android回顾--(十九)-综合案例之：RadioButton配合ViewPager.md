上一篇给大家简单回顾了一下ViewPage和Fragment的使用，这里用一个综合的案例来给大家讲解一下目前常用的模式。

### 案例一：RadioGroupAndViewPager
1. 首先在MainActivity.java的onCrate方法中找到对应的控件，同时初始化值  
```
    List<Fragment> fragments = new ArrayList<>();   //fragment的初始化
    RadioGroup mRadioGroup;
    ViewPager mViewPager;
    RadioButton mRadiBtnHome;
    RadioButton mRadiBtnInfor;
    RadioButton mRadiBtnContacts;
    RadioButton mRadiBtnDynamic;
    String[] titles = {"首页", "消息", "动态", "联系人"};
    List<RadioButton> radioBtns = new ArrayList<>();    //把咋们的那个RadioButton给放进来
    Fragment mPreFragment;

    /**
     * 通过控件id找到控件，并将radiobtn添加进radiogroup
     */
    private void initView() {
        mRadioGroup = findViewById(R.id.mRadioGroup);
        mViewPager = findViewById(R.id.mViewPager);
        mRadiBtnHome = findViewById(R.id.mRadioBtnHome);
        mRadiBtnInfor = findViewById(R.id.mRadioBtnInfor);
        mRadiBtnContacts = findViewById(R.id.mRadioBtnContacts);
        mRadiBtnDynamic = findViewById(R.id.mRadioBtnDynamic);
        //按照顺序放进
        mRadioGroup.addView(mRadiBtnHome);
        mRadioGroup.addView(mRadiBtnInfor);
        mRadioGroup.addView(mRadiBtnContacts);
        mRadioGroup.addView(mRadiBtnDynamic);
    }
```  

2. 进行数据的初始化，案例业务需求比较简单，重复使用Fragment  
```
 private void initData() {
        for (int i = 0; i < 4; i++) {   //Fragment内容单一，可以复用才这样写。一般业务中，不会有这么单一的业务需求，所以需要创建多个Fragment
            ContentFragment contentFragment = new ContentFragment();
            Bundle bundle = new Bundle();
            bundle.putString("content", titles[i]);
            contentFragment.setArguments(bundle);
            fragments.add(contentFragment);
        }
    }
```
3. 在MainActivity.java中初始化数据之后，我们在ContenFragment中的onActivityCreated方法中获取前面传递的数据
```
@Override
    public void onActivityCreated(@Nullable Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        mTextView = view.findViewById(R.id.mTextView);
        content = getArguments().getString("content");
        mTextView.setText(content);
    }
```  
4. 设置默认显示的Fragment和默认选中的radioButton
```
 mViewPager.setCurrentItem(1);
 mRadiBtnInfor.setChecked(true);
```
5. 设置RadioGroup的点击事件
```
    /**
     * RadioGroup的点击事件，显示的Fragment随着button改变
     */
    private class OnCheckedListener implements RadioGroup.OnCheckedChangeListener {
        @Override
        public void onCheckedChanged(RadioGroup group, int checkedId) {
            switch (group.getCheckedRadioButtonId()){
                case R.id.mRadioBtnHome:
                    mViewPager.setCurrentItem(0);  // 设置当前viewpager显示的位置
                    break;
                case R.id.mRadioBtnInfor:
                    mViewPager.setCurrentItem(1);
                    break;
                case R.id.mRadioBtnDynamic:
                    mViewPager.setCurrentItem(2);
                    break;
                case R.id.mRadioBtnContacts:
                    mViewPager.setCurrentItem(3);
                    break;
            }
        }
    }
```
6. 设置ViewPager页面改变的监听
```
 private class OnPageListener implements ViewPager.OnPageChangeListener {
        @Override
        public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {

        }
        //就是我们在改变页面的时候，我们对radiobutton进行改变.
        public void onPageSelected(int position) {//当页面被选中的时候
            Log.e("------------------","发生改变的顺序是:"+position);
            radioBtns.get(position).setChecked(true);
        }

        @Override
        public void onPageScrollStateChanged(int state) {

        }
    }
```

### 案例二：FragmentAndTabHost
1. 布局文件如下：
```
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">
    <!-- 切换卡和盛装那个内容的容器必须由哪个FragmentTabHost来进行包裹  否则会出现问题 -->
    <android.support.v4.app.FragmentTabHost
        android:id="@+id/mFragmentTabHost"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="#ccc">
        <!--切换卡  id只能写成这样子  @android:id/tabs这种写法是Android内置的id的写法  所以后面的tabs也不能改变 -->
        <TabWidget
            android:id="@android:id/tabs"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />
        <!--表示的是盛装那个内容的地方  id也是系统给定的  不能改...-->
        <FrameLayout
            android:id="@android:id/tabcontent"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:gravity="center"
            android:orientation="vertical" />
    </android.support.v4.app.FragmentTabHost>
</android.support.constraint.ConstraintLayout>
```
2. 具体的实现步骤
```
   private void initFragmentTabHost() {
        //初始化容器
        mFragmentTabHost.setup(this, getSupportFragmentManager(), android.R.id.tabcontent);
        //初始化选项卡
        for (int i = 0; i < titles.length; i++) {
            // newTabSpec后面的内容是给当前的切换卡一个tag，setIndicator才是设置那个切换卡内容的地方
            TabHost.TabSpec tabSpec = mFragmentTabHost.newTabSpec(titles[i] + i).setIndicator(titles[i]);
            Bundle bundle = new Bundle();
            bundle.putString("content", titles[i]);
            /**
             * 第一个参数:切换卡
             * 第二个参数:内容的fragment的class文件
             * 第三个参数:就是那个Fragment使用的时候需要传递进去的参数
             *
             */
            mFragmentTabHost.addTab(tabSpec,ContentFragment.class,bundle);
        }
```

### 案例三：TabLayoutAndViewPagerAndFragment
1. 首先需要在gradle里面填加design包，才能使用TabLayout。
```
    //导入design包
    implementation 'com.android.support:design:27.1.0'
```
2. 在布局文件中进行控件布局
```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <!--
    TabLayout的一些属性：
    app:tabIndicatorColor="#07f"    //设置指示器颜色
    app:tabMode="scrollable"        //表示此TabLayout中当子view超出屏幕边界时候，将提供滑动以便滑出不可见的那些子view。
    app:tabSelectedTextColor="#f07" //设置滑动块被选中的颜色
    app:tabTextColor="#699"            //设置滑动块字体颜色
    -->
    <android.support.design.widget.TabLayout
        android:id="@+id/mTabLayout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:tabIndicatorColor="#07f"
        app:tabMode="scrollable"
        app:tabSelectedTextColor="#f07"
        app:tabTextColor="#699" />

    <android.support.v4.view.ViewPager
        android:id="@+id/mViewPager"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_below="@+id/mTabLayout" />
</RelativeLayout>

```
3. 在mainActivity中实现TabLayout和ViewPager的联动。其他和上面案例相同的数据初始化就不上代码了。下面上一个联动的关键方法
```
 mTabLayout.setupWithViewPager(mViewPager);//这里一句话就把他们建立连接了
```
4. 对adapter中需要手动重写一个方法，用于获取需要显示在标签上面的数据
```
        /*
         * 用来返回那个页面当前的标题是什么？方便那个Tablayout去使用
         * */
        @Override
        public CharSequence getPageTitle(int position) {//这个方法不会自动实现，要我们自己去写出来
            return titles[position];//得到是标签的内容，就是让我们的标题显示内容
        }
```
好了，关于ViewPager和Fragment一些联动的两个案例就讲到这里了，可能讲得不是很仔细，我会将案例的代码上传到GitHub上面，有兴趣的同学可以去看看哦。欢迎点赞和提出不足之处。

### 项目地址：https://github.com/JonyZeng/ViewPageFgDemo












