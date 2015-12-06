Router
======
WEB 开发离不开 Router. 通常 Router 负责对 HTTP Request URL 进行分析, 匹配到对应的处理对象. URL 可以分为三部分 Host, Path, QueryParams. 在官方包提供的 http.Request 对象中有对应的字段.
以前作者没有关注路由具体实现, 只是拿来用. 很偶然发现一个路由评测项目 [go-http-routing-benchmark][]. 尽管路由的开销很低, 仍旧很惊异不同路由有这么大差异, 那自己也实现一个吧.

Rivet
=====
 
于是 [Rivet][] 诞生了. Rivet 学习了 [httprouter][] 的方法, 用前缀树(Trie)管理路由节点, 这是提高匹配速度的关键. 另外作者发现事实上:

* 带参数的 URL.Path 很普遍, 字符串参数可能被转换类型.
* 路由处在处理请求的前端, 这期间应用应该有机会拒绝请求.
* Host 路由应当被支持.
* Martini 的注入方式确实方便.
* 路由应该可被独立使用, 而不是和框架强耦合 

Rivet 满足了这些需求, 而且性能非常可观. 

Module
======

既然 Rivet 采用了注入方式, 那应该可以开发只使用 Go 自带 pkg 与框架无关的独立模块,  以便应用选取不同的框架. 当然事实上选用支持注入的框架是最方便的.

[mod][] 就是这样的尝试. 目前, 作者也不知道能有多少模块可以采用这种开发方式, 我会继续尝试.

[go-http-routing-benchmark]: https://github.com/julienschmidt/go-http-routing-benchmark
[Rivet]: https://github.com/typepress/rivet
[httprouter]: https://github.com/julienschmidt/httprouter
[mod]: https://github.com/typepress/mod