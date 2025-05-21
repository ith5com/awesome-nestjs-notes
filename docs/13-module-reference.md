# 模块参考

它允许你在运行时动态地访问和解析依赖项（Provider 实例）。

`ModuleRef` 是 Nest 提供的一个类，用来 **手动获取已经注册到依赖注入容器中的实例**



## 应用场景

### 1. 动态解析服务

如果你不想通过构造函数注入依赖，而是根据条件在运行时决定使用哪个服务，可以用 `ModuleRef`。

```js
import { Injectable, ModuleRef } from '@nestjs/core';

@Injectable()
export class SomeService {
  constructor(private moduleRef: ModuleRef) {}

  async doSomething() {
    const userService = await this.moduleRef.resolve('UserService');
    userService.doWork();
  }
}
```

### 2. 解决循环依赖（延迟注入）

```js
@Injectable()
export class ServiceA {
  private serviceB: ServiceB;

  constructor(private moduleRef: ModuleRef) {}

  async onModuleInit() {
    this.serviceB = await this.moduleRef.resolve(ServiceB);
  }
}
```

这样就不会在构造函数里立即注入 `ServiceB`，从而避免了循环依赖问题。



### 3. 访问作用域为 REQUEST 的依赖

对于 `@Injectable({ scope: Scope.REQUEST })` 的服务，必须使用 `moduleRef.resolve()` 来按请求获取

```js
const myScopedService = await this.moduleRef.resolve(MyRequestScopedService, {
  strict: false,
});
```



### 4. 工厂模式：按需加载不同服务

比如说，一个登录的路由`/login`， 你传入的type是`wechat`或者`web` 取决于跑哪个登录的服务。传统的方法，是在控制器上进行type获取，然后调用不同的`servcie`



```js
@Injectable()
export class LoginService {
  constructor(private readonly moduleRef: ModuleRef) {}

  async login(type: 'password' | 'wechat', data: any) {
    let strategy: IAuthStrategy;

    if (type === 'password') {
      strategy = await this.moduleRef.resolve(UsernamePasswordAuthService);
    } else if (type === 'wechat') {
      strategy = await this.moduleRef.resolve(WeChatAuthService);
    } else {
      throw new Error('不支持的登录类型');
    }

    return strategy.validateUser(data);
  }
}
```

| 优势                         | 原因                     |
| ---------------------------- | ------------------------ |
| 支持运行时动态选择依赖       | 不同登录方式用不同类处理 |
| 解耦依赖关系                 | 不用提前注入全部服务     |
| 避免循环依赖或臃肿的构造函数 | 更易维护与扩展           |
