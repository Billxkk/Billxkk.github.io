---
layout: post
title: fragment跳转，实现传递参数和结果回传
category: 安卓
tags:  android
description: 
---


**效果实现：**
*一个activity的同一个容器中的2个fragment*
1.点击第一个fragment中的按钮,加载出第二个fragment，同时传递过去一个参数
2.点击第二个fragment的图片，将点击结果返回到第一个fragment，选择的图片显示到第一个fragment中

### 一、基本界面实现

  Fragment1布局：
  ImageView用来放来自fragment2的图片

        <ImageView
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:id="@+id/firstPic"
        android:scaleType="center"/>
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/BtnToSecond"
        android:text="to2"/>
Fragment2布局：
TextView用来存放Fragment1的参数，下面是3个图片

        <TextView
        android:id="@+id/text2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/hello_blank_fragment" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">

        <ImageView
            android:layout_width="100dp"
            android:layout_height="100dp"
            android:id="@+id/pic1"
            android:scaleType="center"
            android:background="#00ff00"/>

        <ImageView
            android:layout_width="100dp"
            android:layout_height="100dp"
            android:id="@+id/pic2"
            android:scaleType="center"
            android:background="#00ffff"/>

        <ImageView
            android:layout_width="100dp"
            android:layout_height="100dp"
            android:id="@+id/pic3"
            android:scaleType="center"
            android:background="#ff0000"/>

    </LinearLayout>

在Activity中初始化时将Fragment1显示出来

        @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_fragment_test);

        FromFrag fromFrag = new FromFrag();
        //第一个Fragment
        FragmentManager mana = getSupportFragmentManager();
        FragmentTransaction mtran = mana.beginTransaction();
        mtran.add(R.id.FragContainer_infoTrans,fromFrag,"fromFrag");
        //将fragment贴到容器FragContainer_infoTrans中
        mtran.addToBackStack(null);
        mtran.commit();
Fragment1中的操作：
设置Button按键的点击事件，

        @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
         View view = inflater.inflate(R.layout.fragment_from, container, false);
        Button btn = (Button)view.findViewById(R.id.BtnToSecond);
        final ImageView imageView = (ImageView)view.findViewById(R.id.firstPic);
        //按键实现
        btn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                ToFrag toFrag = new ToFrag();                
                FragmentTransaction transaction = getFragmentManager().beginTransaction();
                transaction.add(R.id.FragContainer_infoTrans, toFrag);
                //addFragment2
                transaction.addToBackStack(null);
                transaction.commit();
            }
        });
        return  view;
    }

Fragment2的设置：
将内容显示出来就行了

        @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        View view =  inflater.inflate(R.layout.fragment_to, container, false);
        return view;
    }

### 二、Fragment间的参数传递
**fragment1传到fragment2：**

在fragment2中建立下面的函数，新建实例时获得参数

        public static ToFrag newInstance(String param1,String param2) {
        ToFrag fragment = new ToFrag();
        Bundle args = new Bundle();
        args.putString(ARG_PARAM1, param1);
        args.putString(ARG_PARAM2, param2);
        fragment.setArguments(args);
        return fragment;
    }

在OnCreatView中获取参数

        @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        View view =  inflater.inflate(R.layout.fragment_to, container, false);
        if (getArguments() != null) {
            String mParam1 = getArguments().getString(ARG_PARAM1);
            TextView tv =  (TextView)view.findViewById(R.id.text2);
            tv.setText(mParam1);
        }
        return view;
    }
在Fragment1中贴fragment2时，调用newInstance获取实例

           btn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                ToFrag toFrag = ToFrag.newInstance("参数from 1",null);
                //传送参数
                FragmentTransaction transaction = getFragmentManager().beginTransaction();
                transaction.add(R.id.FragContainer_infoTrans, toFrag);
                transaction.addToBackStack(null);
                transaction.commit();
            }
        });

**Fragment2回调到fragment1**

在Fragment2中实现监听器：

        private ImageClickEventListener imageClickEventListener;
        public interface ImageClickEventListener{
        void ImageClickEvent(int id);
    }

    public void setImageClickEventListener(ImageClickEventListener listener){
        imageClickEventListener = listener;
    }
将Fragment2 实现 View.OnClickListener接口，设置图片的点击事件：

       @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        View view =  inflater.inflate(R.layout.fragment_to, container, false);
        if (getArguments() != null) {
            String mParam1 = getArguments().getString(ARG_PARAM1);
            TextView tv =  (TextView)view.findViewById(R.id.text2);
            tv.setText(mParam1);
        }
        ImageView img1 = (ImageView)view.findViewById(R.id.pic1);
        ImageView img2 = (ImageView)view.findViewById(R.id.pic2);
        ImageView img3 = (ImageView)view.findViewById(R.id.pic3);
        img1.setOnClickListener(this);
        img2.setOnClickListener(this);
        img3.setOnClickListener(this);

        return view;
    }

    @Override
    public void onClick(View v) {
        int id = v.getId();
        int drawableID = -1;
        switch (id){
            case R.id.pic1:
                drawableID = 1;
                break;
            case R.id.pic2:
                drawableID = 2;
                break;
            case R.id.pic3:
                drawableID = 3;
                break;
        }
        if (drawableID != -1) {
            imageClickEventListener.ImageClickEvent(drawableID);
        }
        onDetach();
    }
    
在Fragment1实现监听器：

            btn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                ToFrag toFrag = ToFrag.newInstance("参数from 1",null);
                //点击事件获取
                toFrag.setImageClickEventListener(new ToFrag.ImageClickEventListener() {
                    @Override
                    public void ImageClickEvent(int id) {
                        switch (id){
                            case 1:
                                imageView.setBackgroundColor(new Color().rgb(0,255,0));
                                break;
                            case 2:
                                imageView.setBackgroundColor(new Color().rgb(0,255,255));
                                break;
                            case 3:
                                imageView.setBackgroundColor(new Color().rgb(255,0,0));
                        }
                    }
                });
                FragmentTransaction transaction = getFragmentManager().beginTransaction();
                transaction.add(R.id.FragContainer_infoTrans, toFrag);
                transaction.addToBackStack(null);
                transaction.commit();
            }
        });
        
  
  *程序相关代码*[http://download.csdn.net/detail/qq_35073176/9912212](http://download.csdn.net/detail/qq_35073176/9912212)
*参考资料*[http://blog.csdn.net/harvic880925/article/details/44131865](http://blog.csdn.net/harvic880925/article/details/44131865)