验证器
=====
对客户端提交过来的数据进行合法性验证, 是验证器要做的.
TypePress 确定完数据表结构后要写的一个 Go 代码就是验证器相关.
通常客户端提交的数据是字符串形式.
通常这些数据是和数据库表字段一一对应的.
验证的途径, 常见的有这两种.

tag
===
事实上我们总是用OOP的方法把数据表 `struct` 化.
TypePress 也这么做了 [meta][0].

```go
type Users struct {
	User_id       uint64    //自增唯一id
	User_login    string    //md5 登录名
	User_pass     string    //md5 密码
	User_nicename string    //显示名称
	User_date     time.Time //注册时间
	User_status   uint32    //用户状态
	Site          string    //博客子域名
}
```
`Users`对应的就是数据表`users`. tag 的方式这样做.

```go
type Users struct {
	// 省略其他字段
	User_nicename string `valid:"MinSize:5,MaxSize:50"`
}
```
当然要让这个生效需要其他的代码配合.

函数
====
函数的方法比较暴力, 就是定义一堆的验证函数等待你去调用.
```go
// UTF-8编码的字符串,只能通过遍历计算长度.
func MinSize(s string, l int) bool {
	l--
	for i, _ := range s {
		if i == l {
			return true
		}
	}
	return false
}
// 调用
if !MinSize("要验证的User_nicename", 5) {
	//验证失败的处理
	return
}
```

TypePress 正是采用这种暴力 [函数验证][1] 的方式, 并且你会看到更暴力生硬的代码. 那些代码肯定不是我一行行敲出来的, 编辑器的使用技巧而已. 写这些代码用不了几分钟.
TypePress 把验证器和数据表 `struct` 定义放到一起, 目录命名 `meta`.
你会看到所有的验证函数都有 `Is` 前缀, 加个统一的前缀, 只是为了在编辑器里面自动完成方便.

[0]: https://github.com/achun/typepress/blob/master/meta/meta.go
[1]: https://github.com/achun/typepress/blob/master/meta/validation.go
