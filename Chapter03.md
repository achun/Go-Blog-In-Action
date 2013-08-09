OOP
===
[OOP][0] 的思想, 无疑是非常实用有效的. 事实是, 无论语言是否直接支持面向对象的编程. 程序员在写代码的时候常常会应用 OOP 的思想.

但是 Go 语言下没有类(Class), 没有 `this` 指针, 没有多态, 只有复合.

应用 OOP 的思想, WEB 应用下控制器常见形式祖先类型的写法(示意代码).
```go
type Controller struct {
	Data interface{}         // WEB 应用通常都有输出数据
	Req  *http.Request       // 来自 http 的请求对象
	Res  http.ResponseWriter // 响应对象
}

// 官方 net/http 包要求的接口形式
func (p *Controller) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	p.Req = r
	p.Res = w
	if r.Method == "POST" {
		p.Post()
		p.Out()
	}
}

// 对应 http POST 方式
func (p *Controller) Post() {
	// WEB开发常见的控制器方法, 用于处理客户 http Post 方式的请求
	// 继承者应该覆盖这个方法, 否则认为不允许这样访问, 那就返回 403 拒绝访问
	p.Res.WriteHeader(403)
}

// 可以预计的共性的处理, 比如向 Res 输出 Data, 也许这是一个模板处理的过程
// 没有共性的处理, 复合仅剩下封装属性字段的作用.
func (p *Controller) Out() {
	if p.Data == nil {
		panic("继承者竟然没有任何输出数据, 你让我怎么办?")
	}
	// 假设 Data 非常简单,就是一个string
	p.Res.Write([]byte(p.Data.(string)))
}

// Login 控制器
type Login struct {
	Controller // 复合
}

// 这里必须覆盖 Controller.Post, 以表示 Login 的具体行为
func (p *Login) Post() {
	if p.Req.Form.Get("login_name") == "" {
		p.Data = "无效的登录名"
		return
	}
	// 这里省略登录成功的过程
	p.Data = "登录成功"
}

// 需要一个接口定义, 这里只有简化的 post 请求.
type HttpPostController interface {
	ServeHTTP(http.ResponseWriter, *http.Request)
	Post()
	Out()
}
```

虽然 Go 只有复合, 但这并不妨碍实现类似 OOP 继承的方式.
`Login`的需求可以这样用
```go
http.Handle("/login", &Login{})
```

但是很明显,现实中这样的用法是错误的, 因为 WEB 的请求是并发的, 这样写所有并发的请求都由同一个`&Login{}`去处理.
`Req`,`Res`,`Data` 会在并发中被错误的赋值. 这明显是错误的方式.
我们需要对每一个请求都及时生成一个 `Login` 对象的机制.

重新审视 `http.Handle` 的第二参数. (官方包 server.go 中的代码)
```go
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}

func Handle(pattern string, handler Handler) { DefaultServeMux.Handle(pattern, handler) }

func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	DefaultServeMux.HandleFunc(pattern, handler)
}

func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	mux.Handle(pattern, HandlerFunc(handler))
}

type HandlerFunc func(ResponseWriter, *Request)

// ServeHTTP calls f(w, r).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
	f(w, r)
}
```
一个可以及时生成新的 `http.Handler` 接口对象功能的方法是我们真正需要的. 
你看到了上面的代码中多引用了几个.
`HandlerFunc`的工作方式就是我们要的, 可以这样做

```go
http.HandleFunc("/login", func(w http.ResponseWriter, r *http.Request) {
	p := &Login{}
	p.ServeHTTP(w, r)
})
```
每次请求都有新的`Login`对象产生. 当然这个写法很生硬, 如果有100个控制器,难道还要写100个不同的写法! 多数 WEB 框架会通过反射包的支持, 构建新对象. 的确是个好方法. 这种方法, 这里不打算费笔墨介绍.

重新审视这种 `OOP` 的方法在 `Go` 里面因为没有 this 指针的难堪. 除了复合然后通过反射外的方式生成新对象, 还有其他方法可走.

闭包
====
这是一种看上去蛋疼的用法.
```go
func main() {
	http.HandleFunc("/login", login)
}
func login(w http.ResponseWriter, r *http.Request) {
	var data interface{}
	var post = func() {
		if r.Form.Get("login_name") == "" {
			data = "无效的登录名"
			return
		}
		// 省略具体过程
		data = "登录成功"
	}
	var out = func() {
		if data == nil {
			w.WriteHeader(403)
			return
		}
		w.Write([]byte(p.Data.(string)))
	}
	post()
	out()
}
```

构造函数
=======
Go 没有构造函数的概念的. 没关系我们模拟一个.一般构造用`Constructor`,这用 `New` ,符合 Go 的风格
```go
// 给控制器接口增加一个构造函数
type HttpPostController interface {
	New() HttpPostController
	ServeHTTP(w http.ResponseWriter, r *http.Request)
	Post()
	Out()
}

// 扩充 Login , 实现 HttpPostController 接口
func (p *Login) New() HttpPostController {
	return &Login{}
}

// 需要一个能配合 http.Handler 支持构造函数
type HandlerNew struct {
	Constructor HttpPostController
}

// http.Handler 接口实现
func (p HandlerNew) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	c := p.Constructor.New()
	c.ServeHTTP(w, r)
}
```
这样使用
```go
http.Handle("/login", HandlerNew{new(Login)})
```
其实就是一层层的 `http.Handler` 接口包裹起来.

谁知道呢
=======
闭包和构造函数的方法, 什么时候用? 我只是把他们罗列出来, 我不知道什么样的场景可以用到. 或许后续的实现中就会用到, 谁知道呢!

[0]: http://zh.wikipedia.org/wiki/%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1
[1]: http://gowalker.org/net/http#Handler