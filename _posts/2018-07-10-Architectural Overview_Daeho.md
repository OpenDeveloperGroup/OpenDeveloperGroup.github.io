---
layout: page
title: Architectural Overview
author: "Daeho Han"
permalink: /
tags: architecture
---

## Chapter 2. Architectural Overview
![netty architecture](https://opendevelopergroup.github.io/assets/daeho/netty/architecture.png)

이번 챕터에서는, Netty에서 제공하는 core 기능을 알아보고 어떻게 그 기능들이 완벽한 네트워크 어플리케이션 개발에 도움을 주는지 설명할 것 입니다.

### 2.1 Rich Buffer Data Structure
Netty는 순차적인 byte를 표현하는데 *NIO ByteBuffer* 대신에 자신만의 buffer API를 사용합니다. 이것은 *ByteBuffer*를 사용하는 것 보다 확실한 이점을 가집니다.  
Netty의 새로운 buffer 타입인 *ChannelBuffer* 는 *ByteBuffer*의 문제를 해결하고 네트워크 어플리케이션 개발자의 요구사항을 충족시키기 위해 디자인 되었습니다. 아래에 몇몇 특징들이 나열되어있습니다.

> 필요하다면 자신만의 buffer를 정의할 수 있다.  
> Transpartne zero copy is achieved by a built-in composite buffer type.  
> *StringBuffer*처럼 capacity가 요구에 따라 확장되는 동적인 buffer 타입이 제공된다  
> flip()을 더 이상 call할 필요가 없다.  
> ByteBuffer보다 더 빠르기도 하다.  

### 2.2 Universal Asynchronous I/O API
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