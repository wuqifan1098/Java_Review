# 面试题

## 1.客户端禁用cookie怎么办？ 你说的实现方式安全吗？（头条、阿里）

采用URL重写的方法，解决禁用Cookie

URL地址重写是对客户端不支持Cookie的解决方案。URL地址重写的原理是**将该用户Session的id信息重写到URL地址中。**服务器能够解析重写后的URL获取Session的id。这样即使客户端不支持Cookie，也可以使用Session来记录用户状态。HttpServletResponse类提供了encodeURL(String url)实现URL地址重写，该方法会自动判断客户端是否支持Cookie。如果客户端支持Cookie，会将URL原封不动地输出来。如果客户端不支持Cookie，则会将用户Session的id重写到URL中。

response.encodeRedirectURL(java.lang.String url) 用于对sendRedirect方法后的url地址进行重写。
response.encodeURL(java.lang.String url)用于对表单action和超链接的url地址进行重写

**在URL后面手动拼接jsessionid**


```java
   //------------使用session解决数据共享问题---------------------
	// 获得session对象
	HttpSession session = req.getSession();
	// 获取jsessionId
	String jsessionid = session.getId();
	// 设置超时时间
	//session.setMaxInactiveInterval(5);
	// 设置共享数据
	session.setAttribute("username", name);
	//------------------------------------------------------
	writer.print("欢迎"+name+"<br/>");
	writer.print("<a href='/session/list;jsessionid="+jsessionid+"' >箱子</a>");
```

**使用响应对象HttpServletRequest中的encodeURL(String path)方法实现jsessionid的自动拼接** 

```java
//------------使用session解决数据共享问题---------------------
	// 获得session对象
	HttpSession session = req.getSession();
	//session.setMaxInactiveInterval(5);
	// 设置共享数据
	session.setAttribute("username", name);
	//------------------------------------------------------
	writer.print("欢迎"+name+"<br/>");
	String url = resp.encodeURL("/session/list");
    // session/list;jsessionid=1C8E961E7366838D5390EAD9963B396C
	System.out.println(url);     
	//writer.print("<a href='/session/list;jsessionid="+jsessionid+"' >箱子</a>");
	writer.print("<a href='"+url+"' >箱子</a>");
```

## 2. Session的生命周期

### Session何时生效

Sessinon在用户访问**第一次访问服务器时创建，需要注意只有访问JSP、Servlet等程序时才会创建Session**，只访问HTML、IMAGE等静态资源并不会创建Session,可调用request.getSession(true)强制生成Session。

### Session何时失效
1.服务器会把长时间没有活动的Session从服务器内存中清除，此时Session便失效。Tomcat中Session的默认失效时间为20分钟。

2.调用Session的invalidate方法。

```java
HttpSession session = request.getSession();
session.invalidate();//注销该request的所有session
```

3.session的过期时间是从什么时候开始计算的？是从一登录就开始计算还是说从停止活动开始计算？

答：从session不活动的时候开始计算，如果session一直活动，session就总不会过期。

从该Session未被访问,开始计时; 一旦Session被访问,计时清0;

4.设置session的失效时间

a)web.xml中

```java
<session-config>
    <session-timeout>30</session-timeout>
</session-config>
```

b)在程序中手动设置

session.setMaxInactiveInterval(30 * 60);//设置单位为秒，设置为-1永不过期

request.getSession().setMaxInactiveInterval(-1);//永不过期

## 3. Cookie 与 Session的区别

- Cookie以**文本文件格式**存储在浏览器中，而session**可以存取任何类型的数据**。

- cookie的存储限制了数据量，只允许4KB，而session是无限量的

- **Cookie 一般用来保存用户信息**，**Session 的主要作用就是通过服务端记录用户的状态**

- Cookie 存储在客户端中，而Session存储在服务器上，相对来说 **Session 安全性更高**。

# Cookie

HTTP/1.1 引入 Cookie 来**保存状态信息**。

**1. 用途**

- 会话状态管理（如用户登录状态、购物车、游戏分数或其它需要记录的信息）
- 个性化设置（如用户自定义设置、主题等）
- 浏览器行为跟踪（如跟踪分析用户行为等）

**2. 分类**

- 会话期 Cookie：浏览器关闭之后它会被自动删除，也就是说它仅在会话期内有效。
- 持久性 Cookie：指定一个特定的过期时间（Expires）或有效期（Max-Age）之后就成为了持久性的 Cookie。

# Cookie 与 Session

- Cookie **只能存储 ASCII 码字符串**，而 Session 则**可以存取任何类型的数据**，因此在考虑数据复杂性时首选 Session；

- Cookie 存储在浏览器中，容易被恶意查看。如果非要将一些隐私数据存在 Cookie 中，可以将 Cookie 值进行加密，然后在服务器进行解密；

- 对于大型网站，如果用户所有的信息都存储在 Session 中，那么开销是非常大的，因此不建议将所有的用户信息都存储到 Session 中。

- Cookie可以设置domain属性来实现跨域名，Session只在当前的域名内有效，不可夸域名

## Cookie的作用是什么?和Session有什么区别？

Cookie 和 Session都是用来跟踪浏览器用户身份的会话方式，但是两者的应用场景不太一样。

**Cookie 一般用来保存用户信息** 比如①我们在 Cookie 中保存已经登录过得用户信息，下次访问网站的时候页面可以自动帮你登录的一些基本信息给填了；②一般的网站都会有保持登录也就是说下次你再访问网站的时候就不需要重新登录了，这是因为用户登录的时候我们可以存放了一个 Token 在 Cookie 中，下次登录的时候只需要根据 Token 值来查找用户即可(为了安全考虑，重新登录一般要将 Token 重写)；③登录一次网站后访问网站其他页面不需要重新登录。**Session 的主要作用就是通过服务端记录用户的状态。**典型的场景是购物车，当你要添加商品到购物车的时候，系统不知道是哪个用户操作的，因为 HTTP 协议是无状态的。服务端给特定的用户创建特定的 Session 之后就可以标识这个用户并且跟踪这个用户了。

Cookie 数据**保存在客户端(浏览器端)**，Session 数据**保存在服务器端**。

Cookie 存储在客户端中，而Session存储在服务器上，相对来说 **Session 安全性更高**。如果使用 Cookie 的一些敏感信息不要写入 Cookie 中，最好能将 Cookie 信息加密然后使用到的时候再去服务器端解密。

# Session

## 实现原理

服务器创建session出来后，会**把session的id号，以cookie的形式**回写给客户机，这样，只要客户机的浏览器不关，再去访问服务器时，都会带着session的id号去，服务器发现客户机浏览器带session id过来了，就会使用内存中与之对应的session为之服务。

# Token

**Token是服务端生成的一串字符串，以作客户端进行请求的一个令牌**。当客户端第一次访问服务端，服务端会根据传过来的唯一标识userId，运用一些算法，并加上密钥，生成一个Token，然后通过BASE64编码一下之后将这个Token返回给客户端，客户端将Token保存起来（可以通过数据库或文件形式保存本地）。下次请求时，客户端只需要带上Token，服务器收到请求后，会用相同的算法和密钥去验证Token。

使用基于 Token 的身份验证方法，在服务端不需要存储用户的登录记录。大概的流程是这样的：

- 客户端使用用户名跟密码请求登录
- 服务端收到请求，去验证用户名与密码
- 验证成功后，服务端会签发一个 Token，再把这个 Token 发送给客户端
- 客户端收到 Token 以后可以把它存储起来，比如放在 Cookie 里或者数据库里
- 客户端每次向服务端请求资源的时候需要带着服务端签发的 Token
- 服务端收到请求，然后去验证客户端请求里面带着的 Token，如果验证成功，就向客户端返回请求的数据