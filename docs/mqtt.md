#### 一、为什么使用该种方式？

mqtt的请求数据量很小，灵活便捷，而且单个应用也可很方便的作为一个服务端与其它应用进行协议交互。在物联网领域应用广泛。

### 二、如何使用？

```
 1.引入开源库
 api "org.eclipse.paho:org.eclipse.paho.client.mqttv3:${outer['1.2.1']}"
 api "org.eclipse.paho:org.eclipse.paho.android.service:${outer['1.1.1']}"
 2.初始化
 public synchronized void init(Context context, String serverUri, String userName, String password,
                                  String clientId, MqttCallbackExtended callbackExtended, IMyMqttCallback callback) {
        Log.d(TAG, "mqtt---init--connect");
        if (client != null) {
            if (!client.getClientId().equals(clientId)) {
                try {
                    if (client.isConnected()) {
                        client.disconnect();
                    }
                } catch (MqttException e) {
                    e.printStackTrace();
                }
                //clientId不能使用变化的值，否则sdcard/Android/data/com.t3.driver/files/MqttConnection中文件不断创建，从而导致app占用数据一直增加
                //封装好的MQTTClient供操作，发送和接手等设置都用它
                client = new MqttAndroidClient(context, serverUri, clientId);
            }
        } else {
            //clientId不能使用变化的值，否则sdcard/Android/data/com.t3.driver/files/MqttConnection中文件不断创建，从而导致app占用数据一直增加
            //封装好的MQTTClient供操作，发送和接手等设置都用它
            client = new MqttAndroidClient(context, serverUri, clientId);
        }
        if (client.isConnected()) {
            if (callback != null) {
                callback.onConnectSuccess(client);
            }
            return;
        }
        //设置回调
        client.setCallback(callbackExtended);

        //mqtt连接参数设置
        MqttConnectOptions mqttConnectOptions = new MqttConnectOptions();
        //设置自动重连
        mqttConnectOptions.setAutomaticReconnect(true);
        //设置是否清空session,这里如果设置为false表示服务器会保留客户端的连接记录，设置为true表示每次连接到服务器都以新的身份连接
        mqttConnectOptions.setCleanSession(false);
        //设置连接的用户名
        if (!TextUtils.isEmpty(userName)) {
            mqttConnectOptions.setUserName(userName);
        }
        //设置连接的密码
        if (!TextUtils.isEmpty(password)) {
            mqttConnectOptions.setPassword(password.toCharArray());
        }
        //设置超时时间 单位为秒
        mqttConnectOptions.setConnectionTimeout(10);
        //设置会话心跳时间 单位为秒 服务器会每隔1.5*20秒的时间向客户端发送个消息判断客户端是否在线，但这个方法并没有重连的机制
        mqttConnectOptions.setKeepAliveInterval(20);
        //设置最大重连时间，单位毫秒，默认128000，重连机制：第一次重连在1秒后，第二次2秒后，第三次4秒后，每次翻倍，一直到最大时间
        mqttConnectOptions.setMaxReconnectDelay(8000);
        //选择MQTT版本
        mqttConnectOptions.setMqttVersion(MqttConnectOptions.MQTT_VERSION_3_1_1);
        //证书验证
//        try {
//            KeyStore keyStore = KeyStore.getInstance("PKCS12");
//            InputStream is = context.getAssets().open("mqtt.jks");
//            keyStore.load(is, password.toCharArray());
//            SSLContext sslContext = SSLContext.getInstance("TLS");
//            TrustManagerFactory trustManagerFactory = TrustManagerFactory.getInstance("X509");
//            trustManagerFactory.init(keyStore);
//            sslContext.init(null, trustManagerFactory.getTrustManagers(), null);
//            mqttConnectOptions.setSocketFactory(sslContext.getSocketFactory());
//            is.close();
//        } catch (Exception e) {
//            e.printStackTrace();
//        }
        try {
            LogExtKt.log(TAG, serverUri);
            //设置好相关参数后，开始连接
            final MqttAndroidClient finalClient = client;
            client.connect(mqttConnectOptions, null, new IMqttActionListener() {
                @Override
                public void onSuccess(IMqttToken asyncActionToken) {
                    //连接成功之后设置连接断开的缓冲配置
                    DisconnectedBufferOptions disconnectedBufferOptions = new DisconnectedBufferOptions();
                    //开启缓存
                    disconnectedBufferOptions.setBufferEnabled(true);
                    //离线后最多缓存100调
                    disconnectedBufferOptions.setBufferSize(100);
                    //不一直持续留存
                    disconnectedBufferOptions.setPersistBuffer(false);
                    //删除旧消息
                    disconnectedBufferOptions.setDeleteOldestMessages(false);
                    finalClient.setBufferOpts(disconnectedBufferOptions);

                    //连接成功回调client
                    if (callback != null) {
                        callback.onConnectSuccess(finalClient);
                    }
                }

                @Override
                public void onFailure(IMqttToken asyncActionToken, Throwable exception) {

                    Log.d("connect fail message", exception.getMessage());
                    exception.printStackTrace();
                    if (!TextUtils.isEmpty(exception.getMessage()) && exception.getMessage().contains("已连接客户机")) {
                       /* if (callback != null) {
                            callback.onConnectSuccess(finalClient);
                        }*/
                    } else {
                        //连接失败回调错误
                        if (callback != null) {
                            callback.onConnectFail(exception);
                        }
                    }
                }
            });
        } catch (MqttException ex) {
            ex.printStackTrace();
        }
    }
    
    public boolean isConnected() {
        return client != null && client.isConnected();
    }
    public void disConnect() {
        LogExtKt.log(TAG, "disConnect----");
        if (isConnected()) {
            try {
                LogExtKt.log(TAG, "disConnect----start");
                client.unregisterResources();
                client.close();
                client.disconnect();
                client.setCallback(null);
                client = null;
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
    
    public interface IMyMqttCallback {
        /**
         * 连接成功回调client，注意连接成功后才能订阅主题或发送消息
         *
         * @param client MqttAndroidClient
         */
        void onConnectSuccess(MqttAndroidClient client);

        /**
         * 连接失败回调错误
         *
         * @param throwable 异常信息
         */
        void onConnectFail(Throwable throwable);
    }
    3. 定义发送接口
     /**
     * 发送消息
     * //设置优先级
     * //qos代表服务质量，指的是交通优先级和资源预留控制机制，而不是接收的服务质量。 服务质量是为不同应用程序，用户或数据流提供的不同优先级的能力，或者也可以说是为数据流保证一定的性能水平的能力。
     * //以下是每一个服务质量级别的具体描述
     * //0 ：最多一次传送 (只负责传送，发送过后就不管数据的传送情况)
     * //1 ：至少一次传送 (确认数据交付) 正常使用1
     * //2 ：正好一次传送 (保证数据交付成功)
     */
    public void publish(String topic, String content, int qos, boolean... retained) {
        publish(topic, content, false, qos, retained);
    }
    public void publish(String topic, String content, boolean gzip, int qos, boolean... retained) {
        if (client == null || !client.isConnected()) {
            Log.d(TAG, "publish fail, not connected");
            return;
        }
        Log.d(TAG, "publish topic: " + topic + ", message: " + content); // 执行
        //封装Message
        byte[] datas = content.getBytes();
        if (gzip) {
            datas = GZIPUtils.compress(content);
        }
        MqttMessage message = new MqttMessage(datas);
        //设置优先级
        //qos代表服务质量，指的是交通优先级和资源预留控制机制，而不是接收的服务质量。 服务质量是为不同应用程序，用户或数据流提供的不同优先级的能力，或者也可以说是为数据流保证一定的性能水平的能力。
        //以下是每一个服务质量级别的具体描述
        //0 ：最多一次传送 (只负责传送，发送过后就不管数据的传送情况)
        //1 ：至少一次传送 (确认数据交付)
        //2 ：正好一次传送 (保证数据交付成功)
        message.setQos(qos);
        //设置是否被服务器保留
        if (retained != null && retained.length > 0) {
            message.setRetained(retained[0]);
        } else {
            message.setRetained(false);
        }
        //发送到对应主题
        try {
            client.publish(topic, message);
        } catch (MqttException e) {
            e.printStackTrace();

        }
    }
    4. 订阅主题
    public <T> void subscribe(String topic, String msgId, IMyMqttMessageListener<T> listener) {
        subscribe(topic, msgId, true, listener);
    }
    /**
     * 订阅主题，过滤对应msgId的payload content.
     *
     * @param topic     订阅的主题
     * @param msgId     过滤对应的msgid.当传入的参数为null，则listenr将返回整个payload body
     *                  注意：
     * @param enableLog 是否需要输出日志
     * @param listener  订阅回调
     */
    public void subscribe(String topic, String msgId, boolean enableLog, IMyMqttMessageListener listener) {
        if (client == null || !client.isConnected()) {
            Log.d(TAG, "subscribe fail, not connected");
            return;
        }
        try {
            Log.d(TAG, "subscribe " + topic);  // 执行
//            client.unsubscribe(topic);
            //主题、QOS、context,订阅监听，消息监听
            client.subscribe(topic, 1, null, new IMqttActionListener() {
                @Override
                public void onSuccess(IMqttToken asyncActionToken) {
                    LogUtil.e("Subscribed!");
                    LogExtKt.log(TAG, asyncActionToken.getResponse().getToken().toString());
                }

                @Override
                public void onFailure(IMqttToken asyncActionToken, Throwable exception) {
                    Log.e("Failed to subscribe");
                    Log.d(TAG, asyncActionToken.getResponse().getToken().toString());
                }
            }, (topic1, message) -> {
                fixExecutors.execute(() -> {
                    if (enableLog) { // 执行
                        LogExtKt.log("incoming-->mqtt", "topic=" + topic1 + ", message=" + message.toString());
                    }
                    if (listener == null || message == null) {
                        return;
                    }
                    String payloadContent = message.toString();
                    if (TextUtils.isEmpty(msgId)) {
                        listener.messageArrived(topic1, payloadContent);
                        return;
                    }
                    try {
                        PayloadEntity entity = (PayloadEntity) JSON.parseObject(payloadContent, listener);
                        if (entity != null && TextUtils.equals(entity.msgHdr.msgID, msgId)) {
                            listener.messageArrived(topic1, entity);
                        }
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                });
            });
        } catch (MqttException ex) {
            ex.printStackTrace();
        }
    }
    public void unsubscribe(String topic) {
        if (client == null || !client.isConnected()) {
            Log.d(TAG, "subscribe fail, not connected");
            return;
        }
        Log.d(TAG, "unsubscribe " + topic);
        try {
            client.unsubscribe(topic);
        } catch (MqttException e) {
            e.printStackTrace();
        }
    }
    public static void cleanCache() {
        // 清除mqtt数据，否则占用空间会越来越大
        FileUtil.deleteFile(AppUtils.context.getExternalFilesDir(null) + "/MqttConnection");
    }
    public static abstract class IMyMqttMessageListener<T> extends TypeReference<T> {
        /**
         * 订阅收到消息回调
         *
         * @param topic  主题
         * @param entity 消息
         */
        protected abstract void messageArrived(String topic, T entity);
    }
```

