





# Controller

在`Nestjs`中，**controller**是一个处理路由的文件。通常由`@Controller()`装饰器。

```js

@Controller('user') // 表示控制器的路由， http://localhost:3000/user 访问
```

## RESTful API

在了解控制器`Controller`之前，需要了解 **RESTful API**

是一种 **标准化的接口设计思想**，主要用于客户端和服务器之间的数据通信。

> 核心原则

| 原则                 | 含义                                                     |
| -------------------- | -------------------------------------------------------- |
| **资源（Resource）** | 用 URL 表示的实体，比如 `/users/1` 表示 id 为 1 的用户。 |
| **使用 HTTP 方法**   | 不同操作用不同 HTTP 动作：GET、POST、PUT、DELETE 等。    |
| **无状态性**         | 每次请求都包含所有必要信息，服务器不记录客户端状态。     |
| **统一接口**         | 所有 API 风格一致、接口清晰、易于理解。                  |

> 常用 HTTP 方法及含义（以 `/users` 资源为例）

| 动作         | HTTP 方法 | 路径       | 说明                |
| ------------ | --------- | ---------- | ------------------- |
| 获取所有用户 | GET       | `/users`   | 获取用户列表        |
| 获取单个用户 | GET       | `/users/1` | 获取 id 为 1 的用户 |
| 创建用户     | POST      | `/users`   | 新增一个用户        |
| 更新用户     | PUT       | `/users/1` | 更新 id 为 1 的用户 |
| 删除用户     | DELETE    | `/users/1` | 删除 id 为 1 的用户 |

## 控制器



一句话：**控制器的目的是处理请求，一个控制器可以有多个路由，每个路由去执行不同的操作，它负责 处理客户端请求并返回响应**。

说白了，就是路由，获取参数，执行方法。

### 创建

> 方法1：通过cli 创建

```js
 nest g controller [name] [--no-spec] // 如果加上--no-spec参数，则不会
```

> 方法2：手动创建

```js
// 创建控制器
import { controller,Get } from '@nestjs/common'

@Controller('user')
function userController(){
  Get()
  function getList(){
    return {}
  }
}

export userController;


// 在module中引入
@Module({
  imports:[userController]
})
```



### 控制器装饰器

| 装饰器          | 说明                   |
| --------------- | ---------------------- |
| `@Controller()` | 声明控制器类和路由前缀 |

### 路由

#### 路由装饰器

| 装饰器      | 说明                  |
| ----------- | --------------------- |
| `@Get()`    | 处理 HTTP GET 请求    |
| `@Post()`   | 处理 HTTP POST 请求   |
| `@Put()`    | 处理 HTTP PUT 请求    |
| `@Delete()` | 处理 HTTP DELETE 请求 |

```js
import { Controller, Get } from '@nestjs/common';

@Controller('cats')
export class CatsController {
  @Get('user')
  findAll(): string {
    return 'This action returns all cats';
  }
}
// 请求路由就是：cats/user
// 后面的3个以此类推
```

### 路由对象

#### 路由对象装饰器

| 参数                      | 说明                                               |
| ------------------------- | -------------------------------------------------- |
| `@Req()` / `@Res()`       | 获取原始的 request/response 对象（不推荐直接使用） |
| `@Session()`              | `req.session`                                      |
| `@Headers(name?: string)` | `req.headers` / `req.headers[name]`                |
| `@Ip()`                   | `req.ip`                                           |
| `@HostParam()`            | `req.hosts`                                        |



### 路由参数

#### 参数装饰器

| 装饰器     | 说明                                             |
| ---------- | ------------------------------------------------ |
| `@Param()` | 获取路径参数，例如 `/users/:id` 中的 `id`        |
| `@Query()` | 获取查询参数，例如 `/users?name=Tom` 中的 `name` |
| `@Body()`  | 获取请求体，用于 POST/PUT 等请求                 |

```js
```







### 路由通配符

**允许你使用通配的 URL 路径规则来匹配多个类似请求，非常适合处理动态或批量路由。**

| 通配符          | 含义                               | 示例匹配                              |
| --------------- | ---------------------------------- | ------------------------------------- |
| `*`             | 匹配任意字符（0 个或多个）         | `/ab*cd` 可匹配 `/abcd`、`/ab123cd`   |
| `?`             | 匹配一个字符                       | `/ab?c` 可匹配 `/abc` 或 `/acc`       |
| `()`            | 匹配可选部分                       | `/ab(cd)?e` 可匹配 `/abe` 或 `/abcde` |
| `:`（路径参数） | 匹配动态段落（不是通配符，但常用） | `/users/:id` 可匹配 `/users/123`      |

### 路由响应

| 参数                                     | 说明                       |
| ---------------------------------------- | -------------------------- |
| @HttpCode()                              | 状态码：响应的默认状态代码 |
| @Header('Cache-Control', 'no-store')     | 指定自定义响应标头         |
| @Redirect('https://nest.nodejs.cn', 301) | 重定向                     |
|                                          |                            |

### 子域路由





### 状态共享





## 请求参数校验

### DTO

DTO 就是用来定义接口数据结构的对象，配合类型系统和验证器，能让你的 API 更加安全、清晰、易维护。

**DTO 全称是：Data Transfer Object（数据传输对象）**

它是一个**定义数据结构的类（或对象）**，专门用于：

- **接收客户端传来的数据**
- **传递给 Service、数据库、验证逻辑等处理**

> 为什么要用DTO

1. **明确字段结构**：定义清楚每个接口应该接收哪些字段（比如 username、password）
2. **类型安全**：配合 TypeScript，字段类型清晰，自动补全，避免出错
3. **数据验证**：配合类验证器（`class-validator`）使用，可以自动校验数据合法性
4. **统一管理**：大型项目中便于维护、复用

> 常见配套工具

| 工具名              | 作用                             |
| ------------------- | -------------------------------- |
| `class-validator`   | 提供 @IsEmail、@Length 等验证器  |
| `class-transformer` | 将 JSON 转换为类实例（DTO 使用） |

```js
import { IsString, IsEmail, Length } from 'class-validator';

export class CreateUserDto {
  @IsString()
  @Length(3, 20)
  username: string;

  @IsEmail()
  email: string;

  @IsString()
  @Length(6, 20)
  password: string;
}
```

### 管道



















