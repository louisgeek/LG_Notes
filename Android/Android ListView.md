

## ListView 简介

- 继承自 AbsListView ，它继承自 AdapterView<ListAdapter>，它又继承自 ViewGroup





## ListView 特点

- 自带头尾视图、空视图和 Item 点击事件
- 只支持纵向布局
- 分割线 android:divider
- 不强制要求实现 ViewHolder 机制，需要自行实现
- 没有 Item 动画效果
- 未实现局部刷新，需要自行实现





## Adapter 适配器

- ListView 和数据源之间的桥梁





## 实现局部刷新

```java
public void updateItemView(ListView listView, int position, DataItem dataItem){
    int firstPos = listView.getFirstVisiblePosition();
    int lastPos = listView.getLastVisiblePosition();
    //只要考虑可见的，不可见仍旧由 getView 负责
    if( position >= firstPos && position <= lastPos ){  
        //获得当前可见的 item的 view
        View view = listView.getChildAt(position - firstPos);
        ViewHolder viewHolder = (VH)view.getTag();
        viewHolder.id_tv_title.setText(dataItem.title);
    }
}
```





## 实现 ViewHolder

- 主要是用于缓存 findViewById 的 View

```java
	@Override
    public int getViewTypeCount() {
        //super 默认返回 1
        return super.getViewTypeCount();
    }

    @Override
    public int getItemViewType(int position) {
        //super 默认返回 0
        return super.getItemViewType(position);
    }

	@Override
    public View getView(final int position, View convertView, ViewGroup parent) {
        ViewHolder viewHolder;
        if (convertView == null) {
            convertView = LayoutInflater.from(parent.getContext()).inflate(R.layout.item_list_work, parent, false);
            viewHolder = new ViewHolder();
            //查找 view
            viewHolder.id_tv_title = convertView.findViewById(R.id.id_tv_title);
            viewHolder.id_tv_subtitle = convertView.findViewById(R.id.id_tv_subtitle);
            viewHolder.id_tv_work = convertView.findViewById(R.id.id_tv_work);
            //
            convertView.setTag(viewHolder);
        } else {
            viewHolder = (ViewHolder) convertView.getTag();
        }
        Work work = mWorkList.get(position);
        viewHolder.id_tv_title.setText(work.bed);
        viewHolder.id_tv_subtitle.setText(work.patientName);
        viewHolder.id_tv_work.setText(work.workTitle);
        //
        return convertView;
    }
    //
    private static class ViewHolder {
        private TextView id_tv_title;
        private TextView id_tv_subtitle;
        private TextView id_tv_work;
    }
```















 