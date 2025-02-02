# 第十一章 测试现有的 API 接口



> **本章概要**
>
> - 在 API 接口中查找 Bug 的方法
> - API 测试的自动化
> - 自动化 API 测试演示
> - Postman 测试集合的共享设置

尽管每天都有很多 API 接口问世，但实际上很多接口的测试工作都做得不够充分。

本章将围绕示例项目 `todo-list-testing` [^1] 演示 API 接口自动化的相关配置与测试实践，其中包括在测试脚本中调用其他请求、完成接口鉴权等操作，具有一定的参考价值。

---



## 1 在 API 接口中查找 Bug 的方法

拿到一套 API 接口，应该先探索该接口，并尝试找出当中的漏洞（Bug）。这样做肯定很难，但感觉困难的时候也往往是学东西的时候（*It’s OK to feel a bit frustrated and lost sometimes as that is often when the most learning is happening!*）。

为此，作者还建议大家尝试一下，能否只通过接口调用让服务端返回 500 报错（还真有点难度）。

查找 Bug 常用的几个思路：

- What kinds of inputs to the system might break things?
  对系统的哪些输入可能会造成破坏？
- What would happen if I tried to modify or change objects that don’t exist?
  如果我试图修改或更改不存在的对象，会发生什么情况？
- Are there any invalid or unexpected ways I could interact with this API?
  是否有任何无效或意想不到的方式可以与此应用程序接口交互？
- What are some unexpected things users of this application might do?
  该应用程序的用户可能会做哪些意想不到的事情？







---

[^1]: 详见 `GitHub` 仓库：https://github.com/djwester/todo-list-testing。本地启动该项目需要一定的 Python 基础和 Django 框架基础，我实测时完全按 REAME 操作，但还是运行失败了，最终只能显示接口文档页，其他页面均报 500 错误，待后期 Python 基础打牢后再尝试离线部署。



