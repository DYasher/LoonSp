# Loon脚本类型
Loon支持的脚本类型：

## http-request
### 语法
```
http-request ^https?:\/\/(www.)?(example)\.com script-path=localscript.js,tag = requestScript,enable=true,requires-body = true
```
此类型脚本会在一个http请求发起时调用，在此类脚本中可以使用如下变量

- 所有loon script API
- $request: 一个http请求信息
    - $request.url: String类型，请求URL
    - $request.method: String类型，请求方法
    - $request.headers: js对象，请求头
    - $request.body: String或者Uint8Array类型，当requires-body = true时才有值，请求的body
- $response: undefined
- $done()方法参数说明：
    - $done(): 不传任何参数，表示放弃该请求，请求连接会直接断开
    - $done({}): 空js对象，请求继续，任何请求参数不会有任何变化
    - $done({url:"https://new.example.com/"}): 替换原来的url
    - $done({headers:{}}): 替换原来的request headers
    - $done({response:{
        status:200,
        headers:{},//response headers
        body:"" //response body String type
        }}): 直接向该请求返回一个假的响应

## http-response
### 语法
```
http-response ^https?:\/\/(www.)?(example)\.com script-path=https://example.com/loon.js,timeout=10,requires-body = true,tag = responseScript,enable=true
```
此类脚本会在http请求得到响应后调用，requires-body参数表示是否截取响应体，
- 所有loon script API
- $request: http请求信息
    - $request.url: String类型，请求URL
    - $request.method: String类型，请求方法
    - $request.headers: js对象，请求头
    - $request.body: String或者Uint8Array类型，如果请求带有body，此参数才有值
- $response: http响应信息
    - $response.status: 响应状态
    - $response.headers: 响应头
    - $response.body: String或者Uint8Array类型，如果响应带有body，并且requires-body = true时此参数才有值
- $done()方法参数说明：
    - $done(): 不传任何参数，表示放弃该请求，请求连接会直接断开
    - $done({}): 空js对象，请求继续，任何请求参数不会有任何变化
    - $done({response:{
        status:200,
        headers:{},//response headers
        body:"" //response body String type
        }}): 直接向该请求返回一个假的响应

## cron
### 语法
```
cron "0 8 * * *" script-path=cron.js,tag = responseScript,enable=true
```
- 根据设定的cron表达式触发脚本
- "* * * * *" : 分 时 日 月 周 
- "* * * * * *" :秒 分 时 日 月 周

## network-changed
### 语法
```
network-changed script-path=https://raw.githubusercontent.com/Loon0x00/LoonExampleConfig/master/Script/netChanged.js, tag=changeModel,enable=true
```
当网络环境发生变化时会调用改脚本，如果有多个这种类型的脚本，只会调用配置文件中的第一个

## generic
### 语法
```
generic script-path=https://raw.githubusercontent.com/Loon0x00/LoonExampleConfig/master/Script/generic_example.js,tag=GeoLocation,timeout=10,img-url=location.fill.viewfinder.system
```
以节点、策略组、规则等配置为参数的脚本，需要在app内部页面手动进行触发，不会主动触发。

# Loon Script API

## 工具
- **console.log(msg))**

打印日志，参数msg:String

- **setInterval(function, milliseconds)**
- **setTimeout(function, milliseconds)**

延迟执行，两个方法是的使用方式和参数和浏览器中一致


## 基本信息
- **$loon**

`$loon`: device Name, system version, app version, build version

- **$script**

`$script.name`: 被执行的脚本名称

`$script.startTime`: 执行脚本的时间

- **$config**

`$config.getConfig()`: 获取当前配置信息，返回json字符串
 ```
 {
    "running_model": 1,//运行模式，0:全局直连 1:分流模式 2:全局代理
    "all_buildin_nodes": [
        "DIRECT",
        "REJECT"
    ],
    "global_proxy": "节点选择",
    "all_policy_groups": [
        "宝贝支付",
        "奈飞影视",
        "运营劫持",
        "负载均衡",
        "全球直连",
        "国内媒体",
        "HK",
        "广告拦截",
        "漏网之鱼",
        "WiFi",
        "节点选择",
        "JP",
        "苹果服务",
        "测速",
        "健康模式",
        "BliBliArea",
        "TW",
        "谷歌服务",
        "油管视频",
        "国外网站",
        "网易解锁"
    ],
    "ssid": "loon-wifi-5g",
    "final": "节点选择",
    "policy_select": {
        "苹果服务": "全球直连",
        "广告拦截": "REJECT",
        "BliBliArea": "HK",
        "油管视频": "节点选择",
        "宝贝支付": "🇺🇲 v1|美国|A|6台负载",
        "奈飞影视": "DIRECT",
        "测速": "DIRECT",
        "谷歌服务": "健康模式",
        "国内媒体": "全球直连",
        "运营劫持": "REJECT",
        "节点选择": "JP",
        "漏网之鱼": "节点选择",
        "JP": "🇯🇵 v1|日本|A|7台负载",
        "网易解锁": "DIRECT",
        "全球直连": "DIRECT",
        "TW": "🇭🇰 v1|香港|D|7台负载|原生",
        "国外网站": "节点选择"
    }
}
 ```
`$config.getConfig(policyName,selectName)`: 设置policyName策略组所选策略为selectName，参数都为String，失败返回false，成功返回true

`$config.getSubPolicies(policyName, function(subPolicies){})`:获取policyName策略组的所有子策略，获取成功后回调，subPolicies为字符串子数组

`$config.getSelectedPolicy(policyName)`: 返回policyName策略组所选的子策略组名称

`$config.setRunningModel(model)`: 设置Loon当前运行模式，model类型为int，0:全局直连 1:分流模式 2:全局代理

## 本地存储
- **$persistentStore**

`$persistentStore.read(value,[key])`: 将value以key为键存储在本地，value和key类型都为字符串，key不传时为当前脚本名字的hash值，存储成功后返回true，失败返回false

`$persistentStore.read([key])`: 读取保存在本地中key映射的值，key不传时为当前脚本名字的hash值，返回相应的value，key和value都为字符串

`$persistentStore.remove()`: 清除所有使用脚本API保存在本地的数据

- **$notification**

`$notification.post(title,subtitle,content,[attach])`: 发起一个ios的本地通知，前三个参数分别为标题、副标题、通知内容，都为String类型，attach为可选内容，最为通知的附件，如通知带的一个图片\视频url或者点击通知时的触发的openurl
```
//当attach为一个字符串时，表示点击通知的跳转链接
$notification.post("title","subtitle","content","loon://switch")

//如果既要支持附件和点击跳转，传入js对象
var attach = {
    "openUrl":"loon://switch",
    "mediaUrl":"https://example.com/img"
}
$notification.post("title","subtitle","content",attach)
```

## 网络请求

- **$httpClient**

`$httpClient.get(params, function(errormsg,response,data){})`:  发起一个http get请求，params是请求参数，callback是请求结束的回调

```
//params为请求参数：如下
{
    url:"https://example.com/",
    timeout: 2000, //请求超时，单位ms
    headers:{
        Content-Type:"application/json"
    },
    body:"{}",//仅仅在post请求中有效
    node:"HK - v1.0",//指定该请求使用哪一个节点或者策略组
}

//回调参数
errormsg: 失败原因，String类型，请求成功为null
response: js对象
{
    status:200,
    headers:{
        content-length:200
    }
}
data: String类型，响应body

```

`$httpClient.post(params, function(errormsg,response,data){})`: 发起post请求，参数、callback参数同get

`$httpClient.head(params, function(errormsg,response,data){})`: 发起head请求，参数、callback参数同get

`$httpClient.delete(params, function(errormsg,response,data){})`: 发起delete请求，参数、callback参数同get

`$httpClient.put(params, function(errormsg,response,data){})`: 发起put请求，参数、callback参数同get

`$httpClient.options(params, function(errormsg,response,data){})`: 发起options请求，参数、callback参数同get

`$httpClient.patch(params, function(errormsg,response,data){})`: 发起patch请求，参数、callback参数同get

## 其他

- **$done()**

在一般的脚本中，调用$done()表示结束脚本的执行，loon内部会进行脚本资源的释放，所以为了loon的js资源请在脚本结束时调用$done()释放资源；在http-request、http-response类型的脚本中，$done()的调用请参考相关脚本类型的说明：[Loon脚本类型](https://github.com/Loon0x00/LoonExampleConfig/blob/master/Script/script_README.md)

- **$envirnoment**

仅用于generic类型的脚本中，当generic类型的脚本运用于某个节点时，`$envirnoment`对象有如下几个属性

`$environment.params.node`: 表示节点名称（build 410版本后推荐用nodeInfo）

`$environment.params.nodeInfo`: 节点简洁信息（为了安全起见，不会返回所有节点信息）
