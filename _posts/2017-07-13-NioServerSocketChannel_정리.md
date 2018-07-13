
## class diagram
   
   
   ![NioServerSocketChannel 구성](https://opendevelopergroup.github.io/assets/NioServerSocketChannel_Diagram.png)
   
   
## 클래스 상세 설명

 ### SelectorProvider
  셀렉터 및 선택 가능한 채널을위한 서비스 공급자 클래스
  
 ### NioServerSocketChannel
   생성자 : public NioServerSocketChannel() {this(newSocket(DEFAULT_SELECTOR_PROVIDER));}
   
 ### interface ServerSocketChannel 
  
  : 들어오는 TCP / IP 연결을 허용하는 TCP/IP ServerChannel
   InetSocketAddress remoteAddress();
   InetSocketAddress remoteAddress();
   InetSocketAddress remoteAddress();
   InetSocketAddress remoteAddress();
   
 ### interface ServerChannel
    메서드 없음
    
 ### Channel
  
  read , write , connect , bind 과 같은 i/o 동작을 처리 가능한
   컴포넌트 또는 network socket

   channel은 사용자에게 다음을 제공한다.
    - channel의 현재 상태
    - channelConfig
    - i/o 동작 read , write .bind 
    -  채널과 관련된 모든 i/o이벤트를 처리하ChannelPipeLine 

   모든 I / O 작업은 비동기입니다.
   
 ###  abstract class AbstractNioMessageChannel
   메세지에서 동작하는 channel에 대한 기본 클래스 AbstractNioChannel
    
   
 ### AbstractNioChannel
   selector 사용에 대한 기본
   
 ### AbstractChannel
   는 network socket

   channel은 사용자에게 다음을 제공한다.
    - channel의 현재 상태
    - channelConfig
    - i/o 동작 read , write .bind 
    -  채널과 관련된 모든 i/o이벤트를 처리하ChannelPipeLine 

   모든 I / O 작업은 비동기입니다.

 ###  abstract class AbstractNioMessageChannel
   메세지에서 동작하는 channel에 대한 기본 클래스 AbstractNioChannel

 ### AbstractNioChannel
   selector 사용에 대한 기본

 ### AbstractChannel
     채널의 뼈대

   
   
   ## 시퀀스 다이어그램
   !(https://github.com/OpenDeveloperGroup/OpenDeveloperGroup.github.io/assets/NioServerSocketChannel_Sequence.png)
   
    
    
