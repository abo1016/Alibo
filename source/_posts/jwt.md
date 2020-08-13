---
title: JWT
tags: []
id: '338'
categories:
  - - all-blog
    - 网络
date: 2018-11-15 14:19:34
---

#### JWT：Json Web Token

跨域身份认证有多种方式，JWT将用户信息加密到token里，服务器不保存任何用户信息。服务器通过使用保存的密钥验证token的正确性，只要正确即通过验证。 在互联网服务器的交互中离不开验证，此次对接Google assistant 智能家居Api时，使用的身份验证方式，我选择的就是JWT。 **首先**，我们来看最普遍的web服务器用户验证流程。

*   用户发送用户名和密码。
*   服务器验证用户名和密码，服务器通过验证后在当前对话（session）里面保存相关数据。
*   服务器向用户返回一个session\_id，将session\_id写入用户cookie里
*   用户后面的每一个请求中都会将包含这个session\_id
*   服务器通过session\_id，查找对应的用户数据

这种模式在web服务中普遍存在，基本是默认的用户身份验证方式，用起来也很方便。但是它的问题在于扩张性非常差，在服务器集群或者是跨域的服务导向架构，就涉及到了session共享的问题，每台服务器都需要读取到session信息。 要解决session共享的方案有： 1. session 数据持久化，写入数据库或别的持久层。各种服务收到请求后，都向持久层请求数据。这种方案的优点是架构清晰，缺点是工程量比较大。另外，持久层万一挂了，就会单点失败。 2. 服务器索性不保存 session 数据了，所有数据都保存在客户端，每次请求都发回服务器。JWT 就是这种方案的一个代表。 **JWT**基本原理 服务器认证以后，生成一个 JSON 对象，发回给用户，就像下面这样

```json
{
  "姓名": "张三",
  "角色": "管理员",
  "到期时间": "2018年7月1日0点0分"
}
```

以后，用户与服务端通信的时候，都要发回这个 JSON 对象。服务器完全只靠这个对象认定用户身份。为了防止用户篡改数据，服务器在生成这个对象的时候，会加上**签名** JWT通常由三个部分组成

*   Header（头部）
*   Payload（负载）
*   Signature（签名）

通过签名加密生后的样子如下 **Header.Payload.Signature**

```json
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

> header

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

alg属性表示签名的算法（algorithm），默认是 HMAC SHA256（写成 HS256）；typ属性表示这个令牌（token）的类型（type），JWT 令牌统一写为JWT

> Payload

Payload 部分也是一个 JSON 对象，用来存放实际需要传递的数据。JWT 规定了7个官方字段，供选用。

*   iss (issuer)：签发人
*   exp (expiration time)：过期时间
*   sub (subject)：主题
*   aud (audience)：受众
*   nbf (Not Before)：生效时间
*   iat (Issued At)：签发时间
*   jti (JWT ID)：编号

例如：

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022
}
```

**并且除了官方定义的字段，还可以自定义字段名** 注意，JWT 默认是不加密的，任何人都可以读到，所以不要把秘密信息放在这个部分。 这个 JSON 对象也要使用 Base64URL 算法转成字符串

> Signature

Signature 部分是对前两部分的签名，防止数据篡改。首先，需要指定一个密钥（secret）。这个密钥只有服务器才知道，不能泄露给用户。然后，使用 Header 里面指定的签名算法（默认是 HMAC SHA256），按照下面的公式产生签名。算出签名以后，把 Header、Payload、Signature 三个部分拼成一个字符串，每个部分之间用"点"（.）分隔，就可以返回给用户。

> 特点

1.  JWT 默认是不加密，但也是可以加密的。生成原始 Token 以后，可以用密钥再加密一次。
2.  JWT 不加密的情况下，不能将秘密数据写入 JWT。
3.  JWT 不仅可以用于认证，也可以用于交换信息。有效使用 JWT，可以降低服务器查询数据库的次数。
4.  JWT 的最大缺点是，由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑。
5.  JWT 本身包含了认证信息，一旦泄露，任何人都可以获得该令牌的所有权限。为了减少盗用，JWT 的有效期应该设置得比较短。对于一些比较重要的权限，使用时应该再次对用户进行认证。
6.  为了减少盗用，JWT 不应该使用 HTTP 协议明码传输，要使用 HTTPS 协议传输。

> 代码示例

```php
<?php
/**
 * Created by PhpStorm.
 * User: dengbo
 * Date: 2018/10/29
 * Time: 16:44
 */
namespace Comm;
require_once __DIR__.'/../vendor/autoload.php';
use \Firebase\JWT\JWT;
use Log\LogCat;
use RedisManager\RedisManager;

class ReportStates{

    private $secretRaw = __DIR__ . '/../Config/jwt.json';//签名密钥
    private $googleToken;
    private $jwt;
    private $secret;
    private $Redis;
    //获取jwt
    private function getJwt()
    {
        $defaultServiceAccount = $this->secret['client_email'];
        $privateKey = $this->secret['private_key'];
        $scope = 'https://www.googleapis.com/auth/homegraph';
        $token = array(
            "iss" => $defaultServiceAccount,
            "scope" => $scope,
            "aud" => $this->secret['token_uri'],
            "iat" => time(),
            "exp" => time() + 3600
        );
        //获取加密后的token，转为字符串
        return JWT::encode($token, $privateKey, 'RS256');
    }

    //获取Google jwt token
    private function getToken()
    {
        $options = array(
            'http' => array(
                'method' => 'POST',
                'header' => 'Content-type:application/x-www-form-urlencoded',
                'content' => 'grant_type=urn%3Aietf%3Aparams%3Aoauth%3Agrant-type%3Ajwt-bearer&assertion='.$this->jwt,
                'timeout' => 60 // 超时时间（单位:s）
            )
        );
        $context = stream_context_create($options);
        $result = file_get_contents($this->secret['token_uri'], false, $context);
        $token =  json_decode($result, JSON_UNESCAPED_UNICODE)['access_token'];
        if (!empty($token)){
            $this->Redis->setex("google_token:{$this->agentUserId}", 3599, $token);
            return $token;
        }else{
            return false;
        }
    }
}
```