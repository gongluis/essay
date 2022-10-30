> 视频抽帧方法总结

视频文件可以分解成一帧一帧的图片，本文要分享的部分就是根据具体的时间戳从制定的视频文件中抽取对应的帧图片。尝试过三种方式。

```
1. 采用安卓原生api的抽帧方式
2. 采用FFmpegMediaMetadataRetriever三方github开源库的方式
3. 采用MediaCodec 调用系统硬件编解码器类进行抽帧 速度最快
```

#### 一、安卓原生api的抽帧方式

> 缺点是在某些机型会存在截图失败率高的问题，在频繁操作的业务场景下也会出现失败的情况。

```
fun getFrameBitmap(uri: String, isSd: Boolean = true): Bitmap? {
        var bitmap: Bitmap? = null
        //MediaMetadataRetriever 是android中定义好的一个类，提供了统一的接口，用于从输入的媒体文件中取得帧和元数据
        val retriever = MediaMetadataRetriever()
        try {
            if (isSd) {
                //（）根据文件路径获取缩略图
                retriever.setDataSource(context, Uri.fromFile(File(uri)))
            } else {
                //根据网络路径获取缩略图
                retriever.setDataSource(uri, HashMap())
            }
            //获得第一帧图片
            bitmap = retriever.frameAtTime
        } catch (e: IllegalArgumentException) {
            LogUtils.e(TAG, e)
        } finally {
            retriever.release()
        }
        return bitmap
    }
```



#### 二、三方库FFMPEG

> 该库是对原生方案的升级，截图速度仍不理想，超过1s。

```
 public static Bitmap getPicFromMp4(long timeMillis, File file) {
        if (file == null) {
            Log.d(TAG, "file == null： ");
            return null;
        }
        //1.先解密视频
        Bitmap bitmap = null;
        try {

            //Mp4FileUtils.deCodeRecoveryMp4File(file.getAbsoluteFile());
            LogExtKt.log(TAG, "getPicFromMp4: " + file.getAbsoluteFile());
            FFmpegMediaMetadataRetriever mmr = new FFmpegMediaMetadataRetriever();
            mmr.setDataSource(file.getAbsolutePath());

            long start = System.currentTimeMillis();
            //单位微秒 OPTION_CLOSEST 最近的i帧,  OPTION_PREVIOUS_SYNC 前一个i帧 ，OPTION_NEXT_SYNC后一个i帧，OPTION_CLOSEST 最近的帧，需要decode多张frame才能接近输入的timestamp，因此速度会慢些。
            bitmap = mmr.getFrameAtTime(timeMillis * 1000, FFmpegMediaMetadataRetriever.OPTION_CLOSEST); // frame at 2 seconds
            if(bitmap == null){
                LogExtKt.log(TAG, "getFrameAtTime bitmap == null： ");
            }
            LogExtKt.log(TAG, "interceptPic spend time： " + (System.currentTimeMillis() - start));
            mmr.release();
            //3.将视频加密回去
            //Mp4FileUtils.enCodeRecoveryMp4File(file);
        } catch (Exception e) {
            e.printStackTrace();
            mmr.release();
            Mp4FileUtils.enCodeRecoveryMp4File(file);
            LogExtKt.log(TAG, "getPicFromMp4 exception: "+e.getMessage());
            return null;
        }
        return bitmap;
    }
```

#### 三、MediaCodeC方案调动系统硬件编解码器进行抽帧

> 目前最优解，截图速度最快差不多三百毫秒，对于系统内存和cpu占用率也比较理想

```
/**
     * 视频抽帧
     * context:[Context]
     * path:[Long]视频路径
     * timestamps:[Long]截取视频的时间戳(微妙)
     * callBack:[MediaCallBack]Bitmap回调
     */
    fun cutVideoFrame(
        path: String,
        timeUs: Long,
        callBack: MediaCallBack
    ) {
        var count = 0
        log(TAG, "start path: $path")
        val file = File(path)
        val exists = file.exists()
        val canRead = file.canRead()
        val canWrite = file.canWrite()
        log(TAG, "media path:$path")
        log(TAG, "media exists:$exists canRead:$canRead canWrite:$canWrite")
        var videoFormat: MediaFormat? = null
        val mediaExtractor = MediaExtractor()
        try {
            mediaExtractor.setDataSource(path)
        } catch (e: Exception) {
            callBack.onFailure("file not exits")
            return
        }
        val trackCount = mediaExtractor.trackCount
        log(TAG, "trackCount : $trackCount")
        for (i in 0 until trackCount) {
            val trackFormat = mediaExtractor.getTrackFormat(i)
            if (trackFormat.getString(MediaFormat.KEY_MIME)?.contains("video") == true) {
                videoFormat = trackFormat
                mediaExtractor.selectTrack(i)
                break
            }
        }
        log(TAG, "MediaFormat : $videoFormat")
        if (videoFormat == null) {
            callBack.onFailure("videoFile cannot read")
            return
        }
        videoFormat.setInteger(
            MediaFormat.KEY_COLOR_FORMAT,
            MediaCodecInfo.CodecCapabilities.COLOR_FormatYUV420Flexible
        )
        videoFormat.setInteger(MediaFormat.KEY_WIDTH, videoFormat.getInteger(MediaFormat.KEY_WIDTH))
        videoFormat.setInteger(
            MediaFormat.KEY_HEIGHT,
            videoFormat.getInteger(MediaFormat.KEY_HEIGHT)
        )
        if (videoFormat.containsKey(MediaFormat.KEY_ROTATION)) {
            val rotate = videoFormat.getInteger(MediaFormat.KEY_ROTATION)
        }
        val duration = videoFormat.getLong(MediaFormat.KEY_DURATION)
        var frameRate = videoFormat.getInteger(MediaFormat.KEY_FRAME_RATE)
        val diff = 1000_000 / frameRate
        log(TAG, "frameRage:$frameRate diff:$diff")
        val codec =
            MediaCodec.createDecoderByType(videoFormat.getString(MediaFormat.KEY_MIME)!!)
        codec.configure(videoFormat, null, null, 0)
        codec.start()

        val bufferInfo = MediaCodec.BufferInfo()
        var inputDone = false
        var outputDone = false
        mediaExtractor.seekTo(timeUs, MediaExtractor.SEEK_TO_PREVIOUS_SYNC)
        Log.i(TAG, "seekTo: $timeUs")
        var getFrame = false
        while (!outputDone) {
            if (!inputDone) {
                //获取输入缓存
                val inputBufferIndex = codec.dequeueInputBuffer(30)
                if (inputBufferIndex >= 0) {
                    Log.i(TAG, "inputBufferIndex: $inputBufferIndex")
                    var keyTime = mediaExtractor.sampleTime
                    var sampleFlags = mediaExtractor.sampleFlags
                    Log.i(TAG, "sampleTime: $keyTime  sampleFlags:$sampleFlags")
                    val inputBuffer = codec.getInputBuffer(inputBufferIndex)
                    val readSampleData = mediaExtractor.readSampleData(inputBuffer!!, 0)
                    Log.i(TAG, "readSampleData: $readSampleData")
                    if (readSampleData > 0) {
                        if (keyTime == timeUs) {
                            getFrame = true
                        }
                        codec.queueInputBuffer(
                            inputBufferIndex,
                            0,
                            readSampleData,
                            keyTime,
                            if (getFrame) MediaCodec.BUFFER_FLAG_END_OF_STREAM else 0
                        )
                        Log.i(TAG, "queueInputBuffer nowTime: $keyTime ")

                        if (!getFrame) {
                            mediaExtractor.advance()
                            keyTime = mediaExtractor.sampleTime
                            sampleFlags = mediaExtractor.sampleFlags
                            if (keyTime >= timeUs - diff / 2 && keyTime <= timeUs + diff / 2 || keyTime >= timeUs) {
                                //当前时间到达指定时间戳附件，即帧间隔的一半
                                getFrame = true
                                Log.i(TAG, "getFrame ")
                            }
                        } else {
                            inputDone = true
                            Log.i(TAG, "inputDone ")
                        }
                    } else {
                        codec.queueInputBuffer(
                            inputBufferIndex,
                            0,
                            0,
                            0,
                            MediaCodec.BUFFER_FLAG_END_OF_STREAM
                        )
                        inputDone = true
                    }
                }
            }
            val outputBufferIndex = codec.dequeueOutputBuffer(bufferInfo, 30)

            if (outputBufferIndex >= 0) {
                count++
                Log.i(
                    TAG,
                    "outputBufferIndex: $outputBufferIndex count:${count} flag:${bufferInfo.flags}"
                )
                val image = codec.getOutputImage(outputBufferIndex)
                Log.i(TAG, "getOutputImage: $image")
                if (bufferInfo.flags == MediaCodec.BUFFER_FLAG_END_OF_STREAM || bufferInfo.flags == 5) {
                    outputDone = true
                    if (image != null) {
                        log(
                            TAG,
                            "getOutputImage count:$count  width:${image.width}  height:${image.height}"
                        )
                        val imageToBitmap = MediaUtil.imageToBitmap(image)
                        callBack.onSuccess(imageToBitmap)
                    } else {
                        callBack.onFailure("")
                    }
                    Log.i(TAG, "outputDone")
                }
                codec.releaseOutputBuffer(outputBufferIndex, false)
            }
        }
        codec.stop()
        codec.release()
        mediaExtractor.release()
        Log.i(TAG, "release")
    }

    interface MediaCallBack {
        fun onSuccess(bitmap: Bitmap?)

        fun onFailure(msg: String)
    }
```

调用：

```
MediaManager.INSTANCE.cutVideoFrame(interceptFile.getAbsolutePath(), interceptTimeMillis * 1000, new MediaManager.MediaCallBack() {
            @Override
            public void onSuccess(@Nullable Bitmap picFromMp4) {
            
            }
            @Override
            public void onFailure(@NonNull String msg) {
            }
        });
```

> 附注：参考博客
>
> https://www.jianshu.com/p/12c3da125ece