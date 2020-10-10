# JWT学习
## JWT简介
Json web token是为了在网络应用环境间传递声明而执行的一种基于JSON的开放标准，该token被设计为紧凑且安全。JWT的声明一般被用来在身份提供者和服务提供者之间传递被认证的用户身份信息，以便于从服务器获取资源，也可以增加一些额外的其它业务逻辑所必须的声明信息，该token也可直接被用于认证，也可被加密。
### 传统的session认证
http协议本身是一种无状态的协议，而这就意味着如果用户向应用提供了用户名和密码来进行用户认证，那么下一次请求时，用户还要再一次进行用户认证才行，因为根据http协议，应用并不能知道是哪个用户发出的请求，所以为了让应用能识别是哪个用户发出的请求，只能在服务器存储一份用户登录的信息，这份登录信息会在响应时传递给浏览器，告诉其保存为cookie,以便下次请求时发送给应用，这样应用就能识别请求来自哪个用户了,这就是传统的基于session认证。  
但是这种基于session的认证使应用本身很难得到扩展，随着不同客户端用户的增加，独立的服务器已无法承载更多的用户，而这时候基于session认证应用的问题就会暴露出来。
#### 基于session认证暴露的问题
**Session：**每个用户经过应用认证后，应用都要在服务器端做一次记录方便下次用户请求的鉴别，通常session都是保存在内存中，随着用户数量的增加，服务器的开销会明显增加。
**扩展性：**用户认证以后，服务器做认证记录，如果认证的记录被保存在内存中，意味着用户下次请求必须要请求在这台服务器上，这样才能拿到授权的资源，这样在分布式的应用上，相应的限制了负载均衡器的能力，即限制了应用的扩展能力。
**CSRF：**因为是基于cookie来进行用户识别，cookie如果被截获，用户就会很容易收到跨站请求伪造的攻击。
### 基于token的鉴权机制
基于token的鉴权机制类似于http协议也是无状态的，它不需要在服务端去保留用户的认证信息或者会话信息。这就意味着基于token认证机制的应用不需要去考虑用户在哪一台服务器登录了，这就为应用的扩展提供了便利。流程如下：
* 用户使用用户名密码来请求服务器
* 服务器进行验证用户的信息
* 服务器通过验证发送给用户一个token
* 客户端存储token，并在每次请求时附送上这个token值
* 服务器验证token值，并返回数据
这个tokne必须在每次请求时传递给服务器，应保存在请求头里面，另外，服务器要支持跨来源资源共享策略。
## JWT组成
JWT由三段信息组成，将这三段信息文本用```.```链接在一起就构成了JWT字符串，例如：
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```
### JWT的构成
第一部分称为头部（header），第二部分称为载荷（playload），第三部分称为签证（signature）
#### header
JWT的头部包含两部分信息：
* 声明类型，这里是JWT
* 声明加密的算法，通常直接使用HMAC SHA256
完整的头部就像下面的JSON：
```json
{
  'typ': 'JWT',
  'alg': 'HS256'
}
```
然后将头部进行base64加密就构成了第一部分，该加密是可以对称解密的：
```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
```
#### playload
载荷就是存放有效信息的地方，这个有效信息包含三部分：
* 标准中注册的声明
* 公共的声明
* 私有的声明
##### 标准中注册的声明
* iss：JWT签发者
* sub：JWT所面向的用户
* aud：接收JWT的一方
* exp：JWT的过期时间，这个过期时间必须大于签发时间
* nbf：定义在什么时间之前，该JWT都是不可用的
* iatrogenic：JWT的签发时间
* jti：JWT的唯一身份标识，主要用来作为一次性token，从而回避重放攻击
##### 公共的声明
公共的声明可以添加任何的信息，一般添加用户的相关信息或其他业务需要的必要信息.但不建议添加敏感信息，因为该部分在客户端可解密。
##### 私有的声明
私有声明是提供者和消费者所共同定义的声明，一般不建议存放敏感信息，因为base64是对称解密的，意味着该部分信息可以归类为明文信息。
定义一个playload：
```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}

```
然后将其进行base64加密，得到JWT的第二部分：
```
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9
```
#### signature
JWT的第三部分是一个签证信息，这个签证信息由三部分组成：
* header（base64后的）
* playload（base64后的）
* secret
secret是保存在服务器端的，jwt的签发生成也是在服务器端的，secret就是用来进行jwt的签发和jwt的验证，所以，它就是你服务端的私钥，在任何场景都不应该流露出去。一旦客户端得知这个secret, 那就意味着客户端是可以自我签发jwt了。
## JWT的应用
一般是在请求头里添加Authorization，并加上Bearer标注：
```
fetch('api/user/1', {
  headers: {
    'Authorization': 'Bearer ' + token
  }
})
```
服务端会验证token，如果验证通过就会返回相应的资源。整个流程就是这样的:
![JWT](https://upload-images.jianshu.io/upload_images/1821058-2e28fe6c997a60c9.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)
## JWT总结
优点：
* 由于json的通用性，所以JWT可以跨语言支持
* 由于有了playload，所以JWT可以在自身存储一些其他业务逻辑所必要的非敏感信息
* 便于传输，JWT构成简单，占用内存小
* 不需要再服务器保存会话信息，便于应用扩展