# rule
### 接口安全
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
接口url ，  
用户令牌accessToken，  
随机字符串 nonce ，  
时间戳 timestamp ,  
用户id userId   
按照参数字母排序 进行md5 32位大写加密  

accessToken 保存在本地，无需作为参数提交  
deviceId 设备id，作为参数，用于防止多设备登录  
nonce 为客户端随机生成的字符串，需要作为参数提交  
timestamp为时间戳  需要作为参数提交 在app每次启动时，获取服务端时间，计算本地时间和服务端的差值，发送的本地时间戳加上该差值，服务端会校验该时间戳，如果时间差超过5分钟，服务端会拒绝请求.  

例如访问用户信息接口  
```
http://example.com/api/1.0/users?deviceId=abcde&nonce=123213&timestamp=341343131&sing=213231231&userId =3
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

40001,"参数不能为空",  
40002,"解密失败",  
40003,"参数校验失败",  
40004,"用户已经注册",   
40005,"图片必须是 .jpg,.gif,.png,.bmp,.jpeg, 并且文件头必须匹配",  
40006,"文件大小不能超过1M",  
40007,"用户不存在",  
50000,"系统错误";  
