# SaladimQBot docs

## 面向纯接口使用

### 介绍

直接使用`SaladimQBot.GoCqHttp`的实现已满足大部分需求, 但是你可能会有再封装框架的需求, 比如再封装一个指令解析器, 但是如果使用直接的实现将不会兼容之后可能会出现的实现, 所以我们将其抽象为`SaladimQBot.Core`项目, 其中只包含拓展方法以及大量接口和抽象类. 使用接口的好处很多, 这里暂时不赘述.

### 从`GoCqHttp`迁移

所有`GoCqHttp`实现中的实体基本上都实现了它们对应的接口, 比如`GroupUser`实现了`IGroupUser`, `Group`实现了`IGroup`, 所以你要做的只需要改改昵称. 特别地, `CqClient`实现的是`IClient`, 其中`IClient`的实例确保实现了大部分事件和方法, 但是不包括`GoCqHttp`的`OnLog`事件.

一般我们以下类似的方式面向接口使用:
```cs
IClient client = new CqWebSocketClient("ws://127.0.0.1:5000", LogLevel.Trace);
//暂时强转回来以订阅日志事件
((CqClient)client).OnLog += Console.WriteLine();
//大部分事件你依然可以订阅, 但是传递来的参数是接口实例而不是准确的实现
client.OnMessageReceived += Client_OnMessageReceive;
```

在`GoCqHttp`实现中我们提出了*可过期类型*, 但是在面向接口使用中我们直接废除了这个选项, 所以你可以直接获取它的值, 但是注意内部依然是使用`.Value`的方式取值.

比如我们获取一个`IUser`类型`user`实例的`Nickname`:
```cs
var someNickname = user.Nickname;
//不需要再.Value
```

因为我们不能new一个接口实例出来所以我们在使用消息构建器时只能通过client的`CreateMessageBuilder`方法来获取一个构建器, 其余方法与之前相同, 但入参均为接口类型. (怪东西: 其实接口可以通过一些黑科技new出来(((( , [参考这个](https://github.com/ilyfairy/UnsafeHelper/blob/871c2b50ebd202f8772396c1f3b6fdf5bb0adeac/UnsafeHelper/UnsafeHelper.cs#L221-L241))

例如:
```cs
IMessageEntityBuilder builder = client.CreateMessageBuilder();
builder.WithText("some text...");
builder.WithAt(2748166392);
IMessageEntity entity = builder.Build();
user.SendMessageAsync(entity);
```

最后修改: 2022-12-19 14:16:43