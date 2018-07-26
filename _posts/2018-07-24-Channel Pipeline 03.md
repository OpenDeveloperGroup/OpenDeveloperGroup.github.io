---
layout: page
title: ChannelPipeline 정리 3
author: "Daeho Han"
permalink: /Archives
tags: daeho
---

## ChannelPipline 흐름
```
  +---------+  1 .. 1   +----------+  1    n  +---------+  n   m  +---------+
  | Channel | --------  | Pipeline | -------- | Context | ------- | Handler |
  +---------+           +----------+          +---------+         +---------+
                                                   | 1..1              |
                                              +------------+           |
                                              | Attachment | <---------+
                                              +------------+
```

- Client가 접속하면 Channel과 Pipeline이 생성된다. 
- 파이프라인은 여러개의 핸들러를 가질 수 있다.
- 파이프라인과 각각의 핸들러 관계 정보등을 가지고 있는 Context가 있다.

### Channel Lifecycle states
- Active
    connected to remote peer
- Inactive
    disconnected from remote peer
- Unregistered
    Channel이 생성된 상태
- Registerd
    Channel이 event loop와 연관된 상태

### ChannelHandler Lifecycle Events

- handlerAdded()
    ChannelPipeline에 추가됨
- handlerRemoved()
    ChannelPipeline에서 제거됨
- exceptionCaught() 
    ChannelPipeline 처리 중 에러 발생

### ChannelInboudnHandler lifecycle methods for Channel

- channelRegistered() 
    Event loop 와 Channel을 연결
- channelUnRegistered()
    Event loop 와  Channel 연결을 해제
- channelActive()
    connected
- channelInactive()
    disconnected
- channelReadComplete()
    read operation 완료
- channelRead()
    Channel의 데이터를 읽기
- channelWritabilityChanged() 
- userEventTriggered()
    event loop에 POJO를 누군가 전달함

### ChannelOutboundHandler
- bind(channelHandlerContext, localSocketAddress, channelPromise)

- connect(channelHandlerContext, socketAddress, channelPromise)

- disconnect(channelHandlerContext, channelPromise)

- close(channelHandlerContext, channelPromise)
