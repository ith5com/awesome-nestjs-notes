# provider

`provider`是 Nest 中的核心概念。像`services`, `repositories`, `factories`这些都是`provider`。请记住，如果是有这个装饰器`@Injectable()`，就可以作为`provider`。 这些都是能够被**injected**的。



![img](https://nest.nodejs.cn/assets/Components_1.png)

## service服务



### 创建service服务

> 方法1：cli生成

```js
nest g service [service-name]
```

> 方法2： @Injectable() 装饰器

表示 `userService` 是可由 Nest [IoC](https://en.wikipedia.org/wiki/Inversion_of_control) 容器管理的类

```js
@Injectable()
export class UserService {
	list(){}
}
```

### 依赖注入Service

依赖注入是一种 [控制反转 (IoC)](https://en.wikipedia.org/wiki/Inversion_of_control) 技术，其中你将依赖的实例化委托给 IoC 容器（在我们的例子中是 NestJS 运行时系统），而不是在你自己的代码中强制执行。

不管是`controller`还是`service`都是一样的，依赖注入核心代码：

```js
constructor(private userService: UserService) {}
```

#### 情况1：没有被其他模块注入

**如果UserService没有被其他模块注入，直接provider注入即可。**

```js
// 第一步：user.module.ts中
@Module({
  provider:[UserService]
})


// 第二步：user.controller.ts中
import {controller, Get} from '@nestjs/common'

export class UserController {
  // 依赖注入
  constructor(private userService: UserService) {}
  
  @Get()
  function list(){
    this.userService.list()
  }
}

```

#### 情况2：被A模块注入

**如果`UserService`已经被A模块注入，需要在A模块中导出`UserService`, 在B模块中，导入A模块**

```js
// 第一步：在A模块
@Module({
  provider:[UserService],
  exports:[UserService]
})

// 第二步：在B模块
@Module({
  imports: [AModule]
})

// 第三步：依赖注入
import {controller, Get} from '@nestjs/common'

export class UserController {
  // 依赖注入
  constructor(private userService: UserService) {}
  
  @Get()
  function list(){
    this.userService.list()
  }
}

```

> 总结：记住下面3个步骤

1. 利用`@Injectable()`装饰器，将`类`声明为可以被 Nest IoC 容器管理的类。
2. 使用`constructor(private userService: UsersService)`注入controller/service中。
   1. 第一种情况：这个IOC没有被其他模块`provider`引入过，可以直接在该模块下`providers`引入
   2. 第二种情况：这个IOC已经被A模块`provider`引入过，则需要在 **A模块**中`exports`这个类`UserService`；再到需要注入的模块中，用`imports`引入**A模块**
3. 可以在**controller**或者**service**中，注入了：`constructor(private userService: UsersService)`





## 自定义provider

### 标准provider(useClass)

如果你能够看懂上面的代码，就没问题了。其实就是标准的Provider

请仔细观察下面的代码

```js
@Module({
  providers: [UsersService],
})

```

展开后，如下，上方是下方代码的缩写。

我们清楚地将**token** `UsersService` 与**class** `UsersService` 相关联

```js
providers: [
  {
    provide: UsersService,
    useClass: UsersService,
  },
];
```

> 根据不同的环境变量加载不同的服务

```js
const configServiceProvider = {
  provide: ConfigService,
  useClass:
    process.env.NODE_ENV === 'development'
      ? DevelopmentConfigService
      : ProductionConfigService,
};

@Module({
  providers: [configServiceProvider],
})
export class AppModule {}

// 注入
 constructor(private readonly configService: ConfigService) {}
```



### 值 providers(useValue)

这个`value providers` 就是说，直接注入静态值的提供（数字、字符串、对象等）。不像 `useClass`/`useFactory` 会创建类或调用函数，它是**静态值**, 

> 定义提供者

```js
@Module({
  providers:[
    {
      provider:'API_KEY'
      useValue:'111-222-333-444-555'
    }
  ]
})

```

> 依赖注入, 在service中/controller中

```js
consturtor(
private readonly @Inject('API_KEY') private apiKey: string
){}

 getApiKdy() {
    return this.apiKey;
  }

```

### 非基于类的提供器令牌

简单来说：就是你注入依赖时，用的不是某个类名，而是**字符串、symbol、或其他值**作为“标识”。

> 常见的

```js
@Injectable()
export class MyService {}

@Module({
  providers: [
    {
      provider:MyService,
      useClass:MyService
    }
  ],
})
export class AppModule {}
```

其中：`provider`的值，每个 Provider（提供器）都有一个“名字”，我们称它为 **令牌（token）**，现在你要牢记这个令牌概念。

正如标题说的，非基于类提供的令牌，可以是：

- 一个字符串

- 一个 `Symbol`

- 一个 `InjectionToken`（高级写法）

> 这个时候，就是也用到**useValue**

```js
import { connection } from './connection';

@Module({
  providers: [
    {
      provide: 'CONNECTION',
      useValue: connection,
    },
  ],
})
export class AppModule {}


// 依赖注入
//'CONNECTION' 自定义提供程序使用字符串值标记

constrctor(@Inject('CONNECTION') connection:Connection){}
```

### 工厂Providers(useFactory)

`useFactory` 语法允许动态创建提供程序。**工厂函数可以做逻辑判断、读取配置、依赖其他服务等等**。

```js
import { ConfigService } from './config.service';

export const TokenProvider = {
  provide: 'TOKEN',
  useFactory: (configService: ConfigService) => {
    const secret = configService.get('TOKEN_SECRET');
    return `Bearer ${secret}`;
  },
  inject: [ConfigService],
};

constructor(@Inject('TOKEN') private token: any) {
  console.log(config.appName);
}
```

| 使用场景               | 举例                             |
| ---------------------- | -------------------------------- |
| 读取环境变量、动态配置 | 根据环境生成不同配置             |
| 创建第三方客户端实例   | 比如创建数据库连接、Redis 客户端 |
| 依赖其他服务构建资源   | 注入服务到工厂中创建依赖         |
| 运行时逻辑决定实现     | 不同运行时条件返回不同对象       |

### 别名Providers（useExisting）

这个提供器不要新建实例了，直接用另一个已有提供器的实例。

简单来说：“我不再单独创建一个实例，我只是用别名指向已有的那个实例”。



```js
{
  provide: AliasService,
  useExisting: RealService,
}
```

### 非基于Service的Provider

> **不是通过类，而是通过别的方式（如 `useValue`、`useFactory`、`useExisting` 等）来提供依赖的方式。**

我们之前一直都是通过`@Injectable（）`这个装饰器来，定义是可以被注入的。

非基于服务的提供商 = 不是用类，而是用值、函数、引用 等方式提供依赖



> 非基于服务的提供商（值、函数、工厂）

#### `useValue`

```js

const Config = {
  appName: 'MyApp',
  version: '1.0.0',
};

{
  provide: 'CONFIG',
  useValue: Config,
}
```

####  `useFactory`

```js

{
  provide: 'RANDOM_NUMBER',
  useFactory: () => Math.random(),
}
```

####  `useExisting`

```js

{
  provide: 'LOGGER',
  useExisting: LoggerService,
}
```

这些提供者不是类，也没有 `@Injectable()` 修饰，因此它们统称为：**非基于类 / 服务的提供者**



## 导出定制提供器

> 可以通过`令牌`导出

```js
const connectionFactory = {
  provide: 'CONNECTION',
  useFactory: (optionsProvider: OptionsProvider) => {
    const options = optionsProvider.get();
    return new DatabaseConnection(options);
  },
  inject: [OptionsProvider],
};

@Module({
  providers: [connectionFactory],
  exports: ['CONNECTION'],
})
export class AppModule {}
```

> 可以整个导出

```js
const connectionFactory = {
  provide: 'CONNECTION',
  useFactory: (optionsProvider: OptionsProvider) => {
    const options = optionsProvider.get();
    return new DatabaseConnection(options);
  },
  inject: [OptionsProvider],
};

@Module({
  providers: [connectionFactory],
  exports: [connectionFactory],
})
export class AppModule {}
```

## 异步Provider



普通的提供器（Provider）通常同步实例化，比如直接用 `useClass` 或 `useValue`，但是有些场景下，我们需要异步完成初始化步骤后才得到实例，比如：

- 从数据库或远程接口加载配置
- 异步连接数据库或缓存
- 需要等待某些异步依赖准备好

这时就要用异步提供器，用 `useFactory` 并返回一个 `Promise`。

### 例子

```js
@Module({
  providers: [
    {
      provide: 'ASYNC_CONFIG',
      useFactory: async () => {
        // 模拟异步获取配置，比如从远程API或文件
        const config = await fetchConfigFromRemote();
        return config;
      },
    },
  ],
  exports: ['ASYNC_CONFIG'],
})
export class ConfigModule {}

```

### 使用场景

- 读取异步配置
- 连接数据库（TypeORM、Mongoose 等模块通常就是异步配置）
- 初始化第三方 SDK

### 异步模块加载

```js
@Module({})
export class DatabaseModule {
  static forRootAsync(options: AsyncOptions): DynamicModule {
    return {
      module: DatabaseModule,
      imports: options.imports || [],
      providers: [
        {
          provide: 'DATABASE_CONNECTION',
          useFactory: async () => {
            const connection = await createDatabaseConnection();
            return connection;
          },
          inject: options.inject || [],
        },
      ],
      exports: ['DATABASE_CONNECTION'],
    };
  }
}

```





























