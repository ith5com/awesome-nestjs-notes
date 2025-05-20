# Pipe管道

NestJS 的 **“管道（Pipes）”** 是一个非常核心的概念，广泛用于：

- **验证数据**
- **转换数据**
- **过滤/格式化输入**

它们在请求进入控制器处理器之前运行，就像是 HTTP 请求体的“守门员”。简单来说：**验证数据**



## 内置管道

| 管道               | 功能简介                         |
| ------------------ | -------------------------------- |
| `ValidationPipe`   | DTO 验证、字段过滤、类型转换     |
| `ParseIntPipe`     | 字符串转整数                     |
| `ParseFloatPipe`   | 字符串转浮点数                   |
| `ParseBoolPipe`    | 字符串转布尔值                   |
| `ParseEnumPipe`    | 验证值是否属于枚举               |
| `ParseUUIDPipe`    | 验证 UUID 字符串                 |
| `DefaultValuePipe` | 设置默认参数值                   |
| `ParseArrayPipe`   | 字符串转数组                     |
| `ParseFilePipe`    | 上传文件时验证类型、大小、格式等 |

### 1. `ValidationPipe`

用于验证 DTO 数据是否合法，最常用。

> 结合 `class-validator` 使用。

```js

@UsePipes(new ValidationPipe())
@Post()
create(@Body() dto: CreateUserDto) {}
```

选项包括：

- `whitelist: true`：**请求体中提交的字段**，如果 **在你的 DTO 中没有定义**，那么这些字段会被自动“丢弃”，不会传给你的业务逻辑。
- `forbidNonWhitelisted: true`：如果有多余字段直接抛异常
- `transform: true`：自动把字符串转成类型（比如 id: number）

### 2. `ParseIntPipe`

将参数从字符串转换为整数。

```js

@Get(':id')
findOne(@Param('id', ParseIntPipe) id: number) {}
```

非法输入（如 `abc`）会抛出 400 错误。

### 3. `ParseFloatPipe`

类似于 `ParseIntPipe`，用于转换为浮点数。

### 4. `ParseBoolPipe`

将字符串 `"true"` 或 `"false"` 转为布尔值。

```js
@Get()
get(@Query('active', ParseBoolPipe) active: boolean) {}
```

### 5. `ParseEnumPipe`

验证参数是否为枚举中的值。

```js
enum Role {
  User = 'user',
  Admin = 'admin',
}

@Get()
get(@Query('role', new ParseEnumPipe(Role)) role: Role) {}

```

### 6. `ParseUUIDPipe`

验证参数是否为合法 UUID（默认 v4）。

```js
@Get(':uuid')
get(@Param('uuid', ParseUUIDPipe) uuid: string) {}
```

### 7. `DefaultValuePipe`

给参数提供一个默认值。

```js
@Get()
find(@Query('page', new DefaultValuePipe(1)) page: number) {}
```

### 8. `ParseArrayPipe`

将 `a,b,c` 字符串转换为数组。默认用 `,` 分隔。

```js
@Get()
find(@Query('tags', new ParseArrayPipe({ items: String })) tags: string[]) {}
```

### 9. `ParseFilePipe`（上传文件相关）

结合文件上传验证字段类型（如文件大小、格式等）：

```js
@Post('upload')
@UseInterceptors(FileInterceptor('file'))
upload(
  @UploadedFile(
    new ParseFilePipe({
      validators: [new MaxFileSizeValidator({ maxSize: 100000 })],
    }),
  )
  file: Express.Multer.File,
) {}

```

## 自定义管道

虽然内置的管道有这么多，但是可能还会满足不了我们的业务需求。比如：**验证年龄是否为正数**

1. `@Injectable()`
2. 实现`PipeTransform`接口类
3. 实现`transform()`方法

```js
@Injectable()
export class PositiveNumberPipe implements PipeTransform {
   transform(value: any) {
     const num = Number(value);
     if (isNaN(num) || num <= 0) {
       throw new BadRequestException('必须是正数');
     }
     return num;
   }
}
```

> 使用

```js
@Get(':age')
getByAge(@Param('age', PositiveNumberPipe) age: number) {
  return `年龄：${age}`;
}
```

### 验证模式

> 基于模式

在 NestJS 中，“**基于模式的验证（Schema-based validation）**”是指：**不使用类和装饰器**，而是用**模式（Schema）定义方式**来验证请求数据。

你可能会问，为什么要用“基于模式的验证”？使用`class-validator`不也是很好吗？

- 使用 `class-validator` 虽然强大，但它依赖于 **类和装饰器**，不够灵活，对某些场景（如动态结构、函数式编程）不够友好。
- 基于模式（schema）的验证方式：
  - 不用创建类（class）
  - 支持动态结构更灵活
  - 与前端 React/Vue 使用的 Yup/Zod 更统一

> 使用 Joi 做模式验证的示例

1. 安装

```js
npm install joi --save
```

2. 定义 `Joi Schema`

```js
// user.schema.ts
import * as Joi from 'joi';

export const CreateUserSchema = Joi.object({
  username: Joi.string().alphanum().min(3).max(30).required(),
  email: Joi.string().email().required(),
  age: Joi.number().integer().min(0).max(100),
});

```

3. 在管道中使用 `Joi schema` 进行验证

```js
// joi-validation.pipe.ts
import { PipeTransform, Injectable, BadRequestException } from '@nestjs/common';
import { ObjectSchema } from 'joi'; // 引入joi的ObjectSchema属性

@Injectable()
export class JoiValidationPipe implements PipeTransform {
  // 引用
  constructor(private schema: ObjectSchema) {}

  transform(value: any) {
    // 验证
    const { error } = this.schema.validate(value);
    if (error) {
      throw new BadRequestException(`Validation failed: ${error.message}`);
    }
    return value;
  }
}

```

4. 在`controller`中使用

```js
// user.controller.ts
import { Controller, Post, Body, UsePipes } from '@nestjs/common';
import { JoiValidationPipe } from './joi-validation.pipe';
import { CreateUserSchema } from './user.schema';

@Controller('user')
export class UserController {
  @Post()
  @UsePipes(new JoiValidationPipe(CreateUserSchema))
  createUser(@Body() body) {
    return body;
  }
}
```

> 对象模式验证

对象模式验证（Object Schema Validation） **用一个对象（schema）定义结构 + 规则，然后校验传入的数据是否符合这个结构。**

在 NestJS 中，它通常通过中间件、自定义管道、或者 `ConfigModule` 来使用

```js
const schema = {
  username: Joi.string().required(),
  email: Joi.string().email(),
};
```



配置文件校验

```js
import * as Joi from 'joi';
import { ConfigModule } from '@nestjs/config';

@Module({
  imports: [
    ConfigModule.forRoot({
      validationSchema: Joi.object({
        PORT: Joi.number().default(3000),
        DB_HOST: Joi.string().required(),
      }),
    }),
  ],
})
export class AppModule {}
```

| 术语                                          | 含义                                                         |
| --------------------------------------------- | ------------------------------------------------------------ |
| **基于模式的验证（Schema-based Validation）** | 使用结构化“**验证模式**（schema）”来校验数据，而不是类和装饰器。是一种验证**范式**。 |
| **对象模式验证（Object Schema Validation）**  | 一种**实现方式**：将 schema 写成对象结构，对象中的字段就是规则（如 Joi/Zod）。 |

对比

| 项目               | 基于模式的验证                                   | 对象模式验证                            |
| ------------------ | ------------------------------------------------ | --------------------------------------- |
| 是什么             | 一种验证思路/范式                                | 一种具体写法（schema 是对象）           |
| 是否一定是对象结构 | ❌ 不一定。也可以是函数式 schema（Zod 支持）      | ✅ 通常是一个对象                        |
| 使用库             | Joi、Zod、Yup 等                                 | Joi、Zod、Yup 等                        |
| 形式举例           | `Joi.string().email()`、`Zod.object({...})`      | `{ username: Joi.string().required() }` |
| 出现位置           | 理论、架构角度（与 class-validator 区分）        | 编码实践层面，写出的 schema 结构        |
| NestJS 使用方式    | 通过自定义管道或 `ConfigModule` 使用 schema 验证 | 同上（但 schema 是对象）                |

## 绑定管道

### 绑定验证管道

```js
@Post()
@UsePipes(new JoiValidationPipe(createCatSchema))
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
```

### 类验证器

类验证器，其实就是`class-validator` 和 `class-transformer` 

1. 安装

```js
npm i --save class-validator class-transformer
```

2. 创建`DTO`文件

```js
import { IsString, IsInt } from 'class-validator';

export class CreateCatDto {
  @IsString()
  name: string;

  @IsInt()
  age: number;

  @IsString()
  breed: string;
}
```

3. 在控制器中使用

```js
@Post()
async create(
  @Body(new ValidationPipe()) createCatDto: CreateCatDto,
) {
  this.catsService.create(createCatDto);
}
```

4. 全局使用（推荐）

```js
// mian.ts
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true, // 剥离 DTO 中没有定义的字段
    forbidNonWhitelisted: true, // 遇到多余字段直接抛错
    transform: true, // 自动将 JSON 转为 DTO 实例
  }),
);
```



















