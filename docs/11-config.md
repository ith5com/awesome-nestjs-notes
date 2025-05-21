# 配置管理与环境变量

我们在项目中，配置环境变量的情况也好多，比如数据库配置、jwt的密钥配置等等。

## 安装

```js
npm install @nestjs/config 
```

## 配置环境变量

>  创建`.env.devlopment`

```js
# 数据库
NODE_ENV=development
DB_HOST=localhost
DB_PORT=3306
DB_USERNAME=root
DB_PASSWORD=123456
DB_NAME=xxn_api_service
```

> 修改`package.json`

```js
 "start:dev": "NODE_ENV=development nest start --watch",
```

## 创建配置文件

> src/config/database.config.ts

```js
import { ConfigService, registerAs } from '@nestjs/config';
import { TypeOrmModuleOptions } from '@nestjs/typeorm';

const databaseConfig = registerAs('database', () => ({
  host: process.env.DB_HOST,
  port: parseInt(process.env.DB_PORT as string, 10) || 5432,
  username: process.env.DB_USERNAME,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
}));

export const getDatabaseConfig = (
  configService: ConfigService,
): TypeOrmModuleOptions => ({
  type: 'mysql',
  host: configService.get('database.host'),
  port: configService.get('database.port'),
  username: configService.get('database.username'),
  password: configService.get('database.password'),
  database: configService.get('database.database'),
  entities: ['dist/**/*.entity{.ts,.js}'],
  synchronize: configService.get('NODE_ENV') === 'development',
  // synchronize: false,
  logging: configService.get('NODE_ENV') === 'development',
  autoLoadEntities: true,
});

export default databaseConfig;
```

### registerAs

`registerAs` 是 `@nestjs/config` 提供的一个函数，用来**注册命名空间的配置工厂函数**。

它的作用是帮助你将配置项**模块化、分组管理**，让配置更加清晰、结构化，方便在项目中按模块或功能组织配置。



> 为什么用 `registerAs`？

- **配置分组管理**
   通过给配置命名空间，避免所有配置变量都堆在一起，特别是大型项目，配置会变得非常多。
- **避免硬编码字符串**
   访问配置时通过命名空间路径访问，如 `configService.get('database.host')`，更清晰也方便维护。
- **模块化复用**
   可以把数据库配置、JWT配置、第三方服务配置等分开写，按需加载。



### 定义

```js
// database.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('database', () => ({
  host: process.env.DB_HOST,
  port: +process.env.DB_PORT || 3306,
  username: process.env.DB_USERNAME,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
}));
```



### 在 `ConfigModule` 中加载

```js
import databaseConfig from './database.config';

@Module({
  imports: [
    ConfigModule.forRoot({
      load: [databaseConfig],
    }),
  ],
})
export class AppModule {}

```

### 访问配置

```js
constructor(private configService: ConfigService) {}

const dbHost = this.configService.get<string>('database.host');
const dbPort = this.configService.get<number>('database.port');
```



## 配置文件校验

```js
import * as Joi from 'joi';

export const envValidationSchema = Joi.object({
  DB_HOST: Joi.string().required(),
  DB_PORT: Joi.number().default(5432),
  DB_USERNAME: Joi.string().required(),
  DB_PASSWORD: Joi.string().required(),
  DB_NAME: Joi.string().required(),

  NODE_ENV: Joi.string().valid('development', 'production', 'test').default('development'),
});
```

## 导入配置到模块

### 模块配置

创建`EnvConfigModule`

```js
import { ConfigModule } from '@nestjs/config'
@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true, // 全局使用
      envFilePath: `.env.${process.env.NODE_ENV || 'development'}`, // 自动加载
      load: [databaseConfig, jwtConfig],
      validationSchema: envValidationSchema,  // 统一校验所有相关环境变量
    }),
  ],
})
export class EnvConfigModule {}
```

`ConfigModule` 常用的参数选项：

| 参数                | 说明                                         | 默认值      |
| ------------------- | -------------------------------------------- | ----------- |
| `isGlobal`          | 是否全局模块，默认只对当前模块可用           | `false`     |
| `envFilePath`       | 指定环境变量文件路径，支持字符串或字符串数组 | `'.env'`    |
| `ignoreEnvFile`     | 是否忽略 `.env` 文件，只使用 `process.env`   | `false`     |
| `validationSchema`  | 使用 Joi 定义环境变量校验规则                | `undefined` |
| `validationOptions` | Joi 校验的额外选项，比如 `allowUnknown` 等   | `{}`        |
| `load`              | 预加载配置工厂函数数组，模块初始化时自动加载 | `[]`        |

在`app.module.ts`中引入
