---
layout: page
title: ChannelPipeline ����
author: "Daeho Han"
permalink: /archives
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