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

```js
import { Injectable, NextInterceptor, ExecutionContext, CallHeader } from '@nestjs/common'

@Injectable()

export class LoggerInterceptor implements NextInterceptor {
  intercept(context: ExecutionContext, next: CallHeader){
    return next
      .handle()
      .pipe(
        tap(() => console.log(`After... ${Date.now() - now}ms`)),
      );
  }
}
```

