---
layout: page
title: Interceptor Chain Pattern And Event Model
author: "Daeho Han"
permalink: /Archives
tags: pattern
---

> - 참고 : http://wildpup.cafe24.com/archives/596

## 1. Intercepting Filter Pattern
Intercepting Filter Pattern은 J2EE기반 기술에서 적용된 Pattern으로 요청이나 응답의 전/후로 어떤 작업을 수행 하고자 할 때 사용할 수 있다.

 Interception Filter Pattern은 다음과 같은 요소로 이루어져 있다. 

 > - Filter : 요청이나 응답 전/후에 처리해야할 작업
 > - Filter Chain : 여러개의 Filter를 가지고 연속적으로 처리하게 할 수 있으며 Target 설정 및 Filter를 추가
 > - Target : 실제 요청 대상
 > - Filter Manager : Filter들과 FilterChain을 관리 한다.

![Intercepting Filter Pattern Diagram](https://opendevelopergroup.github.io/assets/daeho/netty/intercepting_filter_pattern_diagram.png)

 code는 다음과 같다
 
 ```
 public interface Filter {
     public void execute(String request);
 }
 ```

 ```
 public class AuthenticationFilter Implements Filter {
     @Override
     public void execute(String request) {
         // TODO Authenticate
     }
 }
 public class LoggingFilter Implements Filter {
     @Override
     public void execute(String request) {
         // TODO Logging
     }
 }
 ```

 ```
 public class Target {
     public void execute(String request) {
         System.out.println("Executing request" + request);
     }
 }
 ```

 ```
 public class FilterChain {
     private List<Filter> filters = new ArraList<Filter>();
     private Target target;
     
     public void addFilter(Filter filter) {
         filters.add(filter);
     }
     public void execute(String request) {
         for(Filter filter : filters) {
             filter.execute(request);
         }
         target.execute(request);
     }
     public void setTarget(Target target) {
         this.target = target;
     }
 }
 ```

 ```
 public class FilterManager {
     FilterChain filterChain;

     public FilterManager(Target target) {
         filterChain = new FilterChain();
         filterChain.setTarget(target);
     }

     public void setFilter(Filter filter) {
         filterChain.addFilter(filter);
     }

     public void filterRequest(String request) {
         filterChain.execute(request);
     }
 }
 ```

 ```
 public class Client {
     FilterManager filterManager;

     public void setFilterManager(FilterManager filterManager) {
         this.filterManager = filterManager;
     }

     public void sendRequest(String request) {
         filterManager.filterRequest(request);
     }
 }
 ```

 ```
 public class InterceptingFilter {
     public static void main(String[] args) {
         FilterManager filterManager = new FilterManager(new Target());
         filterManager.setFilter(new AuthenticationFilter());
         filterManager.setFilter(new DebugFilter());

         Client client = new Client();
         client.setFilterManager(filterManager);
         clinet.sendRequest("INTECEPTING FILTER EXAM!");
     }
 }
 ```

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

```
ChannelPipeline p = ...;
p.addLast("1", new InboundHandlerA());
p.addLast("2", new InboundHandlerB());
p.addLast("3", new OutboundHandlerA());
p.addLast("4", new OutboundHandlerB()));
p.addLast("5", new InboudOutboundHandlerX()));
```
위와 같이 추가해주면 pipeline에는 inbound시에는 1,2,3,4,5순으로 outbound시에는 5,4,3,2,1 순으로 작동하게 되지만
inbound 시에는 실제로 1,2,5번 handler만 작동하고 outbound시에는 5,4,3이 작동하게 된다.
