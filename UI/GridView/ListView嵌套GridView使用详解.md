## ListView嵌套GridView使用详解

MainActivity如下:
```java
package cn.testlistviewandgridview;
import java.util.ArrayList;
import java.util.HashMap;
import android.app.Activity;
import android.os.Bundle;
import android.widget.ListView;
/**
 * Demo描述:
 * ListView嵌套GridView使用详解
 * 即ListView的每个Item中都包含一个GridView
 * 
 * 注意事项:
 * 由于ListView和GridView都是可滑动的控件.
 * 所以需要自定义GridView,重写其onMeasure()方法.
 * 在该方法中使GridView的高为wrap_content的大小,否则GridView中
 * 的内容只能显示很小一部分
 * 
 * 参考资料:
 * 1 http://bbs.csdn.net/topics/380245627
 * 2 http://blog.csdn.net/lsong89/article/details/8598856
 *   Thank you very much
 */
public class MainActivity extends Activity {
    private ListView mListView;
    private ListViewAdapter mListViewAdapter;
    private ArrayList<ArrayList<HashMap<String,Object>>> mArrayList;
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.main);
		init();
	}
   private void init(){
	   mListView=(ListView) findViewById(R.id.listView);
	   initData();
	   mListViewAdapter=new ListViewAdapter(mArrayList, MainActivity.this);
	   mListView.setAdapter(mListViewAdapter);
   }
   private void initData(){
	   mArrayList=new ArrayList<ArrayList<HashMap<String,Object>>>();
	   HashMap<String, Object> hashMap=null;
	   ArrayList<HashMap<String,Object>> arrayListForEveryGridView;
	  
	   for (int i = 0; i < 10; i++) {
		   arrayListForEveryGridView=new ArrayList<HashMap<String,Object>>();
		   for (int j = 0; j < 5; j++) {
			   hashMap=new HashMap<String, Object>();
			   hashMap.put("content", "i="+i+" ,j="+j);
			   arrayListForEveryGridView.add(hashMap);
		}
		mArrayList.add(arrayListForEveryGridView);
	 }
	  
   }

}

```
ListViewAdapter如下:
```java
package cn.testlistviewandgridview;
import java.util.ArrayList;
import java.util.HashMap;
import android.content.Context;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.BaseAdapter;
import android.widget.GridView;
import android.widget.ImageView;

public class ListViewAdapter extends BaseAdapter {
    private ArrayList<ArrayList<HashMap<String, Object>>> mList;
    private Context mContext;
	
	public ListViewAdapter(ArrayList<ArrayList<HashMap<String, Object>>> mList, Context mContext) {
		super();
		this.mList = mList;
		this.mContext = mContext;
	}

	@Override
	public int getCount() {
		if (mList == null) {
			return 0;
		} else {
			return this.mList.size();
		}
	}

	@Override
	public Object getItem(int position) {
		if (mList == null) {
			return null;
		} else {
			return this.mList.get(position);
		}
	}

	@Override
	public long getItemId(int position) {
		return position;
	}

	@Override
	public View getView(int position, View convertView, ViewGroup parent) {
		ViewHolder holder = null;
		if (convertView == null) {
			holder = new ViewHolder();			
			convertView = LayoutInflater.from
			(this.mContext).inflate(R.layout.listview_item, null, false);		
			holder.imageView = (ImageView) convertView.findViewById(R.id.listview_item_imageview);
			holder.gridView = (GridView) convertView.findViewById(R.id.listview_item_gridview);
			convertView.setTag(holder);
		} else {
			holder = (ViewHolder) convertView.getTag();
		}
		
		
		if (this.mList != null) {
			if (holder.imageView != null) {
				holder.imageView.setImageDrawable
				(mContext.getResources().getDrawable(R.drawable.e));
			}
			if (holder.gridView != null) {
				ArrayList<HashMap<String, Object>> arrayListForEveryGridView = this.mList.get(position);
				GridViewAdapter gridViewAdapter=new GridViewAdapter(mContext, arrayListForEveryGridView);
				holder.gridView.setAdapter(gridViewAdapter);
			}

		}

		return convertView;

	}

	
	private class ViewHolder {
		ImageView imageView;
		GridView gridView;
	}

}

```
GridViewAdapter如下:
```java
package cn.testlistviewandgridview;
import java.util.ArrayList;
import java.util.HashMap;
import android.content.Context;
import android.view.LayoutInflater;
import android.view.View;
import android.view.View.OnClickListener;
import android.view.ViewGroup;
import android.widget.BaseAdapter;
import android.widget.Button;
import android.widget.Toast;

public class GridViewAdapter extends BaseAdapter{
	private Context mContext;
	private ArrayList<HashMap<String, Object>> mList;
	
	public GridViewAdapter(Context mContext,ArrayList<HashMap<String, Object>> mList) {
		super();
		this.mContext = mContext;
		this.mList = mList;
	}

	@Override
	public int getCount() {
		if (mList == null) {
			return 0;
		} else {
			return this.mList.size();
		}
	}

	@Override
	public Object getItem(int position) {
		if (mList == null) {
			return null;
		} else {
			return this.mList.get(position);
		}
	}

	@Override
	public long getItemId(int position) {
		return position;
	}
	@Override
	public View getView(final int position, View convertView, ViewGroup parent) {
		ViewHolder holder = null;
		if (convertView == null) {
			holder = new ViewHolder();			
			convertView = LayoutInflater.from
			(this.mContext).inflate(R.layout.gridview_item, null, false);		
			holder.button = (Button)convertView.findViewById(R.id.gridview_item_button);
			convertView.setTag(holder);
		
		} else {
			holder = (ViewHolder) convertView.getTag();
		}
		
		
		if (this.mList != null) {
			HashMap<String, Object> hashMap = this.mList.get(position);
			if (holder.button != null) {
				holder.button.setText(hashMap.get("content").toString());
				holder.button.setOnClickListener(new OnClickListener() {
					@Override
					public void onClick(View v) {
						Toast.makeText(mContext, "第"+(position+1)+"个", Toast.LENGTH_SHORT).show();
					}
				});
			}
		}

		return convertView;

	}

	
	private class ViewHolder {
		Button button;
	}

}

```
NoScrollGridView如下:
```java
package cn.testlistviewandgridview;
import android.content.Context;
import android.util.AttributeSet;
import android.widget.GridView;

public class NoScrollGridView extends GridView {

	public NoScrollGridView(Context context) {
		super(context);
		
	}

	public NoScrollGridView(Context context, AttributeSet attrs) {
		super(context, attrs);
	}
	
	@Override
	protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
		int expandSpec = MeasureSpec.makeMeasureSpec(Integer.MAX_VALUE >> 2,MeasureSpec.AT_MOST);
		super.onMeasure(widthMeasureSpec, expandSpec);
	}

}


```
main.xml如下:
```xml
<RelativeLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    >

    <ListView
        android:id="@+id/listView"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        android:focusable="false"
    />

</RelativeLayout>
```
listview_item.xml如下:
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="vertical"
   
     >
    
    <ImageView
        android:id="@+id/listview_item_imageview"
        android:layout_width="fill_parent"
        android:layout_height="20dip"
        android:scaleType="fitXY"
        android:src="@drawable/e"
    />
    <cn.testlistviewandgridview.NoScrollGridView
        android:id="@+id/listview_item_gridview"
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:stretchMode="columnWidth"
        android:verticalSpacing="5dip"
        android:horizontalSpacing="5dip"
        android:numColumns="2"/>
</LinearLayout>
```
gridview_item.xml如下:
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal"
    android:padding="10dip"
    >
    <Button
        android:id="@+id/gridview_item_button"
        android:layout_width="140dip"
        android:layout_height="40dip"
        android:background="@drawable/e"
        android:textColor="@android:color/background_light"
        android:clickable="true"
     />

</LinearLayout>
```

> 参考：http://blog.csdn.net/lfdfhl/article/details/9156241?utm_source=tuicool&utm_medium=referral
