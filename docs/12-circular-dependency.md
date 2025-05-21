# 循环依赖

在 NestJS 中，**循环依赖（Circular Dependency）** 是指：
 两个或多个模块/服务之间**互相依赖对方**，导致 Nest 在构建依赖图时无法解析它们的初始化顺序。



## 哪些情况容易出现循环依赖？

- A Service 中发邮件，依赖 B（邮件服务）
- B 需要拿用户信息，又依赖 A（用户服务）
- 两个模块互相导入，Service 又互相依赖

当你发现「A 依赖 B，B 又依赖 A」，就要小心了，这就是循环依赖。

最推荐的解决方式是：

- ❌ 不推荐硬写 `forwardRef` 到处都是
- ✅ 优先考虑「提取通用依赖」到新的 Service / 模块



## 如何解决循环依赖

### 使用 `forwardRef()` 包裹依赖

> 在 `imports` 中使用 `forwardRef()`

```js
// a.module.ts
@Module({
  imports: [forwardRef(() => BModule)],
  providers: [AService],
  exports: [AService],
})
export class AModule {}

// b.module.ts
@Module({
  imports: [forwardRef(() => AModule)],
  providers: [BService],
  exports: [BService],
})
export class BModule {}
```

> 在`@Inject`服务中使用

```js
// auth.service.ts
@Injectable()
export class AuthService {
  constructor(
    @Inject(forwardRef(() => UserService))
    private readonly userService: UserService,
  ) {}
}

// user.service.ts
@Injectable()
export class UserService {
  constructor(
    @Inject(forwardRef(() => AuthService))
    private readonly authService: AuthService,
  ) {}
}
```

### ✅ 拆分逻辑，避免互相依赖(最佳实践)

这才是**最佳实践**。
 你可以把**通用逻辑**抽到第三个 `SharedService` 中，两个服务都依赖它，而不是互相依赖。



### 使用 `ModuleRef` 动态解析依赖

```js
@Injectable()
export class UserService {
  private authService: AuthService;

  constructor(private moduleRef: ModuleRef) {}

  async onModuleInit() {
    this.authService = await this.moduleRef.resolve(AuthService);
  }
}
```













































