# 装饰器

## 为什么要自定义路由装饰器？

- 简化重复代码（比如每个路由都带相同前缀或元数据）
- 统一路由定义风格
- 给路由绑定自定义元信息，方便拦截器/守卫读取



## 参数装饰器

在Nestjs中，已经有了内置的参数装饰器。

这里就不过多赘述了。可以看下`Controller.md`的文章部分。



## 自定义参数装饰器



为什么要做这样做？比如，在路由守卫上，如果解析出来了JWT, 需要把解析出来的`user`信息，挂载到`req`上面，这个时候，你需要通过`@Req() req`再去`req.user`获取。



但是如果有`@CurrentUser() currentUser`这种参数装饰器，你就可以更快的拿到参数信息。



> 封装`@CurrentUser()`装饰器

1. 导入`createParamDecorator`这个类
   - 参数传入回调函数

```js
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const CurrentUser = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return request.user;
  },
);
```

2. 使用

```js
@Get()
async findOne(@CurrentUser() currentUser: UserEntity) {
  console.log(user);
}
```



## 传递数据



当然，装饰器也能传递参数。

```js
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const User = createParamDecorator(
  (data: string, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    const user = request.user;

    return data ? user?.[data] : user;
  },
);
```

使用：

```js

@Get()
async findOne(@User('firstName') firstName: string) {
  console.log(`Hello ${firstName}`);
}
```





## 装饰器组成

在 **NestJS** 中，装饰器（Decorator）是构建框架最核心的机制之一，它们是用来为类、方法、参数或属性添加元数据的工具，从而让 Nest 能在运行时了解如何组织和处理这些对象。



### 基础结构

```js
function CustomDecorator(target: any, propertyKey?: string, descriptor?: PropertyDescriptor) {
  // 装饰器逻辑
}
```



### 分类

| 类型           | 示例                                | 用法说明                               |
| -------------- | ----------------------------------- | -------------------------------------- |
| **类装饰器**   | `@Controller()`, `@Injectable()`    | 作用于类，用于注册控制器、服务等       |
| **方法装饰器** | `@Get()`, `@Post()`, `@UseGuards()` | 作用于类方法，定义路由或中间逻辑       |
| **属性装饰器** | `@Inject()`, `@HostParam()`         | 装饰类的属性，用于依赖注入等           |
| **参数装饰器** | `@Body()`, `@Param()`, `@User()`    | 装饰方法参数，用于提取请求参数或注入值 |



### 自定义元数据装饰器

```js
Reflect.defineMetadata('role', 'admin', target);
const role = Reflect.getMetadata('role', target);
```



















