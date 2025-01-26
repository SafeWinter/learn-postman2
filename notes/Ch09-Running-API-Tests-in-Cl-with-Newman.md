# 第九章 在命令行用 Newman 实现接口测试

---

> **本章概要**
>
> - `Newman` 的用法
> - 理解 `Newman` 运行的配置参数
> - `Newman` 测试报告的配置
> - 将 `Newman` 集成到 CI/CD 工作流的方法

本章示例代码位置：`https://github.com/PacktPublishing/API-Testing-and-Development-with-Postman-Second-Edition/tree/master/Chapter09`。



## 9.1 Newman 的安装与启动

与在熟悉的 `Postman` 客户端做接口测试不同，`Newman` 可以在命令行环境轻松实现绝大部分接口测试。实测发现，在 `Newman` 中运行类似 `Collection Runner` 的集合测试并不会扣减每月免费额度。加之 `Newman` 在 `npm` 社区的功能不断完善，可以轻松高效地完成很多测试任务。

`Newman` 的安装只需一步（前提是先装好 `Node.js`）：

```bash
npm i -g newman
```

这里采用全局安装，主要是让 `newman` 能像 `npm` 那样直接在命令行中运行。



> [!tip]
>
> 全局安装的 `npm` 依赖项，其默认路径通常在系统盘 `C` 盘，如果后期无需使用，也可以考虑卸载：
>
> ```bash
> npm uninstall -g newman
> ```
>
> 确认是否删除成功，可以找到对应的安装路径：
>
> ```bash
> npm root -g
> ```
>
> 若要查看某个具体的全局依赖包的路径：
>
> ```bash
> npm list -g --depth=0
> ```



`Newman` 是通过导出的集合 `JSON` 文件进行测试的，命令格式为：

```shell
newman run path/to/your/collection/JSON/file.json
```

若该集合带有某个环境，则需导出该环境 `JSON` 文件，并用参数 `-e` 进行指定：

```markdown
newman run path/to/your/collection/JSON/file.json -e path/to/your/environment/JSON/file.json
```

若该集合还涉及上传 CSV 文件的数据驱动测试，则需要使用 `-d` 参数指定：

```markdown
newman run path/to/your/collection/file.json -e path/to/your/environment/file.json -d path/to/your/CSV/file.csv
```

此外，对于集合中缺失的一两个环境变量，也可以用 `--env-var` 参数手动定义，不必导出环境 `JSON` 文件：

```bash
newman run path/to/your/collection/file.json --env-var "param1=value1"
```

另外，如果缺了两个参数，则要写两次 `--env-var`：

```bash
newman run path/to/your/collection/file.json --env-var "param1=value1" --env-var "param2=value2"
```



### 实测1：带两个变量的测试集合

```bash
# URL: {{base_url}}/{{param}}?test=true
# collection: TestCollection.json
newman run TestCollection.json --env-var "base_url=https://postman-echo.com/" --env-var "param=get"

newman

Newman Test

→ Test GET
  GET https://postman-echo.com//get?test=true [404 Not Found, 247B, 1111ms]

┌─────────────────────────┬─────────────────────┬─────────────────────┐
│                         │            executed │              failed │
├─────────────────────────┼─────────────────────┼─────────────────────┤
│              iterations │                   1 │                   0 │
├─────────────────────────┼─────────────────────┼─────────────────────┤
│                requests │                   1 │                   0 │
├─────────────────────────┼─────────────────────┼─────────────────────┤
│            test-scripts │                   0 │                   0 │
├─────────────────────────┼─────────────────────┼─────────────────────┤
│      prerequest-scripts │                   0 │                   0 │
├─────────────────────────┼─────────────────────┼─────────────────────┤
│              assertions │                   0 │                   0 │
├─────────────────────────┴─────────────────────┴─────────────────────┤
│ total run duration: 1195ms                                          │
├─────────────────────────────────────────────────────────────────────┤
│ total data received: 0B (approx)                                    │
├─────────────────────────────────────────────────────────────────────┤
│ average response time: 1111ms [min: 1111ms, max: 1111ms, s.d.: 0µs] │
└─────────────────────────────────────────────────────────────────────┘
```



### 实测2：带环境 JSON 和 CSV 映射文件的测试集合

新建测试集合 `Newman Test`，并在该集合下新建测试 GET 请求 `Test GET`，`URL` 为：`{{base_url}}/get?{{queryParam}}={{queryParamVal}}`。然后导出 `JSON` 文件备用：`TestCollection.json`。

在新建测试环境 `Newman Test Env`，并定义变量 `base_url` 的值为 `https://postman-echo.com`。然后导出 `JSON` 文件备用：`NewmanTestEnvironment.json`。

执行下列 `PowerShell` 命令，在桌面上生成一个示例 `CSV` 文件 `DataDrivenInput.csv`：

```powershell
$csvPath = '~/Desktop/DataDrivenInput.csv'
$data = @(@{queryParam="test"; queryParamVal="true"},@{queryParam="query"; queryParamVal="1"})
$data | Export-Csv -Path $csvPath -NoTypeInformation
```

用 `newman` 进行带环境参数的数据驱动测试：

```bash
> (pwd).Path
C:\Users\ad\Desktop
> newman run TestCollection.json -e NewmanTestEnvironment.json -d DataDrivenInput.csv
newman
                                                           Newman Test

Iteration 1/2

→ Test GET
  GET https://postman-echo.com/get?test=true [200 OK, 873B, 2.4s]

Iteration 2/2

→ Test GET
  GET https://postman-echo.com/get?query=1 [200 OK, 979B, 254ms]

┌─────────────────────────┬─────────────────────┬─────────────────────┐
│                         │            executed │              failed │
├─────────────────────────┼─────────────────────┼─────────────────────┤
│              iterations │                   2 │                   0 │
├─────────────────────────┼─────────────────────┼─────────────────────┤
│                requests │                   2 │                   0 │
├─────────────────────────┼─────────────────────┼─────────────────────┤
│            test-scripts │                   0 │                   0 │
├─────────────────────────┼─────────────────────┼─────────────────────┤
│      prerequest-scripts │                   0 │                   0 │
├─────────────────────────┼─────────────────────┼─────────────────────┤
│              assertions │                   0 │                   0 │
├─────────────────────────┴─────────────────────┴─────────────────────┤
│ total run duration: 2.8s                                            │
├─────────────────────────────────────────────────────────────────────┤
│ total data received: 1.19kB (approx)                                │
├─────────────────────────────────────────────────────────────────────┤
│ average response time: 1331ms [min: 254ms, max: 2.4s, s.d.: 1077ms] │
└─────────────────────────────────────────────────────────────────────┘
```



## 9.2 Newman 的其他运行配置参数

查看 `newman` 内置帮助文档：

```bash
newman run -h
```

对应的官方 [GitHub](https://github.com/postmanlabs/newman#command-line-options) 文档位置：`https://github.com/postmanlabs/newman#command-line-options`

这里强调如下参数：

- 限定范围：
  - `--folder`：
- 延迟、超时相关：
  - `--delay-request`：
  - `--timeout`：
  - `--timeout-request`：
  - `--timeout-script`：
- 测试失败相关：
  - `--bail`：
- 协议相关：
  - `-k`：
- 重定向相关：
  - `--ignore-redirects`：



## 9.3 Newman 测试报表的配置

### 9.3.1 内置报表的配置


### 9.3.2 第三方报表的配置



### 9.3.3 自定义报表模块的配置





## 9.4 Newman 与 CI/CD 工作流的集成
