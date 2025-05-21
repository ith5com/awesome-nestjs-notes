# 数据库配置

## 安装

```js
yarn add @nestjs/typeorm typeorm mysql2 @nestjs/config
```

## 文件配置

> 创建环境变量文件`.env.development`

```shell
# 数据库
NODE_ENV=development
DB_HOST=localhost
DB_PORT=3306
DB_USERNAME=root
DB_PASSWORD=123456
DB_NAME=xxn_api_service

```

> 配置`package.json`

```shell
"scripts":{
	"start:dev": "NODE_ENV=development nest start --watch"
}
```

> 创建`config/database.config.ts`

```js
import { registerAs } from '@nestjs/config';

export const DatabaseConfig = registerAs('database', () => ({
  host: process.env.DB_HOST,
  port: parseInt(process.env.DB_PORT as string, 10) || 5432,
  username: process.env.DB_USERNAME,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
}));

```

## 校验配置文件

### 安装Joi

```js
yarn add joi
```

### 配置schema

```js
// shared/env-config/env-config.schema.ts

import * as Joi from 'joi'

export const EnvValidationSchema = Joi.object({
  DB_HOST: Joi.string().required(),
  DB_PORT: Joi.number().default(5432),
  DB_USERNAME: Joi.string().required(),
  DB_PASSWORD: Joi.string().required(),
  DB_NAME: Joi.string().required(),

  NODE_ENV: Joi.string()
    .valid('development', 'production', 'test')
    .default('development'),
})

```



## 配置EnvModule

### 创建`EnvConfigModule`

```js
nest g module shared/envConfig
```

### 写入环境变量配置

```js
import { Module } from '@nestjs/common';

@Module({})
export class EnvConfigModule {}

```

## 配置databaseModule

### 创建`databaseModule`

```shell
nest g module shared/database
```

### 写入数据库配置

```js
import { Module } from '@nestjs/common';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      useFactory: (configService: ConfigService) => ({
        type: 'mysql',
        host: configService.get('database.host'),
        port: configService.get('database.port'),
        username: configService.get('database.username'),
        password: configService.get('database.password'),
        database: configService.get('database.database'),
        entities: ['dist/**/*.entity{.ts,.js}'],
        synchronize: configService.get('NODE_ENV') === 'development',
        logging: configService.get('NODE_ENV') === 'development',
        autoLoadEntities: true,
      }),
      inject: [ConfigService],
    }),
  ],
})
export class DatabaseModule {}

```

> `forRoot`和`forRootAsync`区别

| 方法名           | 参数个数 | 主要用于 | 说明                       |
| ---------------- | -------- | -------- | -------------------------- |
| `forRoot()`      | 1 个     | 静态配置 | 所有配置直接写在代码里     |
| `forRootAsync()` | 1 个     | 动态配置 | 从服务或异步环境变量中读取 |

> `forRoot` 参数说明

| 参数名             | 类型                        | 是否必填 | 说明                                             |
| ------------------ | --------------------------- | -------- | ------------------------------------------------ |
| `type`             | `'mysql'` | `'postgres'` 等 | ✅ 是     | 数据库类型                                       |
| `host`             | `string`                    | ✅ 是     | 数据库地址                                       |
| `port`             | `number`                    | ✅ 是     | 数据库端口                                       |
| `username`         | `string`                    | ✅ 是     | 用户名                                           |
| `password`         | `string`                    | ✅ 是     | 密码                                             |
| `database`         | `string`                    | ✅ 是     | 数据库名                                         |
| `entities`         | `any[]` / 路径              | ✅ 是     | 实体类（Entity）的位置                           |
| `synchronize`      | `boolean`                   | ❌ 否     | 是否自动同步实体结构到数据库（生产环境建议关闭） |
| `logging`          | `boolean`                   | ❌ 否     | 是否打印 SQL 语句                                |
| `timezone`         | `string`                    | ❌ 否     | 设置时区，如 `'+08:00'`                          |
| `migrations`       | `string[]`                  | ❌ 否     | 迁移文件路径                                     |
| `cli`              | `object`                    | ❌ 否     | TypeORM CLI 配置                                 |
| `autoLoadEntities` | `boolean`                   | ❌ 否     | 自动加载被 `@Entity()` 装饰的类                  |

> `TypeOrmModule.forRootAsync()`列表参数说明

| 参数名        | 类型       | 是否必填 | 说明                       |
| ------------- | ---------- | -------- | -------------------------- |
| `imports`     | `Module[]` | ❌ 否     | 导入其它模块（如配置模块） |
| `inject`      | `any[]`    | ❌ 否     | 要注入到工厂函数中的提供者 |
| `useFactory`  | `Function` | ✅ 是     | 工厂函数，返回配置对象     |
| `useClass`    | `Class`    | ❌ 否     | 可选替代 `useFactory`      |
| `useExisting` | `Provider` | ❌ 否     | 从现有服务中读取配置       |













































