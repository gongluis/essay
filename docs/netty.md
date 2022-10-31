[TOC]

####  一、基础概念

1. IO与NIO

```
阻塞与非阻塞是描述进程在访问某个资源时，数据是否准备就绪的的一种处理方式。当数据没有准备就绪时：

阻塞：线程持续等待资源中数据准备完成，直到返回响应结果。
非阻塞：线程直接返回结果，不会持续等待资源准备数据结束后才响应结果。

同步与异步
同步与异步是指访问数据的机制，同步一般指主动请求并等待IO操作完成的方式。
异步则指主动请求数据后便可以继续处理其它任务，随后等待IO操作完毕的通知。
	普通水壶煮水，站在旁边，主动的看水开了没有？同步的阻塞
	普通水壶煮水，去干点别的事，每过一段时间去看看水开了没有，水没开就走人。 同步非阻塞
	响水壶煮水，站在旁边，不会每过一段时间主动看水开了没有。如果水开了，水壶自动通知他。 异步阻塞
	响水壶煮水，去干点别的事，如果水开了，水壶自动通知他。异步非阻塞
	
```

2. 传统BIO模型 

是一种同步阻塞IO，IO在进行读写时，该线程将被阻塞，线程无法进行其它操作。
IO流在读取时，会阻塞。直到发生以下情况：1、有数据可以读取。2、数据读取完成。3、发生异常

3. 伪异步IO模型

以传统BIO模型为基础，通过线程池的方式维护所有的IO线程，实现相对高效的线程开销及管理

4. NIO模型

NIO（JDK1.4）模型是一种同步非阻塞IO，主要有三大核心部分：Channel(通道)，Buffer(缓冲区), Selector（多路复用器）。传统IO基于字节流和字符流进行操作，而NIO基于Channel和Buffer(缓冲区)进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。Selector(多路复用器)用于监听多个通道的事件（比如：连接打开，数据到达）。因此，单个线程可以监听多个数据通道。

IO是面向流的，NIO是面向缓冲区的，缓冲区数据可以前后移动

IO的各种流是阻塞的，这意味着，当一个线程调用read() 或 write()时，该线程被阻塞，直到有一些数据被读取，或数据完全写入。该线程在此期间不能再干任何事情了。NIO的非阻塞模式，使一个线程从某通道发送请求读取数据，但是它仅能得到目前可用的数据，如果目前没有数据可用时，就什么都不会获取。而不是保持线程阻塞，所以直至数据变的可以读取之前，该线程可以继续做其他的事情。 非阻塞写也是如此。一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。 线程通常将非阻塞IO的空闲时间用于在其它通道上执行IO操作，所以一个单独的线程现在可以管理多个输入和输出通道

5.三大组件

channel

```
FileChannel 文件
datachannel
soketchannel 客户端服务端都可以用
serverSocketchannel 服务端专用
```



Buffer

```
ByteBuffer

```

Selector

```
服务器端soket连接多个客户端，多线程版设计，缺点内存占用高,上下文切换成本高（保存状态恢复状态），16核cpu一次最多只能跑16线程。只适合连接数少场景。
线程池版设计，阻塞模式下，线程仅能处理一个socket连接。连接上啥事不干也会等待，仅适合短连接场景
selector版设计，作用就是配合一个线程管理多个channel获取channel上发生的事，这些channel是在非阻塞模式下工作，不会让线程吊死在一个channnel上，适合连接特别多，但是流量低的场景。



```

6. ByteBuffer

正确使用姿势

```
1.向buffer中写数据 channel.read(buffer)
2.切换到读模式 buffer.flip()
3.从buffer中读数据，buffer.get()
4.切换到写模式 buffer.lear()或buffer.compact()
5.重复1~4步骤
```

重要属性

```
1. capacity 容量
2. position 指针
3. limit 写入限制

flip动作发生后，position切换为读取位置，limit切换为读取限制（前面写的position位置）
clear动作发生后，清空
compact方法，是把未读完的部分向前压缩，然后切换到写模式
```

常见方法

```
1. allocate: 空间分配，定义buffer的大小，不可变的
ByteBuffer.allocate(16); //java对内存 读写效率较低，收到GC影响
ByteBuffer.allocateDirect(16);//直接内存，读写效率高（一次拷贝）不受GC影响 分配的效率低

2. 向ByteBuffer写入数据
	调用channel的read方法：channnel.read(buffer)
	调用buffer的put方法：buffer.put();
3. 从ByteBuffer读数据
	调用channel的write方法：channel.write(buffer)
	调用buffer 自己的get方法：buffer.get() 该方法会让position读指针向后走，如果想重复读取数据可以调用rewind方法将position方法重置为0，或者调用get(position)获取索引i的内容，不会移动指针。
	
4. scatteringReads 作用：“onetwothree”将一串字符串读出来并分开，


```

#### 二、使用方法

> 参考链接 https://www.cnblogs.com/AIPAOJIAO/p/10631551.html

```
/**
     * 初始化连接
     */
    public void connect() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                connectServer();
            }
        }).start();
    }
     private void connectServer() {
        try {
            group = new NioEventLoopGroup();
            bootstrap = new Bootstrap()
                    .option(ChannelOption.SO_KEEPALIVE, true)//长连接
                    .option(ChannelOption.RCVBUF_ALLOCATOR, new FixedRecvByteBufAllocator(65535))//两个字节的最大保存值 255是一个字节的最大保存值
                    .channel(NioSocketChannel.class)
                    .group(group)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        ByteBuf delimiter = Unpooled.buffer(1);

                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            delimiter.writeByte(0x7E);
                            ChannelPipeline pipeline = socketChannel.pipeline();
                            pipeline.addLast(new DelimiterBasedFrameDecoder(65535, delimiter));
                            /**
                             * readerIdleTime 设置读超时时间，说白了就是这个时间内没有读到有报文，userEventTriggered被触发，（设置这个参数常用在服务器端）
                             * writerIdleTime 同样的道理，这个是写超时，这个就是今天提到自定义心跳要用的（设置在客户端）
                             * allIdleTime 这个不多说，读写多超时
                             */
                            pipeline.addLast(new IdleStateHandler(0, HEART_TIME, 0));//第二个参数，表示客户端在15s没有数据发出，会主动发一个心跳包给服务端
                            pipeline.addLast(new JTT808Decoder());
                            pipeline.addLast(new JTT808Encoder());
                            pipeline.addLast(new JTTLanZhouHandler(JTTLanZhouClient.this, listener));//处理数据接收
                            pipeline.addLast(new ConnectorIdleStateTrigger());
                        }
                    });
            ChannelFuture channelFuture = bootstrap.connect(new InetSocketAddress(ip, port));
            channelFuture.addListener(channelFutureListener);
        } catch (Exception e) {
            e.printStackTrace();
            if (listener != null) {
                listener.onConnectionSateChange(OnConnectionListener.DIS_CONNECT);
            }
        }
    }
    private ChannelFutureListener channelFutureListener = new ChannelFutureListener() {
        @Override
        public void operationComplete(ChannelFuture channelFuture) throws Exception {
            if (!channelFuture.isSuccess()) {
                Log.i(TAG, "链接失败----ChannelFutureListener");
               /* if (listener != null) {
                    listener.onConnectionSateChange(OnConnectionListener.DIS_CONNECT);
                }*/
                final EventLoop loop = channelFuture.channel().eventLoop();
                loop.schedule(new Runnable() {
                    @Override
                    public void run() {
                        reConnect();
                        if (listener != null) {
                            listener.onConnectionSateChange(OnConnectionListener.RE_CONNECT);
                        }
                    }
                }, 3, TimeUnit.SECONDS);
            } else {
                Log.i(TAG, "链接成功----ChannelFutureListener ");
                channel = channelFuture.channel();
                //开启周期性上报位置服务 5s
                JTTLanZhouManager.getInstance().periodUploadLocation();
            }
        }
    };
    public interface MyChannelFutureListener {
        void operationComplete(ChannelFuture channelFuture, int reconect);

        void reconect(int count);
    }
```

#### 三、解析905格式报文

1. 组包

   ```
   /**
        * 陕标
        * @return
        */
       public static byte[] generate905(int msgId,byte[] msgBody){
           //=========================标识位==================================//
           byte[] flag = new byte[]{0x7E};
   
           //=========================消息头==================================//
           //[0,1]消息Id
           byte[] msgIdb = BitOperator.numToByteArray(msgId, 2);
           //信息属性--消息包大小
           byte[] length = BitOperator.numToByteArray(msgBody.length, 2);
           //ISU标识 10开头  通讯号 861130040179
               //10开头 2字节
               byte[] start = BitOperator.numToByteArray(01, 1);
               //厂商编号 1字节
               byte[] number = BitOperator.numToByteArray(31, 1);
               //设备类型 1字节
               byte[] type = BitOperator.numToByteArray(23, 1);
               //序列号 3字节
               byte[] serno = BitOperator.numToByteArray(456789, 3);
           byte[] isu = ByteUtil2.byteMergerAll(start, number, type,serno);
           byte[] terminalPhone = BCD8421Operater.string2Bcd("013123456789");
           //流水号
           byte[] flowNum = BitOperator.numToByteArray(SocketConfig.getSocketMsgCount(), 2);
           //消息头
           byte[] msgHeader = ByteUtil2.byteMergerAll(msgIdb, length, terminalPhone, flowNum);
           //=========================数据合并（消息头，消息体）=====================//
           byte[] bytes = ByteUtil2.byteMergerAll(msgHeader, msgBody);
           //=========================计算校验码==================================//
           String checkCodeHexStr = HexUtil2.getBCC(bytes);
           byte[] checkCode = HexUtil2.hexStringToByte(checkCodeHexStr);
           //=========================合并:消息头 消息体 校验码 得到总数据============//
           byte[] AllData = ByteUtil2.byteMergerAll(bytes, checkCode);
   
           //=========================转义 7E和7D==================================//
           // 转成16进制字符串
           String hexStr = HexUtil2.byte2HexStr(AllData);
           // 替换 7E和7D
           String replaceHexStr = hexStr.replaceAll(FLAG_7D, " 7D 01")
                   .replaceAll(FLAG_7E, " 7D 02")
                   // 最后去除空格
                   .replaceAll(" ", "");
   
           //替换好后 转回byte[]
           byte[] replaceByte = HexUtil2.hexStringToByte(replaceHexStr);
   
           //=========================最终传输给服务器的数据==================================//
           return ByteUtil2.byteMergerAll(flag, replaceByte, flag);
       }
   ```

   

2. 解包

```
/**
     * 解析数据
     *
     * @param byteBuf
     */
    private JTT905Bean resolve(ByteBuf byteBuf) {

        ByteBuf escapeBuf = escape7D(byteBuf);
       /* byte[] bytes = new byte[escapeBuf.readableBytes()];
        int readerIndex = escapeBuf.readerIndex();
        escapeBuf.getBytes(readerIndex, bytes);*/
        JTT905Bean jtt905Bean = new JTT905Bean();
        //解析消息头
        JTT905Bean.MsgHeader msgHeader = new JTT905Bean.MsgHeader();
        byte[] msgId = escapeBuf.readBytes(2).array();
        byte[] msgAttributes = escapeBuf.readBytes(2).array();
        byte[] terminalPhone = escapeBuf.readBytes(6).array();
        byte[] flowNum = escapeBuf.readBytes(2).array();

        msgHeader.setMsgId(msgId);
        msgHeader.setMsgAttributes(msgAttributes);
        msgHeader.setTerminalPhone(terminalPhone);
        msgHeader.setFlowNum(flowNum);

        //消息体长度
        int[] msgBodyAttr = resolveMsgBodyLength(msgAttributes);
        if (msgBodyAttr[0] == 1) {
            //TODO 分包
            escapeBuf.readBytes(4);
        }
        //消息体
        ByteBuf msgBody = escapeBuf.readBytes(msgBodyAttr[1]);
        //校验码
        byte checkCode = escapeBuf.readByte();

        jtt905Bean.setMsgHeader(msgHeader);
        jtt905Bean.setMsgBody(msgBody);
        jtt905Bean.setCheckCode(checkCode);
//        int checkSum4JT808 = getCheckSum4JT808(bytes, 0, bytes.length - 1);
       /* if(checkSum4JT808 != checkCode){
            Log.i(TAG,"校验码不一致");
            return null;
        }*/
        return jtt905Bean;
    }
    public int getCheckSum4JT808(byte[] bs, int start, int end) {
        int cs = 0;
        for (int i = start; i < end; i++) {
            cs ^= bs[i];
        }
        return cs;
    }
    /**
     * 转义 7D 02->7E  7D 01->7D
     *
     * @param byteBuf
     */
    private ByteBuf escape7D(ByteBuf byteBuf) {
        ByteBuf escapeBuf = Unpooled.buffer();
        int length = byteBuf.readableBytes();
        for (; byteBuf.readerIndex() < length; ) {
            byte b = byteBuf.readByte();
            if (b == 0x7D) {
                byte nextB = byteBuf.readByte();
                if (nextB == 0x02) {
                    escapeBuf.writeByte(0x7E);
                } else if (nextB == 0x01) {
                    escapeBuf.writeByte(0x7D);
                } else {
                    escapeBuf.writeByte(b);
                }
            } else {
                escapeBuf.writeByte(b);
            }
        }
        return escapeBuf;
    }
    
     /**
     * 解析消息体属性
     *
     * @return
     */
    private int[] resolveMsgBodyLength(byte[] msgAttributes) {
        ByteBuf msgAttr = Unpooled.buffer(16);
        for (byte attribute : msgAttributes) {
            msgAttr.writeBytes(ByteUtil.byteToBit(attribute));
        }
        //保留位
        msgAttr.readBytes(2);
        //是否分包
        byte subpackage = msgAttr.readByte();
        //加密方式
        byte[] encrypt = msgAttr.readBytes(3).array();
        //消息体长度
        byte[] bodyLength = msgAttr.readBytes(10).array();
        String bits = "";
        for (byte b : bodyLength) {
            bits += b;
        }
        int msgBodyLength = Integer.parseInt(bits, 2);
        return new int[]{subpackage, msgBodyLength};
    }
```

