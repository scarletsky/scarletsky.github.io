---
title: RecyclerView 基本用法
date: 2015-06-29 20:55:00
categories: [android]
tags: [android, recyclerview]
---

#基本用法

- 在 XML 中添加 `<android.support.v7.widget.RecyclerView/>`

- 编写继承 `RecyclerView.Adapter<RecyclerViewAdapter.ViewHolder>` 的 `Adapter`
    - `Constructor`
    - 编写继承 `RecyclerView.ViewHolder` 的 `ViewHolder`
    - `onCreateViewHolder`
    - `onBindViewHolder`
    - `getItemCount`

- 设置 `RecyclerView`
    - `setLayoutManager`
    - `setAdapter`
    - `setItemAnimator`(可选)
    - `addItemDecoration`(可选)

#XML
新建 xml，添加 `RecyclerView`。

```
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:fab="http://schemas.android.com/apk/res-auto"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <android.support.v7.widget.RecyclerView
        android:id="@+id/my_recyclerview"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

</RelativeLayout>

```

#ViewHolder

我们以前使用 `ListView` 的时候，会用以下的方式来提高 `ListView` 的性能:

- 用 `convertView` 来减少 `LayoutInflater.inflate` 的使用
- 用 `ViewHolder` 来减少 `findViewById` 的使用

`RecyclerView` 标准化了 `ViewHolder` 来缓存昂贵 `findViewById` 的结果。

例子：

```
public static MyViewHolder extends RecyclerView.ViewHolder {

    public TextView mTitle;
    public TextView mSubtitle;

    public MyViewHolder (View v) {
        super(v);
        mTitle = (TextView) v.findViewById(R.id.title);
        mSubtitle = (TextView) v.findViewById(R.id.subtitle);
    }
}

```
PS: 一般 `ViewHolder` 会编写在 `Adapter` 的内部。


#Adapter

我们需要继承 `RecyclerView.Adapter<RecyclerView.ViewHolder>` 来编写我们自己的 `Adapter` 。编写我们自己的 `Adapter` 的时候，最重要的是要重写 `onCreateViewHolder` 和 `onBindViewHolder` 方法。

```
public class MyAdapter extends RecyclerView.Adapter<RecyclerView.ViewHolder> {

    @Override
    public RecyclerView.ViewHolder onCreateViewHolder (ViewGroup parent, int viewType) {
        View v = LayoutInflater.from(ctx).inflate(R.layout.my_adapter, parent, false);
        return new ViewHolder(v);
    }

    @Override
    public void onBindViewHolder (RecyclerView.ViewHolder h, int position) {
        MyViewHolder holder = (MyViewHolder) h;
        holder.mTitle.setText("This is my title");
        holder.mSubtitle.setText("This is my subtitle");
    }

    @Override
    public int getItemCount() {
        return data.size();
    }
}
```

注意，在这个例子中我们的 `MyAdapter` 存放的是 `<RecyclerView.ViewHolder>`，所以在 `onCreateViewHolder` 中返回的也必须是 `<RecyclerView.ViewHolder>`，而在 `onBindViewHolder` 的回调中，我们拿到的也是 `RecyclerView.ViewHolder`，要把它强转成我们之前编写的 `MyViewHolder` 之后才能正常使用。


#RecyclerView
最后只要设置一下 `RecyclerView` 就能使用了。

```
RecyclerView mRecyclerView = (RecyclerView) findViewById(R.id.my_recyclerview);
mRecyclerView.setLayoutManager(new LinearLayoutManager(this));
mRecyclerView.setAdapter(adapter);
```


#参考资料
[RecyclerView全攻略](http://wobushi.ren/recyclerview.html)
[Android RecyclerView 使用完全解析 体验艺术般的控件](http://blog.csdn.net/lmj623565791/article/details/45059587)
