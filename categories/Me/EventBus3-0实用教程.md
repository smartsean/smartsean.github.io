---
title: EventBus3.0实用教程
date: 2017.03.02 18:54:02
tags: Android
category: Android
---
> 作为一个`Android`开发者，我们在日常的开发中肯定会使用到`EventBus`，比如说当我们在做`app`的消息模块的时候，接收到后台推送的消息之后，为了方便用户查看，就需要把消息保存到本地，正常情况下在页面上会有个`badge`显示消息数量，如果我们不在`badge`显示界面的话，就需要在接收到后台推送之后更新`badge`上显示的消息数量，这个时候就可以使用`EventBus`发出一个事件，这样订阅者接收到事件之后，就会从数据库拿未读消息数，显示在`badge`上面。说了这么多，下面就简单的介绍下`EventBus`的使用：

`EventBus`地址：[GitHub](https://github.com/greenrobot/EventBus)

<!--more-->

## 一、EventBus 介绍
`ventBus`是一个`Android`端优化的`publish/subscribe`消息总线，简化了应用程序内各组件间、组件与后台线程间的通信。这个消息总线主要有三个部分：
1. 事件（Event）
2. 事件订阅者（Subscriber）（有没有想到`RxJava`里面的订阅者  ==。）
3. 事件发布者（Publisher）

[官方的关系图](https://github.com/greenrobot/EventBus)：

![官方关系图](http://upload-images.jianshu.io/upload_images/1744889-efc8dbfeb0789d26.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

特征叙述：

- 简化组件间的通信
  - 事件发送者和接收者解耦
  - 在活动、片段和后台线程中执行良好
  - 避免了复杂、易出错的依赖关系和生命周期问题
- 使你的代码更加简单
- 快！
- 小！（大约50K）
- 在100,000,000+个程序上使用
- 先进特征，比如指定线程、设置优先级等

## 二、使用EventBus仅需四步

### 1. 添加依赖
使用`Gradle`：
``` java
compile 'org.greenrobot:eventbus:3.0.0'
```
或者`Maven`：
``` java
<dependency>
    <groupId>org.greenrobot</groupId>
    <artifactId>eventbus</artifactId>
    <version>3.0.0</version>
</dependency>
```
又或者下载`Jar`包添加到项目中
[jar包下载](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22org.greenrobot%22%20AND%20a%3A%22eventbus%22)

### 2. 定义事件（Event）
``` java
public class TestMsg {
}

```

这个`TestMsg`由从事件发布者发出，到事件订阅者接收，当然也可以加上额外的信息，比如下面可以传递`name`：

``` java
public class TestMsg {
    private String name;

    public TestMsg(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```
### 3. 定义事件接收者（Subscriber）

首先在所属的`Activity`的`onCreate()`里面注册
``` java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_my_lib);
        EventBus.getDefault().register(this);
    }
```

在`onDestory`里面取消注册
``` java
@Override
    protected void onDestroy() {
        super.onDestroy();
        EventBus.getDefault().unregister(this);
    }
```
然后定义处理事件：
``` java
@Subscribe(threadMode = ThreadMode.MAIN)
    public void onEventMainThread(TestMsg testMsg) {

        if (testMsg != null) {
            Log.d("MainActivity", "你收到的名字为： "+testMsg.getName());
        }
    }
```
这里的定义了事件接收者以及使用注解`@Subscribe(threadMode = ThreadMode.MAIN)`指定了执行的线程。`ThreadMode`有下面四种类型：

1. ` MAIN UI`主线程
2. `POSTING` 默认调用方式，在调用post方法的线程执行，避免了线程切换，性能开销最少

3. `BACKGROUND` 如果调用`post`方法的线程不是主线程，则直接在该线程执行。
如果是主线程，则切换到后台单例线程，多个方法公用同个后台线程，按顺序执行，避免耗时操作

4. `ASYNC` 开辟新独立线程，用来执行耗时操作，例如网络访问。

当然这里可以在注解里面设置优先级，比如下面设置优先级为100，越大就越线先接收到事件：
``` java
@Subscribe(threadMode = ThreadMode.POSTING,priority = 100)
    public void onEventMainThread(TestMsg testMsg) {

        if (testMsg != null) {
            Log.d("MainActivity", "你收到的名字为： "+testMsg.getName());
        }
    }
```

如果你有三个接收事件，并且设置了不同的优先级，比如100、50、10,你也可以在`priority = 100`的接收到之后取消事件的传递，那么`priority` = 50和10的就不会接收到事件了。但是请注意，只能在`ThreadMode.PostThread`类型的才能取消，其他的三种`ThreadMode`类型是不能取消的。

如何设置：

``` java
EventBus.getDefault().cancelEventDelivery(event) ;
```

### 4.定义事件发布者（Publisher）发出事件

``` java
sendMessage.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                EventBus.getDefault().post(new TestMsg("测试EventBus"));
            }
        });
```

我写了一个小`Demo`，这个`Demo`是我在主`module`，也就是在`app`下面定义了事件接收者，在主`app`依赖的`module`下面发出事件来测试的。结果是可行的。有兴趣的小伙伴可以去看下：

[Demo地址](https://github.com/Seancss/EventBusTest)
