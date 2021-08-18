简介



## 特点

- 原生支持列表、网格、瀑布流等布局
- 分割线 ItemDecoration，较灵活
- 自带 ViewHolder 机制
- 自带 Item 动画
- 自带局部刷新
- 自带拖拽、侧滑接口
- 未实现头尾视图、空视图和 Item 点击事件
- 改进了缓存机制，有四级缓存
- 可扩展性强



## RecyclerView 常规使用

### DataItem.java

```java
public class DataItem {
    public String id;
    public String title;
    public String subtitle;
    public String content;
    public int imageResId;
}

```

- 可选实现 Cloneable 接口，可解决后面遇到对象引用问题



### listitem_image_text.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content">


    <TextView
        android:id="@+id/id_tv_title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:ellipsize="end"
        android:padding="10dp"
        android:singleLine="true"
        android:textSize="22sp"
        app:layout_constraintEnd_toStartOf="@id/id_iv"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        tools:text="标题标题标题标题标题标题标题标题标题标题end" />

    <TextView
        android:id="@+id/id_tv_subtitle"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:ellipsize="end"
        android:padding="10dp"
        android:singleLine="true"
        android:textSize="18sp"
        app:layout_constraintEnd_toStartOf="@id/id_iv"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/id_tv_title"
        tools:text="副标题副标题副标题副标题副标题副标题副标题副标题副标题副标题副标题end" />

    <TextView
        android:id="@+id/id_tv_content"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:padding="10dp"
        android:textSize="14sp"
        app:layout_constraintEnd_toStartOf="@id/id_iv"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/id_tv_subtitle"
        tools:text="内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容内容end" />

    <!--在 id_tv_title,id_tv_subtitle,id_tv_content 三个控件右侧设置一道屏障-->
    <androidx.constraintlayout.widget.Barrier
        android:id="@+id/id_barrier"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:barrierDirection="right"
        app:constraint_referenced_ids="id_tv_title,id_tv_subtitle,id_tv_content" />

    <!--id_iv 控件依赖上面这道屏障-->
    <ImageView
        android:id="@+id/id_iv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@mipmap/ic_launcher"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toEndOf="@id/id_barrier" />


</androidx.constraintlayout.widget.ConstraintLayout>
```



### DataItemViewHolder.java

- 也可以放在 Adapter 里，作为静态内部类

```java
public class DataItemViewHolder extends RecyclerView.ViewHolder {
    TextView id_tv_title;
    TextView id_tv_subtitle;
    TextView id_tv_content;
    ImageView id_iv;

    public DataItemViewHolder(@NonNull View itemView) {
        super(itemView);
        id_tv_title = itemView.findViewById(R.id.id_tv_title);
        id_tv_subtitle = itemView.findViewById(R.id.id_tv_subtitle);
        id_tv_content = itemView.findViewById(R.id.id_tv_content);
        id_iv = itemView.findViewById(R.id.id_iv);
    }

}
```





### DataItemRVAdapter.java

```java
public class DataItemRVAdapter extends RecyclerView.Adapter<DataItemViewHolder> {
    public static final String KEY_TITLE = "KEY_TITLE";
    public static final String KEY_SUBTITLE = "KEY_SUBTITLE";
    public static final String KEY_CONTENT = "KEY_CONTENT";
    public static final String KEY_IMAGE = "KEY_IMAGE";

    private List<DataItem> mDataItemList;

    public void setDataItemList(List<DataItem> dataItemList) {
        this.mDataItemList = new ArrayList<>(dataItemList);
        /*this.mDataItemList = new ArrayList<>();
        this.mDataItemList.addAll(dataItemList);*/
    }

    public List<DataItem> getDataItemList() {
        return mDataItemList;
    }

    @NonNull
    @Override
    public DataItemViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View itemView = LayoutInflater.from(parent.getContext()).inflate(R.layout.listitem_image_text, parent, false);
        return new DataItemViewHolder(itemView);
    }

    @Override
    public void onBindViewHolder(@NonNull DataItemViewHolder holder, int position) {
        DataItem dataItem = mDataItemList.get(position);
        holder.id_tv_title.setText(dataItem.title);
        holder.id_tv_subtitle.setText(dataItem.subtitle);
        holder.id_tv_content.setText(dataItem.content);
        holder.id_iv.setImageResource(dataItem.imageResId);
    }

    @Override
    public void onBindViewHolder(@NonNull DataItemViewHolder holder, int position, @NonNull List<Object> payloads) {
        if (payloads.isEmpty()) {
            //【列表刷新】 或者 【列表局部 Item 刷新】
            super.onBindViewHolder(holder, position, payloads);
        } else {
            //【Item 局部 View 刷新】
            Object payload = payloads.get(0);
            if (payload instanceof Map) {
                //【key-value 键值对】
                Map<String, String> map = (Map<String, String>) payload;
                for (Map.Entry<String, String> stringObjectEntry : map.entrySet()) {
                    String key = stringObjectEntry.getKey();
                    String value = stringObjectEntry.getValue();
                    //简单封装
                    bindViewHolder(holder, position, key, value);
                }
            } else if (payload instanceof String) {
                //【key 标记】
                String key = (String) payload;
                //简单封装
                bindViewHolder(holder, position, key, null);
            }

        }
    }

    private void bindViewHolder(DataItemViewHolder holder, int position, String key, String value) {
        DataItem dataItem = mDataItemList.get(position);
        if (key.equals(KEY_TITLE)) {
            holder.id_tv_title.setText(value != null ? value : dataItem.title);
        } else if (key.equals(KEY_SUBTITLE)) {
            holder.id_tv_subtitle.setText(value != null ? value : dataItem.content);
        } else if (key.equals(KEY_CONTENT)) {
            holder.id_tv_content.setText(value != null ? value : dataItem.content);
        } else if (key.equals(KEY_IMAGE)) {
            holder.id_iv.setImageResource(value != null ? Integer.parseInt(value) : dataItem.imageResId);
        }
    }

    @Override
    public int getItemCount() {
        return mDataItemList.size();
    }

}
```

### MainActivity.java

```java
public class MainActivity extends AppCompatActivity {
    private List<DataItem> mDataItemList;
    private DataItemRVAdapter mDataItemRVAdapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Button id_btn = findViewById(R.id.id_btn);

        id_btn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //
                refreshData();
            }
        });
        //
        initData();
        //
        RecyclerView id_rv = findViewById(R.id.id_rv);
   		LinearLayoutManager linearLayoutManager = new LinearLayoutManager(this);
        id_rv.setLayoutManager(linearLayoutManager);
        id_rv.setItemAnimator(new DefaultItemAnimator());
        id_rv.addItemDecoration(new DividerItemDecoration(this, 					linearLayoutManager.getOrientation()));
        //初始化
        id_rv.setAdapter(mDataItemRVAdapter);
        mDataItemRVAdapter.setDataItemList(mDataItemList);
    }

    private void initData() {
        mDataItemList = new ArrayList<>();
        for (int i = 0; i < 15; i++) {
            DataItem dataItem = new DataItem();
            dataItem.id = String.valueOf(i + 1);
            dataItem.title = dataItem.id + " title";
            dataItem.subtitle = dataItem.id + " subtitle";
            dataItem.content = dataItem.id + " content";
            dataItem.imageResId = R.mipmap.ic_launcher;
            mDataItemList.add(dataItem);
        }

    }

    private void refreshData() {
        //mDataItemList 存放最新的数据
        //1 修改数据
        //引用传递，会影响 Adapter 里 list（虽然 Adapter 里 list 已经是 new ArrayList 了）对应索引的对象的值，但是因为这里不影响
        mDataItemList.get(1).title = "222 title new";
        mDataItemList.get(1).subtitle = "222 subtitle new";
        mDataItemList.get(1).content = "222 content new";
        //2 更新 Adapter 数据源
        mDataItemRVAdapter.setDataItemList(mDataItemList);
        //3 通过 Adapter#notifyDataSetChanged 方式通知数据刷新
        mDataItemRVAdapter.notifyDataSetChanged();

    }
}
```





### 1 全局刷新

- 列表刷新

```java
//列表可见部分全部刷新
mAdapter.notifyDataSetChanged()
```



### 2 局部刷新 Item 刷新

- Item 整体刷新

```java
//修改指定 position 上 Item 的刷新
mAdapter.notifyItemChanged(int position)
//添加指定 position 上 Item 的刷新
mAdapter.notifyItemInserted(int position)
//移除指定 position 上 Item 的刷新
mAdapter.notifyItemRemoved(int position)
//修改从 positionStart 开始到 itemCount 数量的 Item 的刷新
mAdapter.notifyItemRangeChanged(int positionStart, int itemCount)
//添加从 positionStart 开始到 itemCount 数量的 Item 的刷新
mAdapter.notifyItemRangeInserted(int positionStart, int itemCount)
//移除从 positionStart 开始到 itemCount 数量的 Item 的刷新
mAdapter.notifyItemRangeRemoved(int positionStart, int itemCount)

//移到指定 fromPosition 到 toPosition 上 Item 的刷新
mAdapter.notifyItemMoved(int fromPosition, int toPosition)
```

- PS: 点击一个 Item 的删除按钮进行数据删除 notifyItemRemoved ，如果采用事件监听时会传 position 给点击事件接口方法，此时不能直接写 position，应为它其实变成了  final 类型 （注意如果是 java 代码，在 AS 里显示下划线），这样就造成错乱了，需要直接写 holder.getBindingAdapterPosition (其实这里 holder 变成 final 了) ，这样才能保证点下一个删除按钮时候的 pos 是正确的，notifyItemInserted 也是类似

- 还有一种解决方案，就是删除或新增后用 mAdapter.notifyItemRangeChanged(pos, mAdapter.getData().size() - pos) 强行处理一波，及时刷新 pos ，让 pos 能够正确

  

### 3 局部刷新 View 刷新

- Item 的部分 View 控件刷新

```java
//修改指定 position 上 Item 局部 View 的刷新，
mAdapter.notifyItemChanged(int position,Object payload)
//修改从 positionStart 开始到 itemCount 数量的 Item 局部 View 的刷新
mAdapter.notifyItemRangeChanged(int positionStart, int itemCount,Object payload)
```

- 假设 payload 只传 【key 标记】的话，刷新前的数据源就需要先完成更新，这样 Adapter 才直接用数据源
- 另外 payload 可以传 【key-value 键值对】，比如 Map ，那么 Adapter 可以不用数据源而改用 value



#### 修改 DataItemRVAdapter

- 覆写带 Payload 参数的 onBindViewHolder 方法

```java
 @Override
    public void onBindViewHolder(@NonNull DataItemViewHolder holder, int position, @NonNull List<Object> payloads) {
        if (payloads.isEmpty()) {
            //【列表刷新】 或者 【列表局部 Item 刷新】
            super.onBindViewHolder(holder, position, payloads);
        } else {
            //【Item 局部 View 刷新】
            Object payload = payloads.get(0);
            if (payload instanceof Map) {
                //【key-value 键值对】
                Map<String, String> map = (Map<String, String>) payload;
                for (Map.Entry<String, String> stringObjectEntry : map.entrySet()) {
                    String key = stringObjectEntry.getKey();
                    String value = stringObjectEntry.getValue();
                    //简单封装
                    bindViewHolder(holder, position, key, value);
                }
            } else if (payload instanceof String) {
                //【key 标记】
                String key = (String) payload;
                //简单封装
                bindViewHolder(holder, position, key, null);
            }

        }
    }

    private void bindViewHolder(DataItemViewHolder holder, int position, String key, String value) {
        DataItem dataItem = mDataItemList.get(position);
        if (key.equals(KEY_TITLE)) {
            holder.id_tv_title.setText(value != null ? value : dataItem.title);
        } else if (key.equals(KEY_SUBTITLE)) {
            holder.id_tv_subtitle.setText(value != null ? value : dataItem.content);
        } else if (key.equals(KEY_CONTENT)) {
            holder.id_tv_content.setText(value != null ? value : dataItem.content);
        } else if (key.equals(KEY_IMAGE)) {
            holder.id_iv.setImageResource(value != null ? Integer.parseInt(value) : dataItem.imageResId);
        }
    }

```





## LayoutManager

- LinearLayoutManager
- GridLayoutManager
- 



## ItemDecoration

- 继承 RecyclerView.ItemDecoration 实现
- 系统自带一个 DividerItemDecoration 可以直接用
- 通过覆写 getItemOffsets 方法指定 item 的宽度，也可以直接给 item 设置 margin 变相实现
- ItemDecoration 在 RecyclerView 的 Item 绘制之前绘制



## ItemAnimator

- 系统自带一个 DefaultItemAnimator



## HeaderView FooterView

```java
/**
 * Created by louisgeek on 2021/4/28.
 * Decorator 装饰模式
 */
public class AdapterWrapper extends RecyclerView.Adapter<RecyclerView.ViewHolder> {

    private RecyclerView.Adapter<RecyclerView.ViewHolder> mAdapter;
    private View mHeaderView;
    private View mFooterView;
    private static final int HEADER = 0;
    private static final int NORMAL = 1;
    private static final int FOOTER = 2;

    public AdapterWrapper(RecyclerView.Adapter<RecyclerView.ViewHolder> adapter) {
        mAdapter = adapter;
    }

    @NonNull
    @Override
    public RecyclerView.ViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        if (viewType == HEADER) {
            //无需 inflate，直接返回
            return new RecyclerView.ViewHolder(mHeaderView) {
            };
        } else if (viewType == FOOTER) {
            //无需 inflate，直接返回
            return new RecyclerView.ViewHolder(mFooterView) {
            };
        }
        return mAdapter.onCreateViewHolder(parent, viewType);
    }

    @Override
    public void onBindViewHolder(@NonNull RecyclerView.ViewHolder holder, int position) {
        if (position == 0) {
            //第一个头
            //无需绑定操作
        } else if (position == this.getItemCount() - 1) {
            //最后一个尾
            //无需绑定操作
        } else {
            //第二个索引才是原来 NORMAL 列表的的第一个
            int realPosition = position - 1;
            mAdapter.onBindViewHolder(holder, realPosition);
        }
    }

    @Override
    public int getItemCount() {
        //多了头尾
        return mAdapter.getItemCount() + 2;
    }

    @Override
    public int getItemViewType(int position) {
        if (position == 0) {
            //第一个头
            return HEADER;
        } else if (position == this.getItemCount() - 1) {
            //最后一个尾
            return FOOTER;
        } else {
            return NORMAL;
        }
    }

    public void addHeaderView(View view) {
        this.mHeaderView = view;
    }

    public void addFooterView(View view) {
        this.mFooterView = view;
    }
}
```
- 利用装饰模式，减少代码侵入

```java
     mDataItemRVAdapter = new DataItemRVAdapter();
     //新增
     AdapterWrapper adapterWrapper = new AdapterWrapper(mDataItemRVAdapter);
	 adapterRecyclerViewAdapter.addHeaderView(mHeader);
 	 adapterRecyclerViewAdapter.addFooterView(mFooter);
      //修改 setAdapter
	//id_rv.setAdapter(mDataItemRVAdapter);
    id_rv.setAdapter(adapterWrapper);
```







## 使用 DiffUtil 刷新数据

- 通过计算得出新旧数据集的最小变化量

- DiffUtil#calculateDiff 推荐在子线程执行



### 1 局部刷新 Item 刷新

- Item 整体刷新



#### 创建 DiffUtilCallback

```java
public class DiffUtilCallback extends DiffUtil.Callback {
    private List<DataItem> mOldDataItemList;
    private List<DataItem> mNewDataItemList;

    public DiffCallback(List<DataItem> oldDataItemList, List<DataItem> newDataItemList) {
        this.mOldDataItemList = oldDataItemList;
        this.mNewDataItemList = newDataItemList;
    }

    @Override
    public int getOldListSize() {
        return mOldDataItemList != null ? mOldDataItemList.size() : 0;
    }

    @Override
    public int getNewListSize() {
        return mNewDataItemList != null ? mNewDataItemList.size() : 0;
    }

    @Override
    public boolean areItemsTheSame(int oldItemPosition, int newItemPosition) {
        //判断是不是同一个 item 的逻辑
        DataItem oldDataItem = mOldDataItemList.get(oldItemPosition);
        DataItem newDataItem = mNewDataItemList.get(newItemPosition);
        return Objects.equals(oldDataItem.id, newDataItem.id);
    }

    @Override
    public boolean areContentsTheSame(int oldItemPosition, int newItemPosition) {
        //areItemsTheSame 返回 true 的情况下走这个方法，进一步判断这个 item 的内容是否有变化
        DataItem oldDataItem = mOldDataItemList.get(oldItemPosition);
        DataItem newDataItem = mNewDataItemList.get(newItemPosition);
        // id
        if (!Objects.equals(oldDataItem.id, newDataItem.id)) {
            return false;
        }
        // name
        if (!Objects.equals(oldDataItem.name, newDataItem.name)) {
            return false;
        }
        // content
        if (!Objects.equals(oldDataItem.content, newDataItem.content)) {
            return false;
        }
        //如果还有，继续写其他判断条件

        //默认返回内容相等
        return true;
    }
}
```



#### 修改 MainActivity 使用

```java
private void refreshDataByDiffUtil() {
        List<DataItem> oldDataItemList = new ArrayList<>(mDataItemRVAdapter.getDataItemList());
        //mDataItemList 存放最新的数据
        //1 修改数据
        //别这么写！！！引用传递 引用传递会导致 calculateDiff 失效
//        mDataItemList.get(1).name = "222 new";
        DataItem oldDataItem = mDataItemList.get(2);
        DataItem newDataItem = new DataItem();
        newDataItem.id = oldDataItem.id;
        newDataItem.name = "333 new";//就改一个字段
        newDataItem.content = oldDataItem.content;
        mDataItemList.set(2, newDataItem);
        //2 更新 Adapter 数据源
        mDataItemRVAdapter.setDataItemList(mDataItemList);
        //3 DiffUtil 方式通知刷新，推荐在子线程中计算
        DiffUtil.DiffResult diffResult = DiffUtil.calculateDiff(new DiffCallback(oldDataItemList, mDataItemList));
        //内部按不同情况自动调用 Adapter#notifyItemXXX  方法实现的
        diffResult.dispatchUpdatesTo(mDataItemRVAdapter);
}
```

- 能够保留 item 动画





### 2 局部刷新 View 刷新

- Item 的部分 View 控件刷新

- 同上，Adapter 需要覆写带 Payload 参数的 onBindViewHolder 方法



#### 修改 DiffUtilCallback

- 继续覆写 getChangePayload 方法

```java
    @Nullable
    @Override
    public Object getChangePayload(int oldItemPosition, int newItemPosition) {
        DataItem oldDataItem = mOldDataItemList.get(oldItemPosition);
        DataItem newDataItem = mNewDataItemList.get(newItemPosition);
//        areItemsTheSame 返回 true 并且 areContentsTheSame 返回 false 的情况才会进来
        Map<String, String> payloadMap = new HashMap<>();
        // title
        if (!Objects.equals(oldDataItem.title, newDataItem.title)) {
            payloadMap.put(DataItemRVAdapter.KEY_TITLE, newDataItem.title);
        }
        // title
        if (!Objects.equals(oldDataItem.subtitle, newDataItem.subtitle)) {
            payloadMap.put(DataItemRVAdapter.KEY_SUBTITLE, newDataItem.subtitle);
        }
        // content
        if (!Objects.equals(oldDataItem.content, newDataItem.content)) {
            payloadMap.put(DataItemRVAdapter.KEY_CONTENT, newDataItem.content);
        }
        // imageResId
        if (oldDataItem.imageResId != newDataItem.imageResId) {
            payloadMap.put(DataItemRVAdapter.KEY_IMAGE, String.valueOf(newDataItem.imageResId));
        }
        if (!payloadMap.isEmpty()) {
            return payloadMap;
        }
        //默认无变化，内部传的是 null
        return super.getChangePayload(oldItemPosition, newItemPosition);
    }
```

使用

- 同上







## ItemTouchHelper 实现拖拽侧滑

### 创建 ItemTouchHelperCallback

```java
public class ItemTouchHelperCallback extends ItemTouchHelper.Callback {
    private static final String TAG = "ItemTouchHelperCallback";
    private DataItemRVAdapter mDataItemRVAdapter;

    public ItemTouchHelperCallback(DataItemRVAdapter dataItemRVAdapter) {
        this.mDataItemRVAdapter = dataItemRVAdapter;
    }

    @Override
    public int getMovementFlags(@NonNull RecyclerView recyclerView,
                                @NonNull RecyclerView.ViewHolder viewHolder) {
        //拖拽方向
        int dragFlags = 0;
        RecyclerView.LayoutManager layoutManager = recyclerView.getLayoutManager();
        if (layoutManager instanceof LinearLayoutManager) {
            //拖拽方向
            dragFlags = ItemTouchHelper.UP | ItemTouchHelper.DOWN;
        } else if (layoutManager instanceof GridLayoutManager) {
            // Grid 四个方向都可以拖拽
            dragFlags = ItemTouchHelper.UP | ItemTouchHelper.LEFT | ItemTouchHelper.DOWN | ItemTouchHelper.RIGHT;
        }
        //滑动方向 这里向左侧滑
        int swipeFlags = ItemTouchHelper.LEFT;
        //
        return makeMovementFlags(dragFlags, swipeFlags);
//        return 0;
    }

    @Override
    public boolean onMove(@NonNull RecyclerView recyclerView,
                          @NonNull RecyclerView.ViewHolder viewHolder,
                          @NonNull RecyclerView.ViewHolder target) {
        //当拖动效果已经产生了，就会回调此方法
        int fromPosition = viewHolder.getAdapterPosition();
        int toPosition = target.getAdapterPosition();
        List<DataItem> dataItemList = mDataItemRVAdapter.getDataItemList();
        //交换数据
        Log.e(TAG, "onMove: fromPosition " + fromPosition + " toPosition " + toPosition);
        //移动过程中，没有放开就会回调，所以会调用很多次
        Collections.swap(dataItemList, fromPosition, toPosition);
      /*不需要这么写  if (fromPosition < toPosition) {
            for (int i = fromPosition; i < toPosition; i++) {
                Collections.swap(dataItemList, i, i + 1);
            }
        } else {
            for (int i = fromPosition; i > toPosition; i--) {
                Collections.swap(dataItemList, i, i - 1);
            }
        }*/
        StringBuffer stringBuffer = new StringBuffer();
        for (DataItem dataItem : dataItemList) {
            stringBuffer.append(dataItem.title);
        }
        Log.e(TAG, "onMove: " + stringBuffer.toString());
        mDataItemRVAdapter.notifyItemMoved(fromPosition, toPosition);

        //返回 true 可以回调 onMoved 方法
//        return true;
        return false;
    }

    @Override
    public void onMoved(@NonNull RecyclerView recyclerView, @NonNull RecyclerView.ViewHolder viewHolder, int fromPos, @NonNull RecyclerView.ViewHolder target, int toPos, int x, int y) {
        super.onMoved(recyclerView, viewHolder, fromPos, target, toPos, x, y);
//        onMove 方法返回 true 才会走这
        Log.e(TAG, "onMoved: ");
    }

    @Override
    public void onSwiped(@NonNull RecyclerView.ViewHolder viewHolder,
                         int direction) {
        //侧滑删除
        int position = viewHolder.getAdapterPosition();
        mDataItemRVAdapter.getDataItemList().remove(position);
        //
        mDataItemRVAdapter.notifyItemRemoved(position);
    }

    @Override
    public void onSelectedChanged(@Nullable RecyclerView.ViewHolder viewHolder, int actionState) {
        super.onSelectedChanged(viewHolder, actionState);
        //
        Log.e(TAG, "onSelectedChanged: " );
        if (viewHolder == null) {
            return;
        }

        int position = viewHolder.getAdapterPosition();
        //
        if (actionState == ItemTouchHelper.ACTION_STATE_IDLE) {
            //
            viewHolder.itemView.setBackgroundColor(Color.RED);
//            mDataItemRVAdapter.getDataItemList().get(position).bgColor = Color.RED;
//            mDataItemRVAdapter.notifyItemChanged(position);
        } else if (actionState == ItemTouchHelper.ACTION_STATE_SWIPE) {
            //开始滑动
            //如果和item默认颜色不一致的话 注意要解决复用后导致的错乱
            viewHolder.itemView.setBackgroundColor(Color.GREEN);
//            mDataItemRVAdapter.getDataItemList().get(position).bgColor = Color.GREEN;
//            mDataItemRVAdapter.notifyItemChanged(position);
        } else if (actionState == ItemTouchHelper.ACTION_STATE_DRAG) {
            //开始拖拽
            //如果和item默认颜色不一致的话 注意要解决复用后导致的错乱
            viewHolder.itemView.setBackgroundColor(Color.BLUE);
//            mDataItemRVAdapter.getDataItemList().get(position).bgColor = Color.BLUE;
//            mDataItemRVAdapter.notifyItemChanged(position);
//            viewHolder.itemView.setTranslationZ(10);

        }
    }

    @Override
    public void clearView(@NonNull RecyclerView recyclerView, @NonNull RecyclerView.ViewHolder viewHolder) {
        super.clearView(recyclerView, viewHolder);
        Log.e(TAG, "clearView: " );
        if (viewHolder == null) {
            return;
        }
        //拖拽或滑动结束时调用
        int position = viewHolder.getAdapterPosition();
        //如果和item默认颜色不一致的话 注意要解决复用后导致的错乱
        viewHolder.itemView.setBackgroundColor(Color.GRAY);
//        viewHolder.itemView.setBackgroundColor(Color.WHITE);
//        mDataItemRVAdapter.getDataItemList().get(position).bgColor = Color.GRAY;
//            viewHolder.itemView.setTranslationZ(0);
    }

    @Override
    public boolean isItemViewSwipeEnabled() {

        //默认内部返回 true
        return super.isItemViewSwipeEnabled();
    }

    @Override
    public boolean isLongPressDragEnabled() {

        //默认内部返回 true
        return super.isLongPressDragEnabled();
    }

    @Override
    public float getSwipeThreshold(@NonNull RecyclerView.ViewHolder viewHolder) {
        //是否判定为滑动的闸值
        return super.getSwipeThreshold(viewHolder);
    }

    @Override
    public float getMoveThreshold(@NonNull RecyclerView.ViewHolder viewHolder) {
        //是否判定为拖拽的闸值
        return super.getMoveThreshold(viewHolder);
    }
}
```



使用

```
ItemTouchHelper itemTouchHelper = new ItemTouchHelper(new ItemTouchHelperCallback(mDataItemRVAdapter));
itemTouchHelper.attachToRecyclerView(id_rv);
```





ConcatAdapter（原 MergeAdapter）



## ViewHolder#getLayoutPosition、ViewHolder#getBindingAdapterPosition、ViewHolder#getAbsoluteAdapterPosition

```java
//@Deprecated 本意就是 getLayoutPosition，由于命名太不明确，标记过时了
androidx.recyclerview.widget.RecyclerView.ViewHolder#getPosition
androidx.recyclerview.widget.RecyclerView.ViewHolder#getLayoutPosition
//@Deprecated 本意就是 getBindingAdapterPosition 
androidx.recyclerview.widget.RecyclerView.ViewHolder#getAdapterPosition
androidx.recyclerview.widget.RecyclerView.ViewHolder#getBindingAdapterPosition
//int globalPosition = mOwnerRecyclerView.getAdapterPositionInRecyclerView(this);
androidx.recyclerview.widget.RecyclerView.ViewHolder#getAbsoluteAdapterPosition
```

所以只需要考虑 getLayoutPosition、getBindingAdapterPosition、getAbsoluteAdapterPosition

1 正常情况下 LayoutPosition 和 BindingAdapterPosition (原 AdapterPosition) 的 position 是相等的，当 Adapter 设置完新数据，并且调用了 notifyXXX 方法后，RecyclerView 需要重新布局（不超过16ms），在没有完成新布局之前，LayoutPosition 和 AdapterPosition 可能就会出现不相等的情况 了

2 没有使用 ConcatAdapter，那么 getBindingAdapterPosition 和 getAbsoluteAdapterPosition 方法的效果是一样的，使用了ConcatAdapter，getBindingAdapterPosition 得到的是元素位于当前绑定 Adapter 的位置，而getAbsoluteAdapterPosition 方法得到的是元素位于合并后 Adapter 的全局绝对位置

- getBindingAdapterPosition 返回相对于其所绑定的 Adapter 的位置

- getAbsoluteAdapterPosition 返回相对于整个 RecyclerView 的位置

3 调用的是 notifyDataSetChanged，因为要重新绘制所有 Item，而且在绘制完成之前 RecyclerView 是不知道 AdapterPosition 的，所以此时会返回-1（NO_POSITION），但是调用 notifyItemInserted  这样的 AdapterPosition 会立马变更生效

总结：

大多数情况下用 AdapterPosition 进行获取，最好先用 RecyclerView.NO_POSITION 作判断再使用

侧滑删除时，可考虑选用 LayoutPosition ，肉眼可见的情况，还有 mRecyclerView.findViewHolderForLayoutPosition(myViewHolder.getLayoutPosition() - 1)

另外如果再 onBindViewHolder 处理事件相关，推荐用 AdapterPosition 替换 onBindViewHolder 方法的 position 参数











## 闪屏问题

在rv.setAdapter()之前调用((SimpleItemAnimator)rv.getItemAnimator()).setSupportsChangeAnimations(false)禁用change动画

