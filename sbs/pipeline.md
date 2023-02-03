# 处理管线

这一部分类似于 asp.net core 的中间件设计模式, 这里只是简单的说说:

首先我们需要实例化一个泛型管道, 这里以`IClientEvent`为例:
```cs
Pipeline<IClientEvent> eventPipeline = new();
```

向其添加中间件(`Pipeline.Middleware`):
```cs
eventPipeline.AppendMiddleware(async (obj,next)=>
{
    // use obj(IClientEvent type) do something...
    await next(); // await others...
    // do some other thing...
})
```

或者在所有中间件之前插入使用`InsertMiddlewareToFront`方法

然后我们就能以这种方式处理事件了:
```cs
void Client_OnClientEventOccurred(IClientEvent clientEvent)
{
    Task.Run(() => eventPipeline.ExecuteAsync(clientEvent));
}
```

最后修改: 2023-1-29