---
layout: post # needs to be post
title: Realm 在 Android 上的应用
summary: Let's Realm!
featured-img: design
categories: [Android]
---


![Let's Realm](https://ww4.sinaimg.cn/large/006tNc79gy1fe9omgy5jxj30xc0hfabm.jpg)

## 什么是Realm?

<!--more-->

在Android平台上，有很多基于SQLite的ORM框架，例如[GreenDAO](http://greenrobot.org/greendao/),  [OrmLite](http://ormlite.com/), [SugarORM](http://satyan.github.io/sugar/), [LitePal](https://github.com/LitePalFramework/LitePal)等等，对于写习惯了SQL语句的小伙伴们来说，看到SQLite这样的数据库肯定是倍感亲切了，有了这些框架更是如虎添翼。但是，在我们日常的开发中，数据量并不会特别的大，表的结构也不会特别复杂，用SQL语句有种大(过)材(于)小(繁)用(琐)的感觉，我们需要做的事情可能仅仅是把用户生成的数据对象快速的缓存起来。这个时候NoSQL就派上了用场，以Mongodb，Redis为代表的NoSQL都引入了一些相对现代化的方式存储数据，比如支持Json，Document的概念，流式api，数据变更通知等等，极大程度的降低了我们学习的成本提高了我们的开发效率。而Realm作为一款移动端的NoSQL框架，官方定位就是替代SQLite等关系型数据库。

Realm是一个由Y Combinator孵化的创业团队开源出来的MVCC(多版本并发控制)的数据库，支持运行在手机，平板和可穿戴设备等嵌入式设备上。

## Realm的优点
+ [简单易用](#简单易用)
+ [速度快](#速度快)
+ [跨平台](#跨平台)
+ [高级功能](#高级功能)
+ [可视化](#可视化)
+ [开源](#开源)

### 简单易用
上面我们已经说过，Realm并不是基于SQLite上的ORM，它有自己的数据库引擎，使用也非常简单。我们先来看看一段简单的代码。

```java
// 通过继承定义我们自己的model类
public class Dog extends RealmObject {
    private String name;
    private int age;

    // ... 生成 getter 和 setter ...
}

public class Person extends RealmObject {
    @PrimaryKey
    private long id;
    private String name;
    private RealmList<Dog> dogs; // 生命一对多的关系

    // ... 生成 getter 和 setter ...
}

// 像普通的Java对象一样使用他们
Dog dog = new Dog();
dog.setName("Rex");
dog.setAge(1);

// 初始化Realm
Realm.init(context);

// 在当前线程下获取Realm实例
Realm realm = Realm.getDefaultInstance();

// 查询Realm中所有年龄小于2岁的狗狗
final RealmResults<Dog> puppies = realm.where(Dog.class).lessThan("age", 2).findAll();
puppies.size(); // => 0 因为现在还没有狗狗添加到Realm中

// 在事物中持久化你的数据
realm.beginTransaction();
final Dog managedDog = realm.copyToRealm(dog);
Person person = realm.createObject(Person.class);  
person.getDogs().add(managedDog);
realm.commitTransaction();

// 当数据改变后，Listener会被通知
puppies.addChangeListener(new RealmChangeListener<RealmResults<Dog>>() {
    @Override
    public void onChange(RealmResults<Dog> results) {
        // 查询结果会被实时更新
        puppies.size(); // => 1
    }
});

// 在后台线程中异步地更新对象
realm.executeTransactionAsync(new Realm.Transaction() {
    @Override
    public void execute(Realm bgRealm) {
        Dog dog = bgRealm.where(Dog.class).equalTo("age", 1).findFirst();
        dog.setAge(3);
    }
}, new Realm.Transaction.OnSuccess() {
    @Override
    public void onSuccess() {

      // Realm对象会被自动地更新
    	puppies.size(); // => 0 因为没有小狗狗的年龄小于两岁
    	managedDog.getAge();   // => 3 狗狗的年龄已经被更新了
    }
});
```

### 速度快
我们直接看Realm官方的对比测试数据吧。

每秒能在200K数据记录中进行查询后count的次数: Realm能够达到30.9次，而SQLite只能达到13.6次，Core Data只能达到可怜的一次。

![Record of counts](https://i.loli.net/2018/03/27/5aba2be25358e.jpg)

在200K条数据记录进行一次遍历查询，数据和前面的count相似: Realm能够达到每秒遍历200K数据记录31次，SQLite只能达到14次，而Core Data只有可怜的2次。
![Record of queries](https://ww3.sinaimg.cn/large/006tNc79gy1fe9onmh6bqj30qe0g0wf5.jpg)

在一次事物中每秒插入数据量的对比，SQLite可以达到178K条记录，性能最好，Realm可以达到94K，而Core Data再次垫底，只有18K。
![Record of inserts](https://ww3.sinaimg.cn/large/006tNc79gy1fe9onv5dvuj30qe0g0754.jpg)

我自己也进行一次简单的测试，以JSON格式插入641条记录(好吧，我知道数据量比较小，仅仅只是一个参考，具体的数据可以参考[这里](https://github.com/TonnyL/Espresso/blob/master/mobile/src/main/java/io/github/marktony/espresso/data/source/local/CompaniesLocalDataSource.java))。

```
04-03 19:06:13.837 11090-11245/io.github.marktony.espresso D/TAG: 1491217573837

04-03 19:06:14.044 11090-11245/io.github.marktony.espresso D/TAG: 1491217574044
```
![My Record](https://ww2.sinaimg.cn/large/006tNc79gy1fe9p947u3yj30vw02g0tc.jpg)
207毫秒(Android 7.1.1, Realm 3.0)。

### 跨平台
Realm目前支持Objective-C(iOS), Swift(iOS), Java(Android), JavaScript, Xamarin等平台。

现在很多应用都需要兼顾iOS和Android两个平台，使用Realm可以使用Realm提供的API，在数据持久化层实现两个平台的无差异化移植，无需对内部数据的架构进行改动。

### 高级功能
Realm支持加密，格式化查询，流式API，JSON，数据变更通知等等。

### 可视化
Realm官方提供了一个名为「[Realm Browser](https://itunes.apple.com/us/app/realm-browser/id1007457278?mt=12)」轻量级的数据库查看工具(目前还只支持macOS平台)，利用Realm Browser我们可以进行简单的插入，删除等基本操作。
![Realm Browser](https://ww3.sinaimg.cn/large/006tNc79gy1fe9q6fqk7nj31kw0uth2j.jpg)

第三方开发者也提供了一些移动端的数据库查看工具，例如：

[Realm Browser by Jonas Rottmann (Android)](https://github.com/jonasrottmann/realm-browser)

![Realm Browser by Jonas Rottmann (Android)](https://ww3.sinaimg.cn/large/006tNc79gy1fe9rb6igm3j30u00zkq73.jpg)

[ealm Browser by Max Baumbach (iOS)](https://github.com/bearjaw/RBSRealmBrowser)

![Realm Browser by Max Baumbach (iOS)](https://ww2.sinaimg.cn/large/006tNc79gy1fe9rfz16e0g30ai0iub2a.gif)

### 开源
[Realm](https://github.com/realm)已经将[Realm Core](https://github.com/realm/realm-core), [Realm Java](https://github.com/realm/realm-java)，[Realm Cocoa](https://github.com/realm/realm-cocoa), [Realm JS](https://github.com/realm/realm-js), [Realm Dotnet](https://github.com/realm/realm-dotnet)等等项目开源，这也就意味着，你可以向Realm团队提bug，提建议等等，和Realm团队一起见证Realm的成长。

另外，还有一点一定要说明的是，Realm团队的[博客](https://realm.io/cn/news/)，干货满满，而且都有中文翻译哦~

## Realm的不足
说完了优点，自然还要说说不足的地方。

+ [体积](#体积)
+ [数据类型](#数据类型)
+ [线程限制](#线程限制)

### 体积
引入Realm之后，在不做任何处理的情况下，APK体积增大了一个非常恐怖的数字 **5.6M**，你没有看错，是5.6兆 ~~(5.6M是什么概念，四舍五入就是10M，在四舍五入就是100M啊)~~。直接看图吧。

![APK size before implementing Realm](https://ww4.sinaimg.cn/large/006tNc79gy1fe9t6n20c4j30ky0yqn2w.jpg)

![APK size using default configuration](https://ww1.sinaimg.cn/large/006tNc79gy1fe9t7ajo6lj30ky0yqn2u.jpg)

我们可以通过配置`build.gradle`的split，根据不同的设备类型对APK进行拆分，从而达到缩减体积的目的。下面我配置了split之后，APK体积的变化。

```gradle
splits {
        // Split apks on build target ABI, view all options for the splits here:
        // http://tools.android.com/tech-docs/new-build-system/user-guide/apk-splits
        abi {
            enable true
            reset()
            include 'armeabi', 'armeabi-v7a', 'arm64-v8a', 'mips', 'x86', 'x86_64'
        }
        universalApk true
}
```

![APK size using split](https://ww2.sinaimg.cn/large/006tNc79gy1fe9t7nzglwj30ky0yq44c.jpg)

### 数据类型
1. Realm要求当前Bean对象必须直接继承RealmObject，~~或者间接实现(目前已经不再推荐使用)~~，侵入性非常强。
2. Realm不支持内部类。
3. Realm修改了部分数据类型，例如List --> RealmList。在内部实现上，RealmList与ArrayList等还是有比较大的区别的。
4. 使用RealmList<T>时，泛型`T`类型必须是直接继承了RealmObject的类型，例如，如果是RealmList<String>类型，那么不好意思，这是不支持的😂。官方建议我们自定义一个`RealmString`继承自RealmObject，例如：

	```java
	public class RealmString extends RealmObject {
	
	    private String string;
	
	    public String getString() {
	        return string;
	    }
	
	    public void setString(String string) {
	        this.string = string;
	    }
	}
	```

5. Realm是不支持主键自增长的，所以，我们需要自己维护一个`PrimaryKey`来约束主键。例如：

	```java
	@PrimaryKey
    private String number;
	```
另外，如果没有给RealmObject设置主键，insertOrUpdata的默认操作就是insert，这样 就会导致重复插入数据记录了。
	
6. Intent传值时，也会有一些坑。例如，我想要在a1中查询数据，然后将查询结果(RealmList)传递到a2中。不好意思，Realm不想和你说话，并向你丢了一个crash。这是因为，ArrayList实现了`Serializable`接口，而RealmList并没有。再例如，如果不是RealmList，而是一个普通的继承自RealmObject并实现了`Serializable`接口的实体类呢？也不行。这是因为，查询出来的数据并不是我们想要的对象，而是Realm利用apt帮我们生成的实体类的子类，或者说实体类的代理类，而在Realm中，起作用的就是这个代理类。

那么，怎么解决呢？官方的建议不要传递整个RealmList或者RealmObject，而是传递对象的标识符，然后在接收方(Activity, Service, BroadcastReceiver等等)解析出这个标识符，然后利用Realm再次查询获得相应的结果。

### 线程限制
如果你在UI线程获取到了当前Realm对象，在异步线程中使用当前Realm对象进行操作，就会抛出异常了。RealmObject也是如此。所以，异步很重要。

另外，在调用了`Realm.close()`方法之后，所获取的对象就不能再访问了，所以，在获取到了RealmObject之后，官方提供了一个`copyFromRealm`来复制一份实例供我们使用。

## 在Android中使用Realm
看到这里的客官，应该对Realm是真爱了。下面我们就来正经讨论一下，如何在Android中使用Realm数据库。

+ [先决条件](#先决条件)
+ [安装](#安装)
+ [获取Realm实例](#获取Realm实例)
+ [创建Realm实体](#创建Realm实体)
+ [增删改查](#增删改查)
+ [高级用法](#高级用法)
	- [加密](#加密)
	- [与Android系统结合](#与Android系统结合)
	- [与其他第三方库结合使用](#与其他第三方库结合使用)

### 先决条件
+ [Android Studio](https://developer.android.com/studio/index.html) 1.5.1或以上版本
+ JDK 1.7或以上版本
+ 较新版本的Android SDK
+ Android API Level 9 或者以上(即Android 2.3及以上)

什么，你想要在Eclipse上使用Realm?

![Using Realm in Eclipse](https://ww2.sinaimg.cn/large/006tNc79gy1feapqxnhi6j30dw09a3yu.jpg)

### 安装
Realm是作为一个Gradle插件安装的。

+ 第一步：在project级别的`build.gradle`文件下添加：

	```gradle
	buildscript {
	    repositories {
	        jcenter()
	    }
	    dependencies {
	        classpath "io.realm:realm-gradle-plugin:3.0.0"
	    }
	}
	```
	
	下面是project级别的`build.gradle`文件的位置：
	![build.gradle file of project level](https://ww4.sinaimg.cn/large/006tNc79gy1feapw9tjepj30pi0lu79w.jpg)

+ 第二步：在application级别的`build.gradle`文件的顶部应用`realm-android`插件：

	```gradle
	apply plugin: 'realm-android'
	```
	
	下面是application级别的`build.gradle`文件的位置：
	![build.gradle file of application level](https://ww4.sinaimg.cn/large/006tNc79gy1feaq13ys14j30pa0lsq8j.jpg)
	
添加完成后，刷新gradle依赖即可。

下面是两个级别的`build.gradle`文件示例：
+ [Project level build.gradle](https://github.com/realm/realm-java/blob/master/examples/build.gradle)
+ [Application level build.gradle](https://github.com/realm/realm-java/blob/master/examples/introExample/build.gradle)

除了gradle外，Realm并不支持像Maven和Ant这样的构建工具。如果你有需要的话，可以关注下面的两个issue。

+ [Maven support](https://github.com/realm/realm-java/issues/2342)
+ [Ant support](https://github.com/realm/realm-java/issues/1591)

### 获取Realm实例

一般需要先在Application中完成Realm的初始化工作。例如：

```java
public class App extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        Realm.init(this);
    }

}
```

然后我们可以通过一个Configuration来指定Realm生成的数据库的名字和版本等等。

```java
Realm realm = Realm.getInstance(new RealmConfiguration.Builder()
                .deleteRealmIfMigrationNeeded()
                .name("MyAwsomeRealmName.realm")
                .build());
```

### 创建Realm实体
Realm的实体类可以通过继承`RealmObject`的方式创建：

```java
public class User extends RealmObject {

    private String          name;
    private int             age;

    @Ignore
    private int             sessionId;

	// 通过IDE生成的标准getters和setters...
    public String getName() { return name; }
    public void   setName(String name) { this.name = name; }
    public int    getAge() { return age; }
    public void   setAge(int age) { this.age = age; }
    public int    getSessionId() { return sessionId; }
    public void   setSessionId(int sessionId) { this.sessionId = sessionId; }
}
```

Realm实体类还是支持`public`, `protected`, `private`字段和方法的。

```java
public class User extends RealmObject {

    public String name;

    public boolean hasLongName() {
      return name.length() > 7;
    }

    @Override
    public boolean equals(Object o) {
      // 自定义equals操作
    }
}
```

#### 字段类型
Realm支持的字段类型：

`boolean`, `byte`, `short`, `int`, `long`, `float`, `double`, `String`, `Date` and `byte[]`，其中integer类型`byte`, `short`, `int`，都被自动的包装成了`long`类型。`RealmObject`和`RealmList<? extends RealmObject>`的子类支持实体类之间的关系(一对一，一对多，多对多等)。

装箱类型`Boolean`, `Byte`, `Short`, `Integer`, `Long`, `Float` 和 `Double`等也可以在实体类中使用，不过需要注意的是这些字段的值有可能为`null`。

#### Required字段和null值
在有些情况下，字段值为`null`并不合适。在Realm中，`@Required`注解就是用来强制检查，不允许字段出现`null`值。只有`Boolean`, `Byte`, `Short`, `Integer`, `Long`, `Float` 和 `Double`等可以使用`@Required`注解，如果其他类型的字段使用了此注解，编译时将会出现错误。原始字段类型和`RealmList`类型被隐含的标示为`Required`，而`RealmObject`类型字段是可以为nullable的。

#### 属性忽略
使用注解`@Ignore`意味着此字段可以不被存储到数据库中。

#### 自动更新
对于底层数据而言，`RealmObject`是实时的，自动更新的，这也就意味着我们获取到的对象数据不需要我们手动的刷新。更改数据对查询的影响会被立刻反应在查询结果上。

```java
realm.executeTransaction(new Realm.Transaction() {
    @Override
    public void execute(Realm realm) {
        Dog myDog = realm.createObject(Dog.class);
        myDog.setName("Fido");
        myDog.setAge(1);
    }
});
Dog myDog = realm.where(Dog.class).equalTo("age", 1).findFirst();

realm.executeTransaction(new Realm.Transaction() {
    @Override
    public void execute(Realm realm) {
        Dog myPuppy = realm.where(Dog.class).equalTo("age", 1).findFirst();
        myPuppy.setAge(2);
    }
});

myDog.getAge(); // => 2
```

#### 属性索引
使用注解`@Index`会给字段添加一个搜索索引。这会导致插入速度变慢和数据文件变大，但是查询操作会更快。所以，Realm只推荐你在需要提高读性能的时候添加索引。索引支持的字段类型包括：`String`, `byte`, `short`, `int`, `long`, `boolean` 和 `Date`。

### 增删改查
数据库的使用，最常用的就是增删改查(CRUD)四种操作了，我们一起来看看Realm是如何实现上述四种操作的。

#### 写操作
在讨论具体的CRUD之前，我们要先了解一下写操作。读操作是隐式完成的，也就是说，任何时候你都可以对实体进行访问和查询。而所有的写操作(添加，修改，删除)都必须在写事物中完成。写事物能够被提交和取消。写操作同时也用于保证线程安全。

```java
// 获取Realm实例
Realm realm = Realm.getDefaultInstance();

realm.beginTransaction();

//... 在这里添加或者升级对象 ...
User user = realm.createObject(User.class);

realm.commitTransaction();
// 取消写操作
// realm.cancelTransaction();
```

需要注意的是，写操作是互斥的。所以，如果我们同时在UI线程和后台线程中创建写操作就有可能导致ANR。当我们在UI线程创建写事物时，可以使用[异步事物](https://realm.io/docs/java/latest/#asynchronous-transactions)来避免ANR的出现。

Realm是crash安全的，所以如果在事物中产生了一个异常，Realm本身是不会被破坏的。不过在当前事物中的数据被丢失，不过为了避免异常产生的一系列问题，取消事物就非常重要了。如果使用`executeTransaction()`这些操作都会被自动完成。

由于Realm采用的MVCC架构，在写事物进行的同时，读操作也是被允许的。这也就意味着，除非需要在许多的线程中，同时处理许多的并行事务，我们可以使用大型事物，完成许多细粒度的事物。当我们向Realm提交一个写事物时，其他的Realm实例都会被通知，并且被[自动更新](https://realm.io/docs/java/latest/#auto-updating)。

读和写的操作在Realm中就是[ACID](http://en.wikipedia.org/wiki/ACID).

#### 添加
我们可以使用下面的代码将数据添加到Realm中：

```java
realm.beginTransaction();
User user = realm.createObject(User.class); // 创建一个新的对象
user.setName("John");
user.setEmail("john@corporation.com");
realm.commitTransaction();
```

```java
User user = new User("John");
user.setEmail("john@corporation.com");

// 将对象复制到Realm中，后面的操作必须在realmUser上进行。
realm.beginTransaction();
User realmUser = realm.copyToRealm(user);
realm.commitTransaction();
```

我们也可以使用`realm.executeTransaction()`方法替代手动的跟踪`realm.beginTransaction()`, `realm.commitTransaction()`和`realm.cancelTransaction()`，这个方法自动地处理了begin/commit，和错误发生后的cancel。

```java
realm.executeTransaction(new Realm.Transaction() {
	@Override
	public void execute(Realm realm) {
		User user = realm.createObject(User.class);
		user.setName("John");
		user.setEmail("john@corporation.com");
	}
});
```

异步事物可以帮助我们处理同步事物可能带来的UI线程阻塞的问题。使用异步事物后，事物会在一个后台线程上运行，事物完成后会进行结果通知。

```java
realm.executeTransactionAsync(new Realm.Transaction() {
            @Override
            public void execute(Realm bgRealm) {
                User user = bgRealm.createObject(User.class);
                user.setName("John");
                user.setEmail("john@corporation.com");
            }
        }, new Realm.Transaction.OnSuccess() {
            @Override
            public void onSuccess() {
                // 事物成功完成
            }
        }, new Realm.Transaction.OnError() {
            @Override
            public void onError(Throwable error) {
                // 事物失败，自动取消
            }
        });
```

`onSuccess` 和 `onError` 回调都是可选的，但是如果提供了这些方法，它们会在事物成功完成或者失败后被调用。

我们可以通过`RealmAsyncTask`获取一个异步事物的对象，这个对象可以用在当事物未完成而Activity或者Fragment被销毁时取消事物。如果在回调中进行了更新UI的操作，而又忘记了取消事物，就会造成crash。

```java
public void onStop () {
    if (transaction != null && !transaction.isCancelled()) {
        transaction.cancel();
    }
}
```

Realm还提供了一个和神奇的功能，直接通过JSON(String, JSONObject, InputStream)添加数据，并且Realm会自动忽略没有在RealmObject中定义的字段。单个的对象可以通过`Realm.createObjectFromJson()`方法添加，对象表可以通过`Realm.createAllFromJson()`方法添加。

```java
// 代表city的RealmObject
public class City extends RealmObject {
    private String city;
    private int id;
    // getters and setters left out ...
}

// 通过String添加
realm.executeTransaction(new Realm.Transaction() {
    @Override
    public void execute(Realm realm) {
        realm.createObjectFromJson(City.class, "{ city: \"Copenhagen\", id: 1 }");
    }
});

// 通过一个InputStream添加多个对象
realm.executeTransaction(new Realm.Transaction() {
    @Override
    public void execute(Realm realm) {
        try {
            InputStream is = new FileInputStream(new File("path_to_file"));
            realm.createAllFromJson(City.class, is);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
});
```

Realm解析JSON时遵循下面的规则：

+ 使用包含null值的JSON创建对象
	- 对于非必须字段，设置为默认值null
	- 对于必须字段，直接抛出异常
+ 使用包含null值的JSON更新对象
	- 对于非必须字段，设置为null
	- 对于必须字段，直接抛出异常
+ JSON不包含字段
	- 保持必须和非必须字段的值不变

#### 查询
首先定义一个`User`类：

```java
public class User extends RealmObject {

    @PrimaryKey
    private String          name;
    private int             age;

    @Ignore
    private int             sessionId;

	// 使用IDE生成的标准getters和setters
    public String getName() { return name; }
    public void   setName(String name) { this.name = name; }
    public int    getAge() { return age; }
    public void   setAge(int age) { this.age = age; }
    public int    getSessionId() { return sessionId; }
    public void   setSessionId(int sessionId) { this.sessionId = sessionId; }
}
```

查询所有name为「John」或者「Peter」的User:

```java
// 创建一个RealmQuery用于查找所有符合条件的user
RealmQuery<User> query = realm.where(User.class);

// 添加查询条件
query.equalTo("name", "John");
query.or().equalTo("name", "Peter");

// 执行查询
RealmResults<User> result1 = query.findAll();

// 或者进行简化
RealmResults<User> result2 = realm.where(User.class)
                                  .equalTo("name", "John")
                                  .or()
                                  .equalTo("name", "Peter")
                                  .findAll();
```

上面的代码就可以获取一个`RealmResults`类的实例，包含了名称为John或者Peter的所有user。当`findAll()`方法被调用时，查询便开始执行。`findAll()`是`findAll()`方法大家族的一员，类似的方法还有：`findAllSorted()`返回一个排好序之后的结果集合，`findAllAsync()`会在后台线程中异步的完成查询操作。

需要注意的是，查询得到的结果是没有被复制的。正如Realm的官方文档所言：

> All fetches (including queries) are lazy in Realm, and the data is never copied.

我们得到了一个符合查询条件的对象列表的引用，但是如果我们直接操作，对象将会是原始的对象。所以，还是复制一份吧。

```java
ArrayList array = realm. copyFromRealm(result2);
```

`RealmResults`继承自Java的`AbstractList`，在许多方面的操作类似。例如，`RealmResults`是有序的，我们可以通过索引获取特定的对象。

当查询没有符合条件的结果时，返回值`RealmResults`并不会为null，但是`size()`方法会返回0。

如果我们想要修改或者删除`RealmResults`中的对象，也必须在写事物中进行。

##### 过滤
对于所有的数据类型，都有以下两种查询条件：

+ `equalTo()`
+ `notEqualTo()`

使用`in()`匹配某一特定字段而不是一个的值的列表。例如，查找名字 「Jill」, 「William」, 「Trillian」，我们可以使用`in("name", new String[]{"Jill", "William", "Trillian"})`。`in()`方法接收String, 二进制数据和数值型字段。

数值数据类型包括`Data`，都允许进行下面的查询条件:

+ `between()`(包含边界值)
+ `greaterThan()` - 大于
+ `lessThan()` - 小于
+ `greaterThanOrEqualTo()` - 大于等于
+ `lessThanOrEqualTo()` - 小于等于

String类型字段允许使用以下查询条件：

+ `contains()` - 包含
+ `beginsWith()` - 以...开头
+ `endsWith()` - 以...结尾
+ `like()` - 类似于

所有的String类型都支持添加第三个参数来控制大小写的敏感类型。

+ Case.INSENSITIVE -> 大小写不敏感
+ Case.SENSITIVE -> 大小写敏感(默认值)

使用`like()`进行模糊匹配，匹配条件如下:

+ `*` - 匹配0个或者多个Unicode字符
+ `?` - 匹配单个Unicode字符

举个🌰，假设现在有一个RealmObject有一个`name`字段，其值有「William」, 「Bill」,「Jill」, 和 「Trillian」。 查询条件`like("name", "?ill*")`会匹配开始的3个对象， 而`like("name", "*ia?")`会匹配第一个和最后一个对象。

二进制数据，String，`RealmObject`的列表(`RealmList`)有可能为空，也就是长度为0，下面是检测是否为空的查询条件：

+ `isEmpty()`
+ `isNotEmpty()`

如果一个字段是非必须字段，那么它的值就有可能为`null`，我们可以用以下条件检测：

+ `isNull()`
+ `isNotNull()`

##### 逻辑运算符
每一个查询条件都被隐式地使用`AND`连接。而逻辑运算符`OR`必须使用`or()`显式地声明。

仍然以上面`User`类为例，我们可以使用`beginGroup()` 和 `endGroup()` 来声明一组查询条件。

```java
RealmResults<User> r = realm.where(User.class)
                            .greaterThan("age", 10)  // 隐式地AND
                            .beginGroup()
                                .equalTo("name", "Peter")
                                .or()
                                .contains("name", "Jo")
                            .endGroup()
                            .findAll();
```

查询条件也可以使用`not()`进行否定，`not()`也可以和`beginGroup()` 和 `endGroup()` 用于否定一组子查询条件。再举个🌰，我们想要查询所有名字不为「Peter」和「Jo」的User：

```java
RealmResult<User> r = realm.where(User.class)
                           .not()
                           .beginGroup()
                                .equalTo("name", "Peter")
                                .or()
                                .contains("name", "Jo")
                            .endGroup()
                            .findAll();
```

当然，我们也可以用`in()`进行简化：

```java
RealmResult<User> r = realm.where(User.class)
                           .not()
                           .in("name", new String[]{"Peter", "Jo"})
                           finalAll();

```

##### 排序
当我们的查询完成后，可以使用下面的代码对查询结果进行排序:

```java
RealmResults<User> result = realm.where(User.class).findAll();
result = result.sort("age"); // 升序排序
result = result.sort("age", Sort.DESCENDING);
```

默认采用的是按升序排序，如果需要改变的话，可以将`Sort.DESCENDING`作为可选参数传入。

##### 唯一值
我们可以使用`distinct()`来查询某一字段共有多少类型的值。例如，我们要查询在我们的数据库中有多少不同的名字：

```java
RealmResults<Person> unique = realm.where(Person.class).distinct("name");
```

此操作只支持integer和string类型的字段，对其他类型进行此操作会产生异常。我们也可以对多个字段进行排序。

##### 链式查询
由于查询结果并不会进行复制和计算操作，我们可以一步一步的过滤我们的数据：

```java
RealmResults<Person> teenagers = realm.where(Person.class).between("age", 13, 20).findAll();
Person firstJohn = teenagers.where().equalTo("name", "John").findFirst();
```

我们也可以对子对象进行链式查询。假设上面的`Person`类还有一个`Dog`类型的list类型字段：

```java
public class Dog extends RealmObject {
    private int age;
    // getters & setters ...
}

public class Person extends RealmObject {
    private int age;
    private RealmList<Dog> dogs;
    // getters & setters ...
}
```

我们可以通过链式查询，查找年龄在13至20之间，并且至少有一只年龄为1岁的狗狗的人：

```java
RealmResults<Person> teensWithPups = realm.where(Person.class).between("age", 13, 20).equalTo("dogs.age", 1).findAll();
```

需要注意的是，链式查询的基础并不是`RealmQuery`，而是`RealmResults`。如果我们为一个已经存在的`RealmQuery`添加更多的查询条件，修改的是query本身，而不是链。

OK，到这里，查询的情况我们讨论的也差不多了。由于查询在增删改查四种操作中的使用频率最高，所以篇幅也最长，下面我们来讨论「修改」的情况。

#### 修改
事实上，我们在上面的内容中已经进行过修改的操作了，[戳这里](#自动更新)。我们在查询到符合条件的对象后，开启一个事物，在事物中进行修改，然后提交事物即可:

```java
realm.executeTransaction(new Realm.Transaction() {
    @Override
    public void execute(Realm realm) {
        Dog myPuppy = realm.where(Dog.class).equalTo("age", 1).findFirst();
        myPuppy.setAge(2);
    }
});
```

#### 删除
删除操作和修改操作类似，基本思想都是先查询，然后在事物中进行操作。我们可以通过下面的代码进行删除操作:

```java
// 获取查询结果
final RealmResults<Dog> results = realm.where(Dog.class).findAll();

// 所有对数据的变更必须在事物中进行
realm.executeTransaction(new Realm.Transaction() {
    @Override
    public void execute(Realm realm) {
        // 移除符合条件的单个查询结果
        results.deleteFirstFromRealm();
        results.deleteLastFromRealm();

        // 移除单个对象
        Dog dog = results.get(5);
        dog.deleteFromRealm();

        // 移除所有符合条件的查询结果
        results.deleteAllFromRealm();
    }
});
```

### 高级用法
#### 加密
Realm文件可以通过向`RealmConfiguration.Builder.encryptionKey()`传递一个512位(64字节)的密钥进行加密后存储在磁盘上。

```java
byte[] key = new byte[64];
new SecureRandom().nextBytes(key);
RealmConfiguration config = new RealmConfiguration.Builder()
  .encryptionKey(key)
  .build();

Realm realm = Realm.getInstance(config);
```

这样的措施保证了所有存储在磁盘上的数据都是经过标准AES-256加密和解密的。当Realm文件创建后，每次创建Realm实例时都需要提供相同的密钥。

点击[examples/encryptionExample](https://github.com/realm/realm-java/tree/master/examples/encryptionExample)查看完整的示例，示例展示了如何通过Android KeyStore安全地存储密钥以保证其他应用不能读取此密钥。

#### 与Android系统结合
##### 适配器
Realm提供了一些抽象的工具类来方便地将 `OrderedRealmCollection`
( `RealmResults` 和 `RealmList` 都实现了这个接口)展示到UI控件上。

+ [`RealmBaseAdapter`](https://realm.io/docs/java/latest/adapter/io/realm/RealmBaseAdapter.html) 可以和 `ListView` 搭配使用。参考[示例](https://github.com/realm/realm-android-adapters/blob/master/example/src/main/java/io/realm/examples/adapters/ui/listview/MyListAdapter.java)。
+ [`RealmRecyclerViewAdapter`](https://realm.io/docs/java/latest/adapter/io/realm/RealmRecyclerViewAdapter.html) 可以和 `RecyclerView` 搭配使用。参见[示例](https://github.com/realm/realm-android-adapters/blob/master/example/src/main/java/io/realm/examples/adapters/ui/recyclerview/MyRecyclerViewAdapter.java)。

在使用这些适配器之前，我们需要在application级别的`build.gradle`文件中添加额外的依赖:

```gradle
dependencies {
	compile 'io.realm:android-adapters:2.0.0'
}
```

##### Intents
由于我们并不能直接在Intent之间传递`RealmObject`，所以Realm建议只传递`RealmObject`的标识符。举个很简单的🌰：如果一个一个对象拥有一个主键，那么我就可以通过`Intent`的`Bundle`传递这个值。

```java
// 假设我们现在有一个person类，并且将其id字段设置为@PrimaryKey ...
Intent intent = new Intent(getActivity(), ReceivingService.class);
intent.putExtra("person_id", person.getId());
getActivity().startService(intent);
```

然后在接收组件中(Activity, Service, IntentService, BroadcastReceiver 等等)解析出传递的主键值并打开Realm,查询到该主键对应的`RealmObject`。

```java
// 在onCreate(), onHandleIntent()等方法中完成
String personId = intent.getStringExtra("person_id");
Realm realm = Realm.getDefaultInstance();
try {
    Person person = realm.where(Person.class).equalTo("id", personId).findFirst();
    // 对person进行一些操作 ...
} finally {
    realm.close();
}
```

完整的示例可以在 [`threading example`](https://github.com/realm/realm-java/tree/master/examples/threadExample) 的`Object Passing`部分找到。示例展示了在Android常用的如何传递id并得到对应的`RealmObject`。

##### AsyncTask & IntentService
Realm和`AsyncTask`, `IntentService`搭配使用时，需要特别留心，`AsyncTask`类包含了一个在后台线程执行的`doInBackground()`方法，`IntentService`类包含了在工作线程执行的`onHandleIntent(Intent intent)`方法。如果我们需要在上述两个方法中使用Realm，我们需要先打开Realm，完成工作，然后在退出之前关闭Realm。下面是一些示例：

AsyncTask:在`doInBackground()`方法中打开和关闭Realm。

```java
private class DownloadOrders extends AsyncTask<Void, Void, Long> {
    protected Long doInBackground(Void... voids) {
        // 现在已经在后台线程中了。

        // 打开Realm
        Realm realm = Realm.getDefaultInstance();
        try {
            // 使用Realm
            realm.createAllFromJson(Order.class, api.getNewOrders());
            Order firstOrder = realm.where(Order.class).findFirst();
            long orderId = firstOrder.getId(); // order的id
            return orderId;
        } finally {
            realm.close();
        }
    }

    protected void onPostExecute(Long orderId) {
        // 回到Android主线程
        // 完成一些和orderId有关的操作例如在Realm中
        // 查询order并做一些操作。
    }
}
```

IntentServie: 在`onHandleIntent()`方法中打开和关闭Realm。

```java

public class OrdersIntentService extends IntentService {
    public OrdersIntentService(String name) {
        super("OrdersIntentService");
    }

    @Override
    protected void onHandleIntent(Intent intent) {
        // 现在已经在后台线程中了。

        // 打开Realm
        Realm realm = Realm.getDefaultInstance();
        try {
            // 使用Realm
            realm.createAllFromJson(Order.class, api.getNewOrders());
            Order firstOrder = realm.where(Order.class).findFirst();
            long orderId = firstOrder.getId(); // order的id
        } finally {
            realm.close();
        }
    }
}
```

需要特别注意的是：在`IntentService`中，`ChangeListener`不能够正常的工作。尽管它是一个`Looper`线程，每一次调用`onHandleIntent()`不「loop」的分离事件。这也就意味着我们可以注册register listener，但是它永远也不会被触发。

#### 与其他第三方库结合使用
+ [Realm与GSON](#Realm与GSON)
+ [Realm与Jackson-databind](#Realm与Jackson-databind)
+ [Realm与Kotlin](#Realm与Kotlin)
+ [Realm与Retrofit](#Realm与Retrofit)
+ [Realm与RxJava](#Realm与RxJava)

##### Realm与GSON
[GSON](https://code.google.com/p/google-gson/)是Google开发的JSON处理库，Realm和GSON可以无缝的配合使用。

```java
// 使用User类
public class User extends RealmObject {
    private String name;
    private String email;
    // getters and setters ...
}

Gson gson = new GsonBuilder().create();
String json = "{ name : 'John', email : 'john@corporation.com' }";
User user = gson.fromJson(json, User.class);
```

**序列化(Serialization)**

我们有时需要序列化与反序列化一个Realm对象以便与其它库（比如[Retrofit](http://square.github.io/retrofit/)）相配合。因为GSON使用[成员变量值而非getter和setter](https://groups.google.com/forum/#!topic/google-gson/4G6Lv9PghUY)，所以我们无法通过GSON的一般方法来序列化Realm对象。

我们需要为Realm模型对象自定义一个[JsonSerializer](https://google.github.io/gson/apidocs/com/google/gson/JsonSerializer.html)并且将其注册为一个[TypeAdapter](https://google.github.io/gson/apidocs/com/google/gson/TypeAdapter.html)。

请参考这个[Gist](https://gist.github.com/cmelchior/ddac8efd018123a1e53a)。

**数组(Primitive lists)**

某些JSON API会以数组的形式返回原始数据类型（例如String和integer），[Realm暂时不支持对这种数组的处理](https://github.com/realm/realm-java/issues/575)。但我们可以通过自定义[TypeAdapter](https://google.github.io/gson/apidocs/com/google/gson/TypeAdapter)来处理这种情况。

这个[Gist](https://gist.github.com/cmelchior/1a97377df0c49cd4fca9)展示了如何将JSON中的整型数组存入Realm。类似地，我们可以用这个方法来处理其它原始数据类型数组。

**Troubleshooting**

Realm 对象属性可能会包含循环引用。在这种情况下，GSON 会抛出 StackOverflowError。例如如下 Realm 对象拥有一个 Drawable 属性：

```java
public class Person extends RealmObject {
    @Ignore
    Drawable avatar;
    // 其他字段
}
```

Person 类含有一个 Android Drawable 并且被 @Ignore 修饰。当 GSON 序列化时，Drawable 被读取并且造成了堆栈溢出。([GitHub Issue](https://github.com/realm/realm-java/issues/1798))。添加如下代码以避免类似问题：

```java
public boolean shouldSkipField(FieldAttributes f) {
  return f.getDeclaringClass().equals(RealmObject.class) || f.getDeclaringClass().equals(Drawable.class);
}
```

请注意对 Drawable.class 的判定语句，它告诉 GSON 跳过这个属性的序列化以避免堆栈溢出错误。

##### Realm与Jackson-databind
[Jackson-databind](https://github.com/FasterXML/jackson-databind)是一个实现JSON数据和Java类之间绑定的库。

Jackson使用反射实现了数据绑定，而这与Realm对RxJava的支持产生了冲突因为对class loader而言，RxJava不一定可用。所造成的异常如下所示:

```
java.lang.NoClassDefFoundError: rx.Observable
at libcore.reflect.InternalNames.getClass(InternalNames.java:55)
...
```

可以通过为项目引入RxJava或者在工程创建一个看起来包含下面的代码的空文件来修改上面的问题。

```java
package rx;

public class Observable {
	// 为了支持Jackson-Databind，如果没有引入RxJava依赖，
	// 这样的空文件就是必须的
}
```

issue已经被提交到Jackson项目了，戳[这里](https://github.com/FasterXML/jackson-databind/issues/1070)。

##### Realm与Kotlin
Realm 完全兼容 [Kotlin](https://kotlinlang.org/) 语言，但有些地方需要注意：
+ 我们的模型类需要是 [开放的(open)](https://kotlinlang.org/docs/reference/classes.html#overriding-members)。
+ 我们可能需要在某些情况下添加注解 `@RealmCLass` 以保证编译通过。这是由于[当前 Kotlin 注解处理器的一个限制](http://blog.jetbrains.com/kotlin/2015/06/better-annotation-processing-supporting-stubs-in-kapt/)。
+ 很多 Realm API 引用了 Java 类。我们必须在编译依赖中添加 `org.jetbrains.kotlin:kotlin-reflect:${kotlin_version}`。

[参见示例](https://github.com/realm/realm-java/tree/master/examples/kotlinExample)。


##### Realm与Retrofit
[Retrofit](http://square.github.io/retrofit/) 是一个由 [Square](https://github.com/square) 开发，保证类型安全（typesafe）的 REST API 处理工具。

Realm 可以与 Retrofit 1.x 和 2.x 无缝配合工作。但请注意 Retrofit 不会自动将对象存入 Realm。我们需要通过调用 `Realm.copyToRealm()` 或 `Realm.copyToRealmOrUpdate()` 来将它们存入 Realm。

```
GitHubService service = restAdapter.create(GitHubService.class);
List<Repo> repos = service.listRepos("octocat");

// 从Retrofit复制数据元素到Realm
realm.beginTransaction();
List<Repo> realmRepos = realm.copyToRealmOrUpdate(repos);
realm.commitTransaction();
```

##### Realm与RxJava
[RxJava](https://github.com/ReactiveX/RxJava) 是 Netflix 发布的一个 [Reactive](http://reactivex.io/) 的扩展 库以支持 [观察者模式](https://en.wikipedia.org/wiki/Observer_pattern)。

Realm 包含了对 RxJava 的原生支持。如下类可以被暴露为一个 [`Observable`](https://github.com/ReactiveX/RxJava/wiki/Observable)：[`Realm`](https://realm.io/docs/java/latest/api/io/realm/Realm.html#asObservable--), [`RealmResults`](https://realm.io/docs/java/latest/api/io/realm/RealmResults.html#asObservable--), [`RealmObject`](https://realm.io/docs/java/latest/api/io/realm/RealmObject.html#asObservable--), [`DynamicRealm`](https://realm.io/docs/java/latest/api/io/realm/DynamicRealm.html#asObservable--) 和 [`DynamicRealmObject`](https://realm.io/docs/java/latest/api/io/realm/DynamicRealmObject.html#asObservable--)。

```java
// 综合使用Realm, Retrofit 和 RxJava(使用Retrolambda使语法更简洁)
// 加载所有用户并将它们的GitHub的最新stats合并(如果有的话)
Realm realm = Realm.getDefaultInstance();
GitHubService api = retrofit.create(GitHubService.class);
realm.where(Person.class).isNotNull("username").findAllAsync().asObservable()
    .filter(persons.isLoaded)
    .flatMap(persons -> Observable.from(persons))
    .flatMap(person -> api.user(person.getGithubUserName())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(user -> showUser(user));
```

请注意 [异步查询](https://realm.io/cn/docs/java/latest/#asynchronous-queries) 不会阻塞当前线程，如上代码会立即返回一个 RealmResults 实例。如果我们想确定该 RealmResults 已经加载完成请使用 [filter operator](http://reactivex.io/documentation/operators/filter.html) 和 [`RealmResults<E>.isLoaded()](https://realm.io/cn/docs/java/latest/api/io/realm/RealmResults.html#isLoaded--) 方法。通过判断 RealmResults 是否已经加载可以得知查询是否已经完成。

参考 [RxJava sample project](https://github.com/realm/realm-java/tree/master/examples/rxJavaExample)。

**配置**
RxJava 是可选依赖，这意味着 Realm 不会自动包含它。这样做的好处是我们可以选择需要的 RxJava 版本以及防止过多的无用方法被打包。如果我们要使用相关功能，需要手动添加 RxJava 到 `build.gradle` 文件。

```
dependencies {
  compile 'io.reactivex:rxjava:1.1.0'
}
```

我们也可以通过继承 [`RxObservableFactory`](https://realm.io/cn/docs/java/latest/api/io/realm/rx/RxObservableFactory.html) 来决定 `Observable` 的生成方式，然后通过 `RealmConfiguration` 进行配置。

```java
RealmConfiguration config = new RealmConfiguration.Builder()
  .rxFactory(new MyRxFactory())
  .build()
```

如果没有 `RxObservableFactory` 被定义，`RealmObservableFactory` 会被默认使用，它支持 RxJava <= 1.1.*(也就意味着目前在RxJava2.0上现在还没有办法使用...)。

## 版本迁移
当我们的数据结构发生了变化时，我们就需要对数据库进行升级了。而在Realm上，数据库的升级是通过迁移操作完成的，也就是把原来的数据迁移到具有新数据结构的数据库。通常，这样的操作可以分成两部完成。

1. 创建迁移类
	
	```java
	// 迁移类示例
	public class MyMigration implements RealmMigration{
	
		  @Override
		  public void migrate(DynamicRealm realm, long oldVersion, long newVersion) {
		
		     // DynamicRealm 暴露了一个可编辑的schema
		     RealmSchema schema = realm.getSchema();
		
		     // 迁移到版本 1 : 添加一个新的类
		     // 示例:
		     // public Person extends RealmObject {
		     //     private String name;
		     //     private int age;
		     //     // getters and setters left out for brevity
		     // }
		     if (oldVersion == 0) {
		        schema.create("Person")
		            .addField("name", String.class)
		            .addField("age", int.class);
		        oldVersion++;
		     }
		
		     // 迁移到版本 2 :添加一个primary key + 对象引用
		     // 示例:
		     // public Person extends RealmObject {
		     //     private String name;
		     //     @PrimaryKey
		     //     private int age;
		     //     private Dog favoriteDog;
		     //     private RealmList<Dog> dogs;
		     //     // getters and setters left out for brevity
		     // }
		     if (oldVersion == 1) {
		        schema.get("Person")
		            .addField("id", long.class, FieldAttribute.PRIMARY_KEY)
		            .addRealmObjectField("favoriteDog", schema.get("Dog"))
		            .addRealmListField("dogs", schema.get("Dog"));
		        oldVersion++;
		     }
		  }
	
	}
	```
	
2. 使用`Builder.migration`升级数据库

	将版本号改为2，当Realm发现新旧版本号不一致时，会自动使用该迁移类完成迁移操作。
	
	```java
	RealmConfiguration config = new RealmConfiguration.Builder()
	    .schemaVersion(2) // 在schema改变后，必须进行升级
	    .migration(new MyMigration()) // 开始迁移
	    .build()
	```

## 其他
写到这里，基本的内容就差不多讨论完了，事实上，Realm还有很多其他的玩法，感兴趣的话，可以戳[这里](https://realm.io/)，详细的了解。

我自己写了一个应用[Espresso是一款基于MVP(Model-View-Presenter)架构, Material Design设计风格, 采用RxJava2, Retrofit2, Realm数据库和ZXing开发的快递追踪应用](https://github.com/TonnyL/Espresso)，用到了Realm，感兴趣的话，可以看看。

本文同步发表于[TonnyL的简书](http://www.jianshu.com/p/42584d637fb1), [TonnyL的博客](https://tonnyl.github.io/2017/04/03/Realm-Java-%E7%9A%84%E9%82%A3%E4%BA%9B%E4%BA%8B/), [知乎专栏Tonny的咖啡馆-人·技术·生活](https://zhuanlan.zhihu.com/p/26173366)，转载请注明作者和出处。

## 参考资料

[Realm:Create reactive mobile apps in a fraction of time](https://realm.io/)

[Android下Realm使用的2、3事](https://youngytj.github.io/2017/01/01/Realm-Java/)

[Android Realm入坑指南](http://www.jianshu.com/p/cd368b1996a7)

[说说 Realm 在 Android 上的坑](https://juejin.im/entry/5827f4dac4c971005438378f)

[【Android】Realm详解](http://www.jianshu.com/p/37af717761cc)

[在Android中使用Realm作本地存储](http://www.jianshu.com/p/8cb639a78975)

以上所有测试均基于Realm 3.0, 设备为OnePlus 3(Android 7.1.1), 环境为macOS 10.12.4.