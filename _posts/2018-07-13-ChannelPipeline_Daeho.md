---
layout: page
title: ChannelPipeline 정리
author: "Daeho Han"
permalink: /Archives
tags: pattern
---

## 2. *ChannelPipeline*
*ChannelHandler*들은 *Channel*의 *ChannelEvent*들을 처리하거나 인터셉트 한다.
*ChannelPipeline*은 *Intercepting Filter* 패턴의 진보된 구현체이며 이용자에게 이벤트 와 ChannelHandler들을 컨트롤 할 수 있도록 해준다.

### Creation of pipeline
 각각의 새로운 channel들은 새로운 파이프라인을 만들어 연결해야 한다. 일단 연결된 channel과 pipeline은 영원히 연결되어있게 된다. channel은 다른 pipeline과 붙을 수 없고 현재의 pipeline과 떨어질수도 없다.

새로운 pipline을 생성하는 것은 각각의 생성자를 콜하는것 보다 *Channels* 클래스 안에 있는 메소드를 사용하는 것이 좋다

channel들은 각각 자신의 pipeline을 가지고 있으며 새로운 channel이 생성될때 자동으로 생성된다.

### How an event flows in pipeline

아래의 다이어그램은 *ChannelPipeline*의 *ChannelHandler*들에 의해 어떻게 I/O event들이 처리되는지 보여주고 있다. 
I/O event는 각각 *ChannelInboundHandler* 또는 *ChannelOutboundHandler*에 의해 처리되며 *ChannelHandlerContext.fireChannelRead(Object)*와 *ChannelHandlerContext.write(Object)*와 같은 *ChannelHandlerContext*에 정의된 이벤트 전파 메소드들을 호출함으로써 가장 가까운 handler에게 전달되게 된다. 

```
                                          I/O Request
                                            via Channel or
                                        ChannelHandlerContext
                                                      |
  +---------------------------------------------------+---------------+
  |                           ChannelPipeline         |               |
  |                                                  \|/              |
  |    +---------------------+            +-----------+----------+    |
  |    | Inbound Handler  N  |            | Outbound Handler  1  |    |
  |    +----------+----------+            +-----------+----------+    |
  |              /|\                                  |               |
  |               |                                  \|/              |
  |    +----------+----------+            +-----------+----------+    |
  |    | Inbound Handler N-1 |            | Outbound Handler  2  |    |
  |    +----------+----------+            +-----------+----------+    |
  |              /|\                                  .               |
  |               .                                   .               |
  | ChannelHandlerContext.fireIN_EVT() ChannelHandlerContext.OUT_EVT()|
  |        [ method call]                       [method call]         |
  |               .                                   .               |
  |               .                                  \|/              |
  |    +----------+----------+            +-----------+----------+    |
  |    | Inbound Handler  2  |            | Outbound Handler M-1 |    |
  |    +----------+----------+            +-----------+----------+    |
  |              /|\                                  |               |
  |               |                                  \|/              |
  |    +----------+----------+            +-----------+----------+    |
  |    | Inbound Handler  1  |            | Outbound Handler  M  |    |
  |    +----------+----------+            +-----------+----------+    |
  |              /|\                                  |               |
  +---------------+-----------------------------------+---------------+
                  |                                  \|/
  +---------------+-----------------------------------+---------------+
  |               |                                   |               |
  |       [ Socket.read() ]                    [ Socket.write() ]     |
  |                                                                   |
  |  Netty Internal I/O Threads (Transport Implementation)            |
  +-------------------------------------------------------------------+

  ```

다음과 같이 pipeline에 handler들을 추가해 줄 수 있습니다.

```java
ChannelPipeline p = ...;
p.addLast("1", new InboundHandlerA());
p.addLast("2", new InboundHandlerB());
p.addLast("3", new OutboundHandlerA());
p.addLast("4", new OutboundHandlerB()));
p.addLast("5", new InboudOutboundHandlerX()));
```
위와 같이 추가해주면 pipeline에는 inbound시에는 1,2,3,4,5순으로 outbound시에는 5,4,3,2,1 순으로 작동하게 되지만
inbound 시에는 실제로 1,2,5번 handler만 작동하고 outbound시에는 5,4,3이 작동하게 된다.

-- 2018.07.17
### *다음 Handler로 이벤트 전달*
위의 그림에서 알 수 있듯이, 다른 handler로 이벤트를 전달하기 위해서 *ChannelHandlerContext*의 이벤트 전파 메소드들을 호출해야만 한다. 

- Inbound 이벤트 전파 메소드 : 
  - ChannelHandlerContext.fireChannelRegistered()
  - ChannelHandlerContext.fireChannelActive()
  - ChannelHandlerContext.fireChannelRead(Object)
  - ChannelHandlerContext.fireChannelReadComplete()
  - ChannelHandlerContext.fireExceptionCaught(Throwable)
  - ChannelHandlerContext.fireUserEventTriggered(Object)
  - ChannelHandlerContext.fireChannelWritabilityChanged()
  - ChannelHandlerContext.fireChannelInactive()
  - ChannelHandlerContext.fireChannelUnregistered()
- Outbound 이벤트 전파 메소드 : 
  - ChannelHandlerContext.bind(SocketAddress, ChannelPromise)
  - ChannelHandlerContext.connect(SocketAddress, SocketAddress, ChannelPromise)
  - ChannelHandlerContext.write(Object, ChannelPromise)
  - ChannelHandlerContext.flush()
  - ChannelHandlerContext.read()
  - ChannelHandlerContext.disconnect(ChannelPromise)
  - ChannelHandlerContext.close(ChannelPromise)
  - ChannelHandlerContext.deregister(ChannelPromise) 
예시는 아래와 같다

```java
  public class MyInboundHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelActive(ChannelHandlerContext ctx) {
      System.out.println("Connected!");
      ctx.fireChannelActive();
    }
  }

  public class MyOutboundHandler extends ChannelOutboundHandlerAdapter {
    @Override
    public void close(ChannelHandlerContext ctx, ChannelPromise promise) {
      System.out.println("Closing ...");
      ctx.close(promise);
    }
  }
```

### *pipeline 만들기*
 user는 I/O 명령(e.g. write and close)을 요청하거나 I/O 이벤트(e.g. read)를 받기위해서는 pipeline에 하나이상의 *ChannelHandler*를 가지고 있어야 한다. 
일반적인 서버는 다음과 같은 핸들러들을 가지고 있는다. 
1. Protocol Decoder - binary 데이터를 Java 객체로 변환(e.g. ByteBuf)
2. Protocol Encoder - Java 객체를 binary 데이터로 변환
3. Business Logic Handler - 실제 비즈니스 로직을 수행 (e.g. db 접근)
그리고 이에 따른 예제는 다음과 같습니다.

```java
static final EventExecutorGroup group = new DefaultEventExecutorGroup(16);
...
ChannelPipeline pipeline = ch.pipeline();

pipeline.addLast("decoder", new MyProtocolDecoder());
pipeline.addLast("encoder", new MyProtocolEncoder());
...
pipeline.addLast(group, "handler", new MyBusinessLogicHandler());
```