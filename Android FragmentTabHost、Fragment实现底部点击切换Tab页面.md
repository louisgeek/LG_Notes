

1.aty
    package com.example.louisfragmenttabhostdemo;

	import android.annotation.SuppressLint;
	import android.app.Activity;
	import android.os.Bundle;
	import android.support.v4.app.FragmentTabHost;
	import android.support.v7.app.AppCompatActivity;
	import android.view.View;
	import android.widget.LinearLayout;
	import android.widget.TabHost;
	import android.widget.TabHost.OnTabChangeListener;
	import android.widget.TextView;
	import android.widget.Toast;

	public class MainActivity extends AppCompatActivity {

	private FragmentTabHost mFragmentTabHost;
	private Class<?> mFragmentClasses[] = { AFragment.class, BFragment.class, CFragment.class, DFragment.class };
	// Tab选项卡的drawable
	private int imageIds[] = { R.drawable.tab_news_selector, R.drawable.tab_read_selector,
			R.drawable.tab_found_selector, R.drawable.tab_self_selector };

	// Tab选项卡的文字
	private String tabLabels[] = { "新闻", "阅读", "发现", "个人" };

	@SuppressLint("NewApi")
	@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mFragmentTabHost = (FragmentTabHost) findViewById(R.id.id_f_tabhost);
       // mFragmentTabHost.setup(this,getSupportFragmentManager());
        mFragmentTabHost.setup(this, getSupportFragmentManager(), R.id.id_layout_fl_tab_content);
        for (int i=0;i<tabLabels.length;i++){
            TabHost.TabSpec tabSpec= mFragmentTabHost.newTabSpec(tabLabels[i]+i);

            tabSpec.setIndicator(getBiaojIView(i));
            mFragmentTabHost.addTab(tabSpec,mFragmentClasses[i],null);
        }

        mFragmentTabHost.setOnTabChangedListener(new OnTabChangeListener() {
			
			@Override
			public void onTabChanged(String tabId) {
				// TODO Auto-generated method stub
				Toast.makeText(MainActivity.this, "qiehuan:"+tabId, 2).show();
			}
		});
        mFragmentTabHost.setCurrentTab(1);

    }

	@SuppressLint("NewApi")
	private View getBiaojIView(int i) {
		  TextView textView= (TextView) View.inflate(this,
                  R.layout.layout_indicator_view, null);
        //  TextView textView= (TextView) linearLayout.findViewById(R.id.id_tv_indicator);
          textView.setCompoundDrawablesWithIntrinsicBounds(null, getResources()
                  .getDrawable(imageIds[i], this.getTheme()), null, null);
          textView.setText(tabLabels[i]);
          return textView;
	}

}


2.A、B、C、D Frgment就调用文字不一样的布局
    package com.example.louisfragmenttabhostdemo;

	import android.os.Bundle;
	import android.support.v4.app.Fragment;
	import android.view.LayoutInflater;	
	import android.view.View;
	import android.view.ViewGroup;


	/**
	 * A simple {@link Fragment} subclass.
	 * Use the {@link BFragment#newInstance} factory method to
 	* create an instance of this fragment.
 	*/
	public class AFragment extends Fragment {
    // TODO: Rename parameter arguments, choose names that match
    // the fragment initialization parameters, e.g. ARG_ITEM_NUMBER
    private static final String ARG_PARAM1 = "param1";
    private static final String ARG_PARAM2 = "param2";

    // TODO: Rename and change types of parameters
    private String mParam1;
    private String mParam2;



    public AFragment() {
        // Required empty public constructor
    }

    /**
     * Use this factory method to create a new instance of
     * this fragment using the provided parameters.
     *
     * @param param1 Parameter 1.
     * @param param2 Parameter 2.
     * @return A new instance of fragment AFragment.
     */
    // TODO: Rename and change types and number of parameters
    public static AFragment newInstance(String param1, String param2) {
        AFragment fragment = new AFragment();
        Bundle args = new Bundle();
        args.putString(ARG_PARAM1, param1);
        args.putString(ARG_PARAM2, param2);
        fragment.setArguments(args);
        return fragment;
    }

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        if (getArguments() != null) {
            mParam1 = getArguments().getString(ARG_PARAM1);
            mParam2 = getArguments().getString(ARG_PARAM2);
        }
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.fragment_a, container, false);
    }



}


3.按钮不同状态标记布局 tab_news_selector.xml、tab_found_selector.xml、tab_read_selector.xml、tab_self_selector.xml就对应的图片不一样
    
    <?xml version="1.0" encoding="utf-8"?>
	<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@drawable/tab_news_light" android:state_selected="true"/>
    <item android:drawable="@drawable/tab_news_normal"/>
	</selector>


4.aty布局文件 

    <?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="vertical" >

    <FrameLayout
        android:id="@+id/id_layout_fl_tab_content"
        android:layout_width="fill_parent"
        android:layout_height="0dip"
        android:layout_weight="1" />

    <android.support.v4.app.FragmentTabHost
        android:id="@+id/id_f_tabhost"
        android:layout_width="fill_parent"
        android:layout_height="wrap_content" >

        
        <!--网上例子都有这些  但是我测试会报错 <TabWidget
            android:id="@android:id/tabs"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_weight="0"
            android:orientation="horizontal" />

             <FrameLayout
            android:id="@android:id/tabcontent"
            android:layout_width="0dp"
            android:layout_height="0dp"
            android:layout_weight="0" /> -->
       
    </android.support.v4.app.FragmentTabHost>

	</LinearLayout>


5.A、B、C、Dfragment布局  就文字不一样

    <FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.louis.louisfragmenttabhost.AFragment">

    <!-- TODO: Update blank fragment layout -->
    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:text="A fragment"
        android:gravity="center" />

	</FrameLayout>

6.标记xml

    <?xml version="1.0" encoding="utf-8"?>
	<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
      	android:textColor="#AEAEAE"
        android:text="新闻" 
        android:gravity="center"
        android:layout_marginTop="2dp" />

7。结果

![http://i.imgur.com/VjXrGKZ.png](http://i.imgur.com/VjXrGKZ.png)

