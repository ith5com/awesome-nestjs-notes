# NestJS 是什么，为什么选择它

在现在这个经济复苏的时代，创意往往需要技术去支持。如果你有一个好的创意，正好你也是一名前端开发工程师。NestJS是你快速进入后端的一个框架，学习它不需要Nodejs知识，因为这个框架已经很好的帮我们去做好底层封装了。如果有兴趣对底层感兴趣，可以关注我的github，后续我会出一个源码解读。

对于前端开发者来说非常友好，它支持了`Javascript`和`Typescript` 并结合了 OOP（面向对象编程）、FP（函数式编程）和 FRP（函数式反应式编程）。

## ✅ NestJS 的核心特点包括：

- 基于模块的架构：鼓励良好的代码组织方式，每个功能可以作为模块拆分。

- 使用装饰器（Decorators）：受 Angular 启发，使用装饰器来定义控制器、服务、模块等。

- 内置依赖注入（DI）：类似于 Angular、Spring 的依赖注入机制。

- 支持多种传输层：不仅支持 HTTP，还支持 WebSockets、gRPC、MQTT 等。

- 与 Express（默认）或 Fastify 集成：NestJS 抽象了底层 HTTP 平台，开发者可以选择使用 Express 或 Fastify。

- 强大的 CLI 工具：帮助生成模块、服务、控制器等，提高开发效率。

## 🎯 为什么选择 NestJS？

1. 结构清晰，适合大型项目
- 项目按照模块组织，有助于团队协作和代码维护。
- 遵循 SOLID 原则，提升代码的可维护性和可测试性。

2. TypeScript 原生支持
- 提供更好的类型检查、自动补全和代码导航。
- 提升代码质量，减少运行时错误。

3. 依赖注入机制
- 易于管理依赖关系。
- 有利于编写可测试的代码（例如单元测试和 mock 服务）。

4. 支持多种通信协议
- 不仅限于 REST API，还可以用于构建微服务架构（通过消息队列、gRPC 等）。

5. 生态丰富
- 官方支持的模块很多（如配置管理、验证、缓存、队列、日志等）。
- 与流行库如 TypeORM、Prisma、Mongoose 等无缝集成。

6. 活跃的社区和文档
- 官方文档清晰，社区活跃，问题能迅速得到解决。
- 有大量学习资源和开源项目可参考。

## 💻 安装

```bash
npm i -g @nestjs/cli
nest new project-name

# 你喜欢用哪种安装？
? Which package manager would you ❤️  to use?
  npm
❯ yarn （我常用）
  pnpm

# 选择后，点击回车
▸▹▹▹▹ Installation in progress... ☕

# 看到下面的表示安装成功
🚀  Successfully created project base-nestjs
👉  Get started with the following commands:

$ cd base-nestjs
$ yarn run start

                                         
                          Thanks for installing Nest 🙏
                 Please consider donating to our open collective
                        to help us maintain this package.
                                         
                                         
               🍷  Donate: https://opencollective.com/nest
                                         
```

## 目录树说明

```bash
code
└── base-nestjs
    ├── README.md                  # 📄 项目说明文档，记录项目介绍、安装使用方式等信息
    ├── eslint.config.mjs          # 🔧 ESLint 配置文件，用于代码风格和语法规范检查
    ├── nest-cli.json              # ⚙️ Nest CLI 配置文件，定义编译、生成结构等行为
    ├── package.json               # 📦 Node 项目依赖与脚本配置，记录依赖包、启动脚本等
    ├── src                        # 🧠 主源码目录
    │   ├── app.controller.spec.ts # ✅ app.controller 的单元测试文件（使用 Jest 编写）
    │   ├── app.controller.ts      # 🎮 控制器，处理请求并返回响应（如定义路由处理逻辑）
    │   ├── app.module.ts          # 📦 根模块，组织和导入应用程序中使用的所有模块
    │   ├── app.service.ts         # 🛠️ 服务类，处理具体的业务逻辑，供 controller 调用
    │   └── main.ts                # 🚀 应用程序的入口文件，启动 Nest 应用实例
    ├── test                       # 🧪 E2E 测试目录（端到端测试）
    │   ├── app.e2e-spec.ts        # ✅ E2E 测试用例，测试整个应用接口是否正常运行
    │   └── jest-e2e.json          # 🧾 Jest 的 E2E 测试配置文件
    ├── tsconfig.build.json        # 🛠️ 用于构建（build）的 TypeScript 配置文件
    ├── tsconfig.json              # 🛠️ 全局 TypeScript 配置文件（编译选项、路径别名等）
    └── yarn.lock                  # 📌 Yarn 生成的锁定文件，用于锁定依赖版本以确保一致性

```

> 下面我选相关的文件来详细说下，controller、service、module、test 后面会有文章详细说明

### nestjs-cli.json

这个配置文件用于告诉 Nest CLI 如何运行代码生成器、编译项目等。

```js
{
  "$schema": "https://json.schemastore.org/nest-cli", // ✅ JSON Schema 说明文件，提供编辑器智能提示与校验支持
  "collection": "@nestjs/schematics",                 // 🧱 指定使用的 schematics 集合（NestJS 的代码生成模板）
  "sourceRoot": "src",                                // 📂 源码根目录，CLI 会在这个目录中生成代码（如模块、控制器、服务等）
  "compilerOptions": {
    "deleteOutDir": true                              // 🧹 编译前是否自动删除 dist 目录（即上次编译输出）
  }
}

```

### main.ts (入口文件)

```js
import { NestFactory } from '@nestjs/core';      // 从 @nestjs/core 中导入 Nest 工厂方法，用于创建应用实例
import { AppModule } from './app.module';        // 导入应用的根模块 AppModule

async function bootstrap() {                     // 定义异步启动函数 bootstrap
  const app = await NestFactory.create(AppModule); // 创建 Nest 应用实例（注入 AppModule 模块）
  
  //====常用选项开始====
    
  app.enableCors();                            //  启用跨域支持
  app.setGlobalPrefix('api');                 //  所有路由添加前缀 /api
  
  //====常用选项结束====
  
  
  await app.listen(process.env.PORT ?? 3000);      // 启动监听端口，优先使用环境变量 PORT，否则默认 3000
}

bootstrap();                                     // 调用启动函数，运行应用

```







