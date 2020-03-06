---
title: 使用 JWT 进行认证
comments: true
categories:
- Java Web
- Web
tags: Java Web
abbrlink: fd9d3113
date: 2018-08-25 11:30:31
---

# 什么是 JWT ？

JWT 的全称是 JSON Web Token，是一种跨域认证解决方案。

所谓认证，就是获取用户的身份信息。我们知道，Http 是一种无状态协议，为了实现认证和跟踪用户信息，开发者发明了 cookie-session 方案，该方案流程如下：

1. 用户向服务器发送用户名和密码；
2. 服务器验证通过后，在 session 里保存相关数据；
3. 服务器返回一个 session_id，写入客户端的 Cookie；
4. 之后，用户的每次请求都会通过 Cookie 把 session_id 传回给服务器；
5. 服务器通过 session_id，找到先前保存的数据，得到用户信息。

这种方案存在几个问题：

1. 如果是服务器集群，要求 session 数据要共享，要求每一台服务器都能够读取并同步 session；
2. 前后端分离，跨域访问的情况下，每次请求的 session_id 都会不一样；
3. 如果是多端（ios/Android/Web）共用一套后端 API 服务，移动端无法储存 Cookie，需要另辟蹊径。
4. session 数据是保存在服务器的内存中，无形中增加了服务器的压力。

而 JWT 解决了上述问题，它的思想是：服务器不保存 session 数据了，数据全部保存在客户端，每次请求的时候都发回服务器验证。

<!-- more -->

---

# JWT 的原理

用户提供用户名和密码，服务器认证通过以后，生成一个 JSON 对象，发回给用户，如：

```
{
  "姓名": "张三",
  "角色": "管理员",
  "到期时间": "2018年7月1日0点0分"
}
```

之后，用户的每次请求，都要发回这个 JSON 对象给服务器判断身份。

---

# JWT 的组成结构

为了防止数据篡改，我们不可能明文发送像上面那样的 json，而是进行了签名之后，以字符串的形式发送给前端，大概像这样：

```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJudWxsaiJ9.W8hfFfcMVgmlAhTRUl4GHNAq4tq_MWJGB1bv-r9wMCE
```

它是一个很长的字符串，中间用 `.` 分割成三段，这三段分别代表：

1. **Header**：头部，记录了一些元数据，例如采用何种算法，令牌类型等。
2. **Payload**：负载，存储我们的关键信息。
3. **Signature**：签名，前两部分的签名，防止数据篡改。

我们主要关注 Payload，JWT 官方规定了 7 个供选择的字段：

1. iss (issuer)：签发人
2. exp (expiration time)：过期时间
3. sub (subject)：主题
4. aud (audience)：受众
5. nbf (Not Before)：生效时间
6. iat (Issued At)：签发时间
7. jti (JWT ID)：编号

当然，除了这 7 个字段之外，我们还可以添加自定义字段。这些信息以 json 格式存储，并用 Base64URL 算法转成字符串。

原始 json 数据：
```json
{
  "sub": "1234567890",
  "name": "Jerry",
  "myField": "something here"
}
```

Base64URL加密后：
```
eyJzdWIiOiAiMTIzNDU2Nzg5MCIsIm5hbWUiOiAiSmVycnkiLCJteUZpZWxkIjogInNvbWV0aGluZyBoZXJlIn0=
```

注意，Base64URL是可以解密的，因此不要存储密码等敏感信息。

## Base64 和 Base64URL 的区别

JWT 作为一个令牌（token），有些场合可能会放到 URL（比如 api.example.com/?token=xxx）。Base64 有三个字符+、/和=，在 URL 里面有特殊含义，所以要被替换掉：=被省略、+替换成-，/替换成_ 。这就是 Base64URL 算法。

---

# JWT 如何使用

服务器生成 JWT 之后，把加密字符串发回给客户端，客户端可以把它存储在 Cookie 里面，也可以储存在 localStorage。

此后，客户端每次与服务器通信，都要带上这个 JWT。你可以把它放在 Cookie 里面自动发送，但是这样不能跨域，所以更好的做法是放在 HTTP 请求的头信息 Authorization 字段里面。

或者，把 JWT 放在 POST 请求的数据体里面。

参考：
- [JSON Web Token 入门教程](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)
- [Shiro+JWT+Spring Boot Restful简易教程](https://juejin.im/post/59f1b2766fb9a0450e755993)

---

# SpringBoot 实战

自己实现 JWT 并不难，但是秉着不要重复造轮子的原则，我们使用开源框架 [jjwt](https://github.com/jwtk/jjwt) 简化我们的步骤。

## 引入 jjwt 的依赖

pom.xml
```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.10.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.10.5</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.10.5</version>
    <scope>runtime</scope>
</dependency>
```

## 创建 JwtUtil 类

```java
import io.jsonwebtoken.*;
import io.jsonwebtoken.security.Keys;
import org.apache.tomcat.util.codec.binary.Base64;
import javax.crypto.SecretKey;
import java.util.Date;

public class JwtUtil
{

    private static final String KEY1 = "somethinghereshouldbelong";

    /**
     * 由字符串生成加密key
     * @return
     */
    private static SecretKey generalKey(){
        String stringKey = KEY1 + Constant.JWT_SECRET;
        byte[] encodedKey = Base64.decodeBase64(stringKey);
        SecretKey key = Keys.hmacShaKeyFor(encodedKey);
        return key;
    }

    /**
     * 创建jwt
     * @param subject
     * @return
     * @throws Exception
     */
    public static String createJWT(String subject){

        SecretKey key = generalKey();
        Date now = new Date();
        Date thirtyMinutes = new Date(System.currentTimeMillis() + 30*60*1000);

        String jws = Jwts.builder()
                .setSubject(subject) // 主题
                .setIssuedAt(now) // 签发时间
                .setExpiration(thirtyMinutes) // 过期时间
                .signWith(key)
                .compact();

        return  jws;

    }

    /**
     * 解密jwt
     * @param jwt 密钥
     * @return 主题
     * @throws Exception 如果发生 JwtException，说明该密钥无效
     */
    public static String parseJWT (String jwt) throws JwtException {
        SecretKey key = generalKey();

        try {
            return Jwts.parser()
                    .setSigningKey(key)
                    .parseClaimsJws(jwt)
                    .getBody()
                    .getSubject();
        }catch (JwtException ex){
            System.out.println("签证失效");
            return null;
        }
    }

}
```

该类包含三个静态方法，分别是：

1. **generalKey()**：用于生成密钥
2. **createJWT(String subject)**：用于创建一个JWT
3. **parseJWT(String jwt)**：用于解密JWT

### generalKey()

一般我们都是从服务器配置文件读取某个 Key 字符串，转换成 byte[] ，再转成 SecretKey，如下：

```java
byte[] keyBytes = getSigningKeyFromApplicationConfiguration();
SecretKey key = Keys.hmacShaKeyFor(keyBytes);
```

但是如果你嫌麻烦，可以直接用 jjwt 提供的 Key 算法：

```java
SecretKey key = Keys.secretKeyFor(SignatureAlgorithm.HS256); //or HS384 or HS512
```


### createJWT(String subject)

创建 JWT 的过程，详细可看 [jjwt](https://github.com/jwtk/jjwt) github文档。

```java
String jws = Jwts.builder()
        .setSubject(subject) // 设置主题（也就是我们的关键字）
        .setIssuedAt(now) // 设置签发时间
        .setExpiration(thirtyMinutes) // 设置过期时间
        .signWith(key)  // 密钥
        .compact();
```

### parseJWT(String jwt)

解密 JWT 的过程，这里注意，需要抛出异常。因为一旦解密失败(例如失效或者无效)，jjwt会抛出`JwtException`，需要我们在 catch 块里处理。

```java
Jws<Claims> jws;

try {
    jws = Jwts.parser()         // (1)
    .setSigningKey(key)         // (2)
    .parseClaimsJws(jwsString); // (3)

    // we can safely trust the JWT
} catch (JwtException ex) {       // (4)

    // we *cannot* use the JWT as intended by its creator
}
```

## 在 Service 层调用JwtUtil

在 Servce 里面验证用户名和密码无误后，通过以下语句创建一个JWT（token）

```java
String token = JwtUtil.createJWT(user.getUsername());
```

之后把这个 JWT（token） 返回给前端

## 前端

### 保存 token

在 HTML5 中，localStorage 是一个客户端（浏览器）可以存储数据的地方。

前端收到服务器 token 后，通过 localStorage 存储，直接用 javascript代码写入：

```javascript
localStorage.setItem("token", token);
```

### 将 token 写入 header 中

之后发起一次 ajax 请求，把 token 放进 header 的 Authorization 字段里，例如，我这里是获取登录用户信息。

```javascript
axios.get("/api/user/profile",{
    headers: {
        'Authorization': localStorage.getItem("token")
    }
}).then(function (response) {
    // 其他处理
})
```

之后在这台电脑上的每一次请求 header 都会自动携带 Authorization 字段。直到 token 过期才需要重新登录。或者，你可以用 sessionStorage 。

注意：每次发起 ajax 请求，必须在方法参数里手动带上 Authorization

### 退出登录

退出登录非常简单，只需要把 token 从 localStorage 里面删除即可。

```javascript
localStorage.removeItem("token");
location.reload();
```

### 在 Service 层验证 JWT

前端发回 JWT，Service进行校验

```java
// parseJWT 的返回值可以设置为 token 里面的 subject
String subject =  JwtUtil.parseJWT(token);
```

---

# 引申1：JWT 过期问题

JWT 的一个特点就是无状态，给用户签发一个有效期为 30 分钟的 token，如果用户第29分钟还在浏览，下一分钟可能因为 token 失效而被迫重新登录。因此需要考虑刷新 JWT 问题。参考业界主流做法，AWS、Azure 和 Auth0 都是用 JWT 为载体，ID Token + Access Token + Refresh Token 的模式：

- [亚马逊 AWS](https://docs.aws.amazon.com/cognito/latest/developerguide/amazon-cognito-user-pools-using-tokens-with-identity-providers.html)
- [微软 Azure](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-token-and-claims)
- [Auth0](https://auth0.com/docs/tokens)

---

# 引申2：认证和鉴权

JWT 只是实现了 **认证（Authorization）** 功能，事实上，在现代面向服务的应用中，不同的角色有不同的权限（例如管理员和普通用户），如何 **鉴权（Authentication）** 呢？ 这就要交给 [shiro](http://shiro.apache.org/) 或者 [Spring security](https://spring.io/projects/spring-security) 等框架来做了。

说到鉴权和授权，目前很多网站支持微信授权登录、微博授权登录等，这使用到了 [OAuth2.0](https://oauth.net/2/) 协议。OAuth2.0 主要用于有第三方参与的场景。例如，你是微博的开发者，豆瓣想在他的网站上设置一个“使用微博账号登录”的功能，这就需要你向豆瓣提供一些基本权限信息（例如提供用户的用户名、头像）。

关于 OAuth2 的使用场景，可参考：[你可能并不需要 OAuth2](https://mdluo.com/2018-02-11/you-may-not-need-oauth2/)
