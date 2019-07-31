## URI

URI 包含 URL 和 URN。

- URI（Uniform Resource Identifier，统一资源标识符）
- URL（Uniform Resource Locator，统一资源定位符）
- URN（Uniform Resource Name，统一资源名称）

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/20190731223406.png)

## URL 统一资源定位符

URL 英文全称为 Uniform Resource Locator，中文为翻译“统一资源定位符”，是用于在网络中传播和访问互联网资源的一个地址，只有通过特定的地址才能够访问的指定的资源或者网页，简而言之就是我们所说的网址，当然这样有些不太恰当，但是确实最容易被理解的，就如你必须通过 https://zhangzifan.com/about 这个 URL 地址才能访问到我泪雪博客关于子凡的页面，而通过其他的链接都是不行的，这个地址就被称之为 URL。

## URI 统一资源标识符

URI 英文全称为 Uniform Resource Identifier，中文翻译为“统一资源标识符”，也可以被理解为标识、定位任何资源的字符串，子凡感觉在某些程序或者标注的地方，许多人就喜欢将 URL 和 URI 两者混淆，其实通过反正其实都非常相似，但是却也有所不同，用子凡的简而言之来描述就是 **URI包含相对地址和绝对地址，URL 就属于绝对地址**，所以 URI 包含 URI，简单的举例就是很多的网站可能都有/about 这个路径，但是不同的域名或者 IP 访问到的就是不同的资源页面，所以这就只是一个标识，并不能标识其具体未知或者唯一性。

## IRI 国际化资源标识符

IRI 英文全称为 Internationalized Resource Identifiers，中文翻译为“国际化资源标识符”，和 URI 可以相提并论，其中的区别在于 URI 只能使用英文字符，所以没有办法很好的国际化兼容不同的文字语言，所以 IRI 就引入了 Unicode 字符来解决这个兼容问题，最后就有了国际化资源标识符（IRI）。

## URN 统一资源名称

URN 英文全称为 Uniform Resource Name，中文翻译为“统一资源名称”，这是用于表示唯一一个实体的标识符，但是有不能给出实体的位置，用于标识持久性 Internet 资源，URN 可以提供一种机制，用于查找和检索定义特定命名空间的架构文件。尽管普通的 URL 可以提供类似的功能，但是在这方面，URN 更加强大并且更容易管理，因为 URN 可以引用多个 URL。子凡举个最简单的例子大家就明白了，那就是：磁力链接，它就是 URN 的一种实现，可以持久化的标识一个 BT 资源，资源分布式的存储在 P2P 网络中，无需中心服务器用户即可找到并下载它。