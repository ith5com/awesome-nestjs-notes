# Module



在`NestJS`中，万物都是模块化，每个模块都是一个容器，组织相关的 **控制器（Controllers）**、**服务（Providers）**、**导入的模块（Imports）**、**导出的服务（Exports）** 等。

模块分为`一般模块`和`动态模块`

模块之间可以通过 `imports` 和 `exports` 相互依赖和共享功能。

`Module`的装饰器是`@Module()`

`@Module`接收的参数

| 参数          | 说明           | 代码                                                         |
| ------------- | :------------- | :----------------------------------------------------------- |
| 参数名        | 类型           | 说明                                                         |
| `imports`     | `Module[]`     | 引入其他模块的数组（支持模块复用和依赖注入）,引入别的模块，拿到它们 `exports` 出来的提供者 |
| `controllers` | `Controller[]` | 当前模块中定义的控制器（负责接收请求）                       |
| `providers`   | `Provider[]`   | 当前模块中提供的服务（业务逻辑、数据库操作等）<br />服务：(通常是`@Injectable()`装饰器的文件) |
| `exports`     | `Provider[]`   | 向外暴露提供者，以供其它模块 `imports` 时使用                |

```js
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';

@Module({
  imports: [UserModule],            // 引入其他模块
  controllers: [AppController], // 当前模块提供的控制器（处理路由）
  providers: [AppService],      // 提供服务（业务逻辑层）
  exports: [AppService],            // 暴露服务给其他模块使用
})
export class AppModule {}
```

## ModuleRef 检索实例

`ModuleRef` 是 NestJS 提供的一个工具，**允许你在运行时手动获取或创建服务（Provider）的实例**。

> 平时注入服务(这是 **静态注入**（写死在构造函数里)

```js
constructor(private readonly userService: UserService) {}
```

Nest 提供 `ModuleRef` 类来导航内部提供器列表，并使用其注入令牌作为查找键获取对任何提供器的引用。









## 动态模块

在 NestJS 中，**动态模块（Dynamic Module）** 是一种高级模块设计模式，用于在运行时根据传入的配置动态创建模块内容（如服务、依赖、导出项等）。这是构建可复用、可配置库模块（如数据库、缓存、日志等）的核心能力。





其实就是：**动态模块允许你通过静态方法（如 `forRoot()`）传入参数，**动态生成模块结构和依赖注入内容**。**



```js

import { Module, DynamicModule, Global } from '@nestjs/common';

@Global() // 等价于在 DynamicModule 中写 `{ global: true }`
@Module({})
export class SharedModule {
  static forRoot(options: any): DynamicModule {
    return {
      module: SharedModule,
      providers: [{ /* ... */ }],
      exports: [/* ... */],
      global: true, // 全局模块，只要 import 一次就全局可用
    };
  }
}
```

> 常见用途

- 数据库模块（如 `TypeOrmModule.forRoot(config)`）
- 缓存模块（如 `CacheModule.register({ ttl: 5 })`）
- 国际化模块（根据语言配置不同加载项）
- 日志模块（传入 logLevel、输出格式等）

> 例子：自定义日志模块

**Logger.module.ts**

```js
import { Module, DynamicModule } from 'nestjs/common';
import { LoggerService } from './logger.service';

@Module({})
export class LoggerModule {
  static forRoot(prefix: string): DynamicModule {
    return {
      module: LoggerModule,
      providers: [ // 注入参数
        {
          provide: 'LOGGER_PREFIX',
          useValue: prefix,
        },
        LoggerService,
      ],
      exports: [LoggerService],
    };
  }
}

/*
{
  module: Type<any>;             // 当前模块类
  imports?: ModuleMetadata[];    // 可引入其他模块
  providers?: Provider[];        // 注册的服务
  exports?: Provider[];          // 对外暴露
  global?: boolean;              // 是否设为全局模块
}
*/
```

**Logger.service.ts**

```js
import { Inject, Injectable } from '@nestjs/common';

@Injectable()
export class LoggerService {
  constructor(@Inject('LOGGER_PREFIX') private prefix: string) {}

  log(message: string) {
    console.log(`[${this.prefix}] ${message}`);
  }
}
```

**使用**

```js
@Module({
  imports: [LoggerModule.forRoot('MyApp')],
})
```

