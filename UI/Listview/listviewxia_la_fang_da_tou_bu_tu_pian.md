# ListView下拉放大头部图片

> [ 安卓ListView下拉放大头部图片](http://blog.csdn.net/qq_28508571/article/details/51313770)
> 
> [可以下拉缩放HeaderView的ListView:PullToZoomInListView](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2014/0627/1625.html)
> 
> [Github PullToZoomInListView](https://github.com/matrixxun/PullToZoomInListView/blob/master/src/com/matrixxun/pulltozoomlistsimple/PullToZoomListView.java)


自己实现的，头部有多个控件，且包含TabLayout，效果图：

![效果图](5D8C18AA85F29C9ED834C27F309F9C25.gif)

```java

/**
 * 自定义的ListView
 * 1.下拉放大顶部图片，整个头部高度跟着变化
 * 2.上拉加载更多
 */
public class ListView4Profile extends ListView implements AbsListView.OnScrollListener, View.OnTouchListener {
    private static final int ROOT_HEIGHT = 345;//根布局高度-dp
    private static final int IMG_HEIGHT = 180;//图片高度-dp
    private static final float PIX = 0.4f;//下拉系数
    private static final int DURATION = 200;//动画持续时间-毫秒

    private LayoutInflater inflater;
    private DisplayMetrics displayMetrics;
    private RelativeLayout headerRoot;
    public ImageView profileBg;
    public RoundedImageView profileAvatar;
    public ImageView profileVip;
    public TextView profileNickName;
    public TextView profileAddress;
    public TextView profileEdit;
    public TabLayout profileTabs;
    //下拉放大
    private float mFirstPosition = 0;// 记录首次按下位置
    private Boolean mScaling = false;// 是否正在放大
    private int width;
    //Footer
    private View footer;
    //Load
    public int totalItemCount;
    public int lastVisibleItem;
    public boolean isLoading;
    public ILoadListener iLoadListener;

    public ListView4Profile(Context context) {
        super(context);
        initHeader(context);
        initFooter();
    }

    public ListView4Profile(Context context, AttributeSet attrs) {
        super(context, attrs);
        initHeader(context);
        initFooter();
    }

    public ListView4Profile(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initHeader(context);
        initFooter();
    }

    //初始化头部
    private void initHeader(Context context) {
        this.inflater = LayoutInflater.from(context);
        View header = this.inflater.inflate(R.layout.common_profile_head, this, false);
        //init View
        headerRoot = (RelativeLayout) header.findViewById(R.id.profile_root);
        profileBg = (ImageView) header.findViewById(R.id.profile_bg);
        profileAvatar = (RoundedImageView) header.findViewById(R.id.profile_avatar);
        profileVip = (ImageView) header.findViewById(R.id.profile_vip);
        profileNickName = (TextView) header.findViewById(R.id.profile_nickName);
        profileAddress = (TextView) header.findViewById(R.id.profile_address);
        profileEdit = (TextView) header.findViewById(R.id.profile_edit);
        profileTabs = (TabLayout) header.findViewById(R.id.profile_tabs);

        this.addHeaderView(header);
        this.setHeaderDividersEnabled(false);
        zoom(context);
    }

    //顶部放大
    private void zoom(Context context) {
        this.setOnTouchListener(this);
        displayMetrics = new DisplayMetrics();
        ((Activity) context).getWindowManager()
                .getDefaultDisplay()
                .getMetrics(displayMetrics);

        width = displayMetrics.widthPixels;
    }

    // 回弹动画 (使用了属性动画)
    @SuppressLint("NewApi")
    public void replyImage() {
        final ViewGroup.LayoutParams lp = (ViewGroup.LayoutParams) profileBg.getLayoutParams();
        final float w = profileBg.getLayoutParams().width;// 图片当前宽度
        final float h = profileBg.getLayoutParams().height;// 图片当前高度
        final float newW = width;// 图片原宽度
        final float newH = IMG_HEIGHT * displayMetrics.density + 0.5f;// 图片原高度

        final ViewGroup.LayoutParams lp2 = (ViewGroup.LayoutParams) headerRoot.getLayoutParams();
        final float w2 = headerRoot.getLayoutParams().width;// 当前宽度
        final float h2 = headerRoot.getLayoutParams().height;// 当前高度
        final float newW2 = width;// 原宽度
        final float newH2 = ROOT_HEIGHT * displayMetrics.density + 0.5f;// 原高度

        // 设置动画
        ValueAnimator anim = ObjectAnimator.ofFloat(0.0F, 1.0F).setDuration(DURATION);

        anim.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                float cVal = (Float) animation.getAnimatedValue();
                lp.width = (int) (w - (w - newW) * cVal);
                lp.height = (int) (h - (h - newH) * cVal);
                profileBg.setLayoutParams(lp);

                float cVal2 = (Float) animation.getAnimatedValue();
                lp2.width = (int) (w2 - (w2 - newW2) * cVal2);
                lp2.height = (int) (h2 - (h2 - newH2) * cVal2);
                headerRoot.setLayoutParams(lp2);
            }
        });
        anim.start();

    }

    //初始化底部
    private void initFooter() {
        footer = this.inflater.inflate(R.layout.footer_listview_load_layout, this, false);
        footer.findViewById(R.id.load_layout).setVisibility(View.GONE);
        this.addFooterView(footer, this, false);
        this.setOnScrollListener(this);
    }

    public void loadComplete() {
        isLoading = false;
        footer.findViewById(R.id.load_layout).setVisibility(GONE);
    }

    @Override
    public void onScrollStateChanged(AbsListView view, int scrollState) {
        if (totalItemCount == lastVisibleItem
                && scrollState == SCROLL_STATE_IDLE) {
            if (!isLoading) {
                isLoading = true;
                footer.findViewById(R.id.load_layout).setVisibility(View.VISIBLE);
                iLoadListener.onLoad();
            }
        }
    }

    @Override
    public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount) {
        this.lastVisibleItem = firstVisibleItem + visibleItemCount;
        this.totalItemCount = totalItemCount;
    }

    @Override
    public boolean onTouch(View v, MotionEvent event) {
        ViewGroup.LayoutParams lp = (ViewGroup.LayoutParams) profileBg.getLayoutParams();
        ViewGroup.LayoutParams lp2 = (ViewGroup.LayoutParams) headerRoot.getLayoutParams();
        switch (event.getAction()) {
            case MotionEvent.ACTION_UP:
                //手指离开后恢复图片
                mScaling = false;
                replyImage();
                break;
            case MotionEvent.ACTION_MOVE:
                if (!mScaling) {
                    if (this.getScrollY() == 0) {
                        mFirstPosition = event.getY();// 滚动到顶部时记录位置，否则正常返回
                    } else {
                        break;
                    }
                }
                int distance = (int) ((event.getY() - mFirstPosition) * PIX); // 滚动距离乘以一个系数
                if (distance < 0) { // 当前位置比记录位置要小，正常返回
                    break;
                }

                // 处理放大
                mScaling = true;
                lp.width = width + distance;
                //                lp.height = (width + distance) * 9 / 16;
                lp.height = (int) (IMG_HEIGHT * displayMetrics.density + distance);
                profileBg.setLayoutParams(lp);

                lp2.width = width;
                lp2.height = (int) (ROOT_HEIGHT * displayMetrics.density + distance);
                headerRoot.setLayoutParams(lp2);
                //return true;// 返回true表示已经完成触摸事件，不再处理
                return super.onTouchEvent(event);//如果直接return ture,会导致上滑到下面几个item后不能下滑
        }
        return super.onTouchEvent(event);
    }

    public void setInterface(ILoadListener iLoadListener) {
        this.iLoadListener = iLoadListener;
    }

    public interface ILoadListener {
        void onLoad();
    }
}

```

`headView`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    android:id="@+id/profile_root"
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="345dp"
    android:background="@color/white"
    android:descendantFocusability="blocksDescendants"
    >

    <ImageView
        android:id="@+id/profile_bg"
        android:layout_width="match_parent"
        android:layout_height="180dp"
        android:contentDescription="@string/app_name"
        android:scaleType="centerCrop"
        android:src="@drawable/profile_bg1"
        tools:src="@drawable/profile_bg1"
        />

    <com.github.siyamed.shapeimageview.RoundedImageView
        android:id="@+id/profile_avatar"
        android:layout_width="90dp"
        android:layout_height="90dp"
        android:layout_below="@id/profile_bg"
        android:layout_centerHorizontal="true"
        android:layout_marginTop="-80dp"
        android:src="@drawable/load_avatar"
        app:siBorderColor="@color/white"
        app:siBorderWidth="3dp"
        app:siRadius="45dp"/>

    <ImageView
        android:id="@+id/profile_vip"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignBottom="@id/profile_avatar"
        android:layout_alignRight="@id/profile_avatar"
        android:src="@drawable/profile_vip"/>

    <TextView
        android:id="@+id/profile_nickName"
        android:layout_width="wrap_content"
        android:layout_height="28dp"
        android:layout_below="@id/profile_avatar"
        android:layout_centerHorizontal="true"
        android:drawablePadding="5dp"
        android:layout_marginTop="5dp"
        android:drawableRight="@drawable/icon_gender_f"
        android:paddingLeft="10dp"
        android:textColor="@color/c_333"
        android:textSize="20sp"
        tools:text="Nick Name"
        />

    <TextView
        android:id="@+id/profile_address"
        android:layout_width="wrap_content"
        android:layout_height="20dp"
        android:layout_below="@id/profile_nickName"
        android:layout_centerHorizontal="true"
        android:layout_marginTop="10dp"
        android:singleLine="true"
        android:textColor="@color/c_333"
        android:textSize="15sp"
        tools:text="所在地：未知星球"
        />

    <TextView
        android:id="@+id/profile_edit"
        android:layout_width="wrap_content"
        android:layout_height="30dp"
        android:layout_below="@id/profile_address"
        android:layout_centerHorizontal="true"
        android:layout_margin="10dp"
        android:background="@drawable/profile_edit_bg"
        android:gravity="center"
        android:paddingLeft="20dp"
        android:paddingRight="20dp"
        android:text="@string/profile_edit"
        android:textColor="@color/white"
        android:textSize="12sp"
        />

    <View
        android:id="@+id/profile_v1"
        android:layout_width="match_parent"
        android:layout_height="2px"
        android:layout_below="@id/profile_edit"
        android:background="@color/c_e8e8e8"
        />

    <android.support.design.widget.TabLayout
        android:id="@+id/profile_tabs"
        android:layout_width="wrap_content"
        android:layout_height="40dp"
        android:layout_below="@id/profile_v1"
        android:layout_centerHorizontal="true"
        app:tabGravity="center"
        app:tabIndicatorColor="@color/title_bar"
        app:tabMode="scrollable"
        app:tabPaddingEnd="20dp"
        app:tabPaddingStart="20dp"
        app:tabSelectedTextColor="@color/c_333"
        app:tabTextColor="@color/c_999"
        android:descendantFocusability="blocksDescendants"
        />

    <View
        android:id="@+id/profile_v2"
        android:layout_width="match_parent"
        android:layout_height="2px"
        android:layout_below="@id/profile_tabs"
        android:background="@color/c_e8e8e8"
        />
</RelativeLayout>
```

### 使用

```java
public class Main_ProfileFragment extends Fragment implements ListView4Profile.ILoadListener, Note_lv_adapter.PicClickListener {
    private static final int LOAD_NUM = 20;
    private int[] offsets;//页码，动态-群组-好友

    @Bind(R.id.profile_setting)
    ImageView profileSetting;
    @Bind(R.id.profile_lv)
    ListView4Profile profileLv;

    private OkHttpUtil okHttpUtil;
    private Picasso picasso;
    private Gson gson;
    private String mid;
    //动态
    private NoteTask noteTask;
    private Note_lv_adapter note_adapter;
    private List<DataEntity> NoteDatas;
    //简介
    private SimpleAdapter myInfoSA;
    //群组
    private GroupTask groupTask;
    private BiaoQian_content_adapter group_adapter;
    private List<Gson_biaoqian_content_bean.DataEntity> groupDatas;
    //好友
    private FriendTask friendTask;
    private Profile_my_friend_adapter friend_adapter;
    private List<UserDataEntity> userData;

    private PopupWindow popupWindow;
    private RelativeLayout root;
    private PhotoView photoView;
    private LinearLayout parent;

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        View v = inflater.inflate(R.layout.fragment_main__profile, container, false);
        ButterKnife.bind(this, v);
        init();
        initHead();
        initHeadData();
        initHeadEvent();
        return v;
    }

    //初始化参数和配置
    private void init() {
        picasso = Picasso.with(getActivity());
        NoteDatas = new ArrayList<>();
        groupDatas = new ArrayList<>();
        userData = new ArrayList<>();
        okHttpUtil = OkHttpUtil.getInstance();
        gson = new Gson();
        offsets = new int[]{0, 0, 0};
        initPopWindow();
    }

    private void initHead() {
        //Init data
        int bg = new Random().nextInt(5);
        switch (bg) {
            case 0:
                profileLv.profileBg.setImageResource(R.drawable.profile_bg1);
                break;
            case 1:
                profileLv.profileBg.setImageResource(R.drawable.profile_bg2);
                break;
            case 2:
                profileLv.profileBg.setImageResource(R.drawable.profile_bg3);
                break;
            case 3:
                profileLv.profileBg.setImageResource(R.drawable.profile_bg4);
                break;
            case 4:
                profileLv.profileBg.setImageResource(R.drawable.profile_bg5);
                break;
        }
        profileLv.profileTabs.addTab(profileLv.profileTabs.newTab().setText(getString(R.string.profile_dynamic)), 0);
        profileLv.profileTabs.addTab(profileLv.profileTabs.newTab().setText(getString(R.string.profile_group)), 1);
        profileLv.profileTabs.addTab(profileLv.profileTabs.newTab().setText(getString(R.string.profile_friend)), 2);
        TabLayout.Tab tab4 = profileLv.profileTabs.newTab().setText(getString(R.string.profile_info));
        profileLv.profileTabs.addTab(tab4, 3);
        tab4.select();
    }

    private void initHeadData() {
        ....
    }

    private void initHeadEvent() {
        profileLv.profileEdit.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Profile_detailActivity.startAction(getActivity());
                getActivity().overridePendingTransition(R.anim.in_from_right, R.anim.stand);
            }
        });
        profileLv.profileTabs.setOnTabSelectedListener(new TabLayout.OnTabSelectedListener() {
            @Override
            public void onTabSelected(TabLayout.Tab tab) {
                selectTab(tab.getPosition());
            }

            @Override
            public void onTabUnselected(TabLayout.Tab tab) {

            }

            @Override
            public void onTabReselected(TabLayout.Tab tab) {

            }
        });
        profileLv.setInterface(Main_ProfileFragment.this);
        selectTab(3);
    }

    private void initPopWindow() {
        ...
    }

    private void selectTab(int i) {
        profileLv.setAdapter(null);
        profileLv.loadComplete();
        switch (i) {
            case 0://动态
                requestNote();
                break;
            case 1://群组
                requestGroup();
                break;
            case 2://好友
                requestFriend();
                break;
            case 3://简介
                requestUser();
                break;
        }
        profileLv.invalidate();
    }

    @OnClick(R.id.profile_setting)
    void profile_setting() {
        SettingActivity.startAction(getActivity());
        getActivity().overridePendingTransition(R.anim.in_from_right, R.anim.stand);
    }

    //请求用户的日志动态
    private void requestNote() {
        ...
    }

    private void bindNote() {
      ...
    }

    //请求用户加入的群组
    private void requestGroup() {
        ...
    }

    private void bindGroup() {
        ...
    }

    //请求用户的好友
    private void requestFriend() {
        ...
    }

    private void bindFriend() {
        ...
    }

    //获取用户的个人信息
    private void requestUser() {
      ...
    }

    @Override
    public void onResume() {
        super.onResume();
        initHeadData();
    }

    @Override
    public void onDestroyView() {
        super.onDestroyView();
        ButterKnife.unbind(this);
    }

    @Override
    public void onLoad() {
        ...
    }

    @Override
    public void showImg(PhotoView v, String url) {
        ...
    }

    private class NoteTask extends AsyncTask<String, Void, Boolean> {
        @Override
        protected Boolean doInBackground(String... params) {
            ...
            return false;
        }

        @Override
        protected void onPostExecute(Boolean s) {
            super.onPostExecute(s);
            ...
        }
    }

    private class GroupTask extends AsyncTask<String, Void, Boolean> {
        @Override
        protected Boolean doInBackground(String... params) {
            ...
        }

        @Override
        protected void onPostExecute(Boolean aBoolean) {
            super.onPostExecute(aBoolean);
            ...
        }
    }

    private class FriendTask extends AsyncTask<String, Void, Boolean> {
        @Override
        protected Boolean doInBackground(String... params) {
            ...
        }

        @Override
        protected void onPostExecute(Boolean aBoolean) {
            super.onPostExecute(aBoolean);
            ...
        }
    }

    private class ChangeTask extends AsyncTask<Integer, Void, Integer> {
        @Override
        protected Integer doInBackground(Integer... params) {
           ...
        }

        @Override
        protected void onPostExecute(Integer integer) {
            super.onPostExecute(integer);
            ...
        }
    }

}

```

对应的界面：

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <custom.ListView4Profile
        android:id="@+id/profile_lv"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:divider="@color/space"
        android:dividerHeight="1dp"
        android:overScrollMode="never"
        android:scrollbars="none"
        />

    <ImageView
        android:id="@+id/profile_setting"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentEnd="true"
        android:layout_alignParentRight="true"
        android:layout_marginEnd="12dp"
        android:layout_marginRight="12dp"
        android:layout_marginTop="16dp"
        android:contentDescription="@string/app_name"
        android:src="@drawable/profile_setting"
        />
</RelativeLayout>

```


