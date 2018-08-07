## NioServerSocketChannel 정리2


## NioEventLoopGroup 정리
   ### 이벤트 루프 등록   , Channel : NioServerSocketChannel
final ChannelFuture initAndRegister() {
        Channel channel = null;

        try {
            channel = this.channelFactory.newChannel();
            this.init(channel);
        } catch (Throwable var3) {
            if(channel != null) {
                channel.unsafe().closeForcibly();
                return (new DefaultChannelPromise(channel, GlobalEventExecutor.INSTANCE)).setFailure(var3);
            }

            return (new DefaultChannelPromise(new FailedChannel(), GlobalEventExecutor.INSTANCE)).setFailure(var3);
        }

        ChannelFuture regFuture = this.config().group().register(channel);  ==> 이벤트 루프에 channel을 등록함

        if(regFuture.cause() != null) {
            if(channel.isRegistered()) {
                channel.close();
            } else {
                channel.unsafe().closeForcibly();
            }
        }

        return regFuture;
    }


#### MultithreadEventExecutorGroup 함수 호출
  public ChannelFuture register(Channel channel) {
        return this.next().register(channel);
    }
