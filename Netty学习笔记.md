## Netty学习笔记

### NIO

>  (non-blocking io非阻塞IO)，面向缓冲区的编程

#### 三大组件

> Channel 网络通道，会注册到Selector中，是双向的，可以同时进行读写
>
> - FileChannel ：文件的读写
> - DatagramChannel：UDP数据的读写
> - SocketChannel ：TCP网络数据读写
> - ServerSocketChannel  ：TCP网络数据读写
>
>  Buffer 缓冲区，底层通过数组实现，是双向的流，需要通过flip切换；
>
> ```java
> //Buffer常用属性与方法
> private int mark = -1; // 标记
> private int position = 0; // 读写数据的当前位置
> private int limit; // 要读取数据的位置上限，默认与capacity相等;读取的数据是[position,limit)左闭右开
> private int capacity; // 缓冲区的容量
> 
> /**
> * 数据读与写的切换
> */
> public Buffer flip() {
>   limit = position;
>   position = 0;
>   mark = -1;
>   return this;
> }
> ```
>
> Selector 选择器
>
> ```java
> //常用方法
> public abstract long read(ByteBuffer[] dsts, int offset, int length)
>   throws IOException;
> public abstract long write(ByteBuffer[] srcs, int offset, int length)
>   throws IOException;
> /**
> * 将数据从当前通道拷贝到目标通道
> */
> public abstract long transferTo(long position, long count,
>                                 WritableByteChannel target)
>   throws IOException;
> /**
> * 将数据从目标通道拷贝到通道
> */
> public abstract long transferFrom(ReadableByteChannel src,
>                                   long position, long count)
>   throws IOException;
> ```
>
> 