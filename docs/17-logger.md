# 日志

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
