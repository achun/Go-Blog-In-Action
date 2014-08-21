Router
======
WEB 开发离不开 Router. 通常 Router 负责对 HTTP Request URL 进行分析, 匹配到对应的处理对象. URL 可以分为三部分 Host, Path, QueryParams. 在官方包提供的 http.Request 对象中有对应的字段.
以前作者没有关注路由具体实现, 只是拿来就用. 很偶然发现一个路由评测项目 [go-http-routing-benchmark][]. 尽管路由的开销很低, 仍旧很惊异路由有这么大差异, 那自己也实现一个吧, 看看里面到底有什么玄机.

Rivet
=====
 
于是 [Rivet][] 诞生了. Rivet 学习了 [httprouter][] 的方法, 用带首字母索引的 Trie 管理路由节点, 这是 Rivet 的核心. 在开发 Rivet 的过程中, 作者发现事实上:

* 带参数的 URL.Path 是普遍需求, 参数可能需要合法性(类型)检查.
* 路由处在请求处理环节的前端, 过滤请求是不可缺少的环节.
* 路由本质上也是一种过滤器.

[httprouter][] 评测列表中没有能同时满足这些需求的项目. 有些和框架结合太紧, 甚至不能独立工作, 无法解耦. Rivet 实现了这些需求, 而且做得了解耦. [hostrouter][] 就是基于 Rivet 实现了 Host Router, 很简短的代码展示了底层使用 Rivet 的方法. 同时也证明了 Rivet 是解耦的. Rivet 的速度大概在评测的中间水平, 一个主要原因是 Rivet 采用 map 保存 Path Parmas, httprouter 采用数组. 作者认为采用 map 是必要的, 这点性能损失是值得的.

Module
======

很喜欢 Martini 的注入方式, 在没有找到更好的惯用方法前, 使用注入是个好方法. Rivet 借鉴 Martini 的注入方法, 衍化出下面的结构保存注入对象:

```go
map[uint]interface{}
```

Rivet 集合了些功能. 最重要的是解耦. 现在问题来了:

	Rivet 可以替代 Martini, Rivet 深度解耦, 我们要不要采用开发与框架无关的独立模块, 以便代码复用?

[mod][] 就是这样的尝试. 目前, 作者也不知道能有多少模块可以采用这种开发方式, 我会继续尝试.

[go-http-routing-benchmark]: https://github.com/julienschmidt/go-http-routing-benchmark

[Rivet]: https://github.com/typepress/rivet
[httprouter]: https://github.com/julienschmidt/httprouter
[hostrouter]: https://github.com/typepress/hostrouter
[mod]: https://github.com/typepress/mod