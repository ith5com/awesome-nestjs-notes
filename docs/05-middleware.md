# 中间件

在 NestJS 中，中间件是一个**函数**，它在请求进入路由处理器（Controller）之前被调用，用于

- 日志记录

- 请求体修改

- IP 白名单检查

- 设置请求上下文（如 request.user）

可以访问 [request](https://express.nodejs.cn/en/4x/api.html#req) 和 [response](https://express.nodejs.cn/en/4x/api.html#res) 对象。

## 创建中间件

> 如果是类

1. 需要装饰器`@Injectable()`
2. 需要`implements NestMiddleware`

```js
@Injectable()

export class LoggerMiddleware implements NextMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log('Request...');
    next(); // 必须要调用next, 不然会被挂起
  }
}
```

> 如果是函数

```js
import { Request, Response, NextFunction } from 'express';

export function logger(req: Request, res: Response, next: NextFunction) {
  console.log(`Request...`);
  next();
};
```



## 使用中间件

### 模块中使用

由于`@Module()`没有使用中间件的方法，需要通过`NextModule`中的`configure`方法来实现

可以限制路由，甚至是请求方法。

```js
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
     .forRoutes({ path: 'users', method: RequestMethod.GET });
  }
}
```



### 路由通配符

同样需要通过`NextModule`中的`configure`方法来实现

```js
forRoutes({path: 'abcd/*', method: RequestMethod.ALL,})
```

### 全局中间件

> main.ts

```js
app.use(logger);
```

### 中间件消费者

`MiddlewareConsumer` 是一个辅助类。它提供了几种内置的方法来管理中间件

| 方法             | 用途                                         |
| ---------------- | -------------------------------------------- |
| `apply(...)`     | 应用一个或多个中间件；如果是多个，用逗号分隔 |
| `forRoutes(...)` | 设置作用的路由或控制器                       |
| `exclude(...)`   | 排除某些路径或方法                           |

> 应用中间件

```js
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
     .forRoutes({ path: 'users', method: RequestMethod.GET });
  }
}
```



> 排除某些路径或方法

```js
consumer
  .apply(LoggerMiddleware)
  .exclude(
    { path: 'cats', method: RequestMethod.GET },
    { path: 'cats', method: RequestMethod.POST },
    'cats/{*splat}',
  )
  .forRoutes(CatsController);
```

### 

## 注意事项

- 中间件**只在 HTTP 层生效**，对 WebSocket、GraphQL 无效

- 没有返回值，只能修改 `req`、`res`，然后 `next()` 继续流程

- **顺序执行**，你注册顺序很重要

- 通常用于“无状态逻辑”，不要在中间件里处理业务核心代码

## 中间件 vs 守卫 vs 拦截器

| 类型       | 执行时机               | 典型用途             |
| ---------- | ---------------------- | -------------------- |
| **中间件** | 控制器/路由之前        | 日志、请求加工       |
| **守卫**   | 控制器执行前           | 权限验证、角色判断   |
| **拦截器** | 控制器执行前后（环绕） | 响应包装、日志、缓存 |