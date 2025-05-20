# 守卫

在 NestJS 中，**守卫（Guard）是用于处理权限控制、访问限制、身份认证等逻辑**的一种机制。它是 NestJS 提供的生命周期钩子之一，主要作用是在进入控制器的路由处理器（Handler）之前进行判断。

简单来说：**守卫就是用于“是否允许访问这个路由”的判断逻辑**，类似“门卫”，决定谁能进去。



## 使用场景

- 用户是否登录（检查JWT）

- 用户是否有某个角色

- 是否允许访问某个资源

- 某些路由只允许某些设备或来源访问

  ...等等场景



## 实现守卫

1. 需要`@Injectable()`这个装饰器
2. 需要实现`CanActivate`这个类
3. 返回的是一个`boolean`

### AuthGuard守卫

```js
import {Injectable, CanActivate, ExecutionContext} from '@nestjs/common'

@Injectable()
export AuthGuard implements CanActivate {
  canActivate(contxt: ExecutionContext): boolean{
    const request = context.switchToHttp().getRequest();
    
     // 在这里写逻辑：例如检查 headers 中是否有 token
    return !!request.headers.authorization;
  }
}
```

### 基于Role守卫

```js
@Injectable()
export class RolesGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    const user = request.user;

    return user?.roles?.includes('admin');
  }
}
```

## 使用守卫`UseGuards`



### 路由级别使用 

```js
@UseGuards(AuthGuard)
@Get('profile')
getProfile() {
  return '你的个人信息';
}
```

### 控制器级别

```js
@UseGuards(AuthGuard)
@Controller('user')
export class UserController {}
```

### 全局守卫

```js
// main.ts
app.useGlobalGuards(new AuthGuard());
```

## 配合装饰器使用（推荐）

### 定义装饰器

```js
// roles.decorator.ts
import { SetMetadata } from '@nestjs/common';
export const Roles = (...roles: string[]) => SetMetadata('roles', roles);
```

### 守卫

```js
// roles.guard.ts
@Injectable()
export class RolesGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const roles = this.reflector.get<string[]>('roles', context.getHandler());
    const request = context.switchToHttp().getRequest();
    const user = request.user;

    return roles.some(role => user.roles.includes(role));
  }

  constructor(private reflector: Reflector) {}
}
```

## 守卫、拦截器、中间件的区别

| 项目       | 用于何时         | 是否能终止请求 | 是否能修改响应 |
| ---------- | ---------------- | -------------- | -------------- |
| **中间件** | 请求前（早期）   | ✅ 可以         | ❌ 不处理响应   |
| **守卫**   | 控制器方法执行前 | ✅ 可以         | ❌ 不处理响应   |
| **拦截器** | 控制器方法前后   | ❌ 不行         | ✅ 可以修改响应 |

**守卫用于“是否放行请求”判断，适合登录认证、权限控制等场景，是 NestJS 安全体系的第一道门。**