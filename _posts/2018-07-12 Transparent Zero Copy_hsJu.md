---
layout: post
title: Transparent Zero Copy
author: "Hanseok Ju"
tags: 
  - architecture
  - Netty
  - zero_copy
categories: Hanseok
---

# *Transparent Zero Copy In Netty.* (작성중)
  ### 1.Transparent Zero Copy 란

시스템은 User 영역과 Kernel 영역으로 나뉜다. 일반적으로 User 영역(Application 영역)에서 작업을 하다가 중요한(?) 작업(보통 시스템 함수 호출시) (ex. disk file read, data transfer, etc..)을 할때는 Kernel 영역으로 이동해서 시스템 함수를 호출해서 작업을 한다. 이 때, Context Switching 이라는 비용이 발생한다.
++Kernel <-> User 간의 switching 비용이 많이 든다고 한다.++

User와 Kernel 사이의 Context Switching을 줄이는 작업들 중 하나가 Transparent Zero Copy입니다. 

아래는 일반적인 작업 과정입니다.

![traditional architec](https://opendevelopergroup.github.io/assets/hsJu/traditional.JPG)

![traditional ContextSwitching]
(https://opendevelopergroup.github.io/assets/hsJu/traditionalCS.JPG)


전통적인 방식에서는 **파일을 디스크에서 읽을때 1번** && **파일을 소켓통신으로 전송할때 1번** Kernel 영역으로 진입해야 하기 때문에 총 4번의 context switching이 발생합니다.

![Zero copy]
(https://opendevelopergroup.github.io/assets/hsJu/ZeroCopy.JPG)

![Zero copy CS]
(https://opendevelopergroup.github.io/assets/hsJu/ZeroCopyCS.JPG)

파일을 디스크에서 읽어온 다음 User 영역으로 가지 않고 즉각적으로 소켓을 통해 전달하기 때문에 Zero copy(무복사)를 활용하면 **1번의 context switching만 발생합니다.**



### 2. netty에서의 Zero Copy.
**ByteBuffer  vs. ByteBuf** 
* **ByteBuffer  ** 
If ByteBuffer were used, you would have to create a new big buffer and copy the two parts into the new buffer. Alternatively, you can perform a gathering write operation in NIO, but it restricts you to represent the composite of buffers as an array of ByteBuffers rather than a single buffer, breaking the abstraction and introducing complicated state management. Moreover, it's of no use if you are not going to read or write from an NIO channel.

--> ByteBuffer는 제한사항이 많다고 한다. ( 추후 추가 예정)

* **ByteBuf **
ByteBuf does not have such caveats because it is fully extensible and has a built-in composite buffer type.





### 3. example source code.







==참고 url ==
https://netty.io/4.0/api/io/netty/buffer/ByteBuf.html - netty bytebuf
https://www.ibm.com/developerworks/linux/library/j-zerocopy/ - zero copy 


------ 이후 알아볼 내용 ------
내용 마무리 ( ByteBuf로 구현한 zero copy와 ByteBuffer로 구현한 zero copy의 차이점)
ByteBuf 클래스 까보자.