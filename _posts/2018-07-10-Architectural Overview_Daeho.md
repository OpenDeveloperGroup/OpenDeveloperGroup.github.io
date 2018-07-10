---
layout: page
title: Architectural Overview
author: "Daeho Han"
permalink: /
tags: architecture
---

# Chapter 2. 아키텍쳐 개요
![netty architecture](https://opendevelopergroup.github.io/assets/daeho/netty/architecture.png)

이번 챕터에서는, Netty에서 제공하는 core 기능을 알아보고 어떻게 그 기능들이 완벽한 네트워크 어플리케이션 개발에 도움을 주는지 설명할 것 입니다.

## 2.1 풍부한 Buffer 자료구조
Netty는 순차적인 byte를 표현하는데 *NIO ByteBuffer* 대신에 자신만의 buffer API를 사용합니다. 이것은 *ByteBuffer*를 사용하는 것 보다 확실한 이점을 가집니다.  
Netty의 새로운 buffer 타입인 *ChannelBuffer* 는 *ByteBuffer*의 문제를 해결하고 네트워크 어플리케이션 개발자의 요구사항을 충족시키기 위해 디자인 되었습니다. 아래에 몇몇 특징들이 나열되어있습니다.

> 필요하다면 자신만의 buffer를 정의할 수 있다.  
> Transparent zero copy is achieved by a built-in composite buffer type.  
> *StringBuffer*처럼 capacity가 요구에 따라 확장되는 동적인 buffer 타입이 제공된다  
> flip()을 더 이상 call할 필요가 없다.  
> ByteBuffer보다 더 빠르기도 하다.  

## 2.2 범용 비동기 I/O API
Java에서 제공하는 전통적인 I/O API에서 다른 타입과 메소드는 다른 전송 타입을 제공한다. 예를 들어, *java.net.Socket* 과 *java.net.DatagrmSocket*은 공통으로 상속하는 타입을 가지고 있지 않다. 그러므로 그들은 Socket I/O를 매우 다른 방향으로 수행한다.  

이러한 불일치는 네트워크 어플리케이션이 한 전송을 다른 곳으로 포팅하는데 지루하고 어렵게 합니다. 
전송 간의 이동성 부족은 추가 전송이 필요할 경우 응용프로그램의 네트워크 layer를 재작성 해야하는 문제가 발생합니다.
이론적으로 많은 프로토콜은 TCP/IP, UDP/IP, SCTP 그리고 시리얼 포트 통신과 같은 하나 이상의 전송에서 실행할 수 있습니다.  

설상가상으로, Java의 NIO API(New I/O API)는 OIO API(Old blocking I/O API)와 호환이 되지 않는다고 소개되었고 다음 릴리즈인 NIO.2(AIO)에서도 계속 그럴것이다.
이러한 API들은 서로 설계와 성능 특징들이 가지각색이기 때문에 구현 단계에 이르기도 전에 당신의 어플리케이션이 의존할 API를 결정해야 할때도 있다.   

예를 들어, 만약 제공할 클라이언트 수가 매우 작고 NIO를 사용하는것 보다 OIO를 사용하는것이 훨씬 쉽기 때문에 OIO로 개발을 하고 싶어합니다.
그러나 당신의 비즈니스가 기하급수적으로 성장하고 서버가 수만개의 클라이언트를 동시적으로 지원해야하는 문제가 생길 수도 있습니다.
그렇다면 NIO로 개발하는것을 생각하게 됩니다. NIO Select API의 복잡성으로 인해 개발 시간이 너무 오래 걸릴 수도 있습니다.

Netty는 *Channel* 이라는 범용 비동기 I/O 인터페이스를 가지고 있습니다. 이것은 point-to-point 통신에 필요한 모든 명령을 추상화 한것입니다.
일단 당신의 어플리케이션이 하나의 Netty Transport 에서 작성된다면, 그 어플리케이션은 다른 Netty Tranport들에서도 동작할수있습니다.
Netty는 하나의 범용 API를 통해 여러 필수 전송들을 제공합니다.

> NIO-based TCP/IP transport 
> OIO_based TCP/IP transport  
> OIO_based UDP/IP transport, and  
> Local transport  

하나의 전송에서 다른것으로 전환하는데는 다른 *ChannelFactory* 구현체를 선택하는 것과 같이 몇줄 정도는 변경이 필요하다. 

또한 직렬 포트 통신 전송과 같이 아직 작성되지 않은 새 전송을 활용할 수 있으며 몇 줄의 생성자 호출 만 교체 할 수도 있습니다. 
또한 핵심 API를 확장하여 자체 전송을 작성할 수 있습니다.

## 2.3  인터셉터 체인 패턴을 기반으로한 이벤트 모델
잘 정의되고 확장 가능한 이벤트 모델은 Event-Driven 어플리케이션의 필수 요소입니다. 
Netty는 I/O에 초점을 맞춘 잘 정의된 이벤트 모델을 가지고 있습니다. 
엄격한 계층에 의해 다른것과 구별된 각각의 이벤트 타입 때문에 당신이 기존의 코드를 손상시키지 않고 자신만의 이벤트 타입을 구현할수있도록 해줍니다. 
이것이 다른 프레임워크들과의 또 다들 차별점입니다.
많은 NIO 프레임워크들은 이벤트 모델의 개념이 거의 제한적이거나 없다. 당신이 커스텀 이벤트 타입을 추가하려고하면 그들은 기존 코드들을 손상시키게 된다.

*ChannelEvent*는 *ChannelPipeLine안의 *ChannelHandler*들에 의해 처리됩니다.
파이프라인은 이벤트가 처리되고 파이프라인 내부의 핸들러들이 상호작용하는 방법들을 사용자가  컨트롤 할수 있도록 *Interception Filter* 패턴의 진보된 형태를 구현합니다.
예를 들어, socket에서 데이터를 읽어올때 무엇을 할지 정의 할 수 있습니다.

```
public class MyReadHandler implements SimpleChannelHandler {
    public void messageReceived(ChannelHandlerContext ctx, MessageEvent evt) {
        Object message = evt.getMessage();
        // Do something with the received message
        ...
        // And forward the event to the next handler.
        ctx.sendUpstream(evt);
    }
}
```
또한 핸들러가 write 요청을 받을때 수행할 것을 정의 할 수도 있습니다.
```
public class MyWriteHandler implements SimpleChannelHandler {
    public void writeRequested(ChannelHandlerContext ctx, MessageEvent evt) {
        Object message = evt.getMessage();
        // Do something with the message to be written.
        ...

        // And forward the event to the next handler.
        ctx.sendDownstream(evt);
    }
}
```

## 2.4 빠른 개발을 위한 진보된 컴포넌트
위에 언급 된 핵심 구성 요소 위에는 모든 유형의 네트워크 응용 프로그램 구현이 이미 가능하므로 Netty는 개발 페이지를 더욱 빠르게 할 수있는 일련의 고급 기능을 제공합니다.

### 2.4.1 Codec framework
Chapter 1.8에서 설명한 것처럼 프로토콜 코덱과 비즈니스 로직을 분리하는 것이 좋습니다. 그러나이 아이디어를 처음부터 구현할 때 몇 가지 문제가 있습니다. 메시지의 단편화를 처리해야합니다. 일부 프로토콜은 다층 (즉, 다른 하위 프로토콜의 상단에 구축 됨)입니다. 일부는 단일 상태 시스템에서 구현하기에는 너무 복잡합니다.

결과적으로 훌륭한 네트워크 응용 프로그램 프레임 워크는 유지 보수가 가능한 사용자 코덱을 생성하는 확장 가능하고, 재사용 가능하며, 단위 테스트 가능하고, 다중 계층화 된 코덱 프레임 워크를 제공해야합니다.

Netty는 간단하거나 아니거나, 바이너리 또는 텍스트인지 여부와 관계없이 프로토콜 코덱을 작성할 때 발생할 수있는 대부분의 문제를 해결할 수있는 다양한 기본 및 고급 코덱을 제공합니다.

### 2.4.2 SSL / TLS Support
OIO와 달리 NIO에서 SSL을 지원하는 것은 간단하지 않습니다. 데이터를 암호화하거나 해독하기 위해 단순히 스트림을 래핑 할 수는 없지만 javax.net.ssl.SSLEngine을 사용해야합니다. SSLEngine은 SSL만큼이나 복잡한 상태 시스템입니다. 암호 모음 및 암호화 키 협상 (또는 다시 협상), 인증서 교환 및 유효성 검사와 같은 모든 가능한 상태를 관리해야합니다. 게다가, SSLEngine은 완전히 스레드 안전하지는 않습니다.

Netty에서 *SslHandler*는 *SSLEngine*의 모든 세부적인 내용과 함정을 처리합니다.
*SslHandler*를 구성하여 *ChannelPipeline*에 삽입하기 만하면됩니다. 
또한 *StartTLS*와 같은 고급 기능을 매우 쉽게 구현할 수 있습니다.

### 2.4.3 HTTP Implementation
HTTP는 인터넷에서 가장 많이 사용되는 프로토콜입니다. Servlet 컨테이너와 같은 많은 HTTP 구현이 이미 있습니다. 그렇다면 Netty는 왜 핵심 부분에 HTTP를 가지고 있습니까?

Netty의 HTTP 지원은 기존 HTTP 라이브러리와 매우 다릅니다. 낮은 수준에서 HTTP 메시지를 교환하는 방법을 완벽하게 제어 할 수 있습니다. 기본적으로 HTTP 코덱과 HTTP 메시지 클래스가 결합되어 있으므로 강제 스레드 모델과 같은 제한이 없습니다. 즉, 원하는 방식으로 작동하는 고유 한 HTTP 클라이언트 또는 서버를 작성할 수 있습니다. 스레드 모델, 연결 수명주기 및 청크 분할 (chunked encoding)을 포함하여 HTTP 사양에있는 모든 것을 완벽하게 제어 할 수 있습니다.

> 매우 사용자 정의 할 수있는 특성 덕분에 다음과 같은 매우 효율적인 HTTP 서버를 작성할 수 있습니다.  
> 지속적인 연결과 서버 푸시 기술이 필요한 채팅 서버 (예 : Comet 및 WebSockets)  
> 전체 미디어가 스트리밍 될 때까지 연결을 유지해야하는 미디어 스트리밍 서버 (예 : 2 시간 분량의 비디오)
> 메모리 부족없이 큰 파일을 업로드 할 수있는 파일 서버 (예 : 요청 당 1GB 업로드)  
> 수만 개의 타사 웹 서비스에 비동기 적으로 연결되는 확장 가능한 매시업 클라이언트  