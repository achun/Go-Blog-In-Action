轮子工
=====
`不要重复发明轮子`...停. 对于程序员来说代码就是个`轮子`. 不管怎样你总是在`用轮子`,`修轮子`,`造轮子`的`轮子工`里面转来转去.
如果您关注了这本电子书的进度, 会发现,这次的更新离上次已经有好多天了. 这么多天里面, 作者在`轮子工`工种之间打转转.

修轮子
=====
[工具链选择与修改][0] 中列举了`TypePress`计划选用的工具. [船小好调头][1] 中也说明了这样做的原因. 事实上这些天作者对其中的好几个工具进行了修改. 同时也积极与原作者进行沟通.
正如预料,要取得原作者的共识并认可那些修改, 不是一件容易的事情. 这并非是谁的方法更好, 本来不同的开发者会有不同的思路, 实现方法, 使用方法, 场景可能区别很大. 不同意见是必然的.
选用的这一组工具链, 相互是独立的, 修改一个不必担心影响其他. 如果用框架的话就只有等原作者支持了. 不然项目可能会随框架的更新而不兼容. 所以现在可以安心的`修轮子`. 不必担心原作者能否及时支持的问题.

toml
----
目前 `pelletier/go-toml` 的版本没有支持 `SaveToFile()`. 这是一个很常规的功能. 本来作者已经实现了这个功能, 并且发现[只能从根对象读取][2]的问题.
把每个[SetXXX][3]按照`Tree`的结构一层层的实现成独立完整的`TomlTree`就可以解决这个问题.顺手在`SaveToFile()`和`String()` 的时候对输出进行格式化.
实现这些需要的理论知识并不高深, 需要的仅仅是耐心和时间. 现在 [achun/go-toml][4] 已经可以用于项目了.

log
---
和 `uniqush/log` 作者 [沟通][5] 中发现第一次pull的代码有明显的问题, 继续修吧.

* 把官方 log 包的一些[常量][6]重新定义一下, 这样使用者需要定制 `flags` 的时候不必import官方包了.
* 在常量中增加了`LOGLEVEL_EQUAL  = -2`, 因为`uniqush/log`默认的写日志规则是: `0<=调用方法对应的level<=设定的level` 才会写日志.`设定的level`就是`NewLogger`的第3个参数. `LOGLEVEL_EQUAL` 提供了一个新的规则`调用方法对应的level==设定的level` 才会写日志. 您可以在[NewLogger][7]的代码中发现代码实现.
你会发现使用的时候,这个参数可以融合进`flags`参数, 正负的区别可以保证`equal规则`的实施.

用轮子
=====
工具有了, 正确的使用容易达到, 灵活的运用很难. 这里分享几个使用心得.
mux
---
`mux`提供的支持很多, 这里要说的其实并不限于用到`mux`上. `net/http` 要求使用者实现[Handler][8]接口. 也提供了类型[HandlerFunc][9]便捷的把一个函数转换成符合[Handler][8]接口的类型. 这个方法非常妙, 这得益于go对函数类型的支持. 这种方法可以广泛的运用到任何场景. 因为函数无处不在. 就算用OOP的方法定义的`struct`对象, 也一样需要保留函数接口.`mux` 的 [Router][10] 也是符合[Handler][8]接口. 现实中你定义好了路由, 可能你需要检查系统是否正常, 是否正常安装了, 是否数据库连接正常, 是否用户已经登录了, 是否用户提交的数据符合要求, 是否要过滤掉爬虫(当然这个问题就很复杂了), 在每一个路由函数中去调用这些检查当然可以, 但这样要写太多的

```go
if !Check() {
	return
}
```

使用[HandlerFunc][9]的方法对 `Router.ServeHTTP` 包装一下会更和谐.

```go
type HandlerMx func(http.ResponseWriter, *http.Request)
func (f HandlerMux) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	if !CheckMux(w,r) {
		return
	}
	f(w,r)
}
type HandlerRouter func(http.ResponseWriter, *http.Request)
func (f HandlerRouter) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	if !CheckRouter(w,r) {
		return
	}
	f(w,r)
}
``` 

这和[HandlerFunc][9]的道理其实是一样的. 只不过嵌入了`Check`等代码. 前面说过[HandlerFunc][9]是为函数提供了便捷的方法. 可`mux`的`Router`是个`struct`对象.
没有问题, 别忘记了Go支持`方法值`. 所以下面的用法完全没有问题.
`main`函数中可以这样修改.

```go
// manners.Serve(GracefulListen, Mux)
manners.Serve(GracefulListen, HandlerMux(Mux.ServeHTTP))
```

好吧, 这是个极端的用法,只是表示和`方法值`结合的用法. 一般情况我们不会去直接包装全局的`Mux`对象. 下面的代码就很常见了.

```go
// 便捷函数, 方便注册路由的时候用 HandlerRouter 包装
func Handle(handler http.Handler) *mux.Route {
	return Mux.NewRoute().Handler(HandlerRouter(handler.ServeHTTP))
}
func HandleFunc(f func(http.ResponseWriter, *http.Request)) *mux.Route {
	return Mux.NewRoute().Handler(HandlerRouter(f))
}
func init() {
	HandleFunc(func(w http.ResponseWriter, r *http.Request) {
		// 用户查看或者设置 profile 相关的代码
	}).Path("/profile").Name("profile").Methods("get", "post")
}
```

至于`CheckRouter`部分到底写什么, 你可以根据应用需求, 自由书写.

template
--------

作者曾对于如何深入的使用`template`包很迷惑. 以至于写了 [achun/template][11]. 正如[谁知道呢][12]所说, 作者尝试使用闭包和 `template` 结合.
结果发现以前对官方包`template`的理解和使用错的太离谱.
你可以[在线play][13]下面的代码

```go
package main

import (
	"bytes"
	"os"
	"text/template"
)

// 演示 template FuncMap 和闭包结合使用
func main() {
	tpl := template.New("")
	// 使用闭包让 funcs 可以直接读取到 tpl 对象
	tpl.Funcs(template.FuncMap{
		"import": func(filename string) string {
			tpl.New(filename).Parse("import ok")
			ts := tpl.Templates()
			bs := bytes.NewBufferString("")
			ts[len(ts)-1].Execute(bs, "")
			return bs.String()
		},
	})
	// 这里直接用字符串模拟 ParseFiles, 现实中用那个您随意
	tpl.New("import.tmpl").Parse(`{{import "foo.tmpl"}}`)
	tpl.ExecuteTemplate(os.Stdout, "import.tmpl", "")
}
```
看吧 `import` 模板并执行输出结果都能做到.(注意play上用Parse模拟了ParseFiles,你如果本地执行可以换成ParseFiles)
通过闭包和FuncMap结合, 让关键的对象处于闭包的变量访问作用域, 比如 `tpl` 自身, 又或者是[Template_Execute][14]的参数 `data`. 可以实现太多的变化了.


TypePress不是框架
================
好了按照顺序该说`造轮子`了, 作者却发现一个事实 `TypePress不是框架`, 而是一个应用. 现在 TypePress 里面的import却依然使用了

```go
import "github.com/achun/typepress/global"
```

这样的代码, 这不对啊. 应该使用

```go
import "global"
```
对应的目录结构也要调整为

	src
	├───conf
	├───controllers
	├───models
	└───views
	    ├───login
	    └───signup

对增加顶部的 `src` 目录, 并把`src`的路径加入到 `GOPATH` 中. 这样你下载 TypePress 并使用, 你可以随意的在里面添加自己的代码, 这类似一个本地包. 原先`github.com/achun/typepress/global` 你会担心受我的更新而出问题.
好了TypePress这个轮子就是一个模板, 不是只让你用, 是让你`修轮子`的.
本章更新到GitHub的时候 TypePress 配套的改写代码还没有完成.

[0]: https://github.com/achun/Go-Blog-In-Action/blob/master/Chapter01.md#%E5%B7%A5%E5%85%B7%E9%93%BE%E9%80%89%E6%8B%A9%E4%B8%8E%E4%BF%AE%E6%94%B9
[1]: https://github.com/achun/Go-Blog-In-Action/blob/master/Chapter01.md#%E8%88%B9%E5%B0%8F%E5%A5%BD%E8%B0%83%E5%A4%B4
[2]: https://github.com/achun/Go-Blog-In-Action/blob/master/Chapter07.md#toml
[3]: http://gowalker.org/github.com/achun/go-toml#TomlTree_Set
[4]: https://github.com/achun/go-toml
[5]: https://github.com/uniqush/log/pull/3
[6]: http://gowalker.org/github.com/achun/log#_constants
[7]: http://gowalker.org/github.com/achun/log#NewLogger
[8]: http://gowalker.org/net/http#Handler
[9]: http://gowalker.org/net/http#HandlerFunc
[10]: http://gowalker.org/github.com/gorilla/mux#Router_ServeHTTP
[11]: https://github.com/achun/template
[12]: https://github.com/achun/Go-Blog-In-Action/blob/master/Chapter03.md#%E8%B0%81%E7%9F%A5%E9%81%93%E5%91%A2
[13]: http://play.golang.org/p/Fil_Vi2ZhU
[14]: http://gowalker.org/text/template#Template_Execute