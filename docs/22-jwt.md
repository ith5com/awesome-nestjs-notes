# jwt

## 配置

### 安装

```js
yarn add @nestjs/jwt @nestjs/passport passport-jwt passport
```

### 策略选择

`passport-jwt`这个策略

#### PassportStrategy是什么

`Passport.js` 是 Node.js 中最流行的用户认证中间件，常与 Express 一起使用。它支持各种认证方式，如：

- 本地用户名/密码认证（LocalStrategy）
- OAuth 第三方认证（如 Google、Facebook、GitHub）
- 使用 JWT 的 Token 认证
- 自定义策略

`PassportStrategy` 是一个基础策略类，用于创建自定义的认证策略。

每种认证方式（如 Local, JWT, Google）都是一个继承该类的策略实现。

在 NestJS 中，通过 `@nestjs/passport` 提供的 `PassportStrategy` 类继承封装策略更加方便。

#### 在Nestjs中应用

```js
import { Strategy } from 'passport-jwt';
import { PassportStrategy } from '@nestjs/passport';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      secretOrKey: 'yourSecretKey',
    });
  }

  async validate(payload: any) {
    return { userId: payload.sub, username: payload.username };
  }
}
```

### passport-jwt策略

#### 流程

1. 用户请求 API 时，在 `Authorization` header 中携带 JWT。
2. 后端使用 `passport-jwt` 解密并验证这个 token。
3. 如果有效，就认为这个请求已经认证通过。



### 配置策略

```js
import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { PassportStrategy } from '@nestjs/passport';
import { ExtractJwt, Strategy } from 'passport-jwt';

@Injectable()
export class SystemJwtStrategy extends PassportStrategy(
  Strategy,
  'system-jwt', // 策略别名，用于定义守卫继承需要用，因为后期，可能还有小程序、网站前端登录，所以需要和系统的区分
) {
  constructor(private configService: ConfigService) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      secretOrKey: configService.get('JWT_SECRET'), // jwt密钥
    });
  }

  // 校验
  validate(payload: { userId: number }) {
    // 通常这里是需要查询数据库，返回用户信息，再return， 就会挂在路由的req上面。
    return { userId: payload.userId };
  }
}

```

### 配置JWT

#### 定义isPublic装饰器

如果当前的路由不需要j w t的话，可以给他定义一个装饰器`@isPublic`

```js
import { SetMetadata } from '@nestjs/common';

export const PUBLIC_AUTH_KEY = 'isPublicAuth';
export const PublicAuth = () => SetMetadata(PUBLIC_AUTH_KEY, true);

```



#### 定义JwtSystemGuardGuard守卫



```js
import { ExecutionContext, Injectable } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { AuthGuard } from '@nestjs/passport';
import { PUBLIC_AUTH_KEY } from 'src/common/decorators/public-auth.decorator';

@Injectable()
export class JwtSystemGuardGuard extends AuthGuard('system-jwt') {
  constructor(private reflector: Reflector) {
    super();
  }
  canActivate(context: ExecutionContext) {
    const isPublic = this.reflector.getAllAndOverride<boolean>(
      PUBLIC_AUTH_KEY,
      [context.getHandler(), context.getClass()],
    );
    if (isPublic) {
      return true;
    }
    return super.canActivate(context);
  }
}

```

#### 配置JwtAuthModule

```js
// jwt.module.ts
import { JwtModule } from '@nestjs/jwt';

imports:[
  JwtModule.registerAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (configService: ConfigService) => {
        return {
          secret: configService.get('JWT_SECRET'), // 不会用到，策略里覆盖
          signOptions: { expiresIn: '1h' },
        };
      },
    }),
]
```

#### 应用

#### 控制器级别

```js
@UseGuards(JwtSystemGuardGuard)
export class MenuController {}
```

#### 路由级别

```js
	@UseGuards(JwtSystemGuardGuard)
  @ApiOperation({ summary: '创建菜单' })
  @Post()
  async create(@Body() sysMenuDto: SysMenuDto) {
    return await this.menuService.create(sysMenuDto);
  }
```

#### 全局应用级别

```js
// main.ts
app.useGlobalGuards(new JwtSystemGuardGuard());
```





