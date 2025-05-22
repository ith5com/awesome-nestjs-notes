# 日志

## 内置日志系统

Nest 带有一个内置的基于文本的日志器，它在应用引导和其他几种情况下使用，例如显示捕获的异常（即系统日志记录）。此功能是通过 `@nestjs/common` 包中的 `Logger` 类提供的。你可以完全控制日志系统的行为



### 配置开启

```js
// app.module.ts
NestFactory.create(AppModule, { logger: ??? })
```

#### 参数说明

> `logger` 的 3 种取值类型说明：

| 类型                 | 示例                       | 说明                                                |
| -------------------- | -------------------------- | --------------------------------------------------- |
| `boolean`            | `false`                    | 关闭全部日志输出（不打印任何 log、warn、error）     |
| `LogLevel[]` 数组    | `['log', 'error', 'warn']` | 只启用特定等级的日志（比如生产环境只保留最必要的）  |
| `LoggerService` 实例 | `new MyCustomLogger()`     | 使用你自定义实现的日志服务（如基于 `winston` 的类） |

```js
NestFactory.create(AppModule, { logger: true });  // 默认行为
NestFactory.create(AppModule, { logger: false }); // 禁用所有日志


NestFactory.create(AppModule, {
  logger: ['log', 'warn', 'error'], // 推荐生产环境配置
});

NestFactory.create(AppModule, {
  logger: ['log', 'warn', 'error', 'debug', 'verbose'], // 开发环境
});

严重程度：
┌────────────┐
│ error      │ ← 系统异常、接口失败
├────────────┤
│ warn       │ ← 潜在问题、配置告警
├────────────┤
│ log        │ ← 正常运行信息、状态
├────────────┤
│ debug      │ ← 开发调试日志
├────────────┤
│ verbose    │ ← 详细过程追踪（几乎每一步）
└────────────┘

```

### 实现LoggerService

```js
import { LoggerService } from '@nestjs/common';

class MyCustomLogger implements LoggerService {
  log(message: any) {
    console.log('MY LOG:', message);
  }
  error(message: any, trace?: string) {
    console.error('MY ERROR:', message);
  }
  warn(message: any) {
    console.warn('MY WARN:', message);
  }
  debug?(message: any) {
    console.debug('MY DEBUG:', message);
  }
  verbose?(message: any) {
    console.info('MY VERBOSE:', message);
  }
}

NestFactory.create(AppModule, {
  logger: new MyCustomLogger(),
});
```







## winston日志系统

### 安装

```js
npm install winston winston-daily-rotate-file
```

### 配置

> logger.ts

```js
import { createLogger, format, transports } from 'winston';
import * as DailyRotateFile from 'winston-daily-rotate-file';

const logFormat = format.combine(
  format.timestamp({ format: 'YYYY-MM-DD HH:mm:ss' }),
  format.printf(({ timestamp, level, message, context }) => {
    return `[${timestamp as string}] [${level.toUpperCase()}]${context ? ` [${context as string}]` : ''}: ${message as string}`;
  }),
);

export const logger = createLogger({
  level: 'info',
  format: logFormat,
  transports: [
    new DailyRotateFile({
     	dirname: 'logs',              // 日志文件存放目录（必需）
      filename: 'app-%DATE%.log',   // 日志文件名，%DATE% 会被替换成具体日期
      datePattern: 'YYYY-MM-DD',    // 日期格式（默认 YYYY-MM-DD）
      zippedArchive: true,          // 是否压缩归档旧日志文件
      maxSize: '20m',               // 单个日志文件最大尺寸（如超出将切分）
      maxFiles: '14d',              // 最多保留 14 天的日志（可用天数或数量）
      level: 'info',                // 当前 transport 的日志等级
      handleExceptions: true,       // 是否处理未捕获的异常
      json: false,                  // 是否以 JSON 格式保存日志
    }),
    new transports.Console({
      format: format.combine(format.colorize(), logFormat),
    }),
  ],
});

```

> logger.service.ts

```js
import { Injectable, LoggerService as NestLoggerService } from '@nestjs/common';
import { logger } from './logger';

@Injectable()
export class LoggerService implements NestLoggerService {
  log(message: string, context?: string) {
    logger.info(message, { context });
  }

  error(message: string, trace?: string, context?: string) {
    logger.error(`${message} - ${trace}`, { context });
  }

  warn(message: string, context?: string) {
    logger.warn(message, { context });
  }

  debug(message: string, context?: string) {
    logger.debug(message, { context });
  }

  verbose(message: string, context?: string) {
    logger.verbose(message, { context });
  }
}

```

### 使用

```js
constructor(private readonly logger: LoggerService) {}

this.logger.log('用户创建成功', 'UserService');
this.logger.error('创建失败', err.stack, 'UserService');
```

###  `winston` 常用导出方法和作用

| 方法 / 属性              | 说明                                                         |
| ------------------------ | ------------------------------------------------------------ |
| `createLogger()`         | 创建一个日志实例（Logger），核心入口                         |
| `format`                 | 提供格式化工具，如 `combine`、`printf`、`timestamp`、`colorize` 等 |
| `transports.Console`     | 内置的控制台日志输出方式                                     |
| `transports.File`        | 内置的文件日志输出方式                                       |
| `add()`                  | 添加新的 transport（日志输出通道）                           |
| `remove()`               | 移除指定的 transport                                         |
| `logger.log(level, msg)` | 通用日志记录方式，支持动态级别                               |
| `logger.info()`          | 记录 info 等级日志（也有 `warn`, `error`, `debug`, `verbose`） |
| `logger.error()`         | 专门记录错误日志                                             |
