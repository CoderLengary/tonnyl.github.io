---
layout: post # needs to be post
title: Java中的单例模式
summary: 什么是单例模式？
featured-img: emile-perron-190221
categories: [Java]
---

## 什么是单例模式？

<!--more-->

单例模式是一种对象创建模式，用于产生对象的一个具体实例，并且可以确保系统中一个类只能产生唯一的一个实例。

这样的方式能够带来两大好处：

+ 对于频繁使用的对象，可以省略创建对象所花费的时间，对于复制的对象，这样的创建方式的开销是十分值得的;
+ new 的次数少了，对内存的使用频率也就下降了，当然也就减轻了 GC 的压力。

## 设计思想
这种模式要解决的问题就是要保证应用中只存在一个对象。那怎样实现呢？

1. 不允许其他程序new对象
2. 在该类中创建对象
3. 对外需要暴露一个方法，让其他获取这个对象

体现到代码中，解决办法为：

1. 私有化构造函数
2. 在本类中创建一个本类对象
3. 定义一个方法，能够让其他的类获取这个对象

## Code
1. 最简单的写法为

	```java
	public class Singleton {
	
	    private Singleton(){
	    }
	    
	    private static Singleton instance = new Singleton();
	    
	    public static Singleton getInstance(){
	        return instance;
	    }
	}
	```
	
	这种方式称为饿汉式。
	
	+ 优点
		- 简单明了
		- 在类加载时即完成了实例化，避免了线程同步的问题
	+ 缺点
		- 没有使用懒加载
		- 可能造成内存浪费

2. 变种方式
	
	```java
	public class Singleton {
	
	    private static Singleton instance;
	    
	    private Singleton(){
	    }
	    
	    static {
	        instance = new Singleton();
	    }
	    
	    public static Singleton getInstance(){
	        return instance;
	    }
	}
	```	
	
	相比于第一种写法，本类中将实例化对象的代码放到了 static 代码块中。两种方法的效果其实是一样的。所以优缺点也是一样的。

3. 懒汉式(线程不安全)
	
	```java
	public class Singleton {  
	
	    private static Singleton instance;    
	    
	    private Singleton (){
	    }    
	    
	    public static Singleton getInstance() {    
		    if (instance == null) {        
		        instance = new Singleton();    
		    }
	       return instance;
	    }
	}
	```
	
	+ 优点
		- 使用了懒加载
	+ 缺点
		- 当有多个线程并行调用 `getInstance()` 的时候，就会创建多个实例。也就是说在多线程下不能正常工作。

4. 懒汉式(线程安全)
	
	```java
	public class Singleton { 
	 
	    private static Singleton instance;    
	    
	    private Singleton (){
	    }    
	    
	    public static synchronized Singleton getInstance(){    
	      if (instance == null) {
	           instance = new Singleton();    
	      }
	      return instance;
	    }
	}
	```
	
	+ 优点
		- 线程安全，解决了多实例问题
	+ 缺点
		- 不高效。因为在任何时候只能有一个线程调用 getInstance() 方 法。但是同步操作只需要在第一次调用时才被需要，即第一次创建单例实例对象时。

5. 双重校验锁
	
	```java
	public class Singleton {
	
	    private static Singleton instance = null;
	    
	    private Singleton(){
	    }
	    
	    public static Singleton getInstance(){
	        if (instance == null){
	            synchronized (Singleton.class){
	                if (instance == null){
	                    instance = new Singleton();
	                }
	            }
	        }
	        return instance;
	    }
	}
	```
	
	进行两次 `if(instance == null)` (Double-Check)的检查可以保证线程安全。
	
	+ 优点
		- 并发度高，提升了性能。
	+ 缺点
		- 并不是绝对安全
	
6. 内部类
	
	```java
	public class Singleton{  
	
	    private Singleton() {
	    }
	    
	    private static class SingletonHolder{  
	        private static Singleton instance=new Singleton();  
	    }   
	    
	    public static Singleton getInstance(){  
	        return SingletonHolder.instance;  
	    }
	}
	```
	
	这是Google公司的工程师Bob Lee写的新的懒汉单例模式。
	
	+ 优点
		- 使用JVM本身机制保证了线程安全问题
		- 由于 SingletonHolder 是私有的，除了 getInstance() 之外没有办法访问它，因此它是懒汉式的。
	- 同时读取实例的时候不会进行同步，没有性能缺陷。
	- 不依赖jdk版本。

	这种方式也是《Effective Java》推荐的写法。

7. 枚举
	
	```java
	public enum EasySingleton{
		INSTANCE;
	}
	```
	
	+ 优点
		- 简单明了
		- 调用简单，使用EasySingleton.INSTANCE访问实例，比getInstance()简单
		- 创建枚举本身就是线程安全的，不用担心存在多个实例的问题
	+ 缺点
		- 枚举类型enum在jdk1.5时才引入，所以这种方法并不适用于jdk1.5之前的版本

## 测试
使用多线程并发测试。

```java
public class SingletonPatternTest {

    public static void main(String[] args){
        CountDownLatch latch = new CountDownLatch(1);
        int threadCount = 1000;
        for (int i = 0; i < threadCount; i++){
            new Thread(){
                
				@Override
                public void run() {
                    try {
                        // all thread to wait
                        latch.await();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    // test get instance
                    System.out.println(Singleton.getInstance().hashCode());
                }
            }.start();
        }
        // release lock, let all thread excute Singleton.getInstance() at the same time
        latch.countDown();
    }
}
```

其中 `CountDownLatch latch` 为闭锁，所有线程中都用 `latch.await()` ;等待锁释放，待所有线程初始化完成使用 `latch.countDown()` ; 释放锁，从而达到线程并发执行 `Singleton.getInstance()` 的效果。

结果为：

```
2016228077
2016228077
2016228077
2016228077
...
```

Reference:

http://blog.csdn.net/dmk877/article/details/50311791

http://www.trinea.cn/java/singleton/