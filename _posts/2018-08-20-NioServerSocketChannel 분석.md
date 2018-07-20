


## 개요

   NioServerSocketChannel을 분석하기 위해서 netty-all 소스코드를 로그를 출력하기 위한 시도를함
   
   
   ## 환경 구축
   
   컴파일 및 빌드 환경 구축 방법
   
    ### 소스 다운로드
    
    https://github.com/netty/netty/tree/4.0
   
   
   ### 소스 프로젝트 포팅
   ![빌드환경구성](https://opendevelopergroup.github.io/assets/빌드환경.png)
   
   
   ### 빌드
   clean package -Dmaven.test.skip=true -Dcheckstyle.skip=true

   ![테스트구성](https://opendevelopergroup.github.io/assets/테스트프로그램.png)


   ### package 파일 
   
   
   ### 테스트 프로그램 
   
   ### 테스트 프로그램 설정
   
   
   ### netty source log 추가
   
   public class NioServerSocketChannel extends AbstractNioMessageChannel
                             implements io.netty.channel.socket.ServerSocketChannel {

    private static final ChannelMetadata METADATA = new ChannelMetadata(false, 16);
    private static final SelectorProvider DEFAULT_SELECTOR_PROVIDER = SelectorProvider.provider();
    private static final InternalLogger logger = InternalLoggerFactory.getInstance(NioServerSocketChannel.class);
    private static ServerSocketChannel newSocket(SelectorProvider provider) {

        System.out.println("##################################### newSocket ############################################");
        System.out.println("##################################### newSocket ############################################");
        System.out.println("##################################### newSocket ############################################");
        System.out.println("##################################### newSocket ############################################");
        System.out.println(" #################  provider : " + provider + "###########################################");
        System.out.println(" #####################################################################################");
   
   
   
   
   
   

