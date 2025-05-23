



# Redis

## 配置

### 安装

```js
yarn add ioredis
```

| 包名                       | 说明                                            | 推荐度 |
| -------------------------- | ----------------------------------------------- | ------ |
| `redis`                    | Node 原生 Redis 客户端（更基础）                | ⭐⭐     |
| `@nestjs-modules/ioredis`  | NestJS 社区封装版，依赖 `ioredis`，但灵活性略低 | ⭐⭐⭐    |
| `nestjs-redis`（已不维护） | 早期使用者较多，但已不推荐                      | ❌      |

### 配置Redis环境变量

```shell
// development.env

# redis
REDIS_HOST=localhost
REDIS_PORT= 6379
REDIS_USERNAME=root
REDIS_PASSWORD=123456
REDIS_NAME= 0
```

### 创建Redis模块(配置)

`nest g module shared/redis`

```js
// redis.module.ts

provider:[
  {
    provide:'REDIS_CLIENT', // 定义key
    useFactory:(configService:ConfigService): Redis =>({
    	return new Redis({
					host: configService.get<string>('redis.host'),
          port: configService.get<number>('redis.port'),
          password: configService.get<string>('redis.password'),
          db: configService.get<number>('redis.db'),
  		})
  	}),
    inject: [ConfigService]
  }
],
exports: ['REDIS_CLIENT']
```

#### 配置参数说明

| 参数名                 | 类型                               | 默认值                         | 说明                                         |
| ---------------------- | ---------------------------------- | ------------------------------ | -------------------------------------------- |
| `host`                 | `string`                           | `'127.0.0.1'`                  | Redis 服务地址                               |
| `port`                 | `number`                           | `6379`                         | Redis 端口                                   |
| `password`             | `string`                           | `undefined`                    | Redis 密码（如果有）                         |
| `db`                   | `number`                           | `0`                            | 使用哪个数据库索引                           |
| `keyPrefix`            | `string`                           | `''`                           | 所有 key 增加的前缀（比如多业务隔离）        |
| `enableOfflineQueue`   | `boolean`                          | `true`                         | Redis 离线状态时是否缓存命令                 |
| `connectTimeout`       | `number`                           | `10000`                        | 连接超时毫秒数                               |
| `retryStrategy`        | `(times: number) => number | null` | 重连策略函数，返回重连间隔毫秒 |                                              |
| `maxRetriesPerRequest` | `number`                           | `20`                           | 每个请求最大重试次数（为 `null` 则无限重试） |
| `autoResubscribe`      | `boolean`                          | `true`                         | 重新连接后是否自动订阅 pub/sub               |
| `lazyConnect`          | `boolean`                          | `false`                        | 是否延迟连接（调用 `.connect()` 时才连接）   |
| `tls`                  | `object`                           | `undefined`                    | 使用 TLS/SSL 连接 Redis                      |





### 封装Redis常用方法

```js
// redis.service.ts

import { Inject, Injectable } from '@nestjs/common';
import Redis from 'ioredis';

@Injectable()
export class RedisService {
  constructor(@Inject('REDIS_CLIENT') private readonly redis: Redis) {}
  /**
   *
   * @param param0
   * 设置字符串键值对
   */
  public async set(key, value, type, secoend) {
    await this.redis.set(key, value, type, secoend);
  }

  /**
   *
   * @param key
   * @returns
   * 获取 key 的值
   */
  public async get(key: string) {
    return await this.redis.get(key);
  }

  /**
   *
   * @param key
   * 删除 key
   */
  public async del(key: string) {
    await this.redis.del(key);
  }

  /**
   *
   * @param key
   * @returns
   * 	key 是否存在（返回 0/1）
   */
  public async exists(key: string) {
    return await this.redis.exists(key);
  }

  /**
   *
   * @param param0
   * 设置 key 的过期时间（秒）
   */
  public async expire({ key, seconds }) {
    await this.redis.expire(key, seconds);
  }
}

```

### 在业务模块中使用Redis

#### 引入Redis模块

目的是为了使用封装的方法

```js
// auth.module.ts

imports: [RedisModule],

```

```js
// auth.service.ts
import { RedisService } from 'src/shared/redis/redis.service';
constructor(
 private readonly redisService: RedisService,
){}

// 存储到redis中，后面做挤兑下线功能
await this.redisService.set(`user_${user.id}`, token, 'EX', 1000);
```



