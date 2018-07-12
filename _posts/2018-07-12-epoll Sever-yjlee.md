---
layout: post
title: Epoll Server
author: "yjlee"
tags: 
 - select
 - epoll
 - Netty
categories: Yjlee
---

# select i/o 통지 

  select 는 싱글 쓰레드로 다중 i/o를 처리하는 멀티플렉싱 통지 모델의 대표적인 방법이다.
  select는 많은 파일 디스크립터 들을 한꺼번에 관찰하는 FD_SET 구조체를 사용하여 빠르고 간편하게 유저에게
  파일 디스크립터의 상황을 알려준다.
  
  FD_SET?
  FD_SET은 하나의 FD(파일 디스크립터)의 상태를 하나의 비트로 표현한다. 파일 디스크립터의 번호는 고유하기 때문에, 파일 디스크립터의 번호를 인덱스로하   여 해당 비트가 어떤 값을 가지고 있느냐에 따라서 준비상황을 통지 받을 수 있는 것이다. 먼저 파일 디스크립터의 번호를 FD_SET에 
  등록하면 해당 비트의 값  이 1로 저장된다
  
  select는 read/ write/ error 3가지 I/O에 대한 통지를 받는다. 또한 select에 timeout을 설정하여 대기시간을 설정할 수 있다. 
  signature 는 다음과 같다.
  
  ```
  int select( int maxfdNum, //파일 디스크립터의 관찰 범위 (0 ~ maxfdNum -1)
            fd_set *restrict readfds, //read I/O를 통지받을 FD_SET의 주소, 없으면 NULL
            fd_set *restrict writefds,//write I/O를 통지받을 FD_SET의 주소, 없으면 NULL
            fd_set *restrict errorfds,//error I/O를 통지받을 FD_SET의 주소, 없으면 NULL
            struct timeval *restrict timeout //null이면 변화가 있을 때까지 계속 Block, 
                                             //아니면 주어진 시간만큼 대기후 timeout.
           );  
  ```

  ## 샘플

```
 struct timeval timeout; //타임 아웃에 사용할 timeval 변수 fd_set reads, cpy_reads; //read용 FD_SET과 그 사본을 저장할 변수 int fd_max = 0, fd_num = 0; //관찰 범위, 변경된 fd 개수 ... FD_ZERO(&reads); //reads초기화 FD_SET(server_sock, &reads); //server_socket 등록 max_fd = server_socket; //server_socket부터 관찰 범위에 추가 while(TRUE){ cpy_reads = reads; //FD_SET보존을 위한 복사 timeout.tv_sec = 5; //time out 값 설정 timeout.tv_usec = 5000; fd_num = select(fd_max + 1, &cpy_reads, 0, 0, &timeout); //FD_SET사본으로 select 호출 if(fd_num == -1) break; //에러 if(fd_num == 0) continue; //timeout for(int fd = 0; fd < fd_max + 1 ; ++fd) { if(FD_ISSET(fd, &cpy_reads)) //fd가 준비 완료 { if(i == server_socket) //fd가 서버인 경우  { //accpet 처리 (FD_SET으로 등록할 것) } else //fd가 클라이언트 세션인 경우 { //recv및 closesocket 처리 (FD_CLR로 삭제할 것) } } } close(server_socket); return 0;
 ```
 
 ## 한계
 select가 모든 fd를 순회하면서 recv()를 호출하는 방법보다는 훨씬 잘 구현된 멀티플렉싱인 것은 자명하다. 하지만 만들어진지 오래되다보니 그 한계점이 뚜렷하다. 동작 환경에 따라 다르지만 일반적으로 검사할 수 있는 fd개수가 최대 1024개로 제한된다. 그리고 관찰 영역에 포함되는 모든 파일 디스크립터에 대해서 순회하면서 한번씩 FD_ISSET으로 체크하는 것도 불필요한 체크인것 처럼 보인다. 실제로 상태가 변화된 fd의 목록을 넘겨준다면 더 빠르게 작동할 수 있지 않을까?
 
 select를 사용해서 I/O의 상황을 알기 위해서는 프로세스가 커널에게 직접 상황 체크를 요청해야한다. 프로세스가 커널의 상황을 지속적으로 확인하고 그에 맞는 대응을 하는 형태로 구성되기 때문에 프로세스와 커널이 서로 동기화된 상태에서 정보를 주고 받는 형태로 볼 수 있다. 따라서 select의 통지형태를 동기형 통지방식이라 부를 수 있다. 


# epoll
  epoll은 select의 단점을 보완하여 리눅스환경에서 사용할 수 있도록 만든 I/O 통지 기법이다. 전체 파일 디스크립터에 대한 반복문을 사용하지 않고, 커널에게 정보를 요청하는 함수(select 같은)를 호출할 때마다 전체 관찰 대상에 대한 정보를 넘기지도 않는다
  
  계속해서 정보를 넘기지 않기 위해서 관찰 대상인 fd들의 정보를 담은 저장소를 직접 운영체제가 담당한다
  
  운영체제에게 관찰대상의 저장소를 만들어달라고 요청하면 그 저장소에 해당하는 파일 디스크립터(이하 epoll_fd)를 리턴해준다. 관찰 영역이 변경되면(관찰대상 추가 삭제) epoll_fd를 통해 변경을 요청할 수 있다. 그리고 관찰 대상의 변경사항을 체크할때도 epoll_fd를 통해 확인을 한다. 따라서 전체 파일디스크립터를 순회하면서 FD_ISSET을 하는 문제는 더이상 발생하지 않는다.
  
  
  ## 샘플
  ```
  int epoll_fd = epoll_create(EPOLL_SIZE); 
  struct epoll_event* events = malloc(sizeof(struct epoll_event)*EPOLL_SIZE); 
  struct epoll_event init_event; init_event.events = EPOLLIN; 
  init_event.data.fd = server_socket; 
  epoll_ctl(epoll_fd, EPOLL_CTL_ADD, server_socket, &init_event); 
  while(TRUE) { 
    int event_count = epoll_wait(epoll_fd, events, EPOLL_SIZE, -1); 
    if( event_count = -1 ) break; for( int i = 0 ; i < event_count; ++i ) 
    { if(events[i].data.fd == server_socket) //서버 소켓에 이벤트 
    { //accept 처리  ... init_event.events = EPOLLIN; 
    init_event.data.fd = new_client_socket; 
    epoll_ctl(epoll_fd, EPOLL_CTL_ADD, new_client_socket, &init_event); 
    } else //이벤트가 도착한 소켓들 
    { //read, write, closesocket처리 } } 
  } 
  closesocket(server_socket); close(epoll_fd); return 0;
```

 ## epoll 정리
epoll은 select의 단점을 많이 개선한 형태의 통지방식이다. FD_SET을 운영체제가 직접 관리하는 것으로 많은 부분이 개선되었다. 하지만 그 본질적인 동작 구조는 select와 크게 다르지 않다. 프로세스가 커널에게 지속적으로 I/O 상황을 체크하여 동기화 하는 개념은 여전히 유효하다. 따라서 epoll의 통지모델 역시 동기형 통지모델이다

## 단점

표준 지원사항이 아니라서 다른 유닉스에서는 사용할 수 없다





# 정리


## select
특징으로는 
  - 등록된 file descriptor를 하나하나 체크를 해야하고 커널과 유저 공간 사이에 여러번의 데이터 복사가 있음.
  - 관리 file descriptor 수에 제한이 있음.
  - 사용 쉽고 지원 OS가 많아 이식성 좋음.

file descriptor를 하나 하나에 체크하기 때문에 O(n)의 계산량이 필요합니다. 따라서 관리하는 file descriptor의 수가 증가하면 성능이 떨어진다. 
또한 관리 수가 한정되어 있기 때문에 그 수를 초과하면 사용할 수 없다.

## poll
 poll은 거의 select와 동일하지만 다음과 같은 차이가 있다.
- 관리 file descriptor 무제한.
- 좀더 low level의 처리로 system call의 호출이 select보다 적음. 이식성 나쁨.
- 접속수가 늘어나면 오히려 fd당 체크 마스크의 크기가 select는 3bit인데 비해, poll은 64bit정도이므로 양이 많아지면 성능이 select보다 떨어짐.

## epoll
 linux커널 2.6.x이상 버전에만 지원되고 특징은 다음과 같다.
- 관리 fd의 수는 무제한.
- select, poll과 달리, fd의 상태가 kernel 에서 관리됨.
- 일일이 fd 세트를 kernel 에 보낼 필요가 없음.
- kernel이 fd를 관리하고 있기 때문에 커널과 유저스페이스 간의 통신 오버헤드가 대폭 줄어듬.












  
  




  
