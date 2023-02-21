---
title: Apache Commons Codec
date: 2023-2-21
category: Java
tags: apache-commons
excerpt: Apache Commons Codec 提供了通用编码器和解码器的实现,有提供许多编解码相关的工具类。，如Base64、Hex、Phonetic和url。Codec目前最新版本是1.15，最低要求Java7以上。
---
## Apache Commons Codec简介

Apache Commons Codec 提供了通用编码器和解码器的实现,有提供许多编解码相关的工具类。，如Base64、Hex、Phonetic和url。Codec目前最新版本是1.15，最低要求Java7以上。



## maven坐标如下：



```xml
<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>1.15</version>
</dependency>
```

下面只列举其中常用的加以说明，其余感兴趣的可以自行翻阅源码研究。



## 二进制相关



二进制包主要提供16进制、Base64、Base32等的编解码工具类。

### 16进制（Hex类）

十六进制常用于将二进制以更简短的方式展示，比如MD5是128位，展现起来太长，而转换为16进制后只需要32个字符即可表示出来。示例代码如下

```java
// byte数组转为16进制字符串
String hex = Hex.encodeHexString("123".getBytes());
System.out.println(hex);
// 16进制字符串解码
byte[] src = Hex.decodeHex(hex);
System.out.println(new String(src));
```



### Base64，Base32，Base16

Base64是网络上最常见的用于传输二进制数据的编码方式之一，Base64就是一种基于64个可打印字符来表示二进制数据的方法。Base32就是使用32个可打印字符，Base16就是使用16个（实际上相当于16进制）。

| 名称   | 编码表字符串                                      | 位数不足是否会补全 =      |
| ------ | ------------------------------------------------- | ------------------------- |
| base16 | 数字0~9 和 字母 A~F                               | 不会，位数刚好是 4 的倍数 |
| base32 | 大写字母A~Z 和 数字2~7                            | 会                        |
| base64 | Base大写字母A-Z，小写字母a-z，数字0~9以及"+"，"/" | 会                        |

```java
// base64编码
String base64 = Base64.encodeBase64String("测试".getBytes());
System.out.println(base64);
// base64解码
byte[] src = Base64.decodeBase64(base64);
System.out.println(new String(src));
// 字符串是否是base64
Base64.isBase64(base64);

// Base32 Base16 同理
```

Codec还提供了Base系列的流处理，以流的方式去处理Base编解码，示例如下

```java
// 以流方式提供Base64编码和解码
// 附："123"的base64编码为"MTIz"

// 对输入流做base64编码
InputStream is = new ByteArrayInputStream("123".getBytes());
Base64InputStream ebis = new Base64InputStream(is, true);
String enc = IOUtils.toString(ebis, "UTF-8"); // MTIz

// 对base64数据流做解码
is = new ByteArrayInputStream(enc.getBytes());
Base64InputStream dbis = new Base64InputStream(is, false);
String dec = IOUtils.toString(dbis, "UTF-8"); // 123

// -----------------------

// 将数据做base64编码写入输出流
final String data = "123";
ByteArrayOutputStream baos = new ByteArrayOutputStream();
Base64OutputStream ebos = new Base64OutputStream(baos, true);
IOUtils.write(data, ebos, "UTF-8");
String enc2 = baos.toString(); // MTIz

// 将base64数据做解码写入输出流
baos = new ByteArrayOutputStream();
Base64OutputStream dbos = new Base64OutputStream(baos, false);
IOUtils.write(data, dbos, "UTF-8");
String dec2 = dbos.toString(); // 123
```



## URL相关

URL之所以要进行编码，是因为URL中有些字符会引起歧义。

例如URL参数字符串中使用key=value键值对这样的形式来传参，键值对之间以&符号分隔，如/s?q=abc&ie=utf-8。如果你的value字符串中包含了=或者&，那么势必会造成接收URL的服务器解析错误，因此必须将引起歧义的&和=符号进行转义，也就是对其进行编码。

又如URL的编码格式采用的是ASCII码，而不是Unicode，这也就是说你不能在URL中包含任何非ASCII字符，例如中文。否则如果客户端浏览器和服务端浏览器支持的字符集不同的情况下，中文可能会造成问题。

URL编码的原则就是使用安全的字符（没有特殊用途或者特殊意义的可打印字符）去表示那些不安全的字符。

编解码示例代码如下

```java
URLCodec urlCodec = new URLCodec();
// url编码
String encUrl = urlCodec.encode("http://x.com?f=哈");
System.out.println(encUrl);
// url解码
String decUrl = urlCodec.decode(encUrl);
System.out.println(decUrl);
```



## 摘要算法

摘要算法是一种单向的散列算法，它满足以下几个特点。

- 输入长度是任意的
- 输出长度是固定的
- 对每一个给定的输入，计算输出是很容易的
- 不可逆，无法通过输出推算出原数据
- 输出不依赖于输入。就是输入数据变动一个字节结果会相差很多

由于摘要算法以上特点，主要用于数据完整性校验。例如网上的资源一般会提供一个摘要值（一般用MD5算法），用户下载后可以通过工具对资源做MD5后和网上给定的值比较，如果不一致说明文件不完整了，可能是下载过程网络波动内容有丢失，也可能被人篡改过。

也可以做数据的指纹，比如网盘秒传，就是利用摘要值做判断。客户端上传前先对文件做摘要值，传给服务端，服务端发现有相同摘要的文件说明两个文件内容是一致的，这样就无需上传直接将文件存储路径指向这个文件就可以了，既实现了秒传，还节约了服务器磁盘空间（不同用户相同内容的文件实际上指向的是同一份文件）。

很多系统也将密码做md5后存储，其中这种方式并不安全。md5已经很很多公开结果了，并且使用彩虹表碰撞也很容易破解了。所以并不建议使用md5存储密码。密码推荐使用BCrypt算法。

摘要算法主要有以下几个

- MD(Message Digest)：消息摘要
- SHA(Secure Hash Algorithm)：安全散列
- MAC(Message Authentication Code)：消息认证码

### MD系列

主要有MD2、MD4、MD5，目前一般常用MD5

```java
// 如果使用Java自带的api需要十多行才能实现md5算法

// 对数据做md5，参数支持字符串，字节数据，输入流
String md5 = DigestUtils.md5Hex("测试");
```

### SHA系列

SHA系列有SHA-1、SHA-224、SHA-256、SHA-384、SHA-512，SHA3-224、SHA3-256、SHA3-384、SHA3-512等。目前安全起见一般选择256以上，推荐384以上。当然摘要越长则计算耗时也越长，需要根据需求权衡。

```java
// 参数支持字符串，字节数据，输入流
String sha1 = DigestUtils.sha1Hex("测试");
String sha256 = DigestUtils.sha256Hex("测试");
String sha384 = DigestUtils.sha384Hex("测试");
String sha512 = DigestUtils.sha512Hex("测试");
String sha3_256 = DigestUtils.sha3_256Hex("测试");
String sha3_384 = DigestUtils.sha3_384Hex("测试");
String sha3_512 = DigestUtils.sha3_512Hex("测试");
```

### HMAC系列

HMAC(keyed-Hash Message Authentication Code)系列是包含密钥的散列算法，包含了MD和SHA两个系列的消息摘要算法。融合了MD，SHA：

MD系列：HMacMD2，HMacMD4，HMacMD5

SHA系列：HMacSHA1，HMacSHA224，HMacSHA256，HMacSHA38

，HMacSHA512

```java
String key = "asdf3234asdf3234asdf3234asdf3234";
String valueToDigest = "测试数据"; // valueToDigest参数支持字节数据，流，文件等
// 做HMAC-MD5摘要
String hmacMd5 = new HmacUtils(HmacAlgorithms.HMAC_MD5, key).hmacHex(valueToDigest);
// 做HMAC-sha摘要
String hmacSha256 = new HmacUtils(HmacAlgorithms.HMAC_SHA_256, key).hmacHex(valueToDigest);
String hmacSha384 = new HmacUtils(HmacAlgorithms.HMAC_SHA_384, key).hmacHex(valueToDigest);
String hmacSha512 = new HmacUtils(HmacAlgorithms.HMAC_SHA_512, key).hmacHex(valueToDigest);
```



## 命令行

codec包还提供了一个命令行做摘要算法的入口。

```bash
java -cp ./commons-codec-1.15.jar org.apache.commons.codec.cli.Digest MD5 123
```



## 总结

除了以上介绍的工具类外，还有其他不是很常用的就不多做介绍了。感兴趣的可以自行翻阅源码研究。