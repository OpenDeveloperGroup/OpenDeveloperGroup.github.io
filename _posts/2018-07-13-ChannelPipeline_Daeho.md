---
layout: page
title: ChannelPipeline ����
author: "Daeho Han"
permalink: /Archives
tags: pattern
---

## 2. *ChannelPipeline*
*ChannelHandler*���� *Channel*�� *ChannelEvent*���� ó���ϰų� ���ͼ�Ʈ �Ѵ�.
*ChannelPipeline*�� *Intercepting Filter* ������ ������ ����ü�̸� �̿��ڿ��� �̺�Ʈ �� ChannelHandler���� ��Ʈ�� �� �� �ֵ��� ���ش�.

### Creation of pipeline
 ������ ���ο� channel���� ���ο� ������������ ����� �����ؾ� �Ѵ�. �ϴ� ����� channel�� pipeline�� ������ ����Ǿ��ְ� �ȴ�. channel�� �ٸ� pipeline�� ���� �� ���� ������ pipeline�� ���������� ����.

���ο� pipline�� �����ϴ� ���� ������ �����ڸ� ���ϴ°� ���� *Channels* Ŭ���� �ȿ� �ִ� �޼ҵ带 ����ϴ� ���� ����

channel���� ���� �ڽ��� pipeline�� ������ ������ ���ο� channel�� �����ɶ� �ڵ����� �����ȴ�.

### How an event flows in pipeline

�Ʒ��� ���̾�׷��� *ChannelPipeline*�� *ChannelHandler*�鿡 ���� ��� I/O event���� ó���Ǵ��� �����ְ� �ִ�. 
I/O event�� ���� *ChannelInboundHandler* �Ǵ� *ChannelOutboundHandler*�� ���� ó���Ǹ� *ChannelHandlerContext.fireChannelRead(Object)*�� *ChannelHandlerContext.write(Object)*�� ���� *ChannelHandlerContext*�� ���ǵ� �̺�Ʈ ���� �޼ҵ���� ȣ�������ν� ���� ����� handler���� ���޵ǰ� �ȴ�. 

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

������ ���� pipeline�� handler���� �߰��� �� �� �ֽ��ϴ�.

```java
ChannelPipeline p = ...;
p.addLast("1", new InboundHandlerA());
p.addLast("2", new InboundHandlerB());
p.addLast("3", new OutboundHandlerA());
p.addLast("4", new OutboundHandlerB()));
p.addLast("5", new InboudOutboundHandlerX()));
```
���� ���� �߰����ָ� pipeline���� inbound�ÿ��� 1,2,3,4,5������ outbound�ÿ��� 5,4,3,2,1 ������ �۵��ϰ� ������
inbound �ÿ��� ������ 1,2,5�� handler�� �۵��ϰ� outbound�ÿ��� 5,4,3�� �۵��ϰ� �ȴ�.

-- 2018.07.17
### *���� Handler�� �̺�Ʈ ����*
���� �׸����� �� �� �ֵ���, �ٸ� handler�� �̺�Ʈ�� �����ϱ� ���ؼ� *ChannelHandlerContext*�� �̺�Ʈ ���� �޼ҵ���� ȣ���ؾ߸� �Ѵ�. 

- Inbound �̺�Ʈ ���� �޼ҵ� : 
  - ChannelHandlerContext.fireChannelRegistered()
  - ChannelHandlerContext.fireChannelActive()
  - ChannelHandlerContext.fireChannelRead(Object)
  - ChannelHandlerContext.fireChannelReadComplete()
  - ChannelHandlerContext.fireExceptionCaught(Throwable)
  - ChannelHandlerContext.fireUserEventTriggered(Object)
  - ChannelHandlerContext.fireChannelWritabilityChanged()
  - ChannelHandlerContext.fireChannelInactive()
  - ChannelHandlerContext.fireChannelUnregistered()
- Outbound �̺�Ʈ ���� �޼ҵ� : 
  - ChannelHandlerContext.bind(SocketAddress, ChannelPromise)
  - ChannelHandlerContext.connect(SocketAddress, SocketAddress, ChannelPromise)
  - ChannelHandlerContext.write(Object, ChannelPromise)
  - ChannelHandlerContext.flush()
  - ChannelHandlerContext.read()
  - ChannelHandlerContext.disconnect(ChannelPromise)
  - ChannelHandlerContext.close(ChannelPromise)
  - ChannelHandlerContext.deregister(ChannelPromise) 
���ô� �Ʒ��� ����

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

### *pipeline �����*
 user�� I/O ���(e.g. write and close)�� ��û�ϰų� I/O �̺�Ʈ(e.g. read)�� �ޱ����ؼ��� pipeline�� �ϳ��̻��� *ChannelHandler*�� ������ �־�� �Ѵ�. 
�Ϲ����� ������ ������ ���� �ڵ鷯���� ������ �ִ´�. 
1. Protocol Decoder - binary �����͸� Java ��ü�� ��ȯ(e.g. ByteBuf)
2. Protocol Encoder - Java ��ü�� binary �����ͷ� ��ȯ
3. Business Logic Handler - ���� ����Ͻ� ������ ���� (e.g. db ����)
�׸��� �̿� ���� ������ ������ �����ϴ�.

```java
static final EventExecutorGroup group = new DefaultEventExecutorGroup(16);
...
ChannelPipeline pipeline = ch.pipeline();

pipeline.addLast("decoder", new MyProtocolDecoder());
pipeline.addLast("encoder", new MyProtocolEncoder());
...
pipeline.addLast(group, "handler", new MyBusinessLogicHandler());
```