


# Netty Architectural Overview

## 2.1 Rich Buffer Data Structure

   1) ByteBuffer를 대신한 자체 ByteBuf를 사용한다. 특징은 다음과 같다.
      - 자체 butter type을 정의할수 있다.
      - 투명한 제로 카피는 내장형 복합 버퍼 유형에 의해 달성됩니다
      - 동적 버퍼 유형은 즉시 제공되며, 용량은 필요에 따라 확장되며 StringBuffer와 동일합니다
      - flip() 함수는 필요 없다.
      - 종종 ByteBuffer 보다 빠르다.
      
       * ByteBuffer
       * flip() 함수
       
       
 ## 2.2 Universal Asynchronous I/O API
    1) 일반 적인 Socket 과 DatagramSocket은 Sokcet i/o가 매우 다르다.
    2) 통신 프로그램 개발의 어려움을 해결해 준다.
    
    
    
   NIO , OIO 
   
 ## 2.3 Event Model based on the Interceptor Chain Pattern
   Netty는 I / O에 초점을 맞춘 잘 정의 된 이벤트 모델을 가지고 있습니다. 또한 각 이벤트 유형이 엄격한 
   유형 계층 구조에 의해 다른 유형과 구별되기 때문에 기존 코드를 손상시키지 않고 자신의 이벤트 유형을 구현할 수 있습니다
   
   ChannelEvent는, ChannelPipeline 내의 ChannelHandler의리스트에 의해 처리됩니다
   
   - 이벤트 등록 및 추가
   - 이벤트 감지 및 모니터링 (스레드 )
   - 이벤트 실행 (스레드)
   
 ## 2.4 Advanced Components for More Rapid Development
   빠르게 개발 가능하다.
   
   ### 2.4.1 Codec framework
   프로토콜 코덱과 비즈니스 논리를 분리하는 것이 항상 좋은 생각입니다. 그러나이 아이디어를 처음부터 구현할 때 몇 가지 문제가 있습니다. 메시지의 단편화를 처리해야합니다. 일부 프로토콜은 다층 (즉, 다른 하위 프로토콜의 상단에 구축 됨)입니다. 일부는 너무 복잡하여 단일 상태 시스템에서 구현할 수 없습니다.
   
   ### 2.4.2. SSL / TLS Support
   
   ### 2.4.3  HTTP Implementation
    HTTP는 인터넷에서 가장 많이 사용되는 프로토콜입니다. Servlet 컨테이너와 같은 많은 HTTP 구현이 이미 있습니다. 그렇다면 왜 Netty는 HTTP를 핵심 요소로 사용합니까?
    
    Netty의 HTTP 지원은 기존 HTTP 라이브러리와 매우 다릅니다. 낮은 수준에서 HTTP 메시지를 교환하는 방법을 완벽하게 제어 할 수 있습니다.
    사용자 정의가 가능한 특성 덕분에 다음과 같은 매우 효율적인 HTTP 서버를 작성할 수 있습니다
      
    - 지속적인 연결과 서버 푸시 기술이 필요한 채팅 서버 (예 : Comet 및 WebSockets)
    - 전체 미디어가 스트리밍 될 때까지 연결을 유지해야하는 미디어 스트리밍 서버 (예 : 2 시간 분량의 비디오)
    - 메모리 부족없이 큰 파일을 업로드 할 수있는 파일 서버 (예 : 요청 당 1GB 업로드)
    - 수만 개의 타사 웹 서비스에 비동기 적으로 연결되는 확장 가능한 매시업 클라이언트
    
    ### 2.4.4. Google Protocol Buffer Integration
    
    
    
    
    *** 매우 궁금함
  
    아래 코드에서 null을 리턴하면 수신받을 패킷이 부족하다고 생각하고
    기다리는 로직 코드를 보고 
  
    public class TimeDecoder extends FrameDecoder {

    @Override
    protected Object decode(
            ChannelHandlerContext ctx, Channel channel, ChannelBuffer buffer) {
            
        if (buffer.readableBytes() < 4) {
            return null; 
        }
        
        return buffer.readBytes(4);
    }
  }
  
  
    
   # 정리
     1) java nio 서버 / 클라이언트 구현 ByteBuffer 이해
     2) Netty ByteBuf 와 zero copy  , 왜 flip함수가 필요 없어 졌을까?
     3) Netty 이벤트 모델에 대한 이해 ( select 함수 실행 위치)
     4) epoll 방식의 이해
     5) codec 종류 및 protobuf 사용 예제
     6) http server 개발 실습
     
     
    
    
   
