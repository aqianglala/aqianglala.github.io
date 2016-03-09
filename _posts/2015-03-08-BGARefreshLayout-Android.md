---
layout: post
date:   2015-03-08 22:21:49
title:   BGARefreshLayout-Android  
categories: Android
tags: Android BGARefreshLayout-Android  
---


---
github地址：[https://github.com/bingoogolapple/BGARefreshLayout-Android][1]

**开发者使用BGARefreshLayout-Android可以对各种控件实现多种<font color="red">下拉刷新效果、上拉加载更多以及配置自定义头部广告位</font>**

###**注意：<font color="red">内容控件的高度请使用android:layout_height="0dp"和android:layout_weight="1"</font>**

**目前已经实现了四种下拉刷新效果:**



- 新浪微博下拉刷新风格（可设置各种状态是的文本，可设置整个刷新头部的背景）
- 慕课网下拉刷新风格（可设置其中的logo和颜色成自己公司的风格，可设置整个刷新头部的背景）
- 美团下拉刷新风格（可设置其中的图片和动画成自己公司的风格，可设置整个刷新头部的背景）
- 类似qq好友列表黏性下拉刷新风格（三阶贝塞尔曲线没怎么调好，刚开始下拉时效果不太好，可设置整个刷新头部的背景）


**一种上拉加载更多效果**


- 新浪微博上拉加载更多（可设置背景、状态文本）

**效果图**
<div style="clear: both;"></div>
<img src="https://camo.githubusercontent.com/912ee9a45b5ed7063bd6fe7634f8130953a7051d/687474703a2f2f37786b39646a2e636f6d312e7a302e676c622e636c6f7564646e2e636f6d2f726566726573686c61796f75742f73637265656e73686f74732f6267615f726566726573686c61796f7574312e676966" style="width:50%;float:left;"/>
<img src="https://camo.githubusercontent.com/7539fed2c320aecc0d47320586e8c2ee22a2d762/687474703a2f2f37786b39646a2e636f6d312e7a302e676c622e636c6f7564646e2e636f6d2f726566726573686c61796f75742f73637265656e73686f74732f6267615f726566726573686c61796f7574322e676966" style="width:50%;float:left; "/>
<img src="https://camo.githubusercontent.com/eb127f11ecfee22f87933e4b924b3860568b2b1e/687474703a2f2f37786b39646a2e636f6d312e7a302e676c622e636c6f7564646e2e636f6d2f726566726573686c61796f75742f73637265656e73686f74732f6267615f726566726573686c61796f7574332e676966" style="width:50%;float:left; "/>
<img src="https://camo.githubusercontent.com/1f1333545090daef157c53196f7524a7851bdf35/687474703a2f2f37786b39646a2e636f6d312e7a302e676c622e636c6f7564646e2e636f6d2f726566726573686c61796f75742f73637265656e73686f74732f6267615f726566726573686c61796f7574342e676966" style="width:50%;float:left; "/>
<div style="clear: both;"></div>

<!-- ![此处输入图片的描述][2]    ![此处输入图片的描述][3]
![此处输入图片的描述][4]       ![此处输入图片的描述][5] -->

**基本使用（针对RecyclerView，都一样，只是这里使用了这个库自带的通用适配器）**

**1. 添加Gradle依赖**
{% highlight xml %}
compile 'com.android.support:appcompat-v7:23.1.1'
compile 'com.nineoldandroids:library:2.4.0'
compile 'cn.bingoogolapple:bga-refreshlayout:1.1.3@aar'
{% endhighlight %}

**2. 在布局文件中添加BGARefreshLayout**

<font color="red">注意：内容控件的高度请使用android:layout_height="0dp"和android:layout_weight="1"</font>

我之前就是因为没注意到这个，导致上拉时footView没出现，搞了快两天才发现。。。

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>

<cn.bingoogolapple.refreshlayout.BGARefreshLayout
    android:id="@+id/BGARefreshLayout"
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingLeft="10dp"
    android:paddingRight="10dp">

    <android.support.v7.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:overScrollMode="never"
        />
</cn.bingoogolapple.refreshlayout.BGARefreshLayout>

{% endhighlight %}

**3.在Activity或者Fragment中配置BGARefreshLayout**

{% highlight java %}
public class TeacherFragment extends Fragment implements BGARefreshLayout.BGARefreshLayoutDelegate {
    private TeacherRecyclerViewAdapter mAdapter;
    private BGARefreshLayout mRefreshLayout;
    private RecyclerView mDataRv;

    private TeacherFragmentViewmodel mViewmodel;

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable
    Bundle savedInstanceState) {
        //这里使用了MVVM架构来写的，把逻辑操作都放到viewModel来处理了
        mViewmodel = new TeacherFragmentViewmodel(getActivity(),this);
        
        //初始化view
        View rootView = inflater.inflate(R.layout.fragment_teacher1, container, false);
        mRefreshLayout = (BGARefreshLayout) rootView.findViewById(R.id.rl_recyclerview_refresh);
        mDataRv = (RecyclerView) rootView.findViewById(R.id.rv_recyclerview_data);
        
        //设置监听
        mRefreshLayout.setDelegate(this);
        
        //创建适配器
        mAdapter = new TeacherRecyclerViewAdapter(mDataRv);

        //这里使用的是粘性下拉刷新风格，true表示开启上拉加载更多
        BGAStickinessRefreshViewHolder stickinessRefreshViewHolder = new
                BGAStickinessRefreshViewHolder(getActivity(), true);
        //粘性的颜色
        stickinessRefreshViewHolder.setStickinessColor(R.color.colorPrimary);
        //设置图标
        stickinessRefreshViewHolder.setRotateImage(R.mipmap.talkpal_logo);
        mRefreshLayout.setRefreshViewHolder(stickinessRefreshViewHolder);

        
        mDataRv.setLayoutManager(new LinearLayoutManager(getActivity()));
        mDataRv.setAdapter(mAdapter);
        
        //开始下拉刷新
         mRefreshLayout.beginRefreshing();
        return rootView;
    }

    public boolean isRefresh;
    public void setAdapter(List<TeacherBean.DataEntity> data) {
        if(isRefresh){
            //将数据添加在开头，形成一种下拉加载更多的效果
            //mAdapter.addNewDatas(data);
            mAdapter.clear();
            mAdapter.addMoreDatas(data);
        }else{
            mAdapter.addMoreDatas(data);
        }

        stopWait();
    }

    public void stopWait(){
        mRefreshLayout.endRefreshing();
        mRefreshLayout.endLoadingMore();
    }


    @Override
    public void onBGARefreshLayoutBeginRefreshing(BGARefreshLayout refreshLayout) {
        isRefresh=true;
        //加载数据
        mViewmodel.setPage(1);
        mViewmodel.loadData();
    }

    @Override
    public boolean onBGARefreshLayoutBeginLoadingMore(BGARefreshLayout refreshLayout) {
        isRefresh=false;
        mViewmodel.setPage(mViewmodel.getPage()+1);
        mViewmodel.loadData();
        // 返回true则展示footerView
        return true;
    }

}
{% endhighlight %}

**4.BGAAdapter-Android**

**特点**

- 在AdapterView和RecyclerView中通用的Adapter和ViewHolder，使AdapterView和RecyclerView适配器的使用方式基本一致。
- 使用这个适配器还能自动为列表项添加点击时的水波纹效果，炫酷！！
- 当然，recyclerView的点击事件也帮我们做了，只需要在activity中实现BGAOnRVItemClickListener即可
- 为item的孩子节点设置监听器，首先在在适配器中设置点击监听
    {% highlight java %}
    viewHolderHelper.setItemChildClickListener(R.id.tv_item_normal_delete);
    {% endhighlight %}
    然后在Activity中实现BGAOnItemChildClickListener, BGAOnItemChildLongClickListener即可


    **1. 添加Gradle依赖**
    
    {% highlight java %}
     compile 'com.android.support:appcompat-v7:23.1.1'
     compile 'cn.bingoogolapple:bga-adapter:1.0.8@aar'
    {% endhighlight %}
    
    **2. 代码**
    
    {% highlight java %}
    public class TeacherRecyclerViewAdapter extends BGARecyclerViewAdapter<TeacherBean.DataEntity> {
    public TeacherRecyclerViewAdapter(RecyclerView recyclerView) {
        super(recyclerView, R.layout.item_teacher_list);
    }
    
    /**
     * 为item的孩子节点设置监听器，并不是每一个数据列表都要为item的子控件添加事件监听器，所以在父类中采用了空实现，需要设置事件监听器时重写该方法即可
     *
     * @param viewHolderHelper
     */
    @Override
    public void setItemChildListener(BGAViewHolderHelper viewHolderHelper) {
    }

    @Override
    public void fillData(BGAViewHolderHelper viewHolderHelper, int position, TeacherBean.DataEntity model) {
        //设置文字
        viewHolderHelper.setText(R.id.tv_from, model.getFrom()).setText(R.id.tv_languages, model
                .getLanguages());
        //设置图片,facebook的加载图片框架
        String src = model.getGallery().get(0).getSrc();
        Uri uri = Uri.parse(src);
        SimpleDraweeView draweeView = viewHolderHelper.getView(R.id.iv_teacher);
        draweeView.setImageURI(uri);
    }
}
    {% endhighlight %}

就是这么简单~~
    
    
    
    

  [1]: https://github.com/bingoogolapple/BGARefreshLayout-Android
  [2]: https://camo.githubusercontent.com/912ee9a45b5ed7063bd6fe7634f8130953a7051d/687474703a2f2f37786b39646a2e636f6d312e7a302e676c622e636c6f7564646e2e636f6d2f726566726573686c61796f75742f73637265656e73686f74732f6267615f726566726573686c61796f7574312e676966
  [3]: https://camo.githubusercontent.com/7539fed2c320aecc0d47320586e8c2ee22a2d762/687474703a2f2f37786b39646a2e636f6d312e7a302e676c622e636c6f7564646e2e636f6d2f726566726573686c61796f75742f73637265656e73686f74732f6267615f726566726573686c61796f7574322e676966
  [4]: https://camo.githubusercontent.com/eb127f11ecfee22f87933e4b924b3860568b2b1e/687474703a2f2f37786b39646a2e636f6d312e7a302e676c622e636c6f7564646e2e636f6d2f726566726573686c61796f75742f73637265656e73686f74732f6267615f726566726573686c61796f7574332e676966
  [5]: https://camo.githubusercontent.com/1f1333545090daef157c53196f7524a7851bdf35/687474703a2f2f37786b39646a2e636f6d312e7a302e676c622e636c6f7564646e2e636f6d2f726566726573686c61796f75742f73637265656e73686f74732f6267615f726566726573686c61796f7574342e676966