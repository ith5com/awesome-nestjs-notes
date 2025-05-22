# 拦截器

在 **NestJS** 中，**拦截器（Interceptor）** 是一种非常强大的工具，用于**扩展方法执行的行为**，比如：

- 请求前处理（如日志、缓存判断）
- 方法执行后处理（如响应数据包装、时间统计）
- 异常转换
- 延迟或重试机制

**拦截器是你在请求“经过 Controller”前后想“做点额外事”的最佳入口，适合统一数据封装、日志、缓存等中间逻辑处理。**

## 生命周期

1. 请求到达拦截器 →
2. 调用 `next.handle()` 继续执行 →
3. 控制器处理 →
4. 返回响应给拦截器 →
5. 拦截器可再加工或修改响应

## 实现拦截器

1. 需要装饰器`@Injecttable()`
2. 需要实现`NestInterceptor`类的`intercept`方法
   - `intercept`接收两个参数
     - 第一个参数`context`是`ExecutionContext`类型的上下文
     - 第二个参数`next` 是`CallHandler`，下的`handle`方法，可以使用它在拦截器中的某个点调用路由处理程序方法。（有效地封装了请求/响应流）

```js
import { Injectable, NextInterceptor, ExecutionContext, CallHandler } from '@nestjs/common'

@Injectable()

export class LoggerInterceptor implements NextInterceptor {
  intercept(context: ExecutionContext, next: CallHandler){
    return next
      .handle()
      .pipe(
        tap(() => console.log(`After... ${Date.now() - now}ms`)),
      );
  }
}
```

### 参数说明

| 操作符         | 作用描述                                                     |
| -------------- | ------------------------------------------------------------ |
| `map()`        | 映射返回值。常用于包一层统一格式，如 `{ success: true, data: result }` |
| `catchError()` | 捕获错误，转换为你想要的响应结构或重新抛出                   |
| `tap()`        | 类似中间过程打印调试或记录日志                               |
| `switchMap()`  | 如果你要基于上一个结果再返回另一个流                         |

## 绑定拦截器

**拦截器可以是控制器作用域的、方法作用域的或全局作用域的**

### 控制器绑定`@UseInterceptors`

```js
import {UseInterceptors} from '@nestjs/common';

@UseInterceptors(new LoggingInterceptor())
export class UsersController {}
```

### 全局拦截器`useGlobalInterceptors`

```js
const app = await NestFactory.create(AppModule);
app.useGlobalInterceptors(new LoggingInterceptor());
```

### 依赖注入拦截器`APP_INTERCEPTOR`

**当使用此方法对拦截器执行依赖注入时，请注意，无论采用此构造的模块如何，拦截器实际上都是全局的。**

```typescript
@Module({
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: LoggingInterceptor,
    },
  ],
})
export class AppModule {}
```

## 响应映射

在 **NestJS** 中，**响应映射（Response Mapping）**通常是指：

- 对控制器返回的数据进行格式包装、修改、过滤等处理，
- 让客户端拿到的数据是结构化、统一或经过加工的。

这种响应“加工”的过程，主要通过 **拦截器（Interceptor）** 来实现



**响应映射就是：把原始返回值转成你想返回给前端的格式。**

### 场景举例

> 1. 转换结构格式。

```js
// transform.interceptor.ts
//原始数据
/*
{ "id": 1, "username": "admin" }
*/

import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from '@nestjs/common';
import { map } from 'rxjs/operators';
import { Observable } from 'rxjs';

@Injectable()
export class TransformInterceptor implements NestInterceptor {
  intercept(
    context: ExecutionContext,
    next: CallHandler,
  ): Observable<any> {
    return next.handle().pipe(
      map((data) => ({
        statusCode: 200,
        message: '请求成功',
        data,
      })),
    );
  }
}

/*
{
  "statusCode": 200,
  "message": "操作成功",
  "data": {
    "id": 1,
    "username": "admin"
  }
}
*/
```

> 隐藏一些字段，比如返回用户信息，隐藏password

**正常处理，利用class-transformer**

```js
// 第一步：安装
npm install class-transformer class-validator

// 第二步：定义 DTO 并使用装饰器控制字段
// user.dto.ts
import { Exclude, Expose } from 'class-transformer';

export class UserDto {
  @Expose()
  username: string;

  @Exclude()
  password: string;

  constructor(partial: Partial<UserDto>) {
    Object.assign(this, partial);
  }
}

// 第三步：控制器
import { plainToInstance } from 'class-transformer';
import { UserDto } from './user.dto';

@Get('profile')
getUser() {
  const user = {
    id: 1,
    username: 'admin',
    email: 'admin@example.com',
    password: 'secret',
  };

  return plainToInstance(UserDto, user); // 自动排除 password
}
```

**第二种，利用拦截器，帮你处理**

```js
// serialize.interceptor.ts
import {
  CallHandler,
  ExecutionContext,
  Injectable,
  NestInterceptor,
} from '@nestjs/common';
import { plainToInstance } from 'class-transformer';
import { Observable, map } from 'rxjs';

@Injectable()
export class SerializeInterceptor implements NestInterceptor {
  constructor(private readonly dto: any) {}

  intercept(
    context: ExecutionContext,
    next: CallHandler,
  ): Observable<any> {
    return next.handle().pipe(
      map((data) => plainToInstance(this.dto, data, {
        excludeExtraneousValues: true, // 必须开启
      })),
    );
  }
}
```

## 异常映射

用 **拦截器统一格式**成功和失败返回数据；

用 **异常过滤器兜底**处理未捕获异常，输出统一格式。

这个功能，和异常过滤器差不多。我推荐使用异常过滤器。



## 覆盖流



在 **拦截器内部修改或替换从控制器返回的数据流（Stream）**，包括修改、拦截、延迟、缓存、格式化等行为。





























