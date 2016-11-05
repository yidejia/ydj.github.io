---
title: iOS - 接入WebSocket记录 + 一些个人经验
date: 2016-11-04 14:12:55
tags:
- 曹堃
- websocket
categories:
- 移动端

---

# 闲扯
WebSocket 以前没用过，之前写过一篇博客是基于原生socket的（[查看](http://www.jianshu.com/p/edb50afa250e)）比较复杂，慎入。今天另外一个APP需要接websocket了，然后便找到了facebook的 [SocketRocket](https://github.com/facebook/SocketRocket) 框架，然后用了一天时间接上了，完成了掉线自动重连，自动重登录，心跳等等功能，用法比原生socket简单（原生socket基于TCP/UDP协议）。

# 为什么用 WebSocket
因为APP里面有个聊天功能，需要服务器主动推数据到APP。HTTP 通信方式只能由客户端主动拉取，服务器不能主动推给客户端，如果有实时的消息，要立刻通知客户端就麻烦了，要么客户端每隔几秒钟发一次请求，看看有没有新数据，这种方式想想都知道耗流量电量。还一种方式就是走TCP/UDP协议服务器主动推给你，这种方式省流量。还有就是用websocket，websocket是h5里面的东西，h5我不太会，反正它比原生socket用法简单。

# 用法
用 SocketRocket 框架，记住几个代理方法就好了，很简单。
## 1.创建和设置代理对象

```
SRWebSocket *socket = [[SRWebSocket alloc] initWithURLRequest:
[NSURLRequest requestWithURL:[NSURL URLWithString:@"http://ip地址:端口"]];

socket.delegate = self;    // 实现这个 SRWebSocketDelegate 协议啊

[socket open];    // open 就是直接连接了
```

## 2.连接成功会调用这个代理方法

```
- (void)webSocketDidOpen:(SRWebSocket *)webSocket {
    NSLog(@"连接成功，可以立刻登录你公司后台的服务器了，还有开启心跳");
}
```
## 3.连接失败会调用这个方法，看 NSLog 里面的东西

```
- (void)webSocket:(SRWebSocket *)webSocket didFailWithError:(NSError *)error {
    NSLog(@"连接失败，这里可以实现掉线自动重连，要注意以下几点");
    NSLog(@"1.判断当前网络环境，如果断网了就不要连了，等待网络到来，在发起重连");
NSLog(@"2.判断调用层是否需要连接，例如用户都没在聊天界面，连接上去浪费流量");
NSLog(@"3.连接次数限制，如果连接失败了，重试10次左右就可以了，不然就死循环了。
或者每隔1，2，4，8，10，10秒重连...f(x) = f(x-1) * 2, (x<5)  f(x)=10, (x>=5)");
}
```
## 4.连接关闭调用这个方法，注意连接关闭不是连接断开，关闭是 [socket close] 客户端主动关闭，断开可能是断网了，被动断开的。

```
- (void)webSocket:(SRWebSocket *)webSocket didCloseWithCode:(NSInteger)code reason:(NSString *)reason wasClean:(BOOL)wasClean {
    NSLog(@"连接断开，清空socket对象，清空该清空的东西，还有关闭心跳！");
}
```
## 5.收到服务器发来的数据会调用这个方法

```
- (void)webSocket:(SRWebSocket *)webSocket didReceiveMessage:(id)message  {
NSLog(@"收到数据了，注意 message 是 id 类型的，学过C语言的都知道，id 是 (void *)  
void* 就厉害了，二进制数据都可以指着，不详细解释 void* 了");
NSLog(@"我这后台约定的 message 是 json 格式数据
收到数据，就按格式解析吧，然后把数据发给调用层");
}
```

## 6.向服务器发送数据
发送的时候可能断网，可能socket还在连接，要判断一些情况，写在下面了
发送逻辑是，我有一个 socketQueue 的**串行**队列，发送请求会加到这个队列里，然后一个一个发出去，如果掉线了，重连连上后继续发送，**对调用层透明，调用层不需要知道网络断开了**。

```
- (void)sendData:(id)data {
    WEAKSELF(ws);
    dispatch_async(self.socketQueue, ^{
        if (ws.socket != nil) {
// 只有 SR_OPEN 开启状态才能调 send 方法啊，不然要崩
            if (ws.socket.readyState == SR_OPEN) {
                [ws.socket send:data];    // 发送数据

            } else if (ws.socket.readyState == SR_CONNECTING) {
                NSLog(@"正在连接中，重连后其他方法会去自动同步数据");
// 每隔2秒检测一次 socket.readyState 状态，检测 10 次左右
// 只要有一次状态是 SR_OPEN 的就调用 [ws.socket send:data] 发送数据
// 如果 10 次都还是没连上的，那这个发送请求就丢失了，这种情况是服务器的问题了，小概率的
// 代码有点长，我就写个逻辑在这里好了

            } else if (ws.socket.readyState == SR_CLOSING || ws.socket.readyState == SR_CLOSED) {
// websocket 断开了，调用 reConnect 方法重连
                [ws reConnect:^{
                    NSLog(@"重连成功，继续发送刚刚的数据");
[ws.socket send:data];
                }];
            }
        } else {
            NSLog(@"没网络，发送失败，一旦断网 socket 会被我设置 nil 的");
NSLog(@"其实最好是发送前判断一下网络状态比较好，我写的有点晦涩，socket==nil来表示断网");
        }
    });
}
```

## 7.心跳机制
心跳机制就不难了，开个定时器，问下后台要每隔多少秒发送一次心跳请求就好了。然后注意，断网了或者socket断开的时候把心跳关一下，省资源，不然都断网了，还在循环发心跳，浪费CPU和电量。


终于接完websocket了，下班回家压压惊。我第一次用，其实不难，就是考虑的情况比较多，整个逻辑有点多，主要代码就是上面那些了，其他不重要的代码我就不复制粘贴上来了。
