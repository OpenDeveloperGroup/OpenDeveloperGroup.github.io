---
layout: page
title: Architectural Overview
author: "Daeho Han"
permalink: /
tags: netty architecture
---

## Chapter 2. Architectural Overview
![netty architecture](https://opendevelopergroup.github.io/assets/daeho/netty/architecture.png){:.center-image}

이번 챕터에서는, Netty에서 제공하는 core 기능을 알아보고 어떻게 그 기능들이 완벽한 네트워크 어플리케이션 개발에 도움을 주는지 설명할 것 입니다.

### 2.1 Rich Buffer Data Structure
Netty는 순차적인 byte를 표현하는데 *NIO ByteBuffer* 대신에 자신만의 buffer API를 사용합니다. 이것은 *ByteBuffer*를 사용하는 것 보다 확실한 이점을 가집니다.  
Netty의 새로운 buffer 타입인 *ChannelBuffer* 는 *ByteBuffer*의 문제를 해결하고 네트워크 어플리케이션 개발자의 요구사항을 충족시키기 위해 디자인 되었습니다. 아래에 몇몇 특징들이 나열되어있습니다.

> 필요하다면 자신만의 buffer를 정의할 수 있다.  
> Transpartne zero copy is achieved by a built-in composite buffer type.  
> *StringBuffer*처럼 capacity가 요구에 따라 확장되는 동적인 buffer 타입이 제공된다  
> flip()을 더 이상 call할 필요가 없다.  
> ByteBuffer보다 더 빠르기도 하다.  
