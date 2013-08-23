应用程序框架
===========
经过前几章实践 TypePress 确定了开发方向: 应用模板. 查询了一些资料这种代码组织形式早有名称

application framework

应用程序框架. 好吧, 框架这个词遍地开花了.

使用者拷贝 TypePress 后, 应该可以通过简单的方法就可以迅速搭建自己的应用框架(花儿朵朵开).

`main.go` 已经 package 化了. 现在轮到调整控制器了. 先看看调整后的目录结构


    src
    ├───blog // 既然 typepress 个博客系统, 建立目录名就叫 blog, 其实就是 blog 控制器 base 目录
    │   ├───install // controllers 下的 install 移动到此目录下
    │   ├───root    // controllers 下的 root 移动到此目录下
    │   └───sign    // 新建 signin, singup, singout 到此目录下
    ├───controllers // 整理后的 controllers 只剩下最基本的控制器处理流程
    ├───global
    ├───meta
    ├───models
    └───typepress // 当然其中的 main.go 要`import _ "blog"`了
        ├───conf
        ├───log
        ├───root
        │   ├───css
        │   └───js
        └───views
            └───typepress.org // 如前章所述目录更名
                ├───install
                └───root

您应该看明白了, 如果您使用 TypePress, 这种目录结构可以很方便的把 TypePress 改造成另外的系统.

建立您自己的目录, main.go  中 `import` 相应的 package 即可.

现实中作者发现, 每分离一个 package, 编译后的二进制文件大概会增大2K字节左右. 这不是问题, 您的实现要分多少 package 您说了算.

如果您下载过以前的代码, 目录调整后再次 `go build` 可能需要删除以前的 `.a` 文件才能顺利通过.

使用可以参考 [TypePress使用方法][1].

被GoGet教育了一把
================

开发过程中, 作者把 TypePress 介绍给其他开发者, 希望能得到一些反馈.
得到的第一个反馈竟然是无法编译通过. 用 `go get` 得到的 TypePress 引入的 package 和作者开发环境的版本不一致.
于是我删除了自己机器上的相关 package, `go get` 整个过程, 结果和反馈问题的一样, 版本确实不一致.

悲催的事情开始了.

作者是个 GIT 新手, 以为自己 GIT 操作问题引起的, 这样绕来绕去弄了好几个小时. 最后只留下 `master` 一个分支, 其他都删除了问题才得以解决.

第二天睡醒, 开始怀疑是否是 `go get` 的使用问题造成的, 于是在官方 [Download_and_install_packages_and_dependencies][0] 看到这个

```
When checking out or updating a package,
get looks for a branch or tag that matches the locally installed version of Go.
The most important rule is that if the local installation is running version "go1",
get searches for a branch or tag named "go1".
If no such version exists it retrieves the most recent version of the package. 
```
原来 `go get` 会对分支的名称进行匹配, 优先 `checkout` 与`Go`版本匹配的分支, 比如`go1`, 如果匹配不到才会采用最新的分支. 那些出问题的 package 无一例外都有 `go1` 这样的分支.
用分支名称进行版本匹配这方式很好. 是最简单的 `release` 自动匹配.

so

文档要仔细看

[0]: http://golang.org/cmd/go/#hdr-Download_and_install_packages_and_dependencies
[1]: https://github.com/achun/typepress/#%E4%BD%BF%E7%94%A8