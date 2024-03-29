#### 一、生活中的视音频

1. 常见的视频文件格式

   [TOC]

   ：avi，rmvb，mp4，flv，mkv等等，这些格式代表的是封装格式，就是把视频数据和音频数据打包成一个文件的规范，不同的封装格式之间各有劣势。

2. 基本概念

   ```
   AVC/h264: Advanced Video Coding简称
   视频编解码有两套标准：
   	国际电联：h261、h263、h263+等
   	ISO: MPEG-1、MPEG-2、MPEG-4等
   AVC/h264是两大组织结合H263+和MPEG-4的优点联合推出的标准，更高的数据压缩比，同等图像质量的前提下，H264的压缩比比H263高2倍，比MPEG-4高1.5倍。
   
   ```

3. 工具

   查看媒体信息工具：MediaInfo 

   https://mediaarea.net/download/current/MediaInfo_GUI_20.09_Windows.exe

   

   

   #### 二、视频播放器原理

   解协议、解封装、解码视音频、视音频同步

   ```mermaid
   graph TD;
       数据-->解协议;
       解协议-->封装格式数据;
       封装格式数据-->解封装;
       解封装-->视频压缩数据;
       解封装-->音频压缩数据;
       视频压缩数据-->视频解码;
       音频压缩数据-->音频解码;
       视频解码-->视频原数据;
       音频解码-->音频原数据;
       视频原数据--> 视音频同步;
       音频原数据--> 视音频同步;
       视音频同步--> 音频驱动/设备;
       视音频同步--> 视频驱动/设备;
   ```

   

   1. 解协议

      ```
      就是将流媒体协议的数据，解析为标准的相应的封装格式数据。视音频在网络上传播的时候，常常采用各种流媒体协议，例如HTTP，RTMP，或是MMS等等。这些协议在传输视音频数据的同时，也会传输一些信令数据。这些信令数据包括对播放的控制（播放，暂停，停止），或者对网络状态的描述等。解协议的过程中会去除掉信令数据而只保留视音频数据。例如，采用RTMP协议传输的数据，经过解协议操作后，输出FLV格式的数据。
      ```

   2. 解封装

      ```
      就是将输入的封装格式的数据，分离成为音频流压缩编码数据和视频流压缩编码数据。封装格式种类很多，例如MP4，MKV，RMVB，TS，FLV，AVI等等，它的作用就是将已经压缩编码的视频数据和音频数据按照一定的格式放到一起。例如，FLV格式的数据，经过解封装操作后，输出H.264编码的视频码流和AAC编码的音频码流。
      ```

   3. 解码

      ```
      就是将视频/音频压缩编码数据，解码成为非压缩的视频/音频原始数据。音频的压缩编码标准包含AAC，MP3，AC-3等等，视频的压缩编码标准则包含H.264，MPEG2，VC-1等等。解码是整个系统中最重要也是最复杂的一个环节。通过解码，压缩编码的视频数据输出成为非压缩的颜色数据，例如YUV420P，RGB等等；压缩编码的音频数据输出成为非压缩的音频抽样数据，例如PCM数据。
      ```

   4. 视音频同步

      ```
      就是根据解封装模块处理过程中获取到的参数信息，同步解码出来的视频和音频数据，并将视频音频数据送至系统的显卡和声卡播放出来。
      ```

      

   #### 三、流媒体协议

   流媒体协议是服务器与客户端之间通信遵循的规定。

   | 名称     | 推出机构       | 传输层协议 | 客户端   | 目前使用领域    |
   | -------- | -------------- | ---------- | -------- | --------------- |
   | RTSP+RTP | IETF           | TCP+UDP    | VLC, WMP | IPTV            |
   | RTMP     | Adobe Inc.     | TCP        | Flash    | 互联网直播      |
   | RTMFP    | Adobe Inc.     | UDP        | Flash    | 互联网直播      |
   | MMS      | Microsoft Inc. | TCP/UDP    | WMP      | 互联网直播+点播 |
   | HTTP     | WWW+IETF       | TCP        | Flash    | 互联网点播      |

   

4. UDP传输视音频，支持组播，效率较高。但其缺点是网络不好的情况下可能会丢包，影响视频观看质量。

2. 因为互联网网络环境的不稳定性，RTSP+RTP较少用于互联网视音频传输。互联网视频服务通常采用TCP作为其流媒体的传输层协议，因而像RTMP，MMS，HTTP这类的协议广泛用于互联网视音频服务之中。这类协议不会发生丢包，因而保证了视频的质量，但是传输的效率会相对低一些。

#### 四、封装格式

| 名称 | 推出机构           | 流媒体 | 支持的视频编码                 | 支持的音频编码                        | 目前使用领域   |
| ---- | ------------------ | ------ | ------------------------------ | ------------------------------------- | -------------- |
| AVI  | Microsoft Inc.     | 不支持 | 几乎所有格式                   | 几乎所有格式                          | BT下载影视     |
| MP4  | MPEG               | 支持   | MPEG-2, MPEG-4, H.264, H.263等 | AAC, MPEG-1 Layers I, II, III, AC-3等 | 互联网视频网站 |
| TS   | MPEG               | 支持   | MPEG-1, MPEG-2, MPEG-4, H.264  | MPEG-1 Layers I, II, III, AAC,        | IPTV，数字电视 |
| FLV  | Adobe Inc.         | 支持   | Sorenson, VP6, H.264           | MP3, ADPCM, Linear PCM, AAC等         | 互联网视频网站 |
| MKV  | CoreCodec Inc.     | 支持   | 几乎所有格式                   | 几乎所有格式                          | 互联网视频网站 |
| RMVB | Real Networks Inc. | 支持   | RealVideo 8, 9, 10             | AAC, Cook Codec, RealAudio Lossless   | BT下载影视     |

由表可见，除了AVI之外，其他封装格式都支持流媒体，即可以“边下边播”。

#### 五、视频编码

YUV: Y（亮度）UV(色度)

视频编码的主要作用是将视频像素数据（RGB，YUV等）压缩成为视频码流，从而降低视频的数据量。如果视频不经过压缩编码，体积是非常大的，一部电影可能要上百G的空间，视频编码是视音频技术中最重要的技术之一。视频码流的数据量占了视音频总数据量的绝大部分。高效率的视频编码在同等的码率下，可以获得更高的视频质量。

| 名称        | 推出机构       | 推出时间 | 目前使用领域 |
| ----------- | -------------- | -------- | ------------ |
| HEVC(H.265) | MPEG/ITU-T     | 2013     | 研发中       |
| H.264       | MPEG/ITU-T     | 2003     | 各个领域     |
| MPEG4       | MPEG           | 2001     | 不温不火     |
| MPEG2       | MPEG           | 1994     | 数字电视     |
| VP9         | Google         | 2013     | 研发中       |
| VP8         | Google         | 2008     | 不普及       |
| VC-1        | Microsoft Inc. | 2006     | 微软平台     |

由表可见，有两种视频编码方案是最新推出的：VP9和HEVC。目前这两种方案都处于研发阶段，还没有到达实用的程度。当前使用最多的视频编码方案就是H.264。

##### 主流编码标准

H.264仅仅是一个编码标准，而不是一个具体的编码器，H.264只是给编码器的实现提供参照用的

，基于H.264标准的编码器还是很多的。实际中使用最多的就是x264了，性能强悍（超过了很多商业编码器），而且开源。

##### 下一代编码标准

HEVC/H.265 在未来拥有很多大的优势



空间维度：相似像素点表示（16x16矩阵1）可以用一个1表示

时间维度：相邻两帧图片的大部分相似，只有很少的不同点

#### 六、音频编码

音频编码的主要作用是将音频采样数据（PCM等）压缩成为音频码流，从而降低音频的数据量。音频编码也是互联网视音频技术中一个重要的技术。但是一般情况下音频的数据量要远小于视频的数据量，因而即使使用稍微落后的音频编码标准，而导致音频数据量有所增加，也不会对视音频的总数据量产生太大的影响。高效率的音频编码在同等的码率下，可以获得更高的音质。

| 名称 | 机构           | 推出时间 | 使用领域 |
| ---- | -------------- | -------- | -------- |
| AAC  | MPEG           | 1997     | 各个领域 |
| AC-3 | Dolby Inc.     | 1992     | 电影     |
| MP3  | MPEG           | 1993     | 各个领域 |
| WMA  | Microsoft Inc. | 1999     | 微软平台 |

由表可见，近年来并未推出全新的音频编码方案，可见音频编码技术已经基本可以满足人们的需要。音频编码技术近期绝大部分的改动都是在MP3的继任者——AAC的基础上完成的。

#### 七、现有网络视音频平台对比

现有的网络视音频服务主要包括两种方式：点播和直播。点播意即根据用户的需要播放相应的视频节目，这是互联网视音频服务最主要的方式。绝大部分视频网站都提供了点播服务。直播意即互联网视音频平台直接将视频内容实时发送给用户，目前还处于发展阶段。直播在网络电视台，社交视频网站较为常见。

##### 直播平台参数对比

| 名称             | 协议 | 封装 | 视频编码 | 音频编码 | 播放器 |
| ---------------- | ---- | ---- | -------- | -------- | ------ |
| 华数TV           | RTMP | FLV  | H.264    | AAC      | Flash  |
| 六间房           | RTMP | FLV  | H.264    | AAC      | Flash  |
| 中国教育电视台   | RTMP | FLV  | H.264    | AAC      | Flash  |
| 北广传媒移动电视 | RTMP | FLV  | H.264    | AAC      | Flash  |
| 上海IPTV         | RTMP | FLV  | H.264    | AAC      | Flash  |

直播服务普遍采用了RTMP作为流媒体协议，FLV作为封装格式，H.264作为视频编码格式，AAC作为音频编码格式。

采用RTMP作为直播协议的好处在于其被Flash播放器支持。而Flash播放器如今已经安装在全球99%的电脑上，并且与浏览器结合的很好。因此这种流媒体直播平台可以实现“无插件直播”，极大的简化了客户端的操作。

FLV是RTMP使用的封装格式，H.264是当今实际应用中编码效率最高的视频编码标准，AAC则是当今实际应用中编码效率最高的音频编码标准。视频播放器方面，都使用了Flash播放器。

##### 点播平台参数对比

| 名称     | 协议 | 封装 | 视频编码 | 音频编码 | 播放器 |
| -------- | ---- | ---- | -------- | -------- | ------ |
| CNTV     | HTTP | MP4  | H.264    | AAC      | Flash  |
| 优酷网   | HTTP | FLV  | H.264    | AAC      | Flash  |
| 土豆网   | HTTP | F4V  | H.264    | AAC      | Flash  |
| 乐视网   | HTTP | FLV  | H.264    | AAC      | Flash  |
| 新浪视频 | HTTP | FLV  | H.264    | AAC      | Flash  |

点播服务普遍采用了HTTP作为流媒体协议，H.264作为视频编码格式，AAC作为音频编码格式。采用HTTP作为点播协议有以下两点优势：一方面，HTTP是基于TCP协议的应用层协议，媒体传输过程中不会出现丢包等现象，从而保证了视频的质量；另一方面，HTTP被绝大部分的Web服务器支持，因而流媒体服务机构不必投资购买额外的流媒体服务器，从而节约了开支。点播服务采用的封装格式有多种：MP4，FLV，F4V等，它们之间的区别不是很大。视频编码标准和音频编码标准是H.264和AAC。这两种标准分别是当今实际应用中编码效率最高的视频标准和音频标准。视频播放器方面，无一例外的都使用了Flash播放器。

#### 八、码率（bitrate）帧率（f p s）分辨率和清晰度的联系与区别

##### 码率 ，单位时间内传送的数据位数，单位k p b s/s即千位每秒，影响体积和清晰度，与体积成正比：码率越大，体积越大；码率越小，体积越小。

##### 帧率f p s，1秒内传输的图片数量，影响画面流程度，与画面流畅度成正比，帧率越大越流畅，帧率越小画面越有跳动感，帧率越高，每秒经过的画面越多，需要的码率也越高，体积大

##### 分辨率 ，影响图像大小，分别率越高图像越大，分辨率越低，图像越小。

清晰度：

```
在码率一定的情况下，分辨率与清晰度成反比关系：分辨率越高，图像越不清晰，分辨率越低，图像越清晰
在分辨率一定的情况下，码率与清晰度成正比关系，码率越高，图像越清晰；码率越低，图像越不清晰
在码率一定的情况下，分辨率在一定范围内取值都将是清晰的；同样地，在分辨率一定的情况下，码率在一定范围内取值都将是清晰的。
```



#### 九 、I帧 B帧 P帧

1. 为什么要压缩视频？

   未经压缩的视频，数量巨大，存储困难，传输困难，一张图片2M一秒钟三十帧的情况就是2X30=60M,1秒视频60M。

2. 如何进行压缩？

   去除冗余信息（空间冗余，时间冗余，视觉冗余，等）。空间冗余相同区域像素一样，时间冗余相邻帧极少差异点，视觉冗余，眼镜能分辨出几种颜色。

3. 数据压缩分类？

   无损压缩和有损压缩，H264属于有损压缩，在视频编码中每一帧都代表一幅静止的图像，才会有IPB减少数据容量

I帧：关键帧，一幅完整画面，可独立解码

B帧：双向差别帧，本帧与前后帧的差别，不仅要取得前面的缓存画面，还要取得后面的画面，前后画面与本帧数据叠加去的最终画面。

P帧：前向预测帧，表示这一帧和之前的 I帧或P帧的差别，要解码P帧，只需要用之前缓存的画面加本帧的差别，生成最终的画面。



```
I帧：
I帧又称作内部画面，通常是每个GOP的第一个帧，适度的压缩可以当做图像，一部分压缩成P帧，一部分压缩成B帧

I帧也被称为关键帧，是基于离散余弦变换DCT压缩技术

特点：

1.它是一个全帧压缩编码帧

2.解码时仅用I帧的数据就可重新构造完整图像

3.I帧描述了图像背景和运动主体的详情

4.I帧不需要参考其他画面

5.I帧是PB帧的参考帧

6.I帧是帧组GOP的基础帧（第一帧），一组只有一个I帧

7.I帧不需要考虑运动矢量

8.I帧所占数据信息量比较大
P帧：
P帧是通过充分降低于图像序列中前面已编码帧的时间冗余信息来压缩传输数据量的编码图像，也叫预测帧。

当针对连续动态图像编码时，将连续若干幅图像分成I,P,B三种类型，P帧由前面的I、P帧预测而来，也就是考虑运动的特性进行帧间压缩。

特点：

1.P帧是I帧后面1~2帧的编码帧

2.P帧采用运动补偿的方法传送它与前面的I或P帧的差值及运动矢量

3.解码时必须将I帧中的预测帧与预测误差求和才能重构完整的P帧图像

4.P帧属于向前预测的帧间编码。只参考前面最靠近它的I帧或P帧

5.P帧可以是后面P帧的参考帧，或者是后面B帧的参考帧

6.由于P帧是参考帧，可能会造成解码错误的扩散

7.因为是差值传送，压缩比例比较高


```

