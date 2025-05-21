# 接口自动文档生成

## 安装

```js
yarn add @nestjs/swagger
```

## 配置

```js
import {DocumentBuilder, SwaggerModule} from '@nestjs/common'

async function bootstrap(){
    const config = new DocumentBuilder()
    .setTitle('若依管理系统')
    .setVersion('1.0')
    .setDescription('接口文档')
    .build();
  
  const documentFactory = ()=> SwaggerModule.createDocument(app,config)
  SwaggerModule.setup('docs', app, documentFactory)
}
```

## 参数说明

### DocumentBuilder 链式参数

| 方法名                                       | 参数类型                     | 说明                                     |
| -------------------------------------------- | ---------------------------- | ---------------------------------------- |
| `setTitle(title: string)`                    | `string`                     | 设置文档标题（UI 顶部标题）              |
| `setDescription(desc: string)`               | `string`                     | 设置文档描述文字                         |
| `setVersion(version: string)`                | `string`                     | 设置 API 版本号                          |
| `addTag(name: string, description?: string)` | `string, string?`            | 为 API 添加分组标签                      |
| `addBearerAuth(options?, name?)`             | `BearerAuthOptions, string?` | 添加 Bearer JWT 授权（如 token 验证）    |
| `addBasicAuth(options?, name?)`              | `BasicAuthOptions, string?`  | 添加 Basic Auth 授权                     |
| `addApiKey(options, name?)`                  | `ApiKeyOptions, string?`     | 添加 API Key 验证                        |
| `addOAuth2(options?)`                        | `OAuth2Options?`             | 添加 OAuth2 验证支持                     |
| `setContact(name, url, email)`               | `string, string, string`     | 设置联系人信息                           |
| `setLicense(name, url)`                      | `string, string`             | 设置许可协议信息                         |
| `setTermsOfService(url)`                     | `string`                     | 设置服务条款链接                         |
| `addServer(url, desc?)`                      | `string, string?`            | 添加一个服务器地址                       |
| `setExternalDoc(description, url)`           | `string, string`             | 设置外部文档说明                         |
| `build()`                                    | `OpenAPIObject`              | 构建配置对象，供 `createDocument()` 使用 |

### SwaggerModule.setup参数

| 参数位置 | 参数名     | 类型                           | 是否必填 | 说明                                                         |
| -------- | ---------- | ------------------------------ | -------- | ------------------------------------------------------------ |
| 1        | `path`     | `string`                       | ✅ 是     | Swagger UI 的访问路径，例如 `/api-docs`                      |
| 2        | `app`      | `INestApplication`             | ✅ 是     | Nest 应用实例，通常为 `await NestFactory.create(...)` 的结果 |
| 3        | `document` | `OpenAPIObject`                | ✅ 是     | 通过 `SwaggerModule.createDocument()` 生成的 Swagger 文档对象 |
| 4        | `options`  | `SwaggerCustomOptions`（可选） | ❌ 否     | Swagger UI 的自定义选项，控制界面行为、样式等                |

> options

| 属性名            | 类型               | 说明                                                         |
| ----------------- | ------------------ | ------------------------------------------------------------ |
| `swaggerOptions`  | `SwaggerUIOptions` | 传入给 Swagger UI 的配置项（如文档排序、默认展开等）         |
| `customSiteTitle` | `string`           | 自定义文档页面的标题（浏览器 tab 名）                        |
| `customCss`       | `string`           | 注入自定义 CSS 样式                                          |
| `customJs`        | `string`           | 注入自定义 JS 脚本                                           |
| `customfavIcon`   | `string`           | 设置自定义 favicon 图标地址                                  |
| `useGlobalPrefix` | `boolean`          | 是否自动添加全局路由前缀（如果你用了 `app.setGlobalPrefix()`） |
| `explorer`        | `boolean`          | 是否启用 Swagger 的 API Explorer 功能（默认 true）           |

