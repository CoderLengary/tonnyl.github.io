---
layout: post # needs to be post
title: 实现RecyclerView Item的滑动删除
summary: 利用ItemTouchHelper实现recycler view item的侧滑删除
featured-img: shane-rounce-205187
categories: [Android]
---

## 关于ItemTouchHelper
官方文档的解释：

> This is a utility class to add swipe to dismiss and drag & drop support to RecyclerView.

ItemTouchHelper是一个用于在RecyclerView中实现滑动删除和拖拽的工具类。

## 使用
+ 修改build.gradle文件，添加依赖。

	```gradle
	compile 'com.android.support:recyclerview-v7:23.4.0'
	```

+ 构建object与Adapter

	这里假设我已经构建好了实体类CodeLanguage和adapter，在构建adapter时，需要添加一个方法。
	
	```java
	public void remove(int position) {
	   mMovies.remove(position);
	   notifyItemRemoved(position);
	}
	```

+ 创建ItemTouchHelper.SimpleCallback子类

	为了处理拖动和滑动事件，需要创建ItemTouchHelper.SimpleCallback的实现类。这里，只对滑动事件感兴趣，这是我们的callback.
	
	```java
	public class CodeLanguageItemTouchHelper extens ItemTouchHelper.SimpleCallback{
	    private CodeLanguageAdapter adapter;
	    
	    public CodeLanguageItemTouchHelper(CodeLanguageAdapter adapter){
	      super(ItemTouchHelper.UP | ItemTouchHelper.DOWN, ItemTouchHelper.LEFT | ItemTouchHelper.RIGHT);
	      this.adapter = adapter;
	   }
	    /**
	    * If you don't support drag & drop, this method will never be called.
	    * 如果不支持拖拽，那么这个方法就不会被执行。
	    * @param recyclerView The RecyclerView to which ItemTouchHelper is attached to. ItemTouchHelper需要附加到的RecyclerView
	    * @param viewHolder The ViewHolder which is being dragged by the user. 拖动的ViewHolder
	    * @param target The ViewHolder over which the currently active item is being dragged. 目标位置的ViewHolder
	    * @return True if the viewHolder has been moved to the adapter position of target. viewHolder是否被移动到目标位置
	    */
	   
		@Override
	   public boolean onMove(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder, RecyclerView.ViewHolder target) {
	      return false;  
	   }
	    /**
	    * Called when a ViewHolder is swiped by the user.
	    * If you don't support swiping, this method will never be called.
	    * 如果不支持滑动，方法不会被执行。
	    * @param viewHolder The ViewHolder which has been swiped by the user.
	    * @param direction  The direction to which the ViewHolder is swiped.
	    *                    It is one of UP, DOWN, LEFT or RIGHT.
	    *                    If your getMovementFlags(RecyclerView, ViewHolder) method returned relative flags instead of LEFT / RIGHT;
	    *                    `direction` will be relative as well. (START or END).
	    */
	   
		@Override
	   public void onSwiped(RecyclerView.ViewHolder viewHolder, int direction) {
	      //Remove item
	      adapter.remove(viewHolder.getAdapterPosition());
	   }
	   
	}
	```
	
	CodeLanguageItemTouchHelper默认的构造方法需要传入两个参数。
	
	```java
	/**
	* Creates a Callback for the given drag and swipe allowance.
	* @param dragDirs 表示拖拽的方向，有六个类型的值：LEFT、RIGHT、START、END、UP、DOWN
	* @param swipeDirs 表示滑动的方向，有六个类型的值：LEFT、RIGHT、START、END、UP、DOWN
	*/
	ItemTouchHelper.SimpleCallback(int dragDirs, int swipeDirs)
	```
	
	CodeLanguageItemTouchHelper默认需要实现两个方法onMove(),onSwiped()，onMove()是对拖拽的实现，onSwiped()是对滑动的实现。
	
+ 将ItemTouchHelper添加至RecyclerView
	
	创建好自己的ItemTouchHelper类后，将它附加到RecyclerView就很简单了。在Activity或者Fragment的onCreate()方法中：
	
	```java
	ItemTouchHelper.Callback callback = new MovieTouchHelper(adapter);
ItemTouchHelper helper = new ItemTouchHelper(callback);
helper.attachToRecyclerView(codeLanguageRecyclerView);
	```

现在所有的工作已经完成。我们现在还没有添加动画，默认的动画系统已经添加。现在就可以使用了。