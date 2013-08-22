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

