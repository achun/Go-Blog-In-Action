代码中的I18n
===========

I18n 代表国际化. 现实中就是把程序中的各种输出转换成其他语言. 具体实现方法很多.
虽然 TypePress 目前没有完全确定具体实现方法, 可以先把接口预留下来.

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

不得不说的是, 很多应用是没有 I18n 需求的, 好在默认的实现只是包装了下 `fmt` 的输出, 这成本应该可以接受.

重要的是 `url.Values` 提供的支持也非常的丰富, 虽然 `map[string][]string` 限定死最终数据类型是 `string`, 但是足以满足绝大多数需求了.

如果增加修改客户端的提交, 那直接操作 `http.Request.Form` 好了.

如果数据源格式是JSON,那转换到 `http.Request.Form`, 当作容器使用好了.

如果 `string` 真的支持不了你的业务需求, 只有单独写处理接口了.

以`models`中的代码举例
```go
func NotEnoughArguments(r *http.Request, s1, s3 string) string {
	return g.I18n(r, "not enough arguments on %s %s", s1, s3)
}
```
`NotEnoughArguments` 表示提供的参数不够, 信息格式是
```
"not enough arguments on %s %s"
```
需要两个参数表示发生在何种时候, 比如

```
"not enough arguments on Users Append()"
```

如果要对应转义成中文, 可以做成这样

```
"Users Append() 缺少足够的参数"
```

或者

```
"参数不够, 无法完成 新增用户"
```

数据库错误信息I18n
=================

通常数据库错误所返回的信息是用英文写的. 并且文字表述过于技术化, 最终用户难于理解. 业务逻辑一旦确定, 那对应的错误信息是可以转义成最终用户易于理解的文字. 数据库错误信息通常都有固定的格式, 因此对数据库错误信息按照业务逻辑进行 I18n 处理是有可能的.
以`mysql` 的 1062 号错误为例(举例用户注册的时候发生User_login被占用的例子)

```
Error 1062: Duplicate entry '0123456789abcdef0123456789abcdef' for key 'User_login'
```

抽象后的格式是这样.

```
Error 1062: Duplicate entry '%s' for key '%s'
```

最最终用户可读性比较好的转义类似是这样

```
错误编号 1062: '登录名' 发生数据重复 '0123456789abcdef0123456789abcdef'
```

抽象后的格式是这样

```
错误编号 1062: '$1' 发生数据重复 '$0'
```

这里的 `$1` 对应英文中的第2个`%s`, `$0` 对应英文中的第1个`%s`.
用代码吧那两个`%s`提取出来组成数据

```go
[]string{"0123456789abcdef0123456789abcdef","User_login"}
```
就可以用字符串替换的方法替换`$1`,`$0`
甚至可以生成这样的结果.

```
错误编号 1062: '登录名' 已被占用请换一个
```


