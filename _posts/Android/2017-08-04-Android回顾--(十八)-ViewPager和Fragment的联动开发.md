### ViewPager：
是Android3.0以后出来的一个控件，支持页面的左右滑动。是一个容器，需要通过适配器来完成相应的操作。
### ViewPager的事件监听：
```
 viewPager.setOnPageChangeListener(new ViewPager.OnPageChangeListener() {
            @Override
            public void onPageScrolled(int i, float v, int i1) {
                //滚动完成之后调用
            }

            @Override
            public void onPageSelected(int i) {
                //页面改变调用
            }

            @Override
            public void onPageScrollStateChanged(int i) {
                //页面状态改变之后调用
            }
        });
```
滚动控件的适配器：PagerAdapter、FragmentPagerAdapter(是特制给Fragment的适配器，v4包提供，Fragment必须也是v4包才能匹配)
自定义的PagerAdapter如下：
```
public class MyPagerAdapter extends PagerAdapter {
         List<View> views = new ArrayList<>();

        public MyPagerAdapter(List<View> views) {
            this.views = views;
        }

        @Override
        public int getCount() {
            return views.size();
        }

        @Override
        public boolean isViewFromObject(@NonNull View view, @NonNull Object o) {
            return false;
        }

        @NonNull
        @Override
        public Object instantiateItem(@NonNull ViewGroup container, int position) {
            View v = views.get(position);
            if (v.getParent() != null) {
                ViewGroup vg = (ViewGroup) v.getParent();
                vg.removeView(v);
            }
            container.removeView(v);
            container.addView(v);
            return v;
        }

        @Override
        public void destroyItem(@NonNull ViewGroup container, int position, @NonNull Object object) {
            super.destroyItem(container, position, object);
            container.removeView(views.get(position));
        }
    }

```
ViewPager实现图片广告的自动和手动切换
1. 初始化数据  

```
public void initData(){
        for (int i = 0; i < titles.length; i++) {
            ImageView img = new ImageView(this);
            img.setBackgroundResource(imgs[i]); //不设置src而是设置background，为了让图片充满
            img.setLayoutParams(new LinearLayout.LayoutParams(LinearLayout.LayoutParams.MATCH_PARENT,LinearLayout.LayoutParams.MATCH_PARENT));
            listImg.add(img);   //将需要显示的view装进容器里面
            View view = new View(this);     //初始化指示器的小白点的view
            view.setBackgroundResource(R.drawable.dot_normal);  //设置小白点的背景
            LinearLayout.LayoutParams params = new LinearLayout.LayoutParams(25, 25);
            params.setMargins(20,25,0,0);   //设置View外边距
            view.setLayoutParams(params);
            selectedContainer.addView(view);
            points.add(view);
        }
        mTextView.setText(titles[0]);
        points.get(0).setBackgroundResource(R.drawable.dot_enable);
    }
```  

2. 设置适配器,一般这种简单的自定义适配器，使用内部类完成  

```
private void setAdapter() {
        MyPagerAdapter myPagerAdapter = new MyPagerAdapter(listImg);
        mViewPager.setAdapter(myPagerAdapter);
    }

    public class MyPagerAdapter extends PagerAdapter {
        List<View> views = new ArrayList<>();

        public MyPagerAdapter(List<View> views) {
            this.views = views;
        }

        @Override
        public int getCount() {
            return views.size();
        }

        @Override
        public boolean isViewFromObject(@NonNull View view, @NonNull Object o) {
            return view == o;
        }

        @NonNull
        @Override
        public Object instantiateItem(@NonNull ViewGroup container, int position) {
            View v = views.get(position % views.size());
            if (v.getParent() != null) {
                ViewGroup vg = (ViewGroup) v.getParent();
                vg.removeView(v);
            }
            container.removeView(v);
            container.addView(v);
            return v;
        }

        @Override
        public void destroyItem(@NonNull ViewGroup container, int position, @NonNull Object object) {
            super.destroyItem(container, position, object);
            container.removeView(views.get(position % views.size()));
        }
    }
```  

3. 设置页面改变的监听事件  

```
 mViewPager.addOnPageChangeListener(new ViewPager.OnPageChangeListener() {
            @Override
            public void onPageScrolled(int i, float v, int i1) {

            }

            @Override
            public void onPageSelected(int i) {
                mTextView.setText(titles[i / listImg.size()]);
                points.get(prePosition).setBackgroundResource(R.drawable.dot_normal);
                points.get(i / listImg.size()).setBackgroundResource(R.drawable.dot_enable);
                //记录的是改变之前的位置
                prePosition = i % listImg.size();
            }

            @Override
            public void onPageScrollStateChanged(int i) {

            }
        });
```  

4. 设置自动切换  

```
 public void enableAutoFling() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                while (isLooping){
                    try {
                        Thread.sleep(3000);
                        mHandler.sendEmptyMessage(100);  //通过handler传递消息到主线程
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }).start();
    }
Handler mHandler = new Handler(){
    @Override
    public void handleMessage(Message msg) {
        super.handleMessage(msg);
        switch (msg.what){
            case 100:
                //获取viewpager当前的位置
                int position = mViewPager.getCurrentItem();
                //向下一个页面进行滑动
                mViewPager.setCurrentItem(position+1);
                break;
        }
    }
};
```  

### ViewPager+Fragment的联动使用
1. 初始化数据  

```
 public void  initData(){
        for (int i = 0; i < 4; i++) {
            ContentFragment mContentFragment = new ContentFragment();
            Bundle bundle = new Bundle();
            bundle.putString("content","页面 "+i);
            mContentFragment.setArguments(bundle);   // 将数据传递给Fragment
            fragments.add(mContentFragment);
        }
    }
```  

在Fragment中的onActivityCreated方法中接收传递过来的数据  

```
String result = getArguments().getString("content");
mTextView.setText(result);
```  

2. 设置适配器  

```
 private void setAdapter() {
        MyFragmentPagerAdapter myFragmentPagerAdapter = new MyFragmentPagerAdapter(getSupportFragmentManager());
        mViewPager.setAdapter(myFragmentPagerAdapter);
    }

    /**
     * 自定义适配器，适配ViewPage和Fragment.这种简单的适配器，一般都定义内部类，这样可以共享全局资源。
     */
    public class MyFragmentPagerAdapter extends FragmentPagerAdapter {

        MyFragmentPagerAdapter(FragmentManager fm) {
            super(fm);
        }

        @Override
        public Fragment getItem(int i) {
            return fragments.get(i);
        }

        @Override
        public int getCount() {
            return fragments.size();
        }
    }
```  

好了，ViewPage和Fragment的简单联动就回顾到这里了，下一篇我会写一个实际的demo来讲解稍复杂一点的情况。喜欢的朋友可以先点个赞哦。收藏一下。
