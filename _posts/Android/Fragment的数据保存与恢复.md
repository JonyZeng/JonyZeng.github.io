###前言：
>嗨！各位老铁，好久不见。最近工作有点忙，事情比较多，更新可能有点慢了哦。之前的Android回顾系列已经完结了。有兴趣的可以去看看和点赞。

###需求：
> 最近工作上有业务需求，需要从Fragment A 跳转到Fragment B之后，退回到Fragment A，要恢复跳转之前的数据和View。所以，今天要给大家说一下的是Fragment的数据恢复与保存

### Android中数据的保存与恢复
首先提到数据恢复，首先想到的应该是Activity的数据恢复与保存。大家想的都是在onSaveInstanceState方法进行保存。但是，我们今天讲的Fragment的数据恢复与保存，需要手动的调用保存方法进行保存，避免保存失败。
##### 下面是具体的实现步骤：
1. 首先我们需要了解一下setArguments()和getArguments()方法的作用
> 在Fragment中，我们需要使用setArguments(bundle)来传递bundle数据。getArguments()方法获取数据。
```
    /**
     * 在初始化这个Fragment实例的时候，就需要传递这个bundle数据进去。注意：在调用显示这个Fragment之前就要设置这个函数。
     * @return
     */
    @NonNull
    public static NewsFragment newInstance() {
        if (audioFragment==null){  //采用double Check的方式创建单例
            synchronized (NewsFragment.class){
                if (audioFragment==null){
                    audioFragment = new NewsFragment();
                }
            }
        }
        Bundle bundle = new Bundle();
        audioFragment.setArguments(bundle);
        return audioFragment;
    }
```
2. 然后我们就要正式的进行临时数据的获取了。
- 在OnActivityCreated()生命周期方法中判断是否有保存临时数据
```
    @Override
    public void onActivityCreated(@Nullable Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        Log.i(TAG, "isVoce " + isVoiceLaunch());
        if (!reStoreStateFromArguments()) {  //判断是否有保存过临时数据
            Log.i(TAG, "onActivityCreated:");
        }
    }
```
> 在OnActivityCreated中进行判断是因为回退的时候，Fragment会重走这个生命周期
```
    public boolean reStoreStateFromArguments() {
        Bundle bundle = getArguments();  //第一步、拿到之前保存的bundle数据
        saveState = bundle.getBundle("savedViewState");//第二步、通过bundle获取之前保存时候设置的数据bundle
        Log.i(TAG, "sacve " + saveState);
        if (saveState != null) {  //判断是是否有保存过数据 
            Log.i(TAG, "恢复数据");
            reStoreState();
            return true;
        }
        return false;
    }
```
> 在reStoreState()方法里面取出之前存储的数据并绑定到对应的View上面
```
    public void reStoreState() {
        int saveCurrentProgress = saveState.getInt("currentProgress", 0);
        int saveDuration = saveState.getInt("duration", 0);
        String audio_album = saveState.getString("audio_album", null);
        String title = saveState.getString("title", null);
        Glide.with(MateApplication.getInstance()).load(audio_album).error(R.mipmap.play_rdi_album_music).into(play_rdi_album);
        setFragmentTitle(title);
        audioSeek.setProgress(saveCurrentProgress);
        audioSeek.setMax(saveDuration);
        //获取到当前播放的时间
        String time = UIUtil.formatTime(saveCurrentProgress);
        audioCurrentProgress.setText(time);
        String formatTime = UIUtil.formatTime(saveDuration);
        audioDuration.setText(formatTime);
        if (isPlaying) {
            play();
        }
        Log.i(TAG, "生命周期 isPlaying " + AppConstants.isPlaying);
    }
```
3. 最后，我们在Fragment的onSaveInstanceState和onDestroyView()方法中手动的去调用保存数据的方法。
```
    @Override
    public void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);
        saveStateToArguments();
    }
    @Override
    public void onDestroyView() {
        super.onDestroyView();
        Log.i(TAG, "onDestroyView: ");
        saveStateToArguments();
    }
```
> 在onDestroyView中手动调用，是为了防止有时候onSaveInstanceState()方法在Fragment销毁的时候不被调用。
```
    public void saveStateToArguments() {
        Log.i(TAG, "保存数据");
        saveState = saveState();//保存数据
        if (saveState != null) {
            Bundle bundle = getArguments(); // 获取之前初始化时候的bundle
            bundle.putBundle("savedViewState", saveState);//将bundle传入数据
        }
    }
```
> 将需要保存的数据，封装到bundle中进行传递
```
    public Bundle saveState() {
        Bundle outState = new Bundle();
        if (dbdataBeanList != null && dbdataBeanList.size() > 0 && dbdataBeanList.get(getCurrent()) != null) {
            DataBean.DbdataBean audioBean = dbdataBeanList.get(getCurrent());
            outState.putString("title", audioBean.getTrack());
            outState.putString("audio_album", audioBean.getCover_url_large());
            outState.putInt("currentProgress", currentProgress);
            outState.putInt("duration", duration);
            Log.i(TAG, "保存数据");
        }
        return outState;
    }
```
### End
&emsp;&emsp;关于Android中临时数据的保存与恢复，平时在业务中遇见的场景还是挺多的，我们需要好好的理解一下。欢迎大家点赞和指正哦。



