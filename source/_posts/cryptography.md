---
title: 密码学
tags: [article,前端]
description: 前后端信息传输加密方式AES、RSA等方法......
---

### 函数方法

#### 前端 Crypto.js 库

> console.log(crypto);

<p align="center" ><img src="/imgs/Cryptography_01.png" align=center></p>

[github源码地址](https://github.com/brix/crypto-js) 、[API文档](https://cryptojs.gitbook.io/docs/)

##### AES方法

> 通过调用 **AES.encrypt (a, b, c).toString()** 进行**加密**

- a 表示要加密的内容

+ b 表示加密的密钥 ( 随便取 )

+ c 可以不写，未研究

> 通过调用 **AES.decrypt (a, b, c).toString()** 进行**解密**

- a 表示加密后的内容

+ b 表示加密时放入的密钥  ( 和加密时一致 )
<p align="center" ><img src="/imgs/Cryptography_02.png" align=center></p>

> 加密对象

```js
let data = {name:'data'}
let a = crypto.AES.encrypt(JSON.stringify(data),'key').toString();
let b = crypto.AES.decrypt(a,'key').toString(crypto.enc.Utf8)
console.log(a); 
console.log(JSON.parse(b)); //{name:'data'}
```

##### HMAC

> 密钥哈希消息认证码（HMAC）是使用密码哈希函数进行消息认证的机制。
>
> **HMAC可以与任何迭代的加密哈希函数结合使用**。
>
> HMAC中所使用的单向散列函数并不仅限于一种，任何高强度的单向散列函数都可以被用于HMAC，如果将来设计出的新的单向散列函数，也同样可以使用。
>
> 使用SHA-1、SHA-224、SHA-256、SHA-384、SHA-512所构造的HMAC，分别称为HMAC-SHA1、HMAC-SHA-224、HMAC-SHA-384、HMAC-SHA-512。

```js
var hash = Crypto.HmacMD5("Message", "key");
var hash = Crypto.HmacSHA1("Message", "key");
var hash = Crypto.HmacSHA256("Message", "key");
var hash = Crypto.HmacSHA512("Message", "key");
```

##### MD5和HmacMD5

> MD5 可以直接加密，HmacMD5 需要一个密钥

```js
let a = crypto.MD5('123').toString();
let b = crypto.HmacMD5('123','key').toString();
```

#### 非对称加密 RSA

>  1977年，三位数学家Rivest、Shamir 和 Adleman 设计了一种算法，可以实现非对称加密。这种算法用他们三个人的名字命名，叫做RSA算法。从那时直到现在，RSA算法一直是最广为使用的"非对称加密算法"。毫不夸张地说，只要有计算机网络的地方，就有RSA算法。

##### 特性

> RSA 会生成 **公钥** 和 **私钥**；
>
> 通过 **公钥** 加密的密文 只能由 **私钥** 解开，不能被 **公钥** 自己解开；
>
> **公钥** 不能推导出 **私钥**；

##### [参考网址](https://blog.csdn.net/gulang03/article/details/82230408)

#### AES 和 RSA 结合使用

##### 逻辑分析

> **利用RSA来加密传输AES的密钥，用AES来加密数据**

> 1、客户端启动，发送请求到服务端，服务端用RSA算法生成一对公钥和私钥，我们简称为pubkey1,prikey1，将公钥pubkey1返回客户端。
>
> 2、客户端拿到服务端返回的公钥pubkey1后，自己用RSA算法生成一对公钥和私钥，我们简称为pubkey2,prikey2，并将公钥pubkey2利用服务端传过来的公钥pubkey1加密，加密后，传输给服务端。
>
> 3、此时服务端收到客户端传输的密文，用私钥prikey1进行解密，因为数据pubkey2是用服务端的公钥pubkey2加密的，那么，通过解密就可以得到客户端生成的公钥pubkey2
>
> 4、然后服务端自己再生成 对称密钥key,我们取名为aeskey，也就是我们的AES，其实也就是相对于我们配置中的那个16未长度的加密key，生成了这个key以后，我们就用客户端的公钥pubkey2进行加密，返回给客户端。因为被pubkey2加密的数据，只能被客户端对应的prikey2解密，所以，客户端拿到密文后，用prikey2进行解密操作，解密完，得到对称加密AES的密钥key,最后密钥key进行数据传输加密，至此，整个流程结束

##### 图例

<p align="center" ><img src="/imgs/Cryptography_03.svg" align=center></p>

##### 结合使用原因

> 1、单纯的使用非对称加密方式RSA的话，效率会很低，因为非对称加密解密方式虽然很保险，但是过程复杂，需要时间长。
>
> 2、单纯使用对称加密方式AES的话，太死板，因为这种方式使用的密钥是一个固定的密钥，也就是不能改，一旦客户端或者服务端改的话，就必须通知另一方要改，并且改了后，还需要约定一相同的密钥。使用起来非常不灵活。并且，有一种极端的情况就是，一旦密钥被人获取，那么，我们是无法知道的，我们的所发的每一条数据都会被都对方获取。
>
> but,AES有个很大的优点，那就是加密解密效率很高，而我们传输正文数据时，正号需要这种加解密效率高的；所以这种方式适合用于传输量大的数据内容。
>
> 3、基于以上两种特点，所以取其优，就是结合使用；用RSA方式传输AES的密钥，然后客户端和服务端拿到AES的密钥后，再进行传输正式的内容。这样既利用了RSA的灵活性，可以随时改动AES的密钥；又利用了AES的高效性，可以高效传输数据。

