Q:Java NIO
===
### 简介
> Java NIO(New IO)是一个可以替代标准Java IO API的IO API（从Java 1.4开始)，Java NIO提供了与标准IO不同的IO工作方式。Java NIO 核心部分组成：Channels，Buffers，Selectors。
#### Java NIO: Channels and Buffers（通道和缓冲区）
> 标准的IO基于字节流和字符流进行操作的，而NIO是基于通道（Channel）和缓冲区（Buffer）进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。
#### Java NIO: Non-blocking IO（非阻塞IO）
> Java NIO可以让你非阻塞的使用IO，例如：当线程从通道读取数据到缓冲区时，线程还是可以进行其他事情。当数据被写入到缓冲区时，线程可以继续处理它。从缓冲区写入通道也类似。
#### Java NIO: Selectors（选择器）
> Java NIO引入了选择器的概念，选择器用于监听多个通道的事件（比如：连接打开，数据到达）。因此，单个的线程可以监听多个数据通道。要使用Selector，得向Selector注册Channel，然后调用它的select()方法。这个方法会一直阻塞到某个注册的通道有事件就绪。一旦这个方法返回，线程就可以处理这些事件，事件的例子有如新连接进来，数据接收等。

### Channel
> Channel类似流，但又有些不同：
+ 既可以从通道中读取数据，又可以写数据到通道。但流的读写通常是单向的。
+ 通道可以异步地读写。
+ 通道中的数据总是要先读到一个Buffer，或者总是要从一个Buffer中写入。
> Channel的主要实现，这些通道涵盖了UDP 和 TCP 网络IO，以及文件IO。
+ FileChannel 从文件中读写数据。
+ DatagramChannel 能通过UDP读写网络中的数据。
+ SocketChannel 能通过TCP读写网络中的数据。
+ ServerSocketChannel 可以监听新进来的TCP连接，像Web服务器那样。对每一个新进来的连接都会创建一个SocketChannel。

```java
public class test {
    public static void main(String[] args){
        try { 
            RandomAccessFile aFile = new RandomAccessFile("test.txt", "rw");
            FileChannel inChannel = aFile.getChannel();
        
            ByteBuffer buf = ByteBuffer.allocate(1024);
        
            int bytesRead = inChannel.read(buf);
            while (bytesRead != -1) {
        
                System.out.println("Read " + bytesRead);
                buf.flip();
        
                while (buf.hasRemaining()) {
                    System.out.print((char) buf.get());
                }
        
                buf.clear();
                bytesRead = inChannel.read(buf);
            }
            aFile.close();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
        }
    }
}

``` 

### Buffer
![Image text](https://github.com/IceDarron/Note/blob/master/Image/java_nio_capacity&position&limi.png)
> 使用Buffer读写数据一般遵循以下四个步骤：
+ 写入数据到Buffer
+ 调用flip()方法
+ 从Buffer中读取数据
+ 调用clear()方法或者compact()方法
> 当向buffer写入数据时，buffer会记录下写了多少数据。一旦要读取数据，需要通过flip()方法将Buffer从写模式切换到读模式。在读模式下，可以读取之前写入到buffer的所有数据。
一旦读完了所有的数据，就需要清空缓冲区，让它可以再次被写入。有两种方式能清空缓冲区：调用clear()或compact()方法。clear()方法会清空整个缓冲区。compact()方法只会清除已经读过的数据。任何未读的数据都被移到缓冲区的起始处，新写入的数据将放到缓冲区未读数据的后面。

#### capacity,position和limit详解
> 缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。读写涉及到的主要三个属性，详细如下：
##### capacity
> 作为一个内存块，Buffer有一个固定的大小值，也叫“capacity”.你只能往里写capacity个byte、long，char等类型。一旦Buffer满了，需要将其清空（通过读数据或者清除数据）才能继续写数据往里写数据。

##### position
> 当你写数据到Buffer中时，position表示当前的位置。初始的position值为0.当一个byte、long等数据写到Buffer后， position会向前移动到下一个可插入数据的Buffer单元。position最大可为capacity – 1.
当读取数据时，也是从某个特定位置读。当将Buffer从写模式切换到读模式，position会被重置为0. 当从Buffer的position处读取数据时，position向前移动到下一个可读的位置。

##### limit
> 在写模式下，Buffer的limit表示你最多能往Buffer里写多少数据。 写模式下，limit等于Buffer的capacity。
当切换Buffer到读模式时， limit表示你最多能读到多少数据。因此，当切换Buffer到读模式时，limit会被设置成写模式下的position值。换句话说，你能读到之前写入的所有数据（limit被设置成已写数据的数量，这个值在写模式下就是position）

#### flip()方法,从写模式切换到读模式
```java
public class test {
    public final Buffer flip() {
            limit = position;
            position = 0;
            mark = -1;
            return this;
        }
}
```

#### rewind()方法, 重读Buffer中的所有数据
```java
public class test {
    public final Buffer rewind() {
        position = 0;
        mark = -1;
        return this;
    }
}
```

#### clear()方法,准备好再次被写入
```java
public class test {
   public final Buffer clear() {
       position = 0;
       limit = capacity;
       mark = -1;
       return this;
   }
}
```

### Scatter/Gather
> scatter/gather用于描述从Channel中读取或者写入到Channel的操作。

> 分散（scatter）从Channel中读取是指在读操作时将读取的数据写入多个buffer中。因此，Channel将从Channel中读取的数据“分散（scatter）”到多个Buffer中。

> 聚集（gather）写入Channel是指在写操作时将多个buffer的数据写入同一个Channel，因此，Channel 将多个Buffer中的数据“聚集（gather）”后发送到Channel。

>scatter/gather经常用于需要将传输的数据分开处理的场合，例如传输一个由消息头和消息体组成的消息，你可能会将消息体和消息头分散到不同的buffer中，这样你可以方便的处理消息头和消息体。

#### Scattering Reads
> Scattering Reads是指数据从一个channel读取到多个buffer中。注意buffer首先被插入到数组，然后再将数组作为channel.read()的输入参数。read()方法按照buffer在数组中的顺序将从channel中读取的数据写入到buffer，当一个buffer被写满后，channel紧接着向另一个buffer中写。Scattering Reads在移动下一个buffer前，必须填满当前的buffer，这也意味着它不适用于动态消息(消息大小不固定)。换句话说，如果存在消息头和消息体，消息头必须完成填充（例如 128byte），Scattering Reads才能正常工作。

#### Gathering Writes
> Gathering Writes是指数据从多个buffer写入到同一个channel。buffers数组是write()方法的入参，write()方法会按照buffer在数组中的顺序，将数据写入到channel，注意只有position和limit之间的数据才会被写入。因此，如果一个buffer的容量为128byte，但是仅仅包含58byte的数据，那么这58byte的数据将被写入到channel中。因此与Scattering Reads相反，Gathering Writes能较好的处理动态消息。

```java
public class test {
    public static void catterAndGather() throws Exception{
        // catter&gather demo
        RandomAccessFile raf1 = new RandomAccessFile("a.txt", "rw");
        //获取通道
        FileChannel channel1 = raf1.getChannel();
        //设置缓冲区
        ByteBuffer buf1 = ByteBuffer.allocate(50);
        ByteBuffer buf2 = ByteBuffer.allocate(1024);
        //分散读取的时候缓存区应该事有序的，所以把几个缓冲区加入数组中
        ByteBuffer[] bufs = {buf1, buf2};
        //通道进行传输
        channel1.read(bufs);
    
        //查看缓冲区中的内容
        for (int i = 0; i < bufs.length; i++) {
            //切换为读模式
            bufs[i].flip();
        }
    
        System.out.println(new String(bufs[0].array(), 0, bufs[0].limit()));
        System.out.println();
        System.out.println(new String(bufs[1].array(), 0, bufs[1].limit()));
    
        //聚集写入
        RandomAccessFile raf2 = new RandomAccessFile("b.txt", "rw");
        FileChannel channel2 = raf2.getChannel();
        //只能通过通道来进行写入
        channel2.write(bufs);
    }
}
```

### 通道之间的数据传输
> 在Java NIO中，如果两个通道中有一个是FileChannel，那你可以直接将数据从一个channel传输到另外一个channel。

#### transferFrom()
> FileChannel的transferFrom()方法可以将数据从源通道传输到FileChannel中。

#### transferTo()
> transferTo()方法将数据从FileChannel传输到其他的channel中。

> 方法的输入参数position表示从position处开始向目标文件写入数据，count表示最多传输的字节数。如果源通道的剩余空间小于 count 个字节，则所传输的字节数要小于请求的字节数。
此外要注意，在SoketChannel的实现中，SocketChannel只会传输此刻准备好的数据（可能不足count字节）。因此，SocketChannel可能不会将请求的所有数据(count个字节)全部传输到FileChannel中。

### Selector
> Selector（选择器）是Java NIO中能够检测一到多个NIO通道，并能够知晓通道是否为诸如读写事件做好准备的组件。这样，一个单独的线程可以管理多个channel，从而管理多个网络连接。仅用单个线程来处理多个Channels的好处是，只需要更少的线程来处理通道。事实上，可以只用一个线程处理所有的通道。对于操作系统来说，线程之间上下文切换的开销很大，而且每个线程都要占用系统的一些资源（如内存）。因此，使用的线程越少越好。但是，需要记住，现代的操作系统和CPU在多任务方面表现的越来越好，所以多线程的开销随着时间的推移，变得越来越小了。实际上，如果一个CPU有多个内核，不使用多任务可能是在浪费CPU能力。

```java
public class test {
    public static void main(String[] args){
        Selector selector = Selector.open();
        // 与Selector一起使用时，Channel必须处于非阻塞模式下。
        // 这意味着不能将FileChannel与Selector一起使用，因为FileChannel不能切换到非阻塞模式。
        // 而套接字通道都可以。
        channel.configureBlocking(false);
        SelectionKey key = channel.register(selector, SelectionKey.OP_READ);
        
        while(true) {
        
            int readyChannels = selector.select();
            if(readyChannels == 0) continue;
        
            Set selectedKeys = selector.selectedKeys();
            Iterator keyIterator = selectedKeys.iterator();
        
            while(keyIterator.hasNext()) {
              SelectionKey key = keyIterator.next();
        
              if(key.isAcceptable()) {
                  // a connection was accepted by a ServerSocketChannel.
              } else if (key.isConnectable()) {
                  // a connection was established with a remote server.
              } else if (key.isReadable()) {
                  // a channel is ready for reading
              } else if (key.isWritable()) {
                  // a channel is ready for writing
              }
        
              keyIterator.remove();
            }
        }
    }
}
```

#### SelectionKey
> 当向Selector注册Channel时，register()方法会返回一个SelectionKey对象。包含了许多属性，一下一一介绍。

##### interest集合
+ SelectionKey.OP_CONNECT
+ SelectionKey.OP_ACCEPT
+ SelectionKey.OP_READ
+ SelectionKey.OP_WRITE
> 如果你对不止一种事件感兴趣，那么可以用“位或”操作符将常量连接起来，如下：

```java
public class test {
    int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;
}
```

> 通过位与运算确定某个事件是否在interest集合中。

```java
public class test {
    int interestSet = selectionKey.interestOps();
    
    boolean isInterestedInAccept  = (interestSet & SelectionKey.OP_ACCEPT) == SelectionKey.OP_ACCEPT;
    boolean isInterestedInConnect = interestSet & SelectionKey.OP_CONNECT;
    boolean isInterestedInRead    = interestSet & SelectionKey.OP_READ;
    boolean isInterestedInWrite   = interestSet & SelectionKey.OP_WRITE;
}
```

##### ready集合
> 表示通道是否已经准备就绪的操作的集合。

```java
public class test {
   public static void main(String[] args){
       int readySet = selectionKey.readyOps();
       selectionKey.isAcceptable();
       selectionKey.isConnectable();
       selectionKey.isReadable();
       selectionKey.isWritable();
   }
}
```

##### Channel + Selector
> 通过SelectionKey访问Channel和Selector。

```java
public class test {
    Channel  channel  = selectionKey.channel();
    Selector selector = selectionKey.selector();
}
```

##### 附加的对象
> 可以给每个SelectionKey对象附加信息，方便识别每个通道。

```java
public class test {
   public static void main(String[] args){
        selectionKey.attach(theObject);
        Object attachedObj = selectionKey.attachment();
        SelectionKey key = channel.register(selector, SelectionKey.OP_READ, theObject);
   }
}
```

### Pipe
> Java NIO 管道是2个线程之间的单向数据连接。Pipe有一个source通道和一个sink通道。数据会被写到sink通道，从source通道读取。
这里是Pipe原理的图示：

![Image text](https://github.com/IceDarron/Note/blob/master/Image/java_nio_pipe.png)

#### 创建管道

```java
public class test {
    Pipe pipe = Pipe.open();
}
```

#### 向管道写数据

```java
public class test {
    public static void main(String[] args){
        Pipe.SinkChannel sinkChannel = pipe.sink();
        
        String newData = "New String to write to file..." + System.currentTimeMillis();
        ByteBuffer buf = ByteBuffer.allocate(48);
        
        buf.clear();
        buf.put(newData.getBytes());
        buf.flip();
        
        while(buf.hasRemaining()) {
            sinkChannel.write(buf);
        }
    }
}
```

#### 从管道读取数据

```java
public class test {
    Pipe.SourceChannel sourceChannel = pipe.source();
    ByteBuffer buf = ByteBuffer.allocate(48);
    // read()方法返回的int值会告诉我们多少字节被读进了缓冲区。
    int bytesRead = sourceChannel.read(buf);
}
```