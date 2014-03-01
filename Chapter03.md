面向对象
========

[OOP][1] 的思想, 无疑是非常实用有效的. 事实是, 无论语言是否直接支持面向对象的编程. 程序员在写代码的时候常常会应用 OOP 的思想.

Go 语言下没有类(Class), 没有构造函数, 没有 `this` 指针, 没有多态,  只有复合对象(或匿名属性). 复合对象和继承是完全不同的. 在以后的文字中, 继承这个词不再代表一般 OOP 下的继承, 指的是复合对象.
应用 OOP 的思想, WEB 应用下控制器常见形式祖先类型的示意写法(现实中没有太大意义).

```go
// 定义基础控制器结构
type BaseController struct {
	Data interface{}         // 应用要维护的数据
	Req  *http.Request       // 请求对象
	Res  http.ResponseWriter // 响应对象
}

// 官方 net/http 包要求实现的接口
func (p *BaseController) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	p.Req = r // 保存起来供实例使用
	p.Res = w
	if r.Method == "POST" {
		p.Post()
	}
}

// 对应 http POST 方式
func (p *BaseController) Post() {
	// 继承者必须覆盖这个方法, 否则认为不允许 POST 访问
	// BaseController 是不可能知道继承者要做什么, 那就只能返回 403 拒绝访问
	p.Res.WriteHeader(403)
}

// Login 控制器
type Login struct {
	BaseController // 匿名复合
}

// 这里必须覆盖 BaseController.Post, 以实现 Login 的具体行为
func (p *Login) Post() {
	if p.Req.Form.Get("login_name") == "" {
		p.Data = "无效的登录名"
		return
	}
	// 这里省略登录成功的过程
	p.Data = "登录成功"
}

// 把这些行为定义成接口
type Controller interface {
	ServeHTTP(http.ResponseWriter, *http.Request)
	Post()
}
```

用例

```go
http.Handle("/login", &Login{})
```
**但是这有并发问题**

并发下维护上下文
================

很明显现实中这样的用法是错误的, 因为 WEB 的请求是**并发**的, 这样写所有并发的请求都由同一个 `&Login{}` 去处理响应, `Req`,`Res`,`Data` 在并发中都被指向相同的对象. 这是无法正常工作的. 这就是常说的维护上下文, Context.

**并发环境每一个请求都要有维护独占数据的能力.** 除非没有独占数据要维护.

先重新审视 官方包 server.go 中的代码

```go
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}

func Handle(pattern string, handler Handler) { DefaultServeMux.Handle(pattern, handler) }

func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	DefaultServeMux.HandleFunc(pattern, handler)
}

func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	mux.Handle(pattern, HandlerFunc(handler)) // 进行了转换, 只是为了符合接口要求
}

type HandlerFunc func(ResponseWriter, *Request) // 确实没有独占数据要维护

// ServeHTTP calls f(w, r).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
	f(w, r)
}
```

这个`http.Handler`接口其实只是被当作一个函数使用了. 并发问题留给使用者自己解决.
可以这样做

```go
http.HandleFunc("/login", func(w http.ResponseWriter, r *http.Request) {
	p := &Login{}
	p.ServeHTTP(w, r)
})
```

每次请求都有新的`Login`对象产生. 当然这个写法很生硬, 如果有 100 个控制器, 难道还要写 100 个不同的写法?! 可以采用下面的方法.

## 函数法

不用结构体直接使用函数, 所有上下文维护都在函数内部定义成局部变量, 局部变量在函数内部是独占的.

```go
func main() {
	http.HandleFunc("/login", login)
}
func login(w http.ResponseWriter, r *http.Request) {
    // 维护的数据是局部变量
	var data interface{}
	var post = func() { // 用闭包函数访问局部变量
		if r.Form.Get("login_name") == "" {
			data = "无效的登录名"
			return
		}
		data = "登录成功"
	}
	post()
}
```

完全就是个函数, 但是并发下, 这完全没有问题. 问题在于如何和其他的模块进行数据沟通, 方法确实有, 比如可以用 [gorilla/context][2]. 但是无法想象整个项目都用这种写法.

## 约定构造函数

Go 没有构造函数的概念的. 没关系我们约定一个. 其他语言常用`Constructor`, 这里选用 `New` 更符合 Go 风格.

```go
// 给控制器接口增加一个构造函数
type Controller interface {
	New() Controller
	ServeHTTP(w http.ResponseWriter, r *http.Request)
	Post()
}

// 扩充 Login , 实现 New 方法
func (p *Login) New() Controller {
	return &Login{}
}

// 定义一个 http.Handler 接口, 支持构造函数
type HandlerNew struct {
	Controller Controller
}

// http.Handler 接口实现
func (p *HandlerNew) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	c := p.Constructor.New()
	c.ServeHTTP(w, r)
}
```

用例

```go
http.Handle("/login", HandlerNew{new(Login)})
```

## 用反射Value.New

反射包 `reflect` 中的 `reflect.Value` 有 `New` 方法, 可以动态的构造出一个新对象. 有些框架就是采用了这种方法, 但是用 `Value.New` 只是得到一个空属性对象, 要对对象进行初始化依然要约定初始化函数, 这反而比 约定构造函数费事儿. 这里就不具体讨论了.

Martini下的并发
===============

笔者在发现 Martini 之前也很困惑到底用什么办法更好, 所以写了早期的 [TypePress][3], 和对应的 [Go-Blog-In-Action][4]. Martini 巧妙解决了 WEB 并发中的上下文维护.
Martini 发现了 WEB 开发中单个请求响应要维护的上下文有这样的事实:

 - 数据类型是预知的 很显然
 - 数据类型有限的   很显然
 - 数据类型常常是唯一的 就算偶有不唯一, 定义个别名就行了, 这很容易
 - 阶段响应, 完整的响应过程往往分多个阶段, 为了代码复用, 各个阶段有独立的代码

因此 Martini 采用了这样的方案:

 1. Martini 负责动态构建一个 Context 对象, Context 继承自 Injector
 2. Martini 的 Handler 是一组 []Hanlder, 有序执行
 3. 使用者对 Handler 进行阶段性功能划分, 先执行的负责准备好上下文数据 dat
 4. 通过 Map(dat) 保存到 Context. (实际由 Injector 负责)
 5. 后续 Hander 要用 dat, 直接在 Handler 函数中加入参数 dat datType
 6. Injector 通过 reflect 分析 Handler 的参数类型, 并取出 dat, 调用 Handler

这个方法比 gorilla/context 更高效实用, 虽然都是用 map 保存上下文数据, 差别有

 - gorilla/context 的 map 是全局的, Martini 保存到 Context
 - gorilla/context 只是做了 key/value 存储, Martini 完成了 Handler 调用

这和 OOP 有何关系? 关系是

    golang 不是真正的继承, 这给维护上下文数据造成了问题.
    Martini 解决了上下文数据维护问题, 应用可以放心的用复合写逻辑代码.
    上下文数据交给 Martini 就好.



  [1]: http://zh.wikipedia.org/wiki/%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1
  [2]: https://github.com/gorilla/context
  [3]: https://github.com/achun/typepress
  [4]: https://github.com/achun/Go-Blog-In-Action/tree/master