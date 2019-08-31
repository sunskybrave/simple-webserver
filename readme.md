一个简单的web服务器，参考了《linux高性能服务器编程》中第15章的代码  
# 1.基本功能
客户发送http请求到服务器，服务器处理完请求的内容后返回给客户

# 2.基本框架
参考了很多资料，使用半同步半反应堆模式，在主线程中使用epoll进行监听，当有事件发生时对事件进行筛选，如果是可读或可写事件则分发给线程池处理  
针对半同步半异步方式的缺点稍微进行了改进， 最好的改进方法是在主线程只负责监听连接套接字，每个工作线程在自己的epoll中注册此套接字，从而避免了主工作请求队列的频繁加锁解锁   

1)main函数主循环
```c++
while(server running)
{
    epoll等待事件;  //统一I/O和信号事件源
    if(新连接到达且是有效连接)
    {
        accept此连接;
        将此连接设置为non-blocking;
        为此连接设置event(EPOLLIN | EPOLLET ...);
        将此连接加入epoll监听队列;
        从线程池取一个空闲工作者线程并处理此连接;
    }
    else if(读请求)
    {
        线程池处理读请求;
    }
    else if(写请求)
    {
        线程池处理写请求;
    }
    else if(捕捉到信号)
    {
        处理信号
    }
    else
    {
        其他事件
    }           
}
```
2)参考了统一事件源的方式，在epoll上实现对I/O和信号的监听，同时也防止了信号不排队导致的信号丢失  

*信号处理函数中不完成信号的逻辑处理，当信号处理函数被触发时，它只是简单的通知主循环接收到信号，主循环再根据接收到的信号值执行目标信号对应的逻辑代码。信号处理函数使用管道来将信号传递给主循环：信号处理函数往管道的写端写入信号值，主循环从管道的读端读出该信号值。此后在主循环中使用I/O复用系统来监听管道的读端文件描述符上的可读事件。即实现了统一事件源。*

# 3.编写遇到的一些问题及思考  

1)有限状态机    

实现HTTP请求的读取和分析，目前只支持http1.0，后续打算更新到支持http2.0  

# 4.施工进展
1）线程池模块基本功能编写     
2）初探并发下的日志系统设计，主要看了陈硕的《Linux多线程服务器端编程》   

# 5.附加功能计划
1）目前支持http1.0，比较老，后续增加对新协议的支持

2）增加Mysql数据库
