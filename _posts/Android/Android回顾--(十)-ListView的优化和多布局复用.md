### LIstView常用的三方法
1. addHeaderView：这个方法的作用是在LIstView的顶部添加一个View
2. addFooterView：在ListView的底部添加一个View
3. setEmptyView：当ListView数据为空的时候，显示一个对应的View
```
LayoutInflater inflater = getLayoutInflater(); //把这个布局整成View
View headView=inflater.inflate(R.layout.list_item_headview,null);
View footView=inflater.inflate(R.layout.list_item_footview,null);
mListView.addHeaderView(headView);
mListView.addFooterView(footView);
TextView textView=new TextView(GoodListViewActivity.this);
textView.setText("没有数据");
textView.setLayoutParams(new LayoutParams(LayoutParams.MATCH_PARENT,
LayoutParams.MATCH_PARENT));
textView.setGravity(Gravity.CENTER);
mListView.setEmptyView(textView);
mListView.setAdapter(adapter);  //给ListView添加顶部或者底部信息一定是在 setAdapter之前
```
### ListView的优化
1. 复用convertView
```
@Override
public View getView(int position, View convertView, ViewGroup parent) {
	if(convertView==null){    //复用convertView
		convertView=inflater.inflate(R.layout.list_infor_item,null);
		//初始化那个viewHolder的对象
		viewHolder=new ViewHolder();
		//找控件
		viewHolder.img=(ImageView) convertView.findViewById(R.id.mImageView);
		viewHolder.lastInfor=(TextView) convertView.findViewById(R.id.mTextViewLastInfor);
		viewHolder.userName=(TextView) convertView.findViewById(R.id.mTextViewUserName);
		viewHolder.time=(TextView) convertView.findViewById(R.id.mTextViewTime);
				//给这个convertView设置一个tag相当于:是将ViewHolder暂时装进了convertView
		convertView.setTag(viewHolder);
	}else{
		//从convertView里面将viewHolder取出来
		viewHolder=(ViewHolder) convertView.getTag();   //减少了那个findViewById的次数
	}
	//img先丢这里.
	viewHolder.lastInfor.setText(lists.get(position).getLastInfor());
	viewHolder.userName.setText(lists.get(position).getUserName());
	viewHolder.time.setText(lists.get(position).getTime());
	new MyAsyncTask(viewHolder.img).execute(lists.get(position).getImg());
	return convertView;
		}
	}
```
2. 使用静态的ViewHolder来保存模板这样可以减少findViewById的次数。
```
	/**
	 * 静态的ViewHolder模板
	 * @author apple
	 *
	 */
	static class ViewHolder{
		ImageView img;
		TextView userName;
		TextView lastInfor;
		TextView time;
	}
```
### ListView的上拉加载：
1. 通过setOnScrollListener给ListView设置滚动事件
2. 自定义一个类实现OnScrollListener
![自定义OnScrollListener.png](https://upload-images.jianshu.io/upload_images/7156039-a1c638ee04fa8112.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![头部和底部.png](https://upload-images.jianshu.io/upload_images/7156039-e0e1ca5d25e912e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### ListView的多布局复用：
1. 初始化数据源
2. 编写适配器
    - 声明静态模板的对象
    - ListView在实现多布局复用时需要多重写两个方法：
      a. getViewTypeCount：获取ListView中布局的种类
      b. getItemViewType：获取每个布局的类型
3. getView中应该分情况给Item设置内容
### GridView网格视图
解析数据和适配器与ListView相同
GridView布局中特有的属性：
```
 android:numColumns="3"	//列数
 android:verticalSpacing="2dp"	//垂直间距
 android:horizontalSpacing="2dp"   //水平间距
```
