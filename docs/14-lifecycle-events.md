# 生命周期事件

在 **NestJS** 中，生命周期事件是指框架在不同阶段自动调用的 **钩子函数（hooks）**，它们让我们有机会在模块、服务等对象的创建、初始化、销毁过程中插入自定义逻辑。



1. 生命周期钩子是 NestJS 类的“生命周期事件”
2. 常用于初始化连接、销毁资源、注册服务、清理任务等
3. 钩子可用于任何 Provider，包括 Service、Controller、Guard 等
4. 部分钩子需显式启用（如关闭事件）

## 常见的生命周期钩子

| 钩子方法                            | 触发时机                                      |
| ----------------------------------- | --------------------------------------------- |
| `onModuleInit()`                    | 当前类被依赖注入系统初始化之后                |
| `onApplicationBootstrap()`          | 所有模块都已初始化完毕                        |
| `afterModuleInit()`（较少用）       | `onModuleInit()` 之后立即调用（次序更晚）     |
| `onModuleDestroy()`                 | 当前模块即将被销毁时                          |
| `onApplicationShutdown(signal)`     | 应用程序关闭前触发（如收到 `SIGINT` 信号）    |
| `beforeApplicationShutdown(signal)` | 应用关闭前（比 `onApplicationShutdown` 更早） |
| `enableShutdownHooks()`             | 显式启用关闭钩子（默认关闭）                  |

### 示例：服务生命周期钩子

```js
import {
  Injectable,
  OnModuleInit,
  OnModuleDestroy,
  OnApplicationShutdown,
} from '@nestjs/common';

@Injectable()
export class MyService implements OnModuleInit, OnModuleDestroy, OnApplicationShutdown
{
  onModuleInit() {
    console.log('🚀 模块初始化：onModuleInit');
  }

  onModuleDestroy() {
    console.log('🧨 模块销毁：onModuleDestroy');
  }

  onApplicationShutdown(signal: string) {
    console.log(`🛑 应用关闭：onApplicationShutdown (${signal})`);
  }
}

```

## 应用关闭相关钩子

如果你希望在应用退出前执行清理逻辑（如关闭数据库、释放资源），需要：

1. 调用 `app.enableShutdownHooks()` 启用钩子
2. 实现 `onApplicationShutdown()` 或 `beforeApplicationShutdown()`

```js
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.enableShutdownHooks();
  await app.listen(3000);
}
```

## 生命周期在模块中的使用

```js
import { OnApplicationBootstrap, Injectable } from '@nestjs/common';

@Injectable()
export class InitService implements OnApplicationBootstrap {
  onApplicationBootstrap() {
    console.log('✅ 应用初始化完毕，可以开始业务了');
  }
}
```

| 场景                         | 钩子推荐                    |
| ---------------------------- | --------------------------- |
| 初始化连接池、任务调度等     | `onModuleInit`              |
| 所有模块加载完后通知业务系统 | `onApplicationBootstrap`    |
| 关闭数据库、清除缓存         | `onApplicationShutdown`     |
| 监听退出信号做清理           | `beforeApplicationShutdown` |