---
layout: post
title: Channel Buffer
author: "autumnlee"
tags: 
  - Netty
  - NIO
  - Buffer
categories: Autumn
---

# Channel Buffer


### Buffer 란?
사전적 의미는 완충을 말한다.
컴퓨터 시스템의 어떤 장치에서 다른 장치로 전송을 할 때에, 
일시적으로 그 데이터를 보관하는 일종의 메모리 영역이다.
buffering이라 함은 buffer를 채우는 동작을 일컷는 말로서 유사어로 Queue가 있다.

출처: http://lucrative.tistory.com/83 [Lucrative]  



### Java 의 ByteBuffer
- java.nio 에서 제공되는 class
- fields
 ```
  capacity : Buffer 에 저장할 수 있는 최대크기
  position : Buffer 를 읽고(쓰고) 있는 현재 위치. 초기값은 0 이며 읽고 쓸때마다 증가된다. limit 와 capacity 를 넘을 수 없다.
  limit : 읽고 쓸수있는 공간의 최대값. 변경이 가능하지만 capacity 보다 크게 지정될 수는 없다.
 ```  



- flip 메소드

> flip() makes a buffer ready for a new sequence of channel-write or relative get operations: It sets the limit to the current position and then sets the position to zero.
limit 를 현재 position 으로 두고, position 은 0 으로 변경함.

```java

public final Buffer flip() {
    limit = position;
    position = 0;
    mark = -1;
    return this;
}

```
```
     * <p> After a sequence of channel-read or <i>put</i> operations, invoke
     * this method to prepare for a sequence of channel-write or relative
     * <i>get</i> operations.  For example:
     *
     * <blockquote><pre>
     * buf.put(magic);    // Prepend header
     * in.read(buf);      // Read data into rest of buffer
     * buf.flip();        // Flip buffer
     * out.write(buf);    // Write header + data to channel</pre></blockquote>
     *
```
- buffer 에 값을 쓴 뒤, 쓴 값을 읽기 위해 position 을 옮겨주어야 한다.
- 읽기/쓰기를 flip(뒤집다) 한다는 의미로 받아들이면 된다.

[flip에 대한 일침](https://stackoverflow.com/questions/14792968/what-is-the-purpose-of-bytebuffers-flip-method-and-why-is-it-called-flip)
>ByteBuffer is ill designed. There are lots of complaints from decent programmers.
So don't try to reason about it, just study and use the API carefully.





### Netty 의 ByteBuffer

```java

    public abstract class AbstractByteBuf extends ByteBuf {
        private static final InternalLogger logger = InternalLoggerFactory.getInstance(AbstractByteBuf.class);
        private static final String PROP_MODE = "io.netty.buffer.bytebuf.checkAccessible";
        private static final boolean checkAccessible = SystemPropertyUtil.getBoolean("io.netty.buffer.bytebuf.checkAccessible", true);
        static final ResourceLeakDetector<ByteBuf> leakDetector;
        int readerIndex;
        int writerIndex;
        private int markedReaderIndex;
        private int markedWriterIndex;
        private int maxCapacity;

        ...

        public ByteBuf writeInt(int value) {
            this.ensureWritable0(4);
            this._setInt(this.writerIndex, value);
            this.writerIndex += 4;
            return this;
        }

        final void ensureWritable0(int minWritableBytes) {
            this.ensureAccessible();
            if (minWritableBytes > this.writableBytes()) {
                if (minWritableBytes > this.maxCapacity - this.writerIndex) {
                    throw new IndexOutOfBoundsException(String.format("writerIndex(%d) + minWritableBytes(%d) exceeds maxCapacity(%d): %s", this.writerIndex, minWritableBytes, this.maxCapacity, this));
                } else {
                    int newCapacity = this.alloc().calculateNewCapacity(this.writerIndex + minWritableBytes, this.maxCapacity);
                    this.capacity(newCapacity);
                }
            }
        }

    }
```


```
궁금한점
1. buffer 가 사용하는 메모리는 제한되어 있는데, index 를 어떻게 관리할까?
2. netty 의 buffer pool
```

