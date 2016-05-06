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
        UserDataEntity bean = DataHouse.getBean(getActivity());
        mid = bean.getMid() + "";
        String avatar = bean.getAvatar();
        String nickname = CharConvert.convert(bean.getNickname());
        int vip = bean.getVip();
        nickname = CharConvert.convert(nickname);
        String gender = bean.getGender();
        String address = CharConvert.convert(bean.getAddress());


        picasso.load(URLUtil.getInstance().cropImg(avatar, 200, 200))
                .config(Bitmap.Config.RGB_565)
                .into(profileLv.profileAvatar);
        if (vip == 1) {
            profileLv.profileVip.setVisibility(View.VISIBLE);
        } else profileLv.profileVip.setVisibility(View.GONE);

        profileLv.profileNickName.setText(nickname);
        if (gender.equals("m")) {
            Drawable drawable = getResources().getDrawable(R.drawable.icon_gender_m);
            drawable.setBounds(0, 0, drawable.getMinimumWidth(), drawable.getMinimumHeight()); //设置边界
            profileLv.profileNickName.setCompoundDrawables(null, null, drawable, null);//画在右边
        } else if (gender.equals("f")) {
            Drawable drawable = getResources().getDrawable(R.drawable.icon_gender_f);
            drawable.setBounds(0, 0, drawable.getMinimumWidth(), drawable.getMinimumHeight());
            profileLv.profileNickName.setCompoundDrawables(null, null, drawable, null);
        } else {
            profileLv.profileNickName.setCompoundDrawables(null, null, null, null);
        }
        if (address.trim().isEmpty() || address.equals("null"))
            profileLv.profileAddress.setText("所在地：未知星球");
        else profileLv.profileAddress.setText("所在地：" + address);
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
        View cv = getActivity().getLayoutInflater().inflate(R.layout.custom_show_img, null);//动态加载
        root = (RelativeLayout) cv.findViewById(R.id.img_root);
        photoView = (PhotoView) cv.findViewById(R.id.img_view);
        photoView.enable();

        popupWindow = new PopupWindow(cv,
                ViewGroup.LayoutParams.MATCH_PARENT,
                ViewGroup.LayoutParams.MATCH_PARENT);//全屏显示
        popupWindow.setFocusable(true);
        popupWindow.setAnimationStyle(R.style.popWindow);
        popupWindow.setClippingEnabled(false);
        parent = (LinearLayout) getActivity().findViewById(R.id.main_rl);//父窗口view
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
        if (NoteDatas.size() > 0) {
            bindNote();
        } else {
            String url = URLUtil.getInstance().getMyNotes(mid, LOAD_NUM, offsets[0]);
            Logger.d(url);
            if (noteTask == null || !noteTask.getStatus().equals(AsyncTask.Status.RUNNING)) {
                noteTask = new NoteTask();
                Logger.d(noteTask.getStatus().toString());
                noteTask.execute(url);
            }
        }
    }

    private void bindNote() {
        profileLv.setAdapter(note_adapter);
        note_adapter.setShowImg(Main_ProfileFragment.this);
        profileLv.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                if (NoteDatas.size() == 0) return;
                if (position > 0) {
                    DataEntity item = NoteDatas.get(position - 1);
                    if (null == item.getEvent()) {
                        if (item.isHasHtml()) {
                            //网页文章
                            Intent i = new Intent(getActivity(), Note_with_htmlActivity.class);
                            i.putExtra("nid", item.getId());
                            startActivity(i);
                            getActivity().overridePendingTransition(R.anim.in_from_bottom, R.anim.stand);
                        } else {
                            //普通日志
                            Intent i = new Intent();
                            if (null != item.getPicList() && item.getPicList().size() >= 2) {
                                //多图
                                i.setClass(getActivity(), Note_with_picsActivity.class);
                            } else {
                                //无图或单图
                                i.setClass(getActivity(), Note_detailActivity.class);
                            }
                            i.putExtra("id", item.getId());
                            startActivity(i);
                            getActivity().overridePendingTransition(R.anim.in_from_bottom, R.anim.stand);
                        }
                    } else {
                        //活动
                        Event_detail2Activity.startAction(getActivity(), item);
                        getActivity().overridePendingTransition(R.anim.in_from_bottom, R.anim.stand);
                    }
                }
            }
        });
    }

    //请求用户加入的群组
    private void requestGroup() {
        if (groupDatas.size() > 0) {
            bindGroup();
        } else {
            String url = URLUtil.getInstance().userTag(mid, offsets[1], LOAD_NUM);
            if (groupTask == null || !groupTask.getStatus().equals(AsyncTask.Status.RUNNING)) {
                groupTask = new GroupTask();
                groupTask.execute(url);
            }
        }
    }

    private void bindGroup() {
        profileLv.setAdapter(group_adapter);
        profileLv.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                if (groupDatas.size() == 0) return;
                if (position > 0) {
                    position--;
                    Group_detailActivity.startAction(getActivity(),
                            groupDatas.get(position).getTid(),
                            groupDatas.get(position).getName());

                    getActivity().overridePendingTransition(R.anim.in_from_right, R.anim.stand);
                }
            }
        });
    }

    //请求用户的好友
    private void requestFriend() {
        if (userData.size() > 0) {
            bindFriend();
        } else {
            String url = URLUtil.getInstance().getMyFriend(
                    DataHouse.getAPP(getActivity()).bean.getOfusername(),
                    LOAD_NUM,
                    0);
            Logger.d(url);
            if (friendTask == null || !friendTask.getStatus().equals(AsyncTask.Status.RUNNING)) {
                friendTask = new FriendTask();
                Logger.d(friendTask.getStatus().toString());
                friendTask.execute(url);
            }
        }
    }

    private void bindFriend() {
        profileLv.setAdapter(friend_adapter);
        profileLv.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent, View view, int position, long id) {

            }
        });
    }

    //获取用户的个人信息
    private void requestUser() {
        if (myInfoSA == null) {
            UserDataEntity uInfo = DataHouse.getBean(getActivity());
            String[] s1 = new String[]{"年龄", "星座", "身高", "地点", "个性签名"};
            String[] s2 = new String[5];
            s2[0] = uInfo.getAge() + "";
            s2[1] = CharConvert.convert(uInfo.getConstellation());
            s2[2] = uInfo.getHeight() + "";
            s2[3] = CharConvert.convert(uInfo.getAddress());
            s2[4] = CharConvert.convert(uInfo.getSign());
            ArrayList<HashMap<String, Object>> myInfoList = new ArrayList<HashMap<String, Object>>();
            for (int i = 0; i < s1.length; i++) {
                HashMap<String, Object> hs = new HashMap<>();
                hs.put("myinfo_t", s1[i]);
                hs.put("myinfo_c", s2[i]);
                myInfoList.add(hs);
            }
            myInfoSA = new SimpleAdapter(getActivity(), myInfoList, R.layout.item_profile_myinfo,
                    new String[]{"myinfo_t", "myinfo_c"},
                    new int[]{R.id.myinfo_t, R.id.myinfo_c});
        }
        profileLv.setAdapter(myInfoSA);
        profileLv.setOnItemClickListener(null);
        profileLv.loadComplete();
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
        switch (profileLv.profileTabs.getSelectedTabPosition()) {
            case 0://动态
                if (NoteDatas.size() > 0 && NoteDatas.size() % LOAD_NUM == 0) {
                    new ChangeTask().executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR, 0);
                } else profileLv.loadComplete();
                break;
            case 1://群组
                if (groupDatas.size() > 0 && groupDatas.size() % LOAD_NUM == 0) {
                    new ChangeTask().executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR, 1);
                } else profileLv.loadComplete();
                break;
            case 2://好友
                if (userData.size() > 0 && userData.size() % LOAD_NUM == 0) {
                    new ChangeTask().executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR, 2);
                } else profileLv.loadComplete();
                break;
            case 3://简介
                profileLv.loadComplete();
                break;
        }
    }

    @Override
    public void showImg(PhotoView v, String url) {
        //        Info mInfo = v.getInfo();
        popupWindow.showAtLocation(parent, Gravity.BOTTOM, 0, 0);
        //进场动画photoView.animaFrom(mInfo);
        photoView.setImageResource(R.drawable.notification_icon_bg);

        picasso.load(url)
                .config(Bitmap.Config.RGB_565)
                .error(R.drawable.load_image)
                .memoryPolicy(MemoryPolicy.NO_CACHE, MemoryPolicy.NO_STORE)
                .into(photoView);

        root.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (popupWindow.isShowing()) popupWindow.dismiss();
            }
        });

        photoView.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (popupWindow.isShowing()) popupWindow.dismiss();
            }
        });
    }

    private class NoteTask extends AsyncTask<String, Void, Boolean> {
        @Override
        protected Boolean doInBackground(String... params) {
            try {
                String rst = okHttpUtil.run(params[0]);
                Gson_mynote2_bean mynoteBean = gson.fromJson(rst, Gson_mynote2_bean.class);
                if (mynoteBean.getStatus().getCode().equals("0")) {
                    if (null == mynoteBean.getKeys() || mynoteBean.getKeys().size() == 0)
                        return false;
                    NoteDatas.clear();
                    for (int i = 0; i < mynoteBean.getKeys().size(); i++) {
                        List<DataEntity> notes = mynoteBean.getData().get(mynoteBean.getKeys().get(i));
                        for (int j = 0; j < notes.size(); j++) {
                            NoteDatas.add(notes.get(j));
                        }
                    }
                    offsets[0] = NoteDatas.get(NoteDatas.size() - 1).getId();
                    return true;
                }
            } catch (Exception e) {
                return false;
            }
            return false;
        }

        @Override
        protected void onPostExecute(Boolean s) {
            super.onPostExecute(s);
            if (s) {
                note_adapter = new Note_lv_adapter(getActivity(), NoteDatas, picasso);
                bindNote();
            } else {
                NoteDatas.clear();
                profileLv.setAdapter(null);
                profileLv.setOnItemClickListener(null);
            }
        }
    }

    private class GroupTask extends AsyncTask<String, Void, Boolean> {
        @Override
        protected Boolean doInBackground(String... params) {
            try {
                String rst = okHttpUtil.run(params[0]);
                Gson_biaoqian_content_bean bean = gson.fromJson(rst, Gson_biaoqian_content_bean.class);
                if (bean.getStatus().getCode().equals("0")) {
                    groupDatas.clear();
                    groupDatas.addAll(bean.getData());
                    offsets[1] = 1;
                    return true;
                }
            } catch (Exception e) {
                return false;
            }
            return false;
        }

        @Override
        protected void onPostExecute(Boolean aBoolean) {
            super.onPostExecute(aBoolean);
            if (aBoolean) {
                group_adapter = new BiaoQian_content_adapter(getActivity(), groupDatas, picasso);
                bindGroup();
            } else {
                groupDatas.clear();
                profileLv.setAdapter(null);
                profileLv.setOnItemClickListener(null);
            }
        }
    }

    private class FriendTask extends AsyncTask<String, Void, Boolean> {
        @Override
        protected Boolean doInBackground(String... params) {
            try {
                String rst = okHttpUtil.run(params[0]);
                Gson_my_friend_bean bean = new Gson().fromJson(rst, Gson_my_friend_bean.class);
                if (bean.getStatus().getCode().equals("0")) {
                    userData.addAll(bean.getData());
                    offsets[2] = 1;
                    return true;
                }
            } catch (Exception e) {
                Logger.e(e.toString());
                return false;
            }
            return false;
        }

        @Override
        protected void onPostExecute(Boolean aBoolean) {
            super.onPostExecute(aBoolean);
            if (aBoolean) {
                friend_adapter = new Profile_my_friend_adapter(getActivity(), R.layout.item_note_news_c2, userData);
                bindFriend();
            } else {
                profileLv.setAdapter(null);
                profileLv.setOnItemClickListener(null);
            }
        }
    }

    private class ChangeTask extends AsyncTask<Integer, Void, Integer> {
        @Override
        protected Integer doInBackground(Integer... params) {
            try {
                switch (params[0]) {
                    case 0://动态
                        String url0 = URLUtil.getInstance().getMyNotes(mid, LOAD_NUM, offsets[0]);
                        String rst0 = okHttpUtil.run(url0);
                        Gson_mynote2_bean mynoteBean = gson.fromJson(rst0, Gson_mynote2_bean.class);
                        if (mynoteBean.getStatus().getCode().equals("0")) {
                            if (null == mynoteBean.getKeys() || mynoteBean.getKeys().size() == 0)
                                return 0;
                            for (int i = 0; i < mynoteBean.getKeys().size(); i++) {
                                List<DataEntity> notes = mynoteBean.getData().get(mynoteBean.getKeys().get(i));
                                for (int j = 0; j < notes.size(); j++) {
                                    NoteDatas.add(notes.get(j));
                                }
                            }
                            offsets[0] = NoteDatas.get(NoteDatas.size() - 1).getId();
                            return 0;
                        }
                        break;
                    case 1://群组
                        String url1 = URLUtil.getInstance().userTag(mid, offsets[1], LOAD_NUM);
                        String rst1 = okHttpUtil.run(url1);
                        Gson_biaoqian_content_bean bean1 = gson.fromJson(rst1, Gson_biaoqian_content_bean.class);
                        if (bean1.getStatus().getCode().equals("0")) {
                            groupDatas.addAll(bean1.getData());
                            offsets[1] += 1;
                            return 1;
                        }
                        break;
                    case 2://好友
                        String url2 = URLUtil.getInstance().getMyFriend(
                                DataHouse.getAPP(getActivity()).bean.getOfusername(),
                                LOAD_NUM,
                                offsets[2]);
                        String rst2 = okHttpUtil.run(url2);
                        Gson_my_friend_bean bean2 = new Gson().fromJson(rst2, Gson_my_friend_bean.class);
                        if (bean2.getStatus().getCode().equals("0")) {
                            userData.addAll(bean2.getData());
                            offsets[2] += 1;
                            return 2;
                        }
                        break;
                }
            } catch (Exception e) {
                Logger.e(e.toString());
                return -1;
            }
            return -1;
        }

        @Override
        protected void onPostExecute(Integer integer) {
            super.onPostExecute(integer);
            profileLv.loadComplete();
            if (integer == -1) {
                profileLv.setAdapter(null);
                profileLv.setOnItemClickListener(null);
            } else {
                switch (integer) {
                    case 0:
                        note_adapter.notifyDataSetChanged();
                        break;
                    case 1:
                        group_adapter.notifyDataSetChanged();
                        break;
                    case 2:
                        friend_adapter.notifyDataSetChanged();
                        break;
                }
            }
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


