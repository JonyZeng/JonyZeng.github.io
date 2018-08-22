又到了程序猿日常踩坑的时间了，前段时间，做了一个定制化WiFi相关的功能，最近几天空闲，把踩坑的心路历程分享给大家。
# 需求分析
对于很多应用来说，都不愿意跳转到系统的设置去配置WiFi的信息，那样容易造成APP的整体风格出现混乱。所以，我们就需要自己定制化WiFi相关的模块功能。
# 具体实现
### 功能逻辑流程图。
![WiFi流程逻辑 .png](http://upload-images.jianshu.io/upload_images/7156039-278cc4ad6039eecd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 功能实现
1. 开始的开始，我们还是孩子（咳咳。。。。）严肃一点，我们要添加获取WiFi和网络的权限
```
 <uses-permission android:name="android.permission.CHANGE_NETWORK_STATE"/> <!-- 允许程序改变网络链接状态 -->
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/> <!-- 允许程序访问访问WIFI网络状态信息 -->
    <uses-permission android:name="android.permission.CHANGE_WIFI_STATE"/> <!-- 允许程序改变WIFI链接状态 -->
    <!-- 6.0系统需要添加权限才能获得wifi列表 -->
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
    <uses-permission android:name="android.permission.INTERNET"/>
```
2. 我们需要在界面上设置一个switchButton来根据系统的WiFi状态进行区别
```
 mSwitchButton.setChecked(mWifiManager.isWifiEnabled()); //获取当前系统的WiFi状态,设置btn状态
        mSwitchButton.setOnCheckedChangeListener((view, isChecked) -> {
            mWifiManager.setWifiEnabled(isChecked);
            //更新列表
            if (isChecked) {
                Toast.makeText(WifiActivity.this, "打开", Toast.LENGTH_SHORT).show();
                //第二次点击的时候，清除之前的list
                isRefresh = true;
                presenter.subscribe(isRefresh);
            } else {
                Toast.makeText(WifiActivity.this, "关闭", Toast.LENGTH_SHORT).show();
                listView.setVisibility(View.GONE);
            }
        });
          //这里是根据适配器进行适配数据
        mListAdapter = new WifiListAdapter(this, mScanResults);
        listView.setAdapter(mListAdapter);
```
3. 采用MVP的模式对WiFi获取的数据进行加载。别问我为什么，任性。刚学，就是想用。
3.1 View层，在View层调用Presenter层的接口，实现界面的刷新
```
   /**
     * 将数据显示在View上
     *
     * @param list
     */
    @Override
    public void displayWifiList(List<ScanResult> list) {
        listView.setVisibility(View.VISIBLE);
        mListAdapter.addAll(list);
        Log.i(TAG, "displayWifiList: " + list.toString());
    }
```
3.2 model层，通过扫描周边WiFi信号，获取WiFi信号列表。
```
    public void search() {

        new Thread(() -> {
            //注册接收WIFI扫描结果的监听类对象
            mContext.registerReceiver(mWifiReceiver, new IntentFilter(WifiManager.SCAN_RESULTS_AVAILABLE_ACTION));

            //开始扫描
            mWifiManager.startScan();

            mLock.lock();

            //阻塞等待扫描结果
            try {
                mIsWifiScanCompleted = false;
                mCondition.await(WIFI_SEARCH_TIMEOUT, TimeUnit.SECONDS);
                if (!mIsWifiScanCompleted) {
                    mSearchWifiListener.onSearchWifiFailed(ErrorType.SEARCH_WIFI_TIMEOUT);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            mLock.unlock();

            //删除注册的监听类对象
            mContext.unregisterReceiver(mWifiReceiver);
        }).start();
    }
```
3.3 好了，到了MVP最为重要的一层，Presenter层了。这一层就是一个中间枢纽的作用，获取model层传递过来的数据，调用View层刷新界面的接口方法，就可以实现啦。
```
   public void search() {

        new Thread(() -> {
            //注册接收WIFI扫描结果的监听类对象
            mContext.registerReceiver(mWifiReceiver, new IntentFilter(WifiManager.SCAN_RESULTS_AVAILABLE_ACTION));

            //开始扫描
            mWifiManager.startScan();

            mLock.lock();

            //阻塞等待扫描结果
            try {
                mIsWifiScanCompleted = false;
                mCondition.await(WIFI_SEARCH_TIMEOUT, TimeUnit.SECONDS);
                if (!mIsWifiScanCompleted) {
                    mSearchWifiListener.onSearchWifiFailed(ErrorType.SEARCH_WIFI_TIMEOUT);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            mLock.unlock();

            //删除注册的监听类对象
            mContext.unregisterReceiver(mWifiReceiver);
        }).start();
    }
```
4.走到这里，我们其实已经完成了一半的工作了。接下来就是，我们去连接指定的WiFi和断开连接这些细小的逻辑。
4.1 自动连接已经配置好的WiFi，判断WiFi列表中是否有已经配置过得WiFi
```
   //根据ssid判断该wifi是否连接
    public boolean isCurrentConnect(String ssID) {
        ssID = "\"" + ssID + "\"";
        WifiInfo info = getWifiInfo();
        Log.i(TAG, "isCurrentConnect: "+info.getBSSID());
        //当前的ssid为空，未连接wifi
        if (TextUtils.isEmpty(info.getSSID())) {
            return false;
        } else {//已连接
            String tempSSID = info.getSSID();
            return tempSSID.equals(ssID);
        }
    }
```
然后在适配器中，根据返回的值进行匹配，显示已连接
```
  if(wifiItemId != -1){
            if(mWifiSearch.isCurrentConnect(scanResult.SSID)){
                holder.wifiName.setTextColor(mContext.getResources().getColor(R.color.colorBlack));
                holder.wifiStatus.setText("(已连接)");
            }else{
                holder.wifiName.setTextColor(mContext.getResources().getColor(R.color.colorWhite));
                holder.wifiStatus.setText("(已保存)");
            }
        }else{
            holder.wifiName.setTextColor(mContext.getResources().getColor(R.color.colorWhite));
            holder.wifiStatus.setText("");
        }
```
4.2 当我们点击WiFi列表选择连接WiFi的时候，进行搜索判断
```
 /**
     * WiFi的点击事件
     */
    private void settingWifi() {
        listView.setOnItemClickListener((parent, view, position, id) -> {
            //1、点击是已经连接的提示 已连接
            //2、已保存的直接连接
            //3、未保存的 输入密码连接
            final ScanResult scanResult = mListAdapter.getItem(position);
            int wifiItemId = -1;

            if (!TextUtils.isEmpty(scanResult.SSID)) {
                wifiItemId = mWifiSearch.IsConfiguration("\"" + scanResult.SSID + "\"");
            }
            if (wifiItemId != -1) {
                if (mWifiSearch.isCurrentConnect(scanResult.SSID)) {
                    Toast.makeText(mContext, "该wifi已连接！", Toast.LENGTH_SHORT).show();
                    showDisconnDialog(scanResult.SSID);
                } else {
                    //直接连接
                    //  connectWifi(scanResult,);
                    mWifiSearch.ConnectWifi(wifiItemId);
                    Log.i(TAG, "settingWifi: ");
                }
            } else {
                isAuto=false;
                //密码连接
                WifiPswDialog wifiPswDialog = new WifiPswDialog(mContext, new WifiPswDialog.OnCustomDialogListener() {
                    @Override
                    public void back(String str) {
                        showDialog();
                        connectWifi(scanResult, str);
                        isAuto=true;
                    }

                    @Override
                    public void cancel() {
                        isAuto=true;
                    }
                }, scanResult.SSID);
                wifiPswDialog.show();

            }
            Log.i(TAG, "onItemClick: " + scanResult.SSID + "BSSID" + scanResult.BSSID);
        });
    }
```
好了。大体的功能就是这样的，具体代码见git















