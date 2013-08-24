I18n支持
=======
I18n 代表国际化. 现实中就是把程序中的个各种输出转换成其他语言. 具体实现方法很多.
虽然 TypePress 目前没有确定具体方法, 可以先把接口预留下来. 并在程序中广泛的使用他.

I18n 用函数表示的话样子好像这样
```go
func I18n(lang, s string, i ...interface{}) string {
	if len(i) == 0 {
		return s
	}
	if strings.Index(s, "%") == -1 {
		return s + fmt.Sprint(i...)
	}
	return fmt.Sprintf(s, i...)
}
```
这里什么也不做, 包装了 `fmt.Sprint`,`fmt.Sprintf`, 原样返回字符串而已.
我们分析一下场景.

1. I18n 是和客户端有关的, 根据客户端(或许是用户定制)的不同, 来确定语言输出. 这需要能访问到`*http.Request` 才能分析到 `lang`
2. I18n 具体实现需要数据字典并能和从 `*http.Request` 中分析到的目标 `lang` 对应
3. WEB 下的数据处理离不开 `*http.Request`. 无论你是否把数据进行了提取再处理, 源头总有 `http.Request.Form` 的存在.
4. `http.Request.Form` 的类型是 `url.Values`, 并且可以添加删除修改.

总之,WEB下的 I18n 离不开 `*http.Request` 和 `url.Values`.
既然如此 TypePress 可以这样选择方案

1. 直接传递 `*http.Request` 给那些后续需要 I18n 的接口.
2. 传递之前对 `http.Request.Form` 进行处理.
3. I18n 这个函数干脆也直接接收 `*http.Request` 对象

那这样的话预留的I18n看起来应该是这样

```go
var I18n = func(r *http.Request, s string, i ...interface{}) string {
	if len(i) == 0 {
		return s
	}
	if strings.Index(s, "%") == -1 {
		return s + fmt.Sprint(i...)
	}
	return fmt.Sprintf(s, i...)
}
```
很明显控制器相关的代码中需要使用这些措施. 又要改接口了. 没关系 TypePress 写好的接口没几个. 现在就做这个工作, 比以后再做科学多了.

不得不说的是, 很多应用是没有 I18n 需求的, 好在默认的实现只是包装了下 `fmt` 的输出, 这应该可以接受.
