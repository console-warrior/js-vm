
# I 摘要

&ensp;&ensp;Serverless Function（函数计算）是一种云计算架构模型，开发者无需管理服务器即可部署代码，其核心是事件驱动和按需执行。当 HTTP 请求、文件上传等事件发生时，平台自动触发对应函数，动态分配资源，执行完毕后立即释放资源，实现“零闲置成本”。其核心技术依赖沙箱隔离环境（如 Node.js 的 vm 模块），通过vm.createContext()创建独立作用域，限制代码对敏感模块（如 fs、process）的访问，仅允许注入受控接口（如受限文件读取），确保安全性与动态执行能力的平衡。

&ensp;&ensp;Serverless 架构由 FaaS（函数即服务）和 BaaS（后端即服务）组成。FaaS 负责无状态计算，例如用户上传文件后触发函数进行安全扫描或数据处理；BaaS 提供数据库、缓存等后端服务，如 MySQL 存储业务数据（如商品表 goods）、Redis 管理高速缓存（如 Token）。这种分工使开发者专注业务逻辑，例如通过链式调用操作数据库（db("goods").update()），或使用 responseSuccess()/responseError()标准化响应。

&ensp;&ensp;Serverless 通过事件驱动、沙箱隔离和 BaaS 集成，平衡了弹性伸缩与安全运维，适用于各种场景（如 Web 应用、数据处理、小程序、秒杀活动），但需权衡 vm 虚拟机的冷启动与厂商绑定风险。

&ensp;&ensp;关键词：Node.js；Mysql；Serverless；云函数；Vue.js

# II 文档说明

详细文档说明：[Node.js Serverless Function 开发手册](https://www.kdocs.cn/l/cm8X283Zg8lw)

演示系统地址：[Node.js Serverless Function - 函数即服务](https://code.leixf.cn/)

# 1 认识Serverless

此处省略。更多请看上方在线文档说明

# 2 关键技术介绍

此处省略。更多请看上方在线文档说明

# 3 系统使用

## 3.1 系统地址

[https://code.leixf.cn/](https://code.leixf.cn)

## 3.2 登录系统

&ensp;&ensp;打开系统地址，在登录界面输入用户名、密码，如图3-1所示。

&ensp;&ensp;用户名：example

&ensp;&ensp;密码：123456
<p align="center">
<img width="584" height="294" alt="image" src="https://github.com/user-attachments/assets/26695e08-1517-44fc-ae8a-257b5f2be39e" />
</p>
<p align=center>图3-1  系统登录界面</p>

## 3.3 新增接口

&ensp;&ensp;在演示系统的「接口管理」界面，点击「新增」按钮。填写接口名称、选择模块名、选择接口路径，最后编写接口代码保存即可，如图3-2、3-3所示。
<p align="center">
<img width="243" height="136" alt="image" src="https://github.com/user-attachments/assets/496b9934-3621-4165-a4b0-cd7a8ca02d57" />
</p>
<p align="center">图3-2  新增接口按钮</p>

<p align="center">
<img width="586" height="556" alt="image" src="https://github.com/user-attachments/assets/5ab9e9ea-4e44-4238-a480-fec1c66d2908" />
</p>
<p align="center">图3-3  新增数据接口编码示例</p>

## 3.4 接口调试（1）

&ensp;&ensp;在浏览器上进行前端界面编码或使用API工具进行调试，图中的示例接口代码需要传入商品名称、商品价格、商品描述、上传logo图片，按照已编写代码正确传入，此处接口响应新增数据后的自增ID，如图3-4所示。

<p align="center">
<img width="586" height="474" alt="image" src="https://github.com/user-attachments/assets/122ed9e4-3458-4152-bfd4-4d7b1606efce" />
</p>
<p align="center">图3-4  接口调试新增传参示</p>

## 3.5 接口调试（2）

&ensp;&ensp;继续编写新的接口，用于验证此自增ID在商品数据表是否成功插入，将数据库查询结果作为接口响应，如图3-5、图3-6所示。
<p align="center">
<img width="552" height="236" alt="image" src="https://github.com/user-attachments/assets/b24fb83f-c499-4381-8cd2-7ae6b88cdbf5" />
</p>
<p align="center">图3-5  查询数据接口编码示例</p>

<p align="center">
<img width="586" height="473" alt="image" src="https://github.com/user-attachments/assets/f920b67c-1d4e-42d4-98a2-aae6f627b646" />
</p>
<p align="center">图3-6  接口调试查询传参示例</p>

## 3.6 示例代码及数据表结构

```javascript
const params = ctxRequest.body;

if (!params.name) {

  responseError("请输入商品名称");

  return false;

}

if (!params.price) {

  responseError("请输入商品价格");

  return false;

}

if (!params.description) {

  responseError("请输入商品描述");

  return false;

}

if (!ctxRequest?.files?.logoPic) {

  responseError('请选择要上传的文件');

  return;

}

const file = ctxRequest.files.logoPic;

const relativePath = await uploadFile(file);

const result = await db("goods").insert({

  name: params.name,

  price: params.price,

  description: params.description,

  logo: relativePath

});

if (result.length > 0) {

  responseSuccess(result);

} else {

  responseError('新增失败');

}
```

&ensp;&ensp;在演示系统可以对该表进行增删改查操作，需要更多数据表请联系作者创建。

表3-1  商品数据表（表名：goods）


| 字段名          | 类型           | 允许NULL | 默认值  | 键/约束 | 备注   |
| ------------ | ------------ | ------ | ---- | ---- | ---- |
| id           | int(11)      | NO     | -    | -    | 自增ID |
| name         | varchar(255) | NO     | -    | -    | 商品名称 |
| price        | FLOAT        | NO     | 1.00 | -    | 商品价格 |
| logo         | varchar(255) | NO     | -    | -    | 商品首图 |
| description  | longtext     | NO     | -    | -    | 商品描述 |
| create\_time | datetime     | NO     | 当前时间 | -    | 创建时间 |


# 4 编码示例

## 4.1 接口响应

&ensp;&ensp;统一封装成功/失败响应格式，responseSuccess 返回 200 状态码和结构化数据；responseError 返回错误信息与自定义状态码。

```javascript
/**
responseSuccess(响应数据, (可选)提示信息)
举例：const a = 3; const b = 9; const result = (a + b) * a; responseSuccess(result,"计算成功");
响应：{"code":200,"data":36,"msg":"计算成功"}
*/
/**
responseError(提示信息，响应状态码)
举例：responseError("参数有误");
响应：{"code":500,"data":null,"msg":"参数有误"}
*/
responseSuccess(响应数据, (可选)提示信息) // 处理成功或失败，二选一
responseError(提示信息，(可选)响应状态码) // 处理成功或失败，二选一
```

## 4.2 获取 GET 参数

&ensp;&ensp;从 URL 查询字符串中解析参数，通过 ctxRequest.query 获取参数对象，直接返回给前端或用于业务数据处理。

```javascript
// URL：/zhangsan/demo/request-get?id=99&name=张三
// 响应：{"code":200,"data":{"id":"99","name":"张三"},"msg":"操作成功"}
const query = ctxRequest.query;
responseSuccess(query);
```

## 4.3 获取 POST 参数

&ensp;&ensp;接收 JSON 格式的请求体数据，ctxRequest.body 获取请求体数据，适用于表单提交或 API 请求。

```javascript
/**

入参：

{ "id": 9, "name": "张三" }

响应：

{

    "code": 200,

    "data": {

        "id": 9,

        "name": "张三"

    },

    "msg": "操作成功"

}

*/
const params = ctxRequest.body;
responseSuccess(params);
```

## 4.4 查询数据库记录

&ensp;&ensp;从数据库表中读取多条记录，db("表名").select()执行 SQL 查询，返回结果数组。

```javascript
/**

响应：

{"code":200,"data":[{"id":1,"create_time":"2025-08-17 22:28:55","name":"包装盒","price":19.98,"logo":"uploads/2025/08/17/69925491-d5ee-4fa6-adf0-4bb73781ad2d.png","description":"这是一个描述。"}],"msg":"操作成功"}

*/

const result = await db("goods").select();
responseSuccess(result);
```

## 4.5 插入数据库记录

&ensp;&ensp;向数据库新增记录，insert 方法添加数据，通过 result 返回新记录 ID。

```javascript
/**

响应：

{"code":200,"data":2,"msg":"操作成功"}

*/

const result = await db('goods').insert([{ name: '包装盒', price: "16.80", description: "这是一个描述" }]);

if (result.length > 0) {

    responseSuccess(result[0]);

} else {

    responseError("插入失败");

}
```

## 4.6 删除数据库记录

&ensp;&ensp;按条件删除数据库记录，where().del()定位目标数据，返回影响行数。

```javascript
/**

响应：

{"code":200,"data":1,"msg":"操作成功"}

*/

const result = await db('goods').where('id', 3).del();

if (result) {

    responseSuccess(result);

} else {

    responseError("删除失败");

}
```

## 4.7 更新数据库记录

&ensp;&ensp;修改指定数据库记录，where().update()精准更新字段，返回修改行数。

```javascript
/**

响应：

{"code":200,"data":1,"msg":"操作成功"}

*/

const result = await db('goods').where('id', 1).update({

    name: '包装盒二号',

});

if (result) {

    responseSuccess(result);

} else {

    responseError("更新失败");

}
```

## 4.8 设置 Redis 缓存

&ensp;&ensp;存储键值对到 Redis 并设置过期时间，set(key,value,"EX",秒数)实现带时效的缓存。

```javascript
/**

响应：

{"code":200,"data":"OK","msg":"操作成功"}

*/

const result = await redis.set("token", "jfjawieofji3134", "EX", 6000);

responseSuccess(result);
```

## 4.9 获取 Redis 缓存

&ensp;&ensp;从 Redis 获取指定键的值，get(key)直接返回缓存数据，空值返回 null。

```javascript
/**

响应：

{"code":200,"data":"jfjawieofji3134","msg":"操作成功"}

*/

const token = await redis.get("token");

responseSuccess(token);
```

## 4.10 删除 Redis 缓存

&ensp;&ensp;清除 Redis 中的指定键，del(key)删除键值对，返回操作结果。

```javascript
/**

响应：

{"code":200,"data":1,"msg":"操作成功"}

*/

const result = await redis.del("token");

responseSuccess(result);
```

## 4.11 上传文件

&ensp;&ensp;接收前端提交的文件并存储到服务器，通过 ctxRequest.files 获取文件流，uploadFile()返回存储路径，调试过程如图 4-1 所示。

<p align="center">
<img width="552" height="407" alt="image" src="https://github.com/user-attachments/assets/cc675e23-3193-40ee-a2fb-57ba575f17ac" />
</p>
<p align=center>图4-1  上传文件传参</p>

```javascript
/**

前端界面：

<form action="http://localhost:3000/zhangsan/users/upload" method="post" enctype="multipart/form-data">

    <input type="file" name="file" multiple>

    <button type="submit">上传</button>

</form>

响应：

{

    "code": 200,

    "data": {

        "url": "/uploads/2025/08/17/3b4aa2b8-e6bb-46d8-bfca-6eb522930aba.webp"15
    },

    "msg": "操作成功"

}

*/

if (!ctxRequest?.files?.file) {

    responseError('请选择要上传的文件');

    return;

}

const file = ctxRequest.files.file;

const relativePath = await uploadFile(file);

responseSuccess({

    url: relativePath

});
```

## 4.12 浏览上传文件

&ensp;&ensp;通过 URL 直接访问已上传文件，拼接域名+存储路径（如 <http://域名/uploads/路径）实现直读，如图> 4-2 所示，协议://域名/上传成功响应URL。

&ensp;&ensp;示例图片地址：<https://code-api.leixf.cn/uploads/2025/08/18/9b85e244-67e8-46d8-9eb8-bdf0c58535d4.webp>

<p align="center">
<img width="507" height="344" alt="image" src="https://github.com/user-attachments/assets/2e4f903b-4b06-4ef6-b52f-1d82099d6f03" />
</p>
<p align=center>图4-2  浏览上传文件</p>

## 4.13 调用远端接口

&ensp;&ensp;向第三方服务发起 HTTP 请求并处理响应，axios.get()发起 GET 请求，过滤后通过 responseSuccess()返回有效数据。

```javascript
// 接口响应：

// {

//     "code": 200,

//     "data": {

//         "time": "2025-08-22T16:00",

//         "interval": 900,

//         "temperature": 28,

//         "windspeed": 2.6,

//         "winddirection": 146,

//         "is_day": 0,

//         "weathercode": 0

//     },

//     "msg": "操作成功"

// }


try {

    const url = "https://api.open-meteo.com/v1/forecast?latitude=23.1291&longitude=113.2644&current_weather=true";

    const res = await axios.get(url);

    if (res.data) {

        responseSuccess(res.data.current_weather);

    } else {

        responseError("未查询到结果");

    }

} catch (err) {

    responseError(err);

}
```

