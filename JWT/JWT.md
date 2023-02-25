# 1.什么是Jwt

Json Web Token（JWT）是一个开放标准(RFC 7519)，它定义了一种紧凑的、自包含的方式，用于在各方之间以JSON对象安全地传输信息。此信息可以验证和信任，因为它是数字签名的。jwts可以使用秘密(使用HMAC算法)或使用RSA或ECDSA的公钥/私钥对进行签名

通俗解释
JWT简称JSON Web Token，也就是通过JSON形式作为Web应用中的令牌。用于在各方之间安全地将信息作为JSON对象传输。在数据传输过程中还可以完成数据加密、签名等相关处理



# 2.Jwt能做什么

**授权**
这是使用JWT的最常见方案。一旦用户登录，每个后续请求将包括JWT，从而允许用户访问该令牌允计许的路由，服务和资源。单点登录是当今广泛使用JWT的一项功能，因为它的开销很小并且可以在不同的域中轻松使用

**信息交换**
JSON Web Token是在各方之间安全地传输信息的好方法。因为可以对JWT进行签名(例如，使用公钥/私钥对)，所以您可以确保发件人是他们所说的人。此外，由于签名是使用标头和有效负载计算的，因此还可以验证内容是否遭到篡改



# 3.为什么要使用Jwt

## 3.1基于传统的Session认证

**认证方式**
我们知道,http协议本身是一种无状态的协议,而这就意味着如果用户向我们的应用提供了用户名和密码来进行用户认证,那么下一次请求时,用户还要再一次进行用户认证才行,因为根据http协议，我们并不能知道是哪个用户发出的请求,所以为了让我们的应用能识别是哪个用户发出的请求,我们只能在服务器存储一份用户登录的信息,这份登录信息会在响应时传递给浏览器,告诉其保存到cookie,以便下次请求时发送给我们的应用,这样我们的应用就能识别请求来自哪个用户了,这就是传统的基于session认证

**认证流程**

![image-20230215151743664](D:\Java学习\java笔记\JWT\assets\image-20230215151743664.png)



**暴露的问题**

1. 每个用户经过我们的应用认证之后,我们的应用都要在服务端做一次记录,以方便用户下次请求进行鉴别,通常而言session都是保存在内存中,而随着认证用户的增多,服务端的开销会明显增大

2. 用户认证之后,服务端做认证记录,如果认证的记录被保存在内存中的话,这意味着用户下次请求还必须要请求在这台服务器上,这样才能拿到授权的资源。这样在分布式的应用上,相应的限制了负载均衡器的能力。这也意味着限制了应用的扩展能力

3. 由于是基于cookie来进行用户识别的,如果cookie被截获,用户就会很容易受到跨站请求伪造的攻击(CSRF)

4. 在前后端分离系统中就更加痛苦: 如下图所示

    ![image-20230215151918925](D:\Java学习\java笔记\JWT\assets\image-20230215151918925.png)

    - 也就是说前后端分离在应用解耦后增加了部署的复杂性,通常用户一次请求就要转发多次。如果使用session 每次携带sessionId到服务器,服务器都要查询用户信息
    - 如果用户很多,这些信息存储在服务器内存中,给服务器增加负担
    - CSRF(跨站伪造请求攻击)攻击session是基于cookie进行用户识别的,cookie如果被截获,用户就会很容易受到跨站请求伪造的攻击
    - 由于sessionId就是一个特征值,表达的信息不够丰富。不容易扩展,而且如果你后端应用是多节点部署。那么就需要实现session共享机制。不方便集群应用



## 3.2基于JWT认证

![image-20230215152504599](D:\Java学习\java笔记\JWT\assets\image-20230215152504599.png)

- **认证流程**
  
    1. 首先,前端通过Web表单将自己的用户名和密码发送到后端的接口。这一过程一般是一个HTTP的POST请求。建议的方式是通过SSL加密的传输(https协议),从而避免敏感信息被嗅探
    2. 后端核对用户名和密码成功后,将用户的id等其他信息作为JWT Payload(负载),将其与头部分别进行Base64编码拼接后签名,形成一个JWT。形成的JWT就是一个形同111.zzz.xxx的字符串	token => head.payload.signature
    3. 后端将JWT字符串作为登录成功的结果返回给前端。前端可以将返回的结果保存在localStorage或sessionStorage上,退出登录时前端删除保存的JWT即可
    4. 前端在每次请求时将JWT放入HTTP的Header中的Authorization位(解决XSS和XSRF问题)
    5. 后端检查是否存在,如存在验证JWT的有效性。例如,检查签名是否正确; 检查Token是否过期; 检查Token的接收方是否是自己(可选)
    6. 验证通过后,后端使用JWT中包含的用户信息进行其他逻辑操作,返回相应结果
    
- **jwt优势**
  
    1. 简洁(Compact): 可以通过URL,POST参数或者在HTTP的Header发送,因为数据量小,传输速度也很快
    2. 自包含(Self-contained): 负载中包含了所有用户所需要的信息,避免了多次查询数据库
    3. 因为Token是以JSON加密的形式保存在客户端的,所以JWT是跨语言的,原则上任何web形式都支持
    4. 不需要在服务端保存会话信息,特别适用于分布式微服务
    
- **JWT的结构**

    1.令牌组成由- 标头(Header) - 有效载荷(Payload) - 签名(Signature) *# 所以通常情况下JWT如下所示: Header.Payload.Signature*

    2.Header
    标头通常由两部分组成: 令牌的类型(即JWT)和所使用的签名算法,例如HMAC、SHA256或RSA。它会使用 Base64 编码组成 JWT 结构的第一部分
    注意: Base64是一种编码,也就是说,它可以是被翻译回原来的样子,并不是一种加密过程

    3.Payload
    令牌的第二部分是有效负载,其中包含声明。声明是有关实体(通常是普通用户)和其它数据的声明。同样的,它也会使用 Base64 编码组成 JWT 结构的第二部分

    4.Signature

    - 前面两部分都是使用 Base64 进行编码,即前端可以解开知道里面的信息。Signature 需要使用编码后的 Header 和 Payload 以及我们提供的一个密钥,然后使用 Header 中指定的签名算法(HS256)进行签名。签名的作用是保证 JWT 没有被篡改过
    例子如下
    HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload),secret);
    通过token中的header、payload编码base64之后,再通过secret密钥进行加密,比较当前的token中的signature是否与加密之后算出来的一致

    **签名目的**

    - 最后一步签名的过程,实际上是对头部以及负载内容进行签名,防止内容被篡改。如果有人对头部以及负载的内容解码之后进行篡改,在进行编码,最后加上之前的签名组合形成新的 JWT 的话,那么服务端会判断出新的头部和负载形成的签名和 JWT 附带上的签名是不一样的。如果要对新的头部和负载进行签名,在不知道服务器加密时用的密钥的话,得出来的签名也是不一样的

    信息安全问题
    - 问题: Base64编码是可逆的,那么信息不就会暴露了?
    - 思想: 是的,再 JWT 中,不应该再负载里面加入任何敏感的数据。在上面的例子中,我们传输的时用户的User ID。这个值实际上不是什么敏感内容,就算被知道了也是安全的。但是像密码这样的内容就不能被放在 JWT 中了。如果将用户密码放在了 JWT 中,那么怀有恶意的第三方通过Base64解码就很快地知道了你的密码。因此 JWT 适合用于向 Web 应用传递一些非敏感信息。JWT 还经常用于设计用户和授权的系统,甚至实现 Web 应用的单点登录

    **5.整合在一起**

    - 输出的是,三个由"."分隔的Base64-URL字符串,可以在HTML和HTTP环境中轻松传递这些字符串,与基于XML的标准(例如SAML)相比,它更紧凑
    特点
    - 简介(Compact)
    	可以通过URL、POST参数或者在HTTP中的Header发送,因为数据量小,传输速度快
    - 自包含(self-contained)
    	负载中包含了所有用户所需要的信息,bi'ma

# 4.JWT的使用

## 4.1引入依赖

```xml
<!-- JWT相关 -->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>
```

## 4.2封装工具类

```java
package com.husky.common;

import com.husky.entity.User;
import io.jsonwebtoken.*;

import java.util.Date;

/**
 * Created by IntelliJ IDEA.
 * User: 周圣杰
 * Date: 2023/2/15
 * Time: 10:15
 */
public class JwtUtils {

    // 构建token的主题 自定义
    private static final String SUBJECT = "Husky";

    // 设置过期时间
    private static final long EXPIRE = 1000 * 60 * 60 * 24;

    // Jwt签名 自定义
    private static final String SIGNATURE = "Husky-jie";

    // 生成token
    public static String getToken(User user){
        if (user == null || user.getUserId() == null || user.getUserUsername() == null || user.getUserAvatar() == null){
            return null;
        }
        /*
        * builder()：构建Jwt
        * setSubject(SUBJECT)：设置主题，自定义*/
        return Jwts.builder().setSubject(SUBJECT)
                /* payload 通常⽤来存放⽤⼾信息
                 * 下面3行设置token中间字段，携带用户的信息
                 * */
                .claim("userId", user.getUserId())
                .claim("userName", user.getUserUsername())
                .claim("userAvatar", user.getUserAvatar())
                .setIssuedAt(new Date())
                // 设置过期时间
                .setExpiration(new Date(System.currentTimeMillis()+EXPIRE))
                //signature 签名值
                .signWith(SignatureAlgorithm.HS256, SIGNATURE)
                /*
                * compact()用来拼接以下三个值：
                * header可以不写，有默认值
                * payload
                * signature*/
                .compact();
    }

    // 校验token
    public static Claims checkJwt(String token){
        try {
            /*Jwts.parser()解密
            * setSigningKey(签名) 通过签名解密
            * parseClaimsJwt(token) 需要解密的token
            * getBody()获取Claims对象，该对象封装了用户的信息*/
            return Jwts.parser().setSigningKey(SIGNATURE).parseClaimsJws(token).getBody();
        } catch (Exception e) {
            // 若篡改token会导致校验失败，走到异常分支，这里返回null
            return null;
        }
    }
}

```



