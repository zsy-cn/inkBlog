title: JWT
date: 2019-05-13 00:10:10 
author: Xavier
tags: 
    - JWT
type: article
---

# JWT（Json Web Token）验证

## Cookie

Cookie 总是保存在客户端中，按在客户端中的存储位置，可分为 内存Cookie 和 硬盘Cookie。

内存 Cookie 由浏览器维护，保存在内存中，浏览器关闭后就消失了，其存在时间是短暂的。

硬盘 Cookie 保存在硬盘⾥，有⼀个过期时间，除⾮⽤户⼿⼯清理或到了过期时间，硬盘 Cookie 不会被删除，其存在时间 是⻓期的。

所以，按存在时间，可分为 ⾮持久Cookie 和持久Cookie。

那么 cookies 到底是什么呢？

cookie 是⼀个⾮常具体的东⻄，指的就是浏览器⾥⾯能永久存储的⼀种数据，仅仅是浏览器实现的⼀种数据存储功能。

cookie 由服务器⽣成，发送给浏览器 ，浏览器把 cookie 以 key-value 形式保存到某个⽬录下的⽂本⽂件内，下⼀次请求同⼀⽹站时会把该 cookie 发送给服务器。由于 cookie 是存在客户端上的，所以浏览器加⼊了⼀些限制确保 cookie 不会被恶意使⽤，同时不会占据太多磁盘空间，所以每个域的 cookie 数量是有限的。

## Session

Session 字⾯意思是会话，主要⽤来标识⾃⼰的身份。

⽐如在⽆状态的 api 服务在多次请求数据库时，如何知道是同⼀个⽤户，这个就可以通过 session 的机制，服务器要知道当前发请求给⾃⼰的是谁，为了区分客户端请求， 服务端会给具体的客户端⽣成身份标识 session ，然后客户端每次向服务器发请求的时候，都带上这个 “身份标识”，服务器就知道这个请求来⾃于谁了。

⾄于客户端如何保存该标识，可以有很多⽅式，对于浏览器⽽⾔，⼀般都是使⽤ cookie 的⽅式 ，服务器使⽤ session 把⽤户信息临时保存了服务器上，⽤户离开⽹站就会销毁，这种凭证存储⽅式相对于 ，cookie 来说更加安全。

但是 session 会有⼀个缺陷：如果 web 服务器做了负载均衡，那么下⼀个操作请求到了另⼀台服务器的时候 session 会丢失。

因此，通常企业⾥会使⽤ redis,memcached 缓存中间件来实现 session 的共享，此时 web 服务器就是⼀个完全⽆状态的存在，所有的⽤户凭证可以通过共享 session 的⽅式存取，当前 session 的过期和销毁机制需要⽤户做控制。

## Token

token 的意思是 “令牌”，是⽤户身份的验证⽅式，最简单的 token 组成: uid (⽤户唯⼀标识) + time (当前 时间戳) + sign (签名，由 token 的前⼏位 + 盐以哈希算法压缩成⼀定⻓度的⼗六进制字符串) ，同时还可以将不变的参数也放进 token

这里说的 token 只的是 JWT（Json Web Token）

## JWT

⼀般⽽⾔，⽤户注册登陆后会⽣成⼀个 jwt token 返回给浏览器，浏览器向服务端请求数据时携带 token ，服务器端使⽤ signature 中定义的⽅式进⾏解码，进⽽对 token 进⾏解析和验证。

### jwt token 的组成部分

```shell
head payload signature
XXXX.YYYYYYY.ZZZZZZZZZ
```

- header: ⽤来指定使⽤的算法 (HMAC SHA256 RSA) 和 token 类型 (如 JWT)
官网上可以找到各种语言的 jwt 库，例如我们下面使用这个库进行编码，因为这个库使用的人是最多的，值得信赖

```shell
go get github.com/dgrijalva/jwt-go
```

- payload: 包含声明 (要求)，声明通常是⽤户信息或其他数据的声明，⽐如⽤户 id，名称，邮箱等。声明。可分为三种: registered,public,private

- signature: ⽤来保证 JWT 的真实性，可以使⽤不同的算法

### header

token 的第一部分，如

```json
{
    "alg": "HS256",
    "typ": "JWT"
}
```

对上⾯的 json 进⾏ base64 编码即可得到 JWT 的第⼀个部分

### payload

token 第二部分，如

- registered claims: 预定义的声明，通常会放置⼀些预定义字段，⽐如过期时间，主题等 (iss:issuer,exp:expiration time,sub:subject,aud:audience)
- public claims: 可以设置公开定义的字段
- private claims: ⽤于统⼀使⽤他们的各⽅之间的共享信息

不要在 header 和 payload 中放置敏感信息，除⾮信息本身已经做过脱敏处理，因为 payload 部分的具体数据是可以通过 token 来获取到的

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

### Signature

token 的第三部分

为了得到签名部分，必须有编码过的 header 和 payload，以及⼀个秘钥，签名算法使⽤ header 中指定的那 个，然后对其进⾏签名即可

```go
HMACSHA256(base64UrlEncode(header)+”.”+base64UrlEncode(payload),secret)
```

签名是 ⽤于验证消息在传递过程中有没有被更改 ，并且，对于使⽤私钥签名的 token，它还可以验证 JWT

的发送⽅是否为它所称的发送⽅。

签名的⽬的

最后⼀步签名的过程，实际上是对头部以及载荷内容进⾏签名。

⼀般⽽⾔，加密算法对于不同的输⼊ 产⽣的输出总是不⼀样的。对于两个不同的输⼊，产⽣同样的输出的概率极其地⼩。所以，我们就把 “不⼀样的输⼊产⽣不⼀样的输出” 当做必然事件来看待。

所以，如果有⼈对头部以及载荷的内容解码之后进⾏修改，再进⾏编码的话，那么新的头部和载荷的 签名和之前的签名就将是不⼀样的。⽽且，如果不知道服务器加密的时候⽤的密钥的话，得出来的签名也 ⼀定会是不⼀样的。

服务器应⽤在接受到 JWT 后，会⾸先对头部和载荷的内容⽤同⼀算法再次签名。那么服务器应⽤是怎 么知道我们⽤的是哪⼀种算法呢？

在 JWT 的头部中已经⽤ alg 字段指明了我们的加密算法了。

如果服务器应⽤对头部和载荷再次以同样⽅法签名之后发现，⾃⼰计算出来的签名和接受到的签名不 ⼀样，那么就说明这个 Token 的内容被别⼈动过的，我们应该拒绝这个 Token，

注意：在 JWT 中，不应该在载荷⾥⾯加⼊任何敏感的数据，⽐如⽤户的密码。具体原因上文已经给过答案了

jwt.io ⽹站

在 jwt.io (https://jwt.io/#debugger-io) ⽹站中，提供了⼀些 JWT token 的编码，验证以及⽣成 jwt 的⼯具。

下面就是⼀个典型的 jwt-token 的组成部分。

Encoded HS256

`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.keH6T3x1z7mmhKL1T3r9sQdAxxdzB6siemGMr_6ZOwU`

Decodeed

```json
HEADER
{
  "alg": "HS256",
  "typ": "JWT"
}
PAYLOAD
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022
}
VERIFY SIGNATURE
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  123456
)
```

## 什么时候使用 JWT

我们要明白的时候，JWT 是用作认证的，而不是用来做授权的。明白他的功能，那么对应 JWT 的应用场景就不言而喻了

- Authorization (授权): 典型场景，⽤户请求的 token 中包含了该令牌允许的路由，服务和资源。单点登录其实就是现在⼴泛使⽤ JWT 的⼀个特性
- Information Exchange (信息交换): 对于安全的在各⽅之间传输信息⽽⾔，JSON Web Tokens ⽆疑是⼀种很好的⽅式。因为 JWTs 可以被签名

例如，⽤公钥 / 私钥对，你可以确定发送⼈就是它们所说的那个⼈。另外，由于签名是使⽤头和有效负载计算的，您还可以验证内容没有被篡改

JWT 认证过程基本上整个过程分为两个阶段

- 第⼀个阶段，客户端向服务端获取 token
- 第⼆阶段，客户端带着该 token 去请求相关的资源

通常⽐较重要的是，服务端如何根据指定的规则进⾏ token 的⽣成。

在认证的时候，当⽤户⽤他们的凭证成功登录以后，⼀个 JSON Web Token 将会被返回。 此后，token 就是⽤户凭证了，你必须⾮常⼩⼼以防⽌出现安全问题。 ⼀般⽽⾔，你保存令牌的时间不应该超过你所需要它的时间。

⽆论何时⽤户想要访问受保护的路由或者资源的时候，⽤户代理（通常是浏览器）都应该带上 JWT，典型 的，通常放在 Authorization header 中，⽤ Bearer schema: Authorization: Bearer 服务器上的受保护的路由将会检查 Authorization header 中的 JWT 是否有效，如果有效，则⽤户可以访问 受保护的资源。

如果 JWT 包含⾜够多的必需的数据，那么就可以减少对某些操作的数据库查询的需要，尽管可能并不总是如此。 如果 token 是在授权头（Authorization header）中发送的，那么跨源资源共享 (CORS) 将不会成为问题，因为它不使⽤ cookie。

获取 JWT 以及访问 APIs 以及资源

- 客户端向授权接⼝请求授权
- 服务端授权后返回⼀个 access token 给客户端
- 客户端使⽤ access token 访问受保护的资源

## 基于 Token 的身份认证和基于服务器的身份认证

给予服务器的身份认证，通常是基于服务器上的 session 来做用户认证，使用 session 会有如下几个问题

- Sessions：认证通过后需要将⽤户的 session 数据保存在内存中，随着认证⽤户的增加，内存开销会⼤
- 扩展性问题： 由于 session 存储在内存中，扩展性会受限，虽然后期可以使⽤ redis,memcached 来缓存数据
- CORS: 当多个终端访问同⼀份数据时，可能会遇到禁⽌请求的问题
- CSRF: ⽤户容易受到 CSRF 攻击（Cross Site Request Forgery, 跨站域请求伪造）

基于 Token 的身份认证证是⽆状态的，服务器或者 session 中不会存储任何⽤户信息.(很好的解决了共享 session 的问题)

- ⽤户携带⽤户名和密码请求获取 token (接⼝数据中可使⽤ appId,appKey，或是自己协商好的某类数据)
- 服务端校验⽤户凭证，并返回⽤户或客户端⼀个 Token
- 客户端存储 token, 并在请求头中携带 Token
- 服务端校验 token 并返回相应数据

需要注意几点

- 客户端请求服务器的时候，必须将 token 放到 header 中
- 客户端请求服务器每一次都需要带上 token
- 服务器需要设置 为接收所有域的请求: Access-Control-Allow-Origin: *

Session 和 JWT Token 的有啥不一样的

- 他俩都可以存储用户相关的信息
- session 存储在服务器， JWT 存储在客户端

⽤ Token 有什么好处呢

- 他是无状态的 且 可扩展性好
- 他相对安全：防⽌ CSRF 攻击，token 过期重新认证

上文有说说，JWT 是用于做身份认证的而不是做授权的，那么在这里列举一下 做认证和做授权分别用在哪里呢？

- 例如 OAuth2 是⼀种授权框架，是用于授权，主要用在 使⽤第三⽅账号登录的情况 (⽐如使⽤ weibo, qq, github 登录某个 app)
- JWT 是⼀种认证协议 ，⽤在 前后端分离，需要简单的对后台 API 进⾏保护时使⽤
- 无论是授权还是认证，都需要记住使用 HTTPS 来保护数据的安全性

实际看看 JWT 如何做身份验证

- jwt 做身份验证，这里主要讲如何根据 header，payload，signature 生成 token
- 客户端带着 token 来服务器做请求，如何校验？

下面实例代码，主要做了 2 个接口

用到的技术点：

    gin
        路由分组
        中间件的使用
    gorm
        简单操作 mysql 数据库，插入，查询
    jwt
        生成 token
        解析 token

登录接口

访问 url ： 127.0.0.1:9999/v1/login

功能：

    用户登录
    生成 jwt，并返回给到客户端
    gorm 对数据库的操作

认证后 Hello 接口#

访问 url ： 127.0.0.1:9999/v1/auth/hello

功能：

    校验 客户端请求服务器携带 token
    返回客户端所请求的数据

代码结构如下

```go
// main.go
package main

import (
   "github.com/gin-gonic/gin"
)

func main() {
   //连接数据库
   conErr := controller.InitMySQLCon()
   if conErr != nil {
      panic(conErr)
   }

   //需要使用到gorm，因此需要先做一个初始化
   controller.InitModel()
   defer controller.DB.Close()


   route := gin.Default()

   //路由分组
   v1 := route.Group("/v1/")
   {
      //登录（为了方便，将注册和登录功能写在了一起）
      v1.POST("/login", controller.Login)
   }


   v2 := route.Group("/v1/auth/")
   //一个身份验证的中间件
   v2.Use(myauth.JWTAuth())
   {
      //带着token请求服务器
      v2.POST("/hello", controller.Hello)
   }

   //监听9999端口
   route.Run(":9999")

}
```

```go
// controller.go
package main

import (
   "errors"
   "fmt"
   jwtgo "github.com/dgrijalva/jwt-go"
   "github.com/gin-gonic/gin"
   "github.com/jinzhu/gorm"
   "log"
   "net/http"
   "time"
   _ "github.com/jinzhu/gorm/dialects/mysql"
)

//登录请求信息
type ReqInfo struct {
   Name   string `json:"name"`
   Passwd string `json:"passwd"`
}

// 构造用户表
type MyInfo struct {
   Id        int32  `gorm:"AUTO_INCREMENT"`
   Name      string `json:"name"`
   Passwd    string `json:"passwd"`
   CreatedAt *time.Time
   UpdateTAt *time.Time
}

//Myclaims
// 定义载荷
type Myclaims struct {
   Name string `json:"userName"`
   // StandardClaims结构体实现了Claims接口(Valid()函数)
   jwtgo.StandardClaims
}
//密钥
type JWT struct {
   SigningKey []byte
}
//hello 接口
func Hello(c *gin.Context) {
   claims, _ := c.MustGet("claims").(*Myclaims)
   if claims != nil {
      c.JSON(http.StatusOK, gin.H{
         "status": 0,
         "msg":    "Hello wrold",
         "data":   claims,
      })
   }
}

var (
   DB               *gorm.DB
   secret                 = "iamsecret"
   TokenExpired     error = errors.New("Token is expired")
   TokenNotValidYet error = errors.New("Token not active yet")
   TokenMalformed   error = errors.New("That's not even a token")
   TokenInvalid     error = errors.New("Couldn't handle this token:")
)
//数据库连接
func InitMySQLCon() (err error) {
   // 可以在api包里设置成init函数
   connStr := fmt.Sprintf("%s:%s@tcp(%s:%d)/%s?charset=utf8mb4&parseTime=True&loc=Local", "root", "123456", "127.0.0.1", 3306, "mygorm")
   fmt.Println(connStr)
   DB, err = gorm.Open("mysql", connStr)

   if err != nil {
      return err
   }

   return DB.DB().Ping()
}
//初始化gorm对象映射
func InitModel() {
   DB.AutoMigrate(&MyInfo{})
}
func NewJWT() *JWT {
   return &JWT{
      []byte(secret),
   }
}
// 登陆结果
type LoginResult struct {
   Token string `json:"token"`
   Name string `json:"name"`
}
// 创建Token(基于用户的基本信息claims)
// 使用HS256算法进行token生成
// 使用用户基本信息claims以及签名key(signkey)生成token
func (j *JWT) CreateToken(claims Myclaims) (string, error) {
   // 返回一个token的结构体指针
   token := jwtgo.NewWithClaims(jwtgo.SigningMethodHS256, claims)
   return token.SignedString(j.SigningKey)
}
//生成token
func generateToken(c *gin.Context, info ReqInfo) {
   // 构造SignKey: 签名和解签名需要使用一个值
   j := NewJWT()

   // 构造用户claims信息(负荷)
   claims := Myclaims{
      info.Name,
      jwtgo.StandardClaims{
         NotBefore: int64(time.Now().Unix() - 1000), // 签名生效时间
         ExpiresAt: int64(time.Now().Unix() + 3600), // 签名过期时间
         Issuer:    "pangsir",                       // 签名颁发者
      },
   }

   // 根据claims生成token对象
   token, err := j.CreateToken(claims)
   if err != nil {
      c.JSON(http.StatusOK, gin.H{
         "status": -1,
         "msg":    err.Error(),
         "data":   nil,
      })
   }

   log.Println(token)
   // 返回用户相关数据
   data := LoginResult{
      Name:  info.Name,
      Token: token,
   }

   c.JSON(http.StatusOK, gin.H{
      "status": 0,
      "msg":    "登陆成功",
      "data":   data,
   })

   return
}
//解析token
func (j *JWT) ParserToken(tokenstr string) (*Myclaims, error) {
   // 输入token
   // 输出自定义函数来解析token字符串为jwt的Token结构体指针
   // Keyfunc是匿名函数类型: type Keyfunc func(*Token) (interface{}, error)
   // func ParseWithClaims(tokenString string, claims Claims, keyFunc Keyfunc) (*Token, error) {}
   token, err := jwtgo.ParseWithClaims(tokenstr, &Myclaims{}, func(token *jwtgo.Token) (interface{}, error) {
      return j.SigningKey, nil
   })

   fmt.Println(token, err)
   if err != nil {
      // jwt.ValidationError 是一个无效token的错误结构
      if ve, ok := err.(*jwtgo.ValidationError); ok {
         // ValidationErrorMalformed是一个uint常量，表示token不可用
         if ve.Errors&jwtgo.ValidationErrorMalformed != 0 {
            return nil, TokenMalformed
            // ValidationErrorExpired表示Token过期
         } else if ve.Errors&jwtgo.ValidationErrorExpired != 0 {
            return nil, TokenExpired
            // ValidationErrorNotValidYet表示无效token
         } else if ve.Errors&jwtgo.ValidationErrorNotValidYet != 0 {
            return nil, TokenNotValidYet
         } else {
            return nil, TokenInvalid
         }

      }
   }

   // 将token中的claims信息解析出来和用户原始数据进行校验
   // 做以下类型断言，将token.Claims转换成具体用户自定义的Claims结构体
   if claims, ok := token.Claims.(*Myclaims); ok && token.Valid {
      return claims, nil
   }

   return nil, errors.New("token NotValid")
}
//登录
func Login(c *gin.Context) {
   var reqinfo ReqInfo
   var userInfo MyInfo

   err := c.BindJSON(&reqinfo)

   if err == nil {
      fmt.Println(reqinfo)

      if reqinfo.Name == "" || reqinfo.Passwd == ""{
         c.JSON(http.StatusOK, gin.H{
            "status": -1,
            "msg":    "账号密码不能为空",
            "data":   nil,
         })
         c.Abort()
         return
      }
      //校验数据库中是否有该用户
      err := DB.Where("name = ?", reqinfo.Name).Find(&userInfo)
      if err != nil {
         fmt.Println("数据库中没有该用户 ，可以进行添加用户数据")
         //添加用户到数据库中
         info := MyInfo{
            Name:   reqinfo.Name,
            Passwd: reqinfo.Passwd,
         }
         dberr := DB.Model(&MyInfo{}).Create(&info).Error
         if dberr != nil {
            c.JSON(http.StatusOK, gin.H{
               "status": -1,
               "msg":    "登录失败，数据库操作错误",
               "data":   nil,
            })
            c.Abort()
            return
         }
      }else{
         if userInfo.Name != reqinfo.Name || userInfo.Passwd != reqinfo.Passwd{
            c.JSON(http.StatusOK, gin.H{
               "status": -1,
               "msg":    "账号密码错误",
               "data":   nil,
            })
            c.Abort()
            return
         }
      }
      //创建token
      generateToken(c, reqinfo)
   } else {
      c.JSON(http.StatusOK, gin.H{
         "status": -1,
         "msg":    "登录失败，数据请求错误",
         "data":   nil,
      })
   }
}
```

```go
// myauth.go
package main

import (
   "fmt"
   "github.com/gin-gonic/gin"
   "controller"
   "net/http"
)

//身份认证
func JWTAuth() gin.HandlerFunc {
   return func(c *gin.Context) {
      //拿到token
      token := c.Request.Header.Get("token")
      if token == "" {
         c.JSON(http.StatusOK, gin.H{
            "status": -1,
            "msg":    "token为空，请携带token",
            "data":   nil,
         })
         c.Abort()
         return
      }

      fmt.Println("token = ", token)

      //解析出实际的载荷
      j := controller.NewJWT()

      claims, err := j.ParserToken(token)
      if err != nil {
         // token过期
         if err == controller.TokenExpired {
            c.JSON(http.StatusOK, gin.H{
               "status": -1,
               "msg":    "token授权已过期，请重新申请授权",
               "data":   nil,
            })
            c.Abort()
            return
         }
         // 其他错误
         c.JSON(http.StatusOK, gin.H{
            "status": -1,
            "msg":    err.Error(),
            "data":   nil,
         })
         c.Abort()
         return
      }

      // 解析到具体的claims相关信息
      c.Set("claims", claims)
   }
}
```

jwt 是如何将 header,paylaod,signature 组装在一起的？

我们从创建 token 的函数开始看起

```go
func (j *JWT) CreateToken(claims Myclaims) (string, error) {
   // 返回一个token的结构体指针
   token := jwtgo.NewWithClaims(jwtgo.SigningMethodHS256, claims)
   return token.SignedString(j.SigningKey)
}
```

```go
// A JWT Token.  Different fields will be used depending on whether you're
// creating or parsing/verifying a token.
type Token struct {
    Raw       string                 // The raw token.  Populated when you Parse a token
    Method    SigningMethod          // The signing method used or to be used
    Header    map[string]interface{} // The first segment of the token
    Claims    Claims                 // The second segment of the token
    Signature string                 // The third segment of the token.  Populated when you Parse a token
    Valid     bool                   // Is the token valid?  Populated when you Parse/Verify a token
}

func NewWithClaims(method SigningMethod, claims Claims) *Token {
    return &Token{
        Header: map[string]interface{}{
            "typ": "JWT",
            "alg": method.Alg(),
        },
        Claims: claims,
        Method: method,
    }
}
```

CreateToken 用 JWT 对象绑定，对象中包含密钥，函数的参数是载荷

NewWithClaims 函数参数是加密算法，载荷 NewWithClaims 的具体作用是是初始化一个 Token 对象

SignedString 函数，参数为密钥
主要是得到一个完整的 token

SigningString 将 header 与 载荷 处理后拼接在一起

Sign 将密钥计算一个 hash 值，与 header，载荷拼接在一起，进而制作成 token

此处的 Sign 方法具体是调用哪一个实现，请继续往下看

```go
// Get the complete, signed token
func (t *Token) SignedString(key interface{}) (string, error) {
    var sig, sstr string
    var err error
    if sstr, err = t.SigningString(); err != nil {
        return "", err
    }
    if sig, err = t.Method.Sign(sstr, key); err != nil {
        return "", err
    }
    return strings.Join([]string{sstr, sig}, "."), nil
}
```

SigningString

将 header 通过 json 序列化之后使用 base64 加密

同样的也将载荷通过 json 序列化之后使用 base64 加密

将这俩加密后的字符串拼接在一起

```go
// Generate the signing string.  This is the
// most expensive part of the whole deal.  Unless you
// need this for something special, just go straight for
// the SignedString.
func (t *Token) SigningString() (string, error) {
    var err error
    parts := make([]string, 2)
    for i, _ := range parts {
        var jsonValue []byte
        if i == 0 {
            if jsonValue, err = json.Marshal(t.Header); err != nil {
                return "", err
            }
        } else {
            if jsonValue, err = json.Marshal(t.Claims); err != nil {
                return "", err
            }
        }

        parts[i] = EncodeSegment(jsonValue)
    }
    return strings.Join(parts, "."), nil
}
// Encode JWT specific base64url encoding with padding stripped
func EncodeSegment(seg []byte) string {
    return strings.TrimRight(base64.URLEncoding.EncodeToString(seg), "=")
}
```

回到创建 token 函数的位置

```go
func (j *JWT) CreateToken(claims Myclaims) (string, error) {
   // 返回一个token的结构体指针
   token := jwtgo.NewWithClaims(jwtgo.SigningMethodHS256, claims)
   return token.SignedString(j.SigningKey)
}
```

SigningMethodHS256 对应这一个结构 SigningMethodHMAC，如下

```go
// Implements the HMAC-SHA family of signing methods signing methods
// Expects key type of []byte for both signing and validation
type SigningMethodHMAC struct {
     Name string
     Hash crypto.Hash
}

// Specific instances for HS256 and company
var (
     SigningMethodHS256  *SigningMethodHMAC
     SigningMethodHS384  *SigningMethodHMAC
     SigningMethodHS512  *SigningMethodHMAC
     ErrSignatureInvalid = errors.New("signature is invalid")
)
```

看到这里，便解开了上述第 4 点 Sign 方法具体在哪里实现的问题

```go

// Implements the Sign method from SigningMethod for this signing method.
// Key must be []byte
func (m *SigningMethodHMAC) Sign(signingString string, key interface{}) (string, error) {
     if keyBytes, ok := key.([]byte); ok {
          if !m.Hash.Available() {
               return "", ErrHashUnavailable
          }

          hasher := hmac.New(m.Hash.New, keyBytes)
          hasher.Write([]byte(signingString))

          return EncodeSegment(hasher.Sum(nil)), nil
     }

     return "", ErrInvalidKeyType
}
```

效果查看

用 postman 测试
