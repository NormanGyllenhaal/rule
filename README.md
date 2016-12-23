# rule
### 接口规范

1. 协议 生产环境使用 https ,开发，测试环境使用http
2. url 中体现 版本 将版本放在域名之后 https://api.example.com/v1/
3. url 路径 每个网址代表一种资源（resource），而且所用的名词与数据库名对应。数据库中的表都是同种记录的"集合"（collection），API中的名词使用复数
4. HTTP动词
GET（SELECT）：从服务器取出资源（一项或多项）。
POST（CREATE）：在服务器新建一个资源。
PUT（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源）。
PATCH（UPDATE）：在服务器更新资源（客户端提供改变的属性）。
DELETE（DELETE）：从服务器删除资源。
5.	过滤信息 
分页 ?pageNo=2&pageSize =100：指定第几页，以及每页的记录数。
排序 ?sortBy=name&order=asc：指定返回结果按照哪个属性排序，以及排序顺序。
过滤 ?animal_type_id=1：指定筛选条件
6.	http状态码  
200 – OK – 请求成功  
400 – Bad Request – 请求参数错误，数据体体现详细错误信息，在客户端提交的参数错误，加密错误时返回此状态码  
401 – Unauthorized – 未授权，客户端在缺少签名信息时返回此状态码  
403 – Forbidden – 禁止，客户端在时间戳过期，sign 等签名信息验证失败时 返回此状态码  
404 – Not found – 没有发现该资源  
405 - Method Not Allowed - 请求方法不允许  请求时使用错误的http动词时返回次错误码  
406 - Not Acceptable 表示请求资源的MIME类型与客户端中Accept头信息中指定的类型不一致  
411 - SC_LENGTH_REQUIRED 客户端需要发送Content-Length头信息指出发送给服务器的数据的大小  
415 - Unsupported Media Type 不支持的媒体格式 服务端仅支持application/json  
500 - Internal Server Error  - 内部服务器错误   
502 - Bad Gateway 代理服务器错误  
503 - Service Unavailable 服务器由于在维护或已经超载而无法响应  
7.	错误处理  
接口请求发生错误时，返回以下错误信息
```
{
  "timestamp": "当前服务器时间",
  "url": "错误的url",
  "error": "错误信息提示",
  "code": "错误码",
  "requestId":"每个请求的唯一id，为方便查看异常日志",
  "extra":"附加信息"
}
```
8.	业务异常错误码定义
业务错误码统一使用100XX
10001,"用户注册email重复",
10002,"用户名或密码错误",
10003,"金币不足",
10004,"支付失败",
10005,"你已被举报",
10006,"用户已在其他设备上登录",
10007,"你的年龄小于17岁",
10008,"你已经评价过了",
10009,"该设备注册账号已超过限制";

9.	返回结果
GET /collection：返回资源对象的列表（数组）
GET /collection/resource：返回单个资源对象
POST /collection：返回新生成的资源对象
PUT /collection/resource：返回完整的资源对象
PATCH /collection/resource：返回完整的资源对象
DELETE /collection/resource：返回一个空文档

10. HTTP Request Header
Content-Type: application/json;charset=utf-8
Accept-Language: 国家缩写，服务端根据此信息返回国际化信息
Content-Length：数据长度
11. HTTP Responses Header
Content-Language：响应体语言
Content-Type：application/json;charset=utf-8
Content-Length：响应体长度



### 接口数据加密
- 数据加密  
接口数据统一采用des+rsa混合加密方式传输  
1、信息(明文)采用DES密钥加密。  
2、使用RSA加密前面的DES密钥信息。

针对post put patch 请求的数据体  
请求方发送数据：  
1 本地随机生成des密钥  
2 用生成的des密钥加密数据  
3 用rsa加密des密钥  
加密后数据体  
```
{
   Data:des加密的数据体
   Key：rsa加密的des密钥
}
```
而接收方接收到信息后：  
1、用RSA解密DES密钥信息    
2、再用RSA解密获取到的des密钥解密密文信息  
响应数据体
```
{
   Data:des加密的数据体
   Key：rsa加密的des密钥
}
```
### 接口签名  
为保证用户accessToken 的保密性和url的访问权限以及防止通过抓包的方式重复提交请求  
*除用户登录和获取配置信息接口以外，其他接口发送时需要添加签名参数sign*


签名算法为 
接口url， 用户令牌 accessToken，随机字符串 nonce，时间戳 timestamp,  用户id userId   
按照参数字母排序 进行md5 32位大写加密 

例如访问用户信息接口  

```
http://example.com/api/1.0/users?deviceId=abcde&nonce=123213&timestamp=341343131&sign=213231231&userId=3
url  http://example.com/api/1.0/users
accessToken  123456
deviceId=abcde
nonce  abc
timestamp  789
userId 3
```
签名方法 MD5（url+accessToken+ deviceId +nonce+timestamp+userId）
将要进行签名的值http://example.com/api/1.0/users123456abcdeabc7893
对这个值进行MD5 生成sign 

accessToken 保存在本地，无需作为参数提交  
deviceId 设备id，作为参数，用于防止多设备登录  
nonce 为客户端随机生成的字符串，需要作为参数提交  
timestamp为时间戳  需要作为参数提交 在app每次启动时，获取服务端时间，计算本地时间和服务端的差值，发送的本地时间戳加上该差值，服务端会校验该时间戳，如果时间差超过5分钟，服务端会拒绝请求.  

