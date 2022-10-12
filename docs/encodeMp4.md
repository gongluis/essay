1.加密方法

> 视频文件本质就是字节数组文件，加解密的方式很多，本文采用的是当初爱奇艺视频采用过的简单的视频加密方案，优点就是不会损坏视频文件和没有密钥拿到加密后的文件不可解密及无需单独保管密钥。

```
加密mp4的moov box信息。
加密方法：mp4文件中moov信息，除了前四个字节的其余字节与固定一个字节做异或处理
```



2.基本概念

mp4文件：

```
mp4文件是由很多box组成，每个box包含header和data,其中data可以是数据，也可以是别的box。其中主要的box有:ftypbox moovbox mdatbox等。
---ftype  有且只有一个，在文件的开始位置，描述的文件的版本、兼容协议等
---moov   有且只有一个，这个box中不包含具体媒体数据，但包含本文件中所有媒体数据的宏观描述信息，moov box下有mvhd和trak box。
-------mvhd 记录了创建时间、修改时间、时间度量标尺、可播放时长等信息。
-------trak video track包含了mp4中各种信息表，如: stts/ctts/stss/等
-------trak audio track 音频轨道
---free  
---mdat 可以有多个，也可以没有，实际媒体数据。我们最终解码播放的数据都在这里面。
```

字节流：

```
想比较之前的流要么读要么写，RandomAccessFile提供了读写操作.
构造方法：
public RandomAccessFile(String name, String mode)
public RandomAccessFile(File file, String mode)
主要是第二个参数：指定访问模式，有哦4种模式
r:以只读的模式打开，如果调用write方法将会抛出IO异常
rw:以读和写的模式打开
rws:以读和写的模式打开，要求对”文件的内容“和”元数据“的每个更新都同步到存储设备
rwd:以读和写的模式打开，要求对”文件的内容“的每个更新都同步到存储设备
```



3.具体的加密思路

```
box数据组成：box = boxlength + boxTag(moov) + data 我们要处理的是moov数据（TAG+data）   
1.创建视频文件的读写权限流
	RandomAccessFile raf = new RandomAccessFile(file, "rw");
2.读前四个字节即第一个“box ftyp”的总长度
	byte[] boxSizeBuf = new byte[4];
  raf.read(boxSizeBuf, 0, 4);
  int ftypLength = bytesToInt(boxSizeBuf, 0, 4);
3.向后移动指针ftyp长度，到moov的起始处
	raf.seek(ftypLength);
4.读取四个字节即得到“moov box”总长度
	byte[] boxMoovLengthBuf = new byte[4];
  raf.read(boxMoovLengthBuf, 0, 4);
  int moovSize = bytesToInt(boxMoovLengthBuf, 0, 4);
5.再向后读取四个字节即得到moov这几个字节，判断是否为该type,如果不是则返回，表示该mp4文件已被加密
	byte[] boxMoovTypBuf = new byte[4];
  raf.read(boxMoovTypBuf, 0, 4);
  String boxMoovTypStr = byteToString(boxMoovTypBuf);
  if (!TextUtils.equals("moov", boxMoovTypStr)) {//文件已被加密
                return;
            }
6.移动指针到（ftyp的长度加4索引处）刚好是moov box去掉前面的boxLength起始处
	raf.seek(ftypLength + 4);
7.向后读取“moov box总长度” 减4到一个新的长度为“moov box总长度”减4的字节数组中取出整个moov数据，包含type还有data，但是不包含size 因此为 moovSize-4
	byte[] moovData = new byte[moovSize - 4];
  //值全部读取到moovData数组中
  raf.read(moovData, 0, moovSize - 4);
8.对该数组逐个字节做与固定字节的异或处理（这里可选用0x55,原因是101010比较均匀分布亦或结果也比较均匀）
	byte[] enCodeBytes = new byte[moovSize - 4];
for (int i = 0; i < moovData.length; i++) {
      enCodeBytes[i] = (byte) (moovData[i] ^ 0x55);
 }
9.将处理完的字节数组写回去
10.移动指针至box ftyp的总长度加4，修改从该处向后 “moov box总长度”减4的长度
	raf.seek(ftypLength + 4);
  raf.write(enCodeBytes, 0, moovSize - 4);
11.加密完成

```

> 注意：以上是默认mp4文件第二个box是moov的前提下，做的视频加密思路，还有一种情况是box是乱序的，不确定moov box的位置，这个时候就要先找到该box，然后对对应的tag+data进行加密，此处因为box不多采用的是逐个判断的方法，处理了四个box

4.具体的解密思路

```
box数据组成：box = boxlength + boxTag(moov) + data
1.计算出moov^0x55的值，用来判断moov box data是不是已加密的需不需要进行解密
	byte[] encodeMoov = new byte[4];
  encodeMoov[0] = 'm' ^ 0x55;
  encodeMoov[1] = 'o' ^ 0x55;
  encodeMoov[2] = 'o' ^ 0x55;
  encodeMoov[3] = 'v' ^ 0x55;
2.创建视频文件的读写权限流
 RandomAccessFile raf = new RandomAccessFile(file, "rw");
 byte[] boxSizeBuf = new byte[4];
 raf.read(boxSizeBuf, 0, 4);
 int firstBoxLength = bytesToInt(boxSizeBuf, 0, 4);
 byte[] firstBoxTag = new byte[4];
 raf.read(firstBoxTag, 0, 4);
 
 if (Arrays.equals(encodeMoov, firstBoxTag)) {
                //解密然后返回 解密和加密用的同一个方法，一个数亦或固定值两次还是这个数本身
                enCodeMoov(raf, 0);
                return;
            }
 //以次类推，同样处理第二个box           

```

5.流程

```
1.视频落盘成功时会给通知
2.这个时候修改刚落盘的mp4文件（加密）
3.离线视频拉取，首先检测符合条件的文件是否被加密，如果被加密需要解密然后上传
4.高德众包给到抽帧指令后，解密视频然后进行抽帧

```

6.风险

```
1.多线程写文件，也就是加密解密过程需要单线程执行
2.对于锐承tmp文件的处理及博泰不可播放mp4文件处理
```



