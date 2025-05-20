# 异常过滤器

`Exception filters` 这个功能非常厉害，可以捕获所有的错误。当你的应用代码未处理异常时，该层会捕获该异常。



内置的全局异常过滤器执行，该过滤器处理 `HttpException` 类型（及其子类）的异常。当异常无法识别时（既不是 `HttpException` 也不是继承自 `HttpException` 的类），内置异常过滤器会生成以下默认 JSON 响应：

```js
{
  "statusCode": 500,
  "message": "Internal server error"
}
```

## 内置 HTTP 异常

> `HttpException` 类型（及其子类）的异常, 如下：

| 常类                            | 状态码 | 说明                     |
| ------------------------------- | ------ | ------------------------ |
| `BadRequestException`           | 400    | 请求无效，参数不合法     |
| `UnauthorizedException`         | 401    | 未授权（如未登录）       |
| `ForbiddenException`            | 403    | 禁止访问（如权限不足）   |
| `NotFoundException`             | 404    | 资源未找到               |
| `MethodNotAllowedException`     | 405    | 不允许的方法             |
| `NotAcceptableException`        | 406    | 不可接受的请求           |
| `RequestTimeoutException`       | 408    | 请求超时                 |
| `ConflictException`             | 409    | 冲突（如唯一性冲突）     |
| `GoneException`                 | 410    | 资源已被永久删除         |
| `PayloadTooLargeException`      | 413    | 请求体太大               |
| `UnsupportedMediaTypeException` | 415    | 不支持的媒体类型         |
| `UnprocessableEntityException`  | 422    | 请求格式正确但语义有问题 |
| `InternalServerErrorException`  | 500    | 服务器内部错误           |
| `NotImplementedException`       | 501    | 尚未实现的功能           |
| `BadGatewayException`           | 502    | 网关错误                 |
| `ServiceUnavailableException`   | 503    | 服务不可用               |
| `GatewayTimeoutException`       | 504    | 网关超时                 |

```js
throw new BadRequestException('缺少name参数')
/*
{
  "statusCode": 400,
  "message": "缺少name参数"
}
*/
```



## 抛出标准异常

通过实例化`HttpException`这个类，然后传入两个参数：

- 第一个参数：异常的描述
- 第二个参数：HTTPCODE, 异常的状态码
- 第三个参数：构造函数，`options` - 可用于提供错误 [cause](https://nodejs.cn/blog/release/v16.9.0/#error-cause)，它可用于记录目的，提供有关导致 `HttpException` 被抛出的内部错误的有价值信息

```js
import {HttpException, HttpStatus} from '@nestjs/common'

@Get()
async findAll() {
  throw new HttpException('Forbidden', HttpStatus.FORBIDDEN,{cause:error});
}

/*
{
  "statusCode": 403,
  "message": "Forbidden"
}
*/
```

## 异常日志记录

默认情况下，异常过滤器不会记录内置异常（如 `HttpException`）（以及从它继承的任何异常）。抛出这些异常时，它们不会出现在控制台中，因为它们被视为正常应用流程的一部分。意思就是说：**这些是业务逻辑抛出来的错误，给客户端看的。不是应用错误，所以不会在控制台报错，中止流程**



## 自定义异常

自定义一个`ForbiddenException`类，去继承`HttpException`这个类的所有属性和方法

```js
import { HttpException, HttpStatus } from '@nestjs/common';
import { ErrorEnum } from '../constants/error-code.constants';
export class BusinessException extends HttpException {
  /*
  export enum ErrorEnum {
  	DEFAULT = '0:未知错误',
  	SERVER_ERROR = '500:服务繁忙，请稍后再试',
  }
  */
  constructor(error: ErrorEnum | string) {
    if (!error.includes(':')) {
      super(
        {
          code: HttpStatus.OK,
          message: error,
        },
        HttpStatus.OK,
      );
      return;
    }
    const [code, message] = error.split(':');
    super(
      {
        code: code,
        message: message,
      },
      HttpStatus.OK,
    );
  }
}


// 调用方式1
throw new BusinessException(ErrorEnum.SERVER_ERROR)
// 调用方式2
throw new BusinessException('这个是自定义错误')
```

## 异常过滤器

#### 定义

什么是异常过滤器？顾名思义，就是当上面抛出业务异常的时候，去根据某些因素或者场景，去控制流和如何响应给客户端的JSON

比如，我们上面抛出的异常是：

```js
{
  "statusCode": 403,
  "message": "Forbidden"
}
```

而我们需要的是:

```js
{
    code: 500,
    message: '服务器内部错误',
    data: null,
}
```

类似这种。

#### 实现过滤器

1. 异常过滤器都应实现通用 `ExceptionFilter<T>`
2. 如果使用 `@nestjs/platform-fastify`，则可以使用 `response.send()` 而不是 `response.json()`。不要忘记从 `fastify` 导入正确的类型。
3. `@Catch(HttpException)` 装饰器将所需的元数据绑定到异常过滤器，告诉 Nest 这个特定的过滤器正在寻找 `HttpException` 类型的异常，而不是其他任何东西。

```js
import {Catch,ExceptionFilter} form '@nestjs/common'

@Catch() // 这里可以填写Exception参数，如果不填写，就是捕获一切错误
export class HttpExceptionFilter<T> implements ExceptionFilter{
  catch(exception:T,host:ArgumentsHost){
    const ctx = host.switchToHttp()
    const response = ctx.getResponse<Response>()
    const path = ctx.getRequest().url
    
    // 如果是属于HTTP异常
    if (exception instanceof HttpException) {
      // 获取到异常的响应
      const exceptionResponse = exception.getResponse();
      // 获取到异常的status
      const status = exception.getStatus();
      //如果返回的异常是自定义的
      if (
        typeof exceptionResponse === 'object' &&
        'code' in exceptionResponse
      ) {
        return response.status(status).json({
          code: (exceptionResponse as any).code,
          message: (exceptionResponse as any).message,
          data: null,
          path,
        });
      }
			// 否则处理标准异常
     	return response.status(status).json({
        code: status,
        message: exception.message,
        data: null,
      });
    }
    
    // 如果不是HTTP异常，处理未知异常
    return response.status(HttpStatus.INTERNAL_SERVER_ERROR).json({
      code: HttpStatus.INTERNAL_SERVER_ERROR,
      message: '服务器内部错误',
      data: null,
    });
    
    
  }
}
```

#### 使用过滤器

1. 局部使用（单个路由/方法）

```js
import { UseFilters } from '@nestjs/common';

@UseFilters(new HttpExceptionFilter())
@Get('test')
getData() {
  throw new HttpException('Bad Request', 400);
}
```

2. 控制器级使用

```js
@UseFilters(HttpExceptionFilter)
@Controller('users')
export class UsersController {
  @Get()
  findAll() {
    throw new HttpException('Forbidden', 403);
  }
}
```

3. 全局使用

全局使用的话，可以在`main.ts`中使用`app.useGlobalFilters`去绑定全局过滤器

```js
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  // 绑定过滤器
  app.useGlobalFilters(new HttpExceptionFilter());
  await app.listen(3000);
}
```

还可以通过使用 `APP_FILTER` 全局提供（**推荐用于依赖注入**）

在`AppModule`中

```js
@Module({
  providers:[{
    provider:'APP_FILTER'
    useClass:HttpExceptionFilter
  }]
})
```

| 使用方式                 | 应用范围 | 是否支持依赖注入   |
| ------------------------ | -------- | ------------------ |
| `@UseFilters()` 路由级   | 单个方法 | ❌ 否               |
| `@UseFilters()` 控制器级 | 控制器   | ❌ 否               |
| `app.useGlobalFilters()` | 整个应用 | ❌ 否               |
| `APP_FILTER`             | 整个应用 | ✅ **支持依赖注入** |

