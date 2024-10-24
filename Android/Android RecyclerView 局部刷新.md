
# Android RecyclerView 局部刷新

## 使用常规方式刷新数据

### 1 局部刷新之 Item 刷新
- Item 整体的刷新

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

- PS: 点击一个 Item 的删除按钮进行数据删除 notifyItemRemoved ，如果采用事件监听时会传 position 给点击事件接口方法，此时不能直接写 position，因为它其实变成了 final 类型 （注意如果是 java 代码，在 AS 里会显示下划线），这样就造成 pos 错乱了，需要改用 holder.getBindingAdapterPosition (其实这里 holder 变成 final 了) ，这样才能保证点下一个删除按钮时候的 pos 是正确的，notifyItemInserted 也是类似

- 另外还有一种解决方案，就是删除或新增后用 mAdapter.notifyItemRangeChanged(pos, mAdapter.getData().size() - pos) 强行处理一波，及时更新 pos ，让 pos 能够保持正确

  

### 2 常规-局部刷新之 View 刷新
- Item 的部分 View 控件的刷新

#### 通过带 payload 参数的刷新方法
- 自行写逻辑操作 View 实现刷新

```java
//修改指定 position 上 Item 局部 View 的刷新
mAdapter.notifyItemChanged(int position,Object payload)
//修改从 positionStart 开始到 itemCount 数量的 Item 局部 View 的刷新
mAdapter.notifyItemRangeChanged(int positionStart, int itemCount,Object payload)
```

- 假设 payload 只传 【key 标记】的话，那么在执行刷新前就需要保证数据源先完成更新，然后这样 Adapter 才直接用数据源实现刷新
- 另外 payload 也可以传 【key-value 键值对】，比如 Map，那么 Adapter 可以不用数据源而改用 value 进行刷新

#### 修改 DataItemRVAdapter

- 覆写带 Payload 参数的 onBindViewHolder 方法

```java
 @Override
    public void onBindViewHolder(@NonNull DataItemViewHolder holder, int position, @NonNull List<Object> payloads) {
        if (payloads.isEmpty()) {
            //【列表刷新】 或者 【局部刷新之 Item 刷新】
            super.onBindViewHolder(holder, position, payloads);
        } else {
            //不为空【局部刷新之 View 刷新】
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
        //自行写逻辑操作 View 实现刷新
        if (key.equals(KEY_TITLE)) {
            holder.tv_title.setText(value != null ? value : dataItem.title);
        } else if (key.equals(KEY_SUBTITLE)) {
            holder.tv_subtitle.setText(value != null ? value : dataItem.content);
        } else if (key.equals(KEY_CONTENT)) {
            holder.tv_content.setText(value != null ? value : dataItem.content);
        } else if (key.equals(KEY_IMAGE)) {
            holder.iv.setImageResource(value != null ? Integer.parseInt(value) : dataItem.imageResId);
        }
    }

```


## 使用 DiffUtil 方式刷新数据
- 通过 DiffUtil 计算得出新旧数据集的最小变化量
- 最终内部还是会调用 Adapter#notifyItemXXX 方法去实现刷新的，所以也能够保留 Item 的动画
- DiffUtil#calculateDiff 推荐在子线程执行


### 1 DiffUtil-局部刷新之 Item 刷新
- Item 整体的刷新

#### 创建 DiffUtilCallback 继承 DiffUtil.Callback

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
        //areItemsTheSame 返回 true 的情况下会走这个方法，进一步判断这个 item 的内容是否有变化
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

        //默认返回 true 代表内容相等
        return true;
    }
}
```

#### 使用
- 通过 calculateDiff 配合 dispatchUpdatesTo

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
        //2 先更新 Adapter 数据源
        mDataItemRVAdapter.setDataItemList(mDataItemList);
        //3 然后通过 DiffUtil 的方式通知刷新，推荐在子线程中计算
        DiffUtil.DiffResult diffResult = DiffUtil.calculateDiff(new DiffCallback(oldDataItemList, mDataItemList));
        //内部会按不同情况自动调用 Adapter#notifyItemXXX 方法去实现刷新
        diffResult.dispatchUpdatesTo(mDataItemRVAdapter);
}
```
 


### 2 DiffUtil-局部刷新之 View 刷新
- Item 的部分 View 控件的刷新
- 同常规方式一样 Adapter 需要覆写带 Payload 参数的 onBindViewHolder 方法


#### 修改 DiffUtilCallback 类
- 继续覆写 getChangePayload 方法

```java
    @Nullable
    @Override
    public Object getChangePayload(int oldItemPosition, int newItemPosition) {
        DataItem oldDataItem = mOldDataItemList.get(oldItemPosition);
        DataItem newDataItem = mNewDataItemList.get(newItemPosition);
        // areItemsTheSame 返回 true 并且 areContentsTheSame 返回 false 的情况才会进来
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

#### 使用

- 同上


