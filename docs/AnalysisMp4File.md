### mp4文件内容解析

#### 一、mp4文件格式

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



####  二、stts/ctts/stsc/stsz等信息表的解析



##### s t t s :Time To Sample Atoms

```
存储了媒体sample的时长信息，提供了时间和相关sample之间的映射关系。该atom包含了一个表，关于time和sample号之间的索引关系。表的每个entry给出了具有相同时间间隔的连续的sample的个数和这些sample的时间间隔值。将这些时间间隔相加在一起，就可以得到一个完整的time与sample之间的映射。将所有的时间间隔相加在一起，就可以得到该track的时间总长。
```

| SampleCount | SampleDuration |      |
| ----------- | -------------- | ---- |
| 4           | 3              |      |
| 2           | 1              |      |
| 3           | 2              |      |

如上表表示，有４个sample的duration为3，2个sample的duration为1, 3个sample的duration为2……注意，这里的duration并不是真实时间，是由时间戳和timescale计算得到的值。



##### stsc:Sample-To-Chunk Atoms

```
为了优化数据访问，通常把sample封装到chunk中，一个chunk可能会包含一个或者多个sample。每个chunk会有不同的size，每个chunk中的sample也会有不同的size。stsc table提供了从sample到chunk的一个映射，
每个table entry可能包含一个或者多个chunk。Table entry包含的内容包括第一个chunk号(从这个位置开始)、每个chunk包含的sample的个数以及sample的描述ID:
```

| First Chunk | Samples per chunk | Sample descriptionID |
| ----------- | ----------------- | -------------------- |
| 1           | 3                 | 23                   |
| 3           | 1                 | 23                   |
| 5           | 1                 | 24                   |

如上表表示，第1、2个chunk包含的sample个数都是3个，sample的ID都是23，第3、4个chunk包含的sample个数都是1个，sample的ID都是23，第5个到最后一个chunk包含的sample个数都是1个，sample的ID都是24．



##### stsz: Sample Size Atom

该表中存储了每个sample的size

| Size |
| ---- |
| Size |
| Size |
| Size |
| Size |

如上表表示，1-5个sample的size都是"size"。



##### stco/co64: Chunk Offset Atom

```
stco/co64中存储了每个chunk在文件中的位置。stco表示use32BitFileOffset，co64表示use64BitFileOffset
```

| Offset |
| ------ |
| Offset |
| Offset |
| Offset |
| Offset |

如上表表示，1-5个chunk在文件中的位置"offset"。

##### stss: Sync Sample Atom

标记了每个关键帧的sample。如果该表不存在，就表示媒体流每个sample都是关键帧。

| Sample1 | 1    |
| ------- | ---- |
| Sample2 | 31   |
| Sample3 | 61   |
| Sample4 | 91   |

如上表表示第1个关键帧是第1帧，第2个关键帧是第31帧，第3个关键帧是第61帧，第4个关键帧是第91帧……

三、具体应用

```
step1: 具体播放时，比如用户需要seek到某一位置，播放器首先可以根据进度条得到用户seek到的时间seektime。

step2: 利用stts表，根据seektime找到其对应的seeksample

step3: 利用stss表，查看seeksample是否为关键帧，如果是则继续，如果不是则按照播放器规定的方法(查找最邻近关键帧、查找前面一个关键帧、查找后面一个关键帧)更新seeksample。

step4: 利用stsc表，根据通过seeksample找到对应的seekchunk和其在chunk中是第几个sample

step5: 利用stsz表，计算seeksample在其所在chunk中的地址偏移量sampleindex

step6: 利用stco表，查找seekchunk在文件中的位置chunkindex

step7: chunkindex + sampleindex 就是用户seek到的帧在文件中的位置。

step8: 取出数据去解码。

step9: 如果文件存在B帧需要根据ctts去调整pts，否则pts=dts。

step10: rennder
```

