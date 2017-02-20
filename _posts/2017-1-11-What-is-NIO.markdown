---
layout:     post
title:      What is NIO
author:     theankang
tags: 		Java NIO
subtitle:  	
category:  
---
## 2017-1-11 10:58:18
  
### 传统的BIO
  - 传统的同步阻塞I/O(Blocking I/O)
  - 对于**事件**的读，写分别建立线程
  - 线程的建立，切换开销很大
  - 之所以需要多线程，是因为在进行I/O操作的时候，一是没有办法知道到底能不能写、能不能读，只能"傻等"

---
### **NIO**
  - Non-blocking I/O
  - 同步非阻塞I/O模型
  - 使用了Reactor(反应器模式)
> NIO由原来的阻塞读写（占用线程）变成了单线程轮询事件，找到可以进行读写的网络描述符进行读写。除了事件的轮询是阻塞的（没有可干的事情必须要阻塞），剩余的I/O操作都是纯CPU操作，没有必要开启多线程。

``` Java
    interface ChannelHandler{
        void channelReadable(Channel channel);
        void channelWritable(Channel channel);
    }
    class Channel{
        Socket socket;
        Event event;//读，写或者连接
    }
    //IO线程主循环:
    class IoThread extends Thread{
        public void run(){
            Channel channel;
            while(channel=Selector.select()){//选择就绪的事件和对应的连接
                if(channel.event==accept){
                    registerNewChannelHandler(channel);//如果是新连接，则注册一个新的读写处理器
                }
                if(channel.event==write){
                    getChannelHandler(channel).channelWritable(channel);//如果可以写，则执行写事件
                }
                if(channel.event==read){
                    getChannelHandler(channel).channelReadable(channel);//如果可以读，则执行读事件
                }
            }
        }
        Map<Channel，ChannelHandler> handlerMap;//所有channel的对应事件处理器
    }
```

---

#### **NIO线程包括以下3种**
- 事件分发器
  - 单线程选择就绪的事件。
- I/O 处理器
  - 包括connect、read、write等，这种纯CPU操作，一般开启CPU核心个线程就可以。
- 业务线程
  - 在处理完I/O后，业务一般还会有自己的业务逻辑，有的还会有其他的阻塞I/O，如DB操作，RPC等。只要有阻塞，就需要单独的线程。
  
---

- 一般情况下，I/O 复用机制需要事件分发器（event dispatcher）
- 开始时在分发器那里注册感兴趣的事件，并提供相应的处理者（event handler)
- 事件分发器的两种模式
  - Reactor
      - 同步I/O
  - Proactor 
      - 异步I/O

http://tutorials.jenkov.com/java-nio/index.html
最后先扔一个链接, 有时间细细看一下(先把这个坑挖到这里, 不过估计是填不上了...).
