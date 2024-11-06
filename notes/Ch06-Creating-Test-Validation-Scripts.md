# 第六章 Postman 测试脚本的创建

> **本章概要**
>
> - 检查 API 接口响应
> - 设置预请求脚本（pre-request scripts）
> - Postman 环境（environments）的用法

---

## 6.0 概述

如果某个测试脚本运行了很长时间都从没出现过故障，那么不排除脚本本身在写法上可能有问题，例如一些低级断言错误：`true === true`、`5 === 5` 等。

设计良好的测试套件也可能因为糟糕的断言脚本而功败垂成。

所谓 **断言（assertion）**，即测试中检查实际结果是否符合预期的校验逻辑。

`Postman` 的测试脚本是基于 `JavaScript` 实现的，并内置了 `Chai.js` 断言库，可以对请求的 Header 头信息及 body 正文进行断言校验。

本章介绍这些断言的基本写法与用法，并结合 Collection Runner 工具介绍在测试套件层面（即 Collection 集合）自动运行多个请求的具体操作。

相关资源链接：详见 [GitHub 第六章文件夹](https://github.com/PacktPublishing/API-Testing-and-Development-with-Postman-Second-Edition/tree/main/Chapter06)。



## 6.1 检查 API 接口响应

示例项目：电影《星球大战》API 接口（集合名称：`Star Wars API – Chapter 6`）。

相关 JSON 配置文件 `Star Wars API_Chapter6_initial.postman_collection.json` 详见百度网盘：

- 链接地址：`https://pan.baidu.com/s/1cbc8hFxzi2g08egDVU8XAw`
- 提取码：`yo47`

导入配置后，点击 Postman 右上角的 `Variables` 图标查看变量声明情况，会看到预先定义的基础 URL 变量 `base_url`：

![](assets/6.1.png)

**图 6.1 导入演示项目后看到的预定义变量 base_url**

然后将初始请求 `Get People` 的 URL 由 `https://swapi.dev/api/people/1` 替换为 `{{base_url}}/people/1` 就完成了环境搭建。

接着找到 `Get People` 下的 `Scripts` 标签，选中左边的 `Post-response` 选项卡，在脚本编辑区插入一段 Postman 内置的测试用例，例如验证响应码为 200 的示例代码：

```js
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});
```

实测执行情况如下（左下角表示断言已通过）：

![](assets/6.2.png)

**图 6.2 执行修改后的 Get People 请求得到的响应结果与测试结果截图**



### 6.1.1 关于 pm.test 方法

测试脚本中的 `pm.test` 方法，即与绝大多数测试框架中的 `test` 方法用法一致，其中的 `pm` 指代 Postman，是一个全局对象，用于访问 **请求** 或 **响应** 中的数据，或者访问 **变量**、**Cookie** 等。而 `test` 方法接收一个描述信息字符串和一个回调函数；前者用于说明测试用例的意图，后者为具体的测试校验逻辑。`pm.test()` 方法在 `Postman` 中的签名为：

```ts
test(testName: string, specFunction: Function): void
```

其中 `specFunction` 是一个异步函数，本质上是一个返回逻辑 `true` 或 `false` 的条件判定函数（也称 `predicate` 谓词函数）。虽然实测发现，令回调函数返回一个非逻辑值也不会报错，但最好还是按惯例返回逻辑值：

![](assets/6.3.png)

**图 6.3 实测回调函数返回值类型：返回一个非逻辑值也不报错，但最好返回逻辑值**



### 6.1.2 关于 Chai 断言

`Postman` 中的断言是基于断言库 `Chai.js` （`Chai` 读作 `/ʧaɪ/`，同英文单词 chai 的发音，表示“茶叶”）做的二次封装，`Chai.js` 是一款同时支持 `TDD` 风格和 `BDD` 风格的断言库，其名称来自一种印度茶，暗含提供一种友好易用的测试体验的意思。因此刚才的断言逻辑也可以用 `Chai.js` 重构为：

```js
const chai = require('chai');
pm.test("Status code is 200", function () {
    // pm.response.to.have.status(200);
    chai.expect(pm.response.code).equals(200);
});
```

实测结果：

![](assets/6.4.png)

**图 6.4 在 Postman 中使用 chai 断言重写校验逻辑**



## 6.2 检查响应的 body 部分

如果从 Postman 提供的示例脚本中选择 `Response body: Contains string`，则可以校验响应 body 中是否包含字符串，例如 `"skin_color"`：

```js
pm.test("Body matches string 'skin_color'", function () {
   pm.expect(pm.response.text()).to.include("skin_color");
});
```

这里的 `pm.response.text()` 返回字符串形式的响应 body 内容。

如果选择示例代码 `Response body:JSON value check`，则可以校验请求响应的 `JSON` 结果，得到如下模板：

```js
pm.test("Your test name", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.value).to.eql(100);
});
```

从第 2 行可见，Postman 的示例脚本还是用的 `ES5` 语法。这里也可以改为 `ES6` 的写法。关键是无需从零开始手动构建测试脚本。

此外，回调函数还支持 `console.log()` 控制台输出，方便调试测试脚本。控制台标签 `console` 位于主界面的左下角位置。

实测：Luke 的家园 URL 是否正确：

```js
pm.test("Check the homeworld URL", function () {
    const { homeworld } = pm.response.json();
    const expected = 'https://swapi.dev/api/planets/1/';
    pm.expect(homeworld).to.eql(expected);
});
```

运行结果（书中放到 GitHub 的参考文件是错的，期望值写的是 `http`，新版响应中为 `https`）：

![](assets/6.5.png)

**图 6.5 实测响应 body 为 JSON 数据时，校验 homeworld 字段的值是否正确**



## 6.3 检查请求头 Header 

Postman 示例代码：`Response header: Content-Type header check`

```js
pm.test("Content-Type is present", function () {
    pm.response.to.have.header("Content-Type");
});
```

这里和最开始校验响应码的写法很像。这也是 BDD 风格的优势：声明式语法，贴近自然语言。



## 6.4 Postman 常见的断言写法

### 6.4.1 常见自定义断言对象

`Postman` 在 `Chai.js` 的基础上内置了几个常用的自定义断言对象，例如 `pm.response.to.have.status` 中的 `.status`，以及检查 `Header` 时用到的 `pm.response.to.have.header` 中的 `.header`。

常用的自定义断言对象梳理如下：

|     自定义对象     |                           功能描述                           | 断言示例                                                     |
| :----------------: | :----------------------------------------------------------: | ------------------------------------------------------------ |
|   `.statusCode`    |                       校验响应码的取值                       | `pm.response.to.have.statusCode(200);`                       |
| `.statusCodeClass` | 校验响应码所属类别：2 表示以 2 开头的响应码，3 表示以 3 开头的响应码，以此类推 | `pm.response.to.have.statusCodeClass(2);`                    |
|  `.statusReason`   | 校验响应码描述信息，它是与响应码对应的固定文字描述信息，例如 `200` 对应 `OK` | `pm.response.to.have.statusReason('OK');`                    |
|     `.status`      |                   校验响应码或对应描述信息                   | `pm.response.to.have.status(200);`<br/>`pm.response.to.have.status("Created");` |
|     `.header`      |        校验请求头是否存在，或者请求头的键值对是否正确        | `pm.response.to.have.header("Content-Type");`<br/>`pm.response.to.have.header("Content-Type", "application/json;");` |
|    `.withBody`     |                    校验响应是否包含 body                     | `pm.response.to.be.withBody;`                                |
|      `.json`       |                 检查 body 是否为 `JSON` 格式                 | `pm.response.to.be.json;`                                    |
|      `.body`       | 1. 校验 body 是否存在<br/>2. 校验 body 文本是否完整匹配期望内容<br/>3. 校验 body 是否匹配某正则表达式<br/>4. 校验 body 是否与给定 JSON 匹配 | `pm.response.to.have.body;`<br/>`pm.response.to.have.body("JSON text");`<br/>`pm.response.to.have.body(<regex>);`<br/>`pm.response.to.have.body({key1:value1,...});` |
|    `.jsonBody`     | 1. 是否为 JSON 格式的响应 body<br/>2. 是否与给定 JSON 匹配 [^1] | `pm.response.to.have.jsonBody;`<br/>`pm.response.to.have.jsonBody({key: value});` |
|  `.responseTime`   | 1. 响应时间精确匹配给定毫秒时间<br/>2. 响应时间不少于给定毫秒时间<br/>3. 响应时间不超过给定毫秒时间<br/>4. 响应时间介于给定时间范围内（单位：ms） | `pm.response.to.have.responseTime(150);`<br/>`pm.response.to.have.responseTime.above(150);`<br/>`pm.response.to.have.responseTime.below(150);`<br/>`pm.response.to.have.responseTime.within(100,150);` |
|  `.responseSize`   | 1. 响应内容尺寸精确匹配给定字节大小<br/>2. 响应内容尺寸不少于给定字节大小<br/>3. 响应内容尺寸不超过给定字节大小<br/>4. 响应内容尺寸介于给定字节范围内（单位：字节） | `pm.response.to.have.responseSize(50);`<br/>`pm.response.to.have.responseSize.above(50);`<br/>`pm.response.to.have.responseSize.below(100);`<br/>`pm.response.to.have.responseSize.within(50,100);` |
|   `.jsonSchema`    |    校验响应的 JSON 格式的 body 是否与给定 schema 模式匹配    | `pm.response.to.have.jsonSchema(mySchema);`                  |

注意：原书 `.jsonBody` 中的第二个写法是错的，这里为了照顾下一节的实测环节，暂时保留错误写法。正确写法应该是：`pm.response.to.have.jsonBody(key, value);`。



> [!tip]
>
> **关于断言结果取反**
>
> 可以在上述断言中添加 `.not` 让断言逻辑取反。例如：
>
> ```js
> pm.response.to.not.have.jsonSchema(mySchema);
> // 或者
> pm.response.to.have.not.jsonSchema(mySchema);
> ```
>
> 只要加在助动词（`to`）或动词（`have`）后，取反逻辑就成立。



### 6.4.2 实测情况

（1）实测 `.body` 对象：

```js
pm.test('The response contains body', () => {
    pm.response.to.have.body;
    pm.response.to.have.body();
});

pm.test('The response body equals expected string', () => {
    const bodyText = pm.response.text();
    pm.response.to.have.body(bodyText);
});

pm.test('The response body matches some RegExp', () => {
    pm.response.to.have.body(/skin_color/i);
});

pm.test('The response body matches expected JSON object', () => {
    const json = pm.response.json();
    pm.response.to.have.body(json);
});
```

运行结果（匹配文本和 JSON 对象时 **必须完全匹配**）：

![](assets/6.6.png)

**图 6.6 实测自定义断言对象 .body 结果截图**



（2）实测 `.jsonBody` 对象：

```js
pm.test('The response has a JSON-format body', () => {
    pm.response.to.have.jsonBody;
    pm.response.to.have.jsonBody();
});

pm.test('The response JSON match some key-value pair', () => {
    // Error
    pm.response.to.have.jsonBody({
        'films[2]': 'https://swapi.dev/api/films/3/aaabbbccc'
    });
    // Correct
    pm.response.to.have.jsonBody(
        'films[2]', 'https://swapi.dev/api/films/3/'
    );
});
```

运行结果：

![](assets/6.8.png)

**图 6.7 实测自定义对象 jsonBody 的写法运行结果截图。书中所给的第二个写法是错的，参数不能为一个 JSON 对象**

> [!warning]
>
> **注意**
>
> 根据实测结果，校验响应 JSON 中是否包含给定的键值对，不能直接传入一个 JS 对象，而应该按 Postman 提示的方法签名，分别传入 `key` 和 `value` 的值作为参数。这里顺便分享一个小技巧，可以很方便的查看 Postman 方法的内置签名：书写脚本时，Postman 会自动提示当前对象的方法签名，遇到存在多个签名的情况，可以使用 <kbd>Alt</kbd> + <kbd>Up</kbd> 或 <kbd>Alt</kbd> + <kbd>Down</kbd> 进行翻页，查看隐藏的其他签名。例如 `pm.response.to.have.jsonBody()` 方法的签名就有四个，如下图所示：
>
> ![](assets/6.8.1.png)
>
> **图 6.8.1 pm.response.to.have.jsonBody() 方法存在四种签名**
>
> ```ts
> // Signature 1
> jsonBody(): any
> // Signature 2
> jsonBody(optionalExpectEqual: any): any
> // Signature 3
> jsonBody(optionalExpectPath: string): any
> // Signature 4
> jsonBody(optionalExpectPath: string, optionalValue: any): any
> ```
>
> 根据实测的上下文，这里显然应该使用第四种签名写法。



（3）实测 `.jsonSchema` 对象：

校验响应中的 JSON 结构时，旧版 `Postman` 使用 `tv4` 模块（`tv4` 全称 `Tiny Validator for JSON Schema version 4`）；新版中该写法已经废弃，推荐写法为上述列表中的写法，即改用 `Ajv` 依赖（全称 `Another JSON Schema Validator`）并作为 `pm.response` 下的内置对象，实测脚本如下：

```js
// fictional results from pm.response.json();
const json1 = {
    "args": {},
    "films": [],
    "files": {},
    "form": {
        "foo1": "bar1",
        "foo2": "bar2"
    },
    "headers": {},
    "json": null,
    "url": "https://postman-echo.com/post"
};

const schema = {
    type: 'object',
    properties: {
        films: {type: 'array'},
        url: {type: 'string'},
    },
    required: [
        'films', 'url'
    ]
};

const tv4 = require('tv4');
pm.test('Response is a valid JSON (via tv4)', function() {
    pm.expect(tv4.validate(json1, schema)).to.be.true;
});

const Ajv = require('ajv');
const ajv = new Ajv();
pm.test('Response is a valid JSON (via Ajv)', function() {
    pm.response.to.have.jsonSchema(schema);
    pm.expect(ajv.validate(schema, json1)).to.be.true;
});
```

运行结果：

![](assets/6.7.png)

**图 6.8 校验响应 JSON 的 schema 模式时，Postman 新旧两版写法的实测结果对比截图**



（4）本节给出的三个练手测试实战

（4.1）检查服务器是否为 nginx（该信息位于 Header 中）：

```js
pm.test('The server is nginx', () => {
    const server = pm.response.headers.get('Server');
    pm.expect(server).to.contain('nginx');
});
```

实测结果：

![](assets/6.10.png)

**图 6.9 实测从 Header 中读取信息、校验服务器是否为 nginx 的运行结果**



（4.2）检查请求响应时间是否在 500ms 内：

```js
pm.test('The response time for this call is less than 500ms', () => {
    pm.response.to.have.responseTime.below(500, 'responseTime too long');
});
```

实测结果：

![](assets/6.9.png)

**图 6.10 可根据 Chai.js 相关文档，结合 Postman 自动提示，自定义报错提示内容**



（4.3）检查 Luke 是否出演过当中的 4 部电影：

```js
pm.test('Luke appears in 4 films', () => {
    const { films } = pm.response.json();
    pm.expect(films).lengthOf(4);
});
/* "films": [
    "https://swapi.dev/api/films/1/",
    "https://swapi.dev/api/films/2/",
    "https://swapi.dev/api/films/3/",
    "https://swapi.dev/api/films/6/"
],*/
```

实测结果：

![](assets/6.11.png)

**图 6.11 根据 Postman 自动提示校验响应 JSON 中的某数组字段的长度情况**



---

[^1]: 原书这里的写法有误。根据最新版 `Postman`（**v11.18.2**）的实测结果，期望的键值对应该分别作为参数传入 `jsonBody()` 函数内，具体详见下一节第二个实测案例。



