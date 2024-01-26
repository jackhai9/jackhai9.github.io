title: Android之ListView的性能提升
date: 2014-11-21 15:10:14
categories: 技术 
tags: [ListView]
---
## ListView如何运作的？
ListView是设计应用于对**可扩展性和高性能要求**的地方。实际上，这就意味着ListView有以下2个要求：

* 1、尽可能少的创建View；
* 2、只是绘制和布局在屏幕上可见的子View。

<!--more--> 
理解第一点很简单：通过布局xml文件创建View并显示是很耗时耗资源的操作。尽管布局文件已经编译打包成了二进制形式以便于更高效的语法解析，但是创建View仍然需要通过一个特殊的XML树，并实例化所有需要响应的View。

ListView通过回收一些不可见的Views，通常在Android源码中称为“ScrapView(废弃的View)”来解决这个问题。这意味着开发者只需要简单的更新每行的内容而不需要针对每个单独的行的布局来创建View。

为了实现第二点，在我们滑动屏幕时，ListView通过使用View回收器来增加低于或者高于当前窗口的Views，并把当前活动的Views移动到一个可回收池中。这样的话，ListView只需要在内存中保持足够多的Views去填充分配空间中的布局和一些额外的可回收Views，即使当你的Adapter有上百个items的时候，它会使用不同的方法去填充行之间的空间，从顶部或者底部等等，具体取决于窗口是如何变化的。

下面这个图很直观的展示了当你按下ListView的时候发生了什么：
![ListView](http://blogresource.qiniudn.com/2014/Android/listview_tips.png)

通过上述介绍，我们已经熟悉了ListView的这种机制，让我们继续前往技巧部分。正如上述介绍的，在滑动时，ListView通过动态的创建和回收很多View，实现了尽可能地让Adapter的getView()轻量。所有的技巧都是通过多种方法让getView()更快。

## View的回收

当ListView每次需要在屏幕上显示新的一行的时候，会从其Adapter中调用getView()方法。众所周知，getView()方法有3个参数：行的位置， convertView以及父ViewGroup。

参数convertView说穿来就是之前讲述的ScrapView。当ListView更新一行布局时，convertView是一个非空值。因此，当convertView值非空时，你仅仅需要更新内容即可，而不需要重新inflate一个新行的布局。getView()在Adapter中一般是如下的形式：
```java
public View getView(int position, View convertView, ViewGroup parent) {
	    if (convertView == null) {
	        convertView = mInflater.inflate(R.layout.your_layout, null);
	    }
	
	    TextView text = (TextView) convertView.findViewById(R.id.text);
	    text.setText("Position " + position);
	
	    return convertView;
	}
```

## View Holder的使用

Android很常见的一个操作就是在布局文件中找到一个内部的View。通常是使用View的findViewById()方法来实现。这个findViewById()方法在View树中，根据一个View ID，会递归的被调用来找到其子树。虽然在静态UI布局中使用findViewById()是完全正常的。但是，在滑动时，ListView调用其Adapter中的getView()是非常频繁的。findViewById()可能会影响ListView滑动时的性能，尤其是你的行布局是很复杂的时候。

View Holder的模式就是减少在Adapter中getView()方法中调用findViewById()的次数。实际上，View Holder是一个轻量级的内部类，用于直接引用所有内部views。在创建View之后，你可以在每行的View存储一个标签。通过这种方法，只需要在初次创建布局的时候调用findViewById()。下面是一个使用上述方法的View Holder模板的代码示例：
```java
	public View getView(int position, View convertView, ViewGroup parent) {
	    ViewHolder holder;
	
	    if (convertView == null) {
	        convertView = mInflater.inflate(R.layout.your_layout, null);
	
	        holder = new ViewHolder();
	        holder.text = (TextView) convertView.findViewById(R.id.text);
	
	        convertView.setTag(holder);
	    } else {
	        holder = convertView.getTag();
	    }
	
	    holder.text.setText("Position " + position);
	
	    return convertView;
	}

	private static class ViewHolder {
	    public TextView text;
	}
```
## 异步加载
很多时候，Android应用在ListView每行中显示一些多媒体内容，比如图片等。在Adapter中的getView()使用应用内置的图片资源还是不会出什么问题的，因为可以存储在Android的高速缓存中。但当你想动态的显示来自本地磁盘或网络的内容时（例如缩略图，简历图片等）你可能不希望直接在Adapter中的getView()加载它们，因为**IO进程会阻塞UI线程**。如果这样做的话，ListView就看起来非常卡顿。

在一个单独的线程，如果想要运行的所有行的IO操作实现异步的话，技巧就是要做到符合ListView的回收行为。例如，如果在Adapter中的getView()中，使用AsyncTask方式去加载资料图片，在AsyncTask完成之前，你正在加载的图片View就有可能被回收用于其他地方。所以，一旦异步操作完成的同时，需要一种机制来知道相应的View有没有被回收。

一个简单的方法来实现这一目标是通过附加一些标识，来标示该行与它相关的View的信息。然后，当异步操作完成的时候，检查目标行的View和标识的View是否一致。实现这一目标的方法很多。下面是实现这种方法的一个很简单的示例：
```java
	public View getView(int position, View convertView,
	        ViewGroup parent) {
	    ViewHolder holder;
	
	    ...
	
	    holder.position = position;
	
	    new ThumbnailTask(position, holder)
	            .executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR, null);
	
	    return convertView;
	}

	private static class ThumbnailTask extends AsyncTask {
	    private int mPosition;
	    private ViewHolder mHolder;
	
	    public ThumbnailTask(int position, ViewHolder holder) {
	        mPosition = position;
	        mHolder = holder;
	    }
	
	    @Override
	    protected Cursor doInBackground(Void... arg0) {
	        // Download bitmap here
	    }
	
	    @Override
	    protected void onPostExecute(Bitmap bitmap) {
	        if (mHolder.position == mPosition) {
	            mHolder.thumbnail.setImageBitmap(bitmap);
	        }
	    }
	}

	private static class ViewHolder {
	    public ImageView thumbnail;
	    public int position;
	}
```
## 人机交互知识
做到在每一行异步加载很多资源，是一个高性能的ListView的必经之路。但是，在滑动屏幕时，如果你一味的在每一个getView()调用里面都去启动一个异步的操作，造成的结果就是你会浪费大量资源。因为行被频繁回收，造成大部分返回的结果会被丢弃。

**考虑到实际的人机交互情况，在ListView适配器中，在每一行中都不应该去触发任何异步操作。也就是说，在ListView中有fling(快速滑动)操作时，启动任何异步操作都没有任何意义。一旦滚动停止或即将停止，才是开始真正显示每行的内容的时候。**

Romain Guy写了一个很经典的应用：[Shelves app](http://code.google.com/p/shelves/)，其中有一个很好的的[示例](https://code.google.com/p/shelves/source/browse/trunk/Shelves/src/org/curiouscreature/android/shelves/activity/ShelvesActivity.java)。当GridView停止滑动时不做其他事情时，它就开始触发从而去异步加载书的封面资源。即使在滑动时，你也可以展示缓存中的内容，通过使用memory cache来平衡交互。这真是个好主意！

## 以上

强烈推荐你看下Romain Guy和Adam Powell的关于ListView的[讨论](https://www.youtube.com/watch?v=wDBM6wVEO70)，里面涵盖了很多这篇文章的东西。你可以看看[Pattrn](https://play.google.com/store/apps/details?id=org.lucasr.pattrn)这个app，可以看到这里面的几个技巧是如何在应用中运用的。

希望它是你在Android开发中一个很有用的参考:-)

