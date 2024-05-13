# nest-jayson
Nest.js implementation of JSON-RPC 2.0 built on top of jayson

## Installation

```bash
$ npm install nest-jayson
```

## Requirements
- Nest.js with express.js

## Usage

### Module
Register JsonRpcModule in `app.module.ts`
```typescript

@Module({
    imports: [
        JsonRpcModule.forRoot({
            middlewares: [],
        }),
    ],
})
export class AppModule {}
```

### Controller
Create a controller with `@JsonRpcController` decorator
```typescript
@JsonRpcController('vaccination')
export class VaccinationController {
}
```
### Method
Create a method with `@JsonRpcMethod` decorator,
express Request object can be acquired by using `@Req` decorator, 
and JSON-RPC `params` object can be accessed with `@Body` decorator.

```typescript
@JsonRpcController('vaccination')
export class VaccinationController {
    @JsonRpcMethod('getVaccination')
    getVaccination(@Body request: any, @Req req: Request);
}
```

### Method Invocation
Invoke the registered method by calling `controller.methodName` with the JSON-RPC `params` object as the argument.
```json
{
    "jsonrpc": "2.0",
    "method": "vaccination.getVaccination",
    "params": {
        "secureNo": "76481"
    },
    "id": "2"
}
```

### Middleware
Middleware is a nestjs provider that implements the `JsonRpcMiddleware` interface, middleware can intercept the request before it reaches the controller method.
Use `callback` method to return the response immediately, or call `next` method to pass the request.
As middlewares are nestjs provider, it can be injected with other providers.

```typescript
import { JSONRPCCallbackType, JSONRPCResultLike } from 'jayson';
import { Request } from 'express';
import { JsonRpcMiddleware, JsonRpcMiddlewareInterface } from 'nestjs-jayson';

@JsonRpcMiddleware()
export class CacheMiddleware implements JsonRpcMiddlewareInterface {
  async use(
    req: Request,
    callback: JSONRPCCallbackType,
    next: (err: any) => void
  ): Promise<void> {
    const result = this.cache.get();
    if (result) {
      callback(null, result as JSONRPCResultLike);
    } else next(null);
  }
}
```

### Registering Middleware
Register the middleware in the `JsonRpcModule.forRoot` method by providing method filters and the middleware provider.
Use `*.*` to match all methods in all controllers, or `controllerName.*` to match all methods in a specific controller.
Even `*.methodName` can be used to match a specific method in all controllers.
Middlewares are applied in order of registration.

```typescript
@Module({
    imports: [
        JsonRpcModule.forRoot({
            middlewares: [
                {
                    methods: ['*.*'],
                    middleware: AuthMiddleware,
                },
            ],
        }),
    ],
})
```

### Error Handling
nestjs-jayson provides error handling capabilities in order to provide proper JSON-RPC error response.

## License
Distributed under the MIT License

## Acknowledgements
https://www.npmjs.com/package/jayson
