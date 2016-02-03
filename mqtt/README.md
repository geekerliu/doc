## 集成前准备
到[大数点开发者平台](https://dev.dasudian.com/)注册成为大数点合作伙伴并创建应用

## 下载SDK
到[大数点官网](https://dev.dasudian.com/sdk/client)下载MQTT SDK.

## SDK内容

 - dsd-lib-MQTT.jar
 - dsd-lib-BLE.jar
 - libdsd_lib_mqtt.so
 
## 配置工程
### 导入库和jar包
拷贝libdsd_lib_mqtt.so到libs/armeabi目录下，如果没有armeabi目录，请手动创建该目录。
拷贝dsd-lib-MQTT.jar和dsd-lib-BLE.jar到libs目录下，如下图所示。<br/>
![导入.so到工程目录下](doc/1.png)

### 配置权限
在AndroidManifest.xml中加入如下内容使能必要的访问权限.
```xml
<!-- 访问网络权限 -->
<uses-permission android:name="android.permission.INTERNET" />
<!-- 获取mac地址权限 -->
<uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<!-- 使用蓝牙设备的权限 -->
<uses-permission android:name="android.permission.BLUETOOTH" />
<!-- 管理蓝牙设备的权限 -->
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
```

## 使用SDK
### 实现如下接口
```
implements DsdMqttInterface
```
```
@Override
public void mqttCallBack(int event, String topicName, String content, int contentLen);
```
* 注意：不要在该函数中直接刷新UI。
* 参数说明

| event |  topicName | content |  contentLen  |
|-------|------------|---------|-------|
| 0:重连成功| null |null | 0 |
| 1:连接断开|  null |null   | 0 | 
| 2:订阅成功 | null |null | 0 | 
| 3:订阅失败 | null |null | 0 |
| 4:取消订阅成功 | null |null | 0 |
| 5:取消订阅失败 | null |null | 0 |
| 6:发布成功 | null |null | 0 |
| 7:发布失败 | null |null | 0 |
| 8:收到别人发布的消息|  话题名 | 消息内容 | 消息内容长度 |
| 9:收到订阅列表 | null | 该用户的订阅列表，格式举例：[{"t":"topicName0","q":"0"},{"t": "topicName1","q": "1"}...] | 订阅列表长度 |

### 链接服务器
```java
// 获取sdk实例
private DsdMqttClient dsdMqttClient = DsdMqttClient.getInstance();
```
```java
// 注册回调函数到SDK中,将callback函数注册到SDK中,SDK会在事件发生的时候回调该函数.
// 其中this为实现了mqttCallBack()回调函数的类
dsdMqttClient.setMqttCallBack(this);
```

```java
/**
 * 初始化sdk并链接服务器。
 * @param  serverAddress  服务器地址，为空则默认使用大数点提供的公有云服务
 * @param  version      "1.0"
 * @param  appID        app的id，在大数点开发者平台创建应用时获得
 * @param  appKey       app秘钥，在大数点开发者平台创建应用时获得
 * @param  userID       用户名,用户身份唯一标识
 * 						传null表示不统计，或者传一个用户信息的JSON，比如{"name":"jack","gender":"male","region":"Beijing"}
 * @param  clientID	    客户端id（手机mac地址），用于唯一标记一个客户端
 * @param  option       可选的用户信息，用于后台统计；
 * @param  obj          DsdMqttClient的实例对象
 * @return              成功:0,失败:-1
 */
int dsdMqttConnect(String serverAddress, String version, String appID, String appKey,
				String userID, String clientID, String option, DsdMqttClient obj);
```
## 订阅
Qos(Quality of Service):统一说明如下

> 0.”最多一次“，尽操作环境所能提供的最大努力分发消息。消息可能会丢失。例如，这个等级可用于环境传感器数据，单次的数据丢失没关系，因为不久之后会再次发送。<br/>
> 1.“至少一次”，保证消息可以到达，但是可能会重复。 <br/>
> 2.“仅一次”，保证消息只到达一次。例如，这个等级可用在一个计费系统中，这里如果消息重复或丢失会导致不正确的收费。

```java
/**
 * 注意：该函数返回成功仅代表数据成功传入sdk中，
 * 数据是否真正成功发送到服务器需要从回调函数（mqttCallBack）中获得。
 * 订阅一个话题
 * @param  topic   话题名
 * @param  qos     服务质量，0|1|2
 * @return         成功:0,失败:-1。
 */
int dsdMqttSubscribe(String topic, int qos);
```
## 取消订阅
```java
/**
 * 注意：该函数返回成功仅代表数据成功传入sdk中，
 * 数据是否真正成功发送到服务器需要从回调函数（mqttCallBack）中获得。
 * 取消订阅某个话题
 * @param  topic   取消订阅得话题名
 * @return         成功:0,失败:-1
 */
int dsdMqttUnsubscribe(String topic);
```
## 发布话题

```java
/**
 * 注意：该函数返回成功仅代表数据成功传入sdk中，
 * 数据是否真正成功发送到服务器需要从回调函数（mqttCallBack）中获得。
 * 发布一个话题
 * @param  topicName  话题名
 * @param  content    话题内容
 * @param  contentLen 话题内容长度
 * @param  qos        服务质量，0|1|2
 * @param  retained   如果设置为true，最新发布的消息会保留到服务器中，如果有新的订阅者订阅了该话题，则把该话题发布给新的订阅者。
 * @return      成功:0,失败:-1
 */
int dsdMqttPublish(String topicName, String content, int contentLen, int qos, boolean retained);
```
## 获取订阅列表
```java
/**
 * 注意：该函数返回成功仅代表数据成功传入sdk中，
 * 数据是否真正成功发送到服务器需要从回调函数（mqttCallBack）中获得。
 * 该方法为异步方法，调用成功后，订阅的列表内容将在回调函数mqttCallBack中返回。
 * @return	成功：0，失败：-1
 */
int dsdMqttGetSubList();
```

## 与服务器断开链接
```java
/**
 * 与服务器断开链接
 */
int dsdMqttDisconnect();
```
## 下载Android示例程序
[下载Android示例程序](https://www.github.com/Dasudian/imsdk-example-android)
