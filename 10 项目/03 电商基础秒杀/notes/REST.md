## REST

REST是*Representational State Transfer*(在表现层上的状态转化)的缩写，这个词的意思要在文章的后面才能解释清楚。REST是一种WEB应用的架构风格，[它被定义为6个限制](https://link.jianshu.com?t=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FRepresentational_state_transfer%23Architectural_constraints)，满足这6个限制，能够获得诸多优点（详细优点在文章最后总结）。

先用一句话来概括RESTful **API**(具有REST风格的API): **用URL定位资源，用HTTP动词（GET,HEAD,POST,PUT,PATCH,DELETE）描述操作，用状态码表示操作结果。**
 但是REST远远不仅是指API的风格，它是一种WEB应用的架构风格。我们到后面会有所体会。

### 引入：从另一个角度看待前后端

我们浏览一个网站，说到底就是与这个网站中的资源进行互动（获取、提交、修改、删除）。前端的工作，就是为用户从服务端获取资源、展示资源、请求服务端改变资源。

RESTful API有助于客户端和服务端的功能分离，服务器完全扮演着一个“资源服务商”的角色。各种不同的客户端都可以通过同一套API与这个“资源服务商”交流，从而与资源进行互动。





![img](https:////upload-images.jianshu.io/upload_images/4888929-b3c72a5abb3ce4db.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/550/format/webp)

RESTful API

### 指定一个资源

在RESTful架构风格中，URL用来指定一个资源。资源就是服务器上的数据。比如说URL`/api/users`表示的是该网站的所有用户，这是一种资源，可以与之互动（获取、提交、修改、删除）。另外，资源地址使用嵌套的结构，比如`/api/users/csr`表示用户名为'csr'的用户，`/api/users/csr/blogs`表示'csr'的所有博客，`/api/users/csr/blogs/1234567`表示其中的某一篇博客。这些都是资源，后者嵌套在前者之中。

既然URL表示一个资源，自然就不应该包含动词，它应该由名词组成。一个 not RESTful 的例子是通过向`api/delete/resource`发送GET请求来删除一个资源。

> 更详细的URL设计可以查看[这篇博客](https://link.jianshu.com?t=http%3A%2F%2Fwww.ruanyifeng.com%2Fblog%2F2014%2F05%2Frestful_api.html)。URL风格只是REST的外表，不是本文的重点。

### 操作一个资源

既然通过URL能够指定一个服务器上的资源。那么我们应该如何与这个资源进行互动呢？我们对这个资源(URL)使用不同的HTTP方法，就代表对这个资源的不同操作：

- GET       获取资源
- HEAD      获取资源的概况(响应的HTTP只有head，没有body)
- POST      新建资源
- PUT       更新资源(客户端提供完整资源数据)
- PATCH     更新资源(客户端提供需要修改的资源数据)
- DELETE    删除资源

GET、HEAD、PUT、DELETE方法是**幂等方法**(对于同一个内容的请求，发出n次的效果与发出1次的效果相同)。
 GET、HEAD方法是**安全方法**(不会造成服务器上资源的改变)。

> PATCH不一定是幂等的。PATCH的实现方式有可能是"提供一个用来替换的数据"，也有可能是"提供一个更新数据的方法"(比如`data++`)。如果是后者，那么PATCH不是幂等的。

### 通过HTTP状态码来表示操作的结果

虽然HTTP状态码设计的本意就是表示操作结果，但是有时候人们往往没有很好的利用它，RESTful API要求充分利用HTTP状态码

```java
200 OK - [GET]：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）。
201 CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功。
202 Accepted - [*]：表示一个请求已经进入后台排队（异步任务）
204 NO CONTENT - [DELETE]：用户删除数据成功。
400 INVALID REQUEST - [POST/PUT/PATCH]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。
401 Unauthorized - [*]：表示用户没有权限（令牌、用户名、密码错误）。
403 Forbidden - [*] 表示用户得到授权（与401错误相对），但是访问是被禁止的。
404 NOT FOUND - [*]：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。
406 Not Acceptable - [GET]：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。
410 Gone -[GET]：用户请求的资源被永久删除，且不会再得到的。
422 Unprocesable entity - [POST/PUT/PATCH] 当创建一个对象时，发生一个验证错误。
500 INTERNAL SERVER ERROR - [*]：服务器发生错误，用户将无法判断发出的请求是否成功。
```

### 如何设计RESTful API

在过去不使用RESTful架构风格的时候，如果我们要设计一个系统，会以“操作”为出发点，然后围绕它去建设其他需要的东西。
 举个例子，我们要向系统中增加一个用户登陆的功能：

1. 需要一个用户登陆的功能(操作)
2. 约定一个用于登录的API(也就是URL)
3. 约定这个API的使用方式(发送\响应什么数据、格式是什么)
4. 前后端针对这个API进行开发

这种设计方式有如下缺点：

1. 当我们不断为这个系统增加操作，每增加一个操作都要按照上面的流程设计一次，第2和3点的工作实际是可以大大削减的(通过REST)。
2. 操作之间可能是有依赖的，依赖多起来，系统会变得很复杂。
3. 我们的API缺乏一致性(需要一份庞大的文档来记录api的地址、使用方式)。
4. 操作通常被认为是有副作用（Side Effect）的，很难使用缓存技术。

而如果我们设计REST风格的系统，资源是第一位的考虑，首先从资源的角度进行系统的拆分、设计，而不是像以往一样以操作为角度来进行设计。

用两个例子来说明：银行的转账功能和matrix的提交功能。

> matrix是一个课程管理网站，可以提交作业，读者不知道也没关系。

这两个功能非常具有“动作性”，看起来和“资源”联系不大，很容易就会设计成not RESTful的API：`POST /transfer/${amount}/to/${toUserID}`、`POST api/assigments/${assigmentID}/submit`。
 一旦在URL中引入了动词，这个URL的功能就定死了，无法用于别的用途。而且各种功能的API各有各的结构，一致性很差，需要一份详细的API文档才能使用。

这种情况下，要如何通过RESTful架构风格，设计一套一致、多用途的URL呢？
 **简单地说，就是将一个“动作”理解为“操作一个资源”。这里的“操作”是指HTTP的方法。**

对于转账动作，就可以理解为“POST一个转账事务”，因此API就可以设置成这样: `POST /transactions`，请求体为：`to=632&amount=500`。这样的设计不但简洁明了，而且我们可以将这个URL用于别的用途：通过`GET /transactions`来获取该用户的所有转账事务。如果在这个URL的尾部加上transactionID我们还可以获取某一次转账记录。

对于matrix的提交作业动作，我们可以理解为“POST一份作业答案”，所以API设计为`POST /api/courses/${courseId}/assignments/${problemId}/submissions`，在请求体中保存学生要提交的内容。我们可以将这个URL用于别的用途：通过向这个URL发送GET请求来获得该学生在这个题目的所有提交。如果在这个URL的尾部加上submissionID我们还可以获取、删除、更改某一个提交记录。

从以上的两个例子我们可以看出，使用RESTful风格可以克服传统架构风格的那4个缺陷：

1. 设计API工作量减少，因为功能需求一旦出来，需要操作的资源、操作的方式立刻就能分析出来，因此资源URL和API的使用方式(GET, POST...)都很容易得到。
2. 没有了操作之间的依赖。资源之间虽然可能有关联，但是小得多。
3. 对资源的操作也就那么几种(获取、新建、修改、删除)，API的一致性、自我描述性很强，不需要过多解释。
4. 对于GET请求，我们都可以考虑使用缓存，因为在RESTful的架构中，GET请求代表获取数据，必须是安全、幂等的。

### 无状态

根据[REST的架构限制](https://link.jianshu.com?t=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FRepresentational_state_transfer%23Architectural_constraints)，RESTful的服务器必须是无状态的，这意味着来自客户的每一个请求必须包含服务器处理该请求所需的所有信息， 服务器不能利用任何已经存储的“上下文(context，在这里表示用户的状态)”来处理新到来的请求，会话状态只能由客户端来保存，并且在请求时一并提供。

> 这里注意两点。1. 服务器不能存储“上下文”不代表连数据库都不能有，“上下文”指那些在服务器内存中的、非持久化的数据。2.
>  无状态不代表不能有会话(sessions)，无状态仅仅指**服务器无状态**，会话的状态由客户端提供。

我一开始以为无状态与用户登陆是冲突的，后来在[StackOverflow](https://link.jianshu.com?t=https%3A%2F%2Fstackoverflow.com%2Fquestions%2F6068113%2Fdo-sessions-really-violate-restfulness)上找到了一个令我满意的解答。以下两幅图摘录自这个答案。
 无状态的认证机制：
 



![img](https:////upload-images.jianshu.io/upload_images/4888929-fc26b6ac220fa85d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/792/format/webp)

无状态的认证机制



> What you need is storing username and password on the client and send it with every request. You don't need more to do this than HTTP basic auth and an encrypted connection.
>  只需要将用户名和密码存储在客户端，然后客户端每次发送请求都附带上用户名和密码。要做到这点你只需要[HTTP基本认证](https://link.jianshu.com?t=https%3A%2F%2Fzh.wikipedia.org%2Fwiki%2FHTTP%E5%9F%BA%E6%9C%AC%E8%AE%A4%E8%AF%81)和一个加密的连接(HTTPS)。
>  如果每次认证，都要去数据库查询用户的信息来核对，那么响应会非常慢，而且服务器也会有很大的性能损失。为了加快认证的速度，最好在内存中使用认证缓存。这并不违背“无状态”的限制，因为缓存的作用仅仅起加速的作用，没有缓存照样工作。

无状态的第三方鉴权机制：





![img](https:////upload-images.jianshu.io/upload_images/4888929-ed5a07c637d467b3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/981/format/webp)

无状态的第三方鉴权机制

> What about 3rd party clients? They cannot have the username and password and all the permissions of the users. So you have to store separately what permissions a 3rd party client can have by a specific user. So the client developers can register they 3rd party clients, and get an unique API key and the users can allow 3rd party clients to access some part of their permissions. Like reading the name and email address, or listing their friends, etc... After allowing a 3rd party client the server will generate an access token. These access token can be used by the 3rd party client to access the permissions granted by the user.
>  通过这个方式，用户可以给第三方应用授权，让第三方应用拿着用户的“令牌”访问网站的一些服务。

> 以上两幅图讲的是RESTful风格的身份验证机制。在实践中最好使用[OAuth 2.0框架](https://link.jianshu.com?t=http%3A%2F%2Fwww.ruanyifeng.com%2Fblog%2F2014%2F05%2Foauth_2_0.html)。

无状态增强了系统的故障恢复能力，因为在服务器上没有保存session的状态，所以恢复起来更容易。
 更重要的是，无状态意味着分布式系统能够更好地工作，负载均衡器可以自由地将请求分发到任意的服务器。因为请求中都已经包含了服务器所需的所有信息，任何服务器都可以处理。
 并且，无状态让系统的横向拓展能力强大。因为不需要在不同的服务器之间同步session状态，所以服务器之间的沟通开销很低。增加服务器的数量不会带来明显的性能损失（“1+1”更接近于“2”了）。

### HATEOAS



![img](https:////upload-images.jianshu.io/upload_images/4888929-981aaac89332aac9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/673/format/webp)

REST的4个层次

前面已经讨论了level 1和level 2，实际上REST还有一个更高的层次：HATEOAS(Hypermedia As The Engine Of Application State)。

对于客户端的资源请求，服务器不仅要返回所请求的资源，而且要根据请求返回用户所处的状态和可转移的状态。
 客户端不需要提前知道应用有哪些状态，而是根据服务端响应的“可转移的状态”，提供给用户选择，从而发生状态转移。

[这个例子](https://link.jianshu.com?t=https%3A%2F%2Fwww.zhihu.com%2Fquestion%2F28557115%2Fanswer%2F48120528)和[这个例子](https://link.jianshu.com?t=https%3A%2F%2Fstackoverflow.com%2Fquestions%2F671118%2Fwhat-exactly-is-restful-programming%2F671132%23671132)都是不错的解释。

用简单的话来说，在严格的RESTful架构中，客户端不需要提前知道服务端的API有哪些、怎么调用，在客户端与服务器通信的过程中，服务端会告诉客户端：在你当前所处的状态下，有哪些API可以使用、可以转移到哪些状态。

> 既然服务器是无状态的，那么它要如何知道发起请求的用户处于什么状态呢？这就要求客户端在发送请求的时候要携带上足够的信息，让服务器能够判断客户端的状态。

这就很像10086的“电话自动语音应答服务”：你想要查询你的手机流量，只需要会拨打“10086”，对方会提示你按下哪些按键就能进入哪些状态。进入下一个状态以后，又会有语音提示你接下来能够按哪些按键……最终，你能进入到你想要的那个状态（流量查询服务）。你需要记住的仅仅是“10086”这个号码而已！

> 10086的语音提示相当于Hypermedia，是驱动应用状态转换的“引擎”。

再进一步想想，在RESTful架构中，所有的状态其实就组成了一颗树(更准确地说是网)：根节点就是网站的基地址。在你获得一个节点中的资源的同时，服务器还会返回给你这个节点的边：Hypermedia(超链接就是一种Hypermedia)。通过Hypermedia，你能够知道如何跳转到相邻的节点。
 结果就是：你能够访问到这颗树的所有节点，而你所需要提前知道的只是“如何到达根节点”而已！

> 每个节点就是一个状态。客户端就在这个状态网中不断跳转。

这种架构的优势非常明显：前后端之间的耦合更加微弱。
 随着应用功能的升级改变，“树”的样子会大大改变，但是只需要让后端修改返回的资源内容和Hypermedia，前端几乎不用改动。功能的演化更加灵活了。

### REST字面意思

现在你应该明白Representational State Transfer中的State Transfer(状态转化)是什么意思了。那么Representational(表现层的)是什么意思呢？
 资源可以有很多种表示(representation)。表示是资源的外在形式，资源是表示的真正内容。不管用什么形式来表示，始终描述的是这个资源。当我们讨论“文章列表”这个**资源**时，我们并不在乎它是json格式还是xml格式。但是当我们真的要在服务器与客户端之间传输数据的时候，不能说“传输资源”，因为资源太抽象了，发送方必须要以某一种形式来传递它，接收方才能很好地解析。发送方与接收方之间传输的就是representation。
 在[Roy Fielding的论文](https://link.jianshu.com?t=http%3A%2F%2Fwww.ics.uci.edu%2F~fielding%2Fpubs%2Fdissertation%2Frest_arch_style.htm)中有这个定义：*A representation is a sequence of bytes, plus representation metadata to describe those bytes.*  HTTP报文是一种representation，header包含描述信息（尤其是`Cnotent-Type`这种），body就是一个字节序列。除此之外，document, file也是representation。
 Representational State Transfer的结构是(Representational (State Transfer))，在这里我们用的是representation的形容词形式，表示**在表现的层次上讨论状态转化**。这个词的字面意思是**通过传输representation来触发的客户端状态转化**。

------

### 总结

至此，我们应该能够体会到REST已经不仅仅是一种API风格了，它是一种软件架构风格(REST本身不是一种架构)。REST风格的软件架构具有很强的演化、拓展能力：

1. 一致的URL和HTTP动词使用：确保系统能够接纳多样而又标准的客户端，保证客户端的演化能力。
2. 无状态：保证了系统的横向拓展能力、服务端的演化能力。
3. HATEOAS：保证了应用本身的演化能力(功能增加、改变)。

这3点是单单对演化拓展优势的说明，[这个回答](https://link.jianshu.com?t=(https%3A%2F%2Fzhihu.com%2Fquestion%2F28557115%2Fanswer%2F79275672))总结了REST的6个约束分别对应的优点。

------

## 参考资料

[https://stackoverflow.com/questions/671118/what-exactly-is-restful-programming](https://link.jianshu.com?t=https%3A%2F%2Fstackoverflow.com%2Fquestions%2F671118%2Fwhat-exactly-is-restful-programming)
 [https://stackoverflow.com/questions/6068113/do-sessions-really-violate-restfulness](https://link.jianshu.com?t=https%3A%2F%2Fstackoverflow.com%2Fquestions%2F6068113%2Fdo-sessions-really-violate-restfulness)
 [https://martinfowler.com/articles/richardsonMaturityModel.html](https://link.jianshu.com?t=https%3A%2F%2Fmartinfowler.com%2Farticles%2FrichardsonMaturityModel.html)
 [https://www.zhihu.com/question/28557115](https://link.jianshu.com?t=https%3A%2F%2Fwww.zhihu.com%2Fquestion%2F28557115)
 [https://www.zhihu.com/question/33959971](https://link.jianshu.com?t=https%3A%2F%2Fwww.zhihu.com%2Fquestion%2F33959971)
 [RESTful API 设计指南](https://link.jianshu.com?t=http%3A%2F%2Fwww.ruanyifeng.com%2Fblog%2F2014%2F05%2Frestful_api.html)
 [https://stackoverflow.com/a/10421579/8175856](https://link.jianshu.com?t=https%3A%2F%2Fstackoverflow.com%2Fa%2F10421579%2F8175856)
 [https://en.wikipedia.org/wiki/Representational_state_transfer](https://link.jianshu.com?t=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FRepresentational_state_transfer)