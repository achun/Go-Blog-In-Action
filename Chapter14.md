最简单的models实现
================

前章分析过控制器, models, meta验证器三者的关系. 这里讨论 models 的实现方式.

首先这里 models 只实现最简单的单表操作业务逻辑, 对于复杂逻辑还是单独写接口比较好.

TypePress 是通过定义 `CURD` 操作相关字段顺序来实现简单 models.

```go
// Users 表查询 可通过的 Fields, 注意次序是有优先级的
Users.Q.Fields = []string{"User_id", "User_login", "User_pass", "Site", "User_nicename", "User_status"}
// Users 表添加 可通过的字段值
Users.A.Fields = []string{"User_login", "User_pass", "User_nicename", "Site"}
// Users 表添加 必须要有的字段值, 这里和 Fields 完全一致
Users.A.Exists = Users.A.Fields
// Users 表更新 可设定的字段值
Users.U.Fields = []string{"User_pass", "User_nicename", "Site", "User_status"}
// Users 表更新 条件必须给定的字段值
Users.U.Exists = []string{"User_id"}
```

很明显 `Fields` 和 `Exists` 在不同的操作中所代表的含义是不同的.

基于简单的目标, models 中的代码都就是这些字段的定义. 执行代码的逻辑反而很简单这里就不贴了.
