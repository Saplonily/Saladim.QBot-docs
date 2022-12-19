# Saladim.QBot docs

## 消息转发

消息转发是一种特殊的消息, 也是一种特殊的CQ码, 所以我们一般不会直接使用消息链直接加入消息转发. 一般地, 我们使用`消息转发`实体(`ForwardEntity`)直接传入`SendMessageAsync`来发送转发. 同样地, 消息转发实体也有对应的构建器.
```cs
//使用构造器
ForwardEntityBuilder fb = new(client);
//或者使用client的方法
var fb = client.CreateForwardBuilder();
```

假设我们现在有三个消息实体`msg1`,`msg2`,`msg3`, 我们要将其发送到一个叫`groupTo`的群实体中:
```cs
var fb = client.CreateForwardBuilder();
var forwardEntity = fb
        .AddMessage(msg1)
        .AddMessage(msg2)
        .AddMessage(msg3)
        .Build();
await groupTo.SendMessageAsync(forwardEntity);
```

同时我们是支持使用自定义消息转发的(假设`userABC`都是一个用户实体):
```cs
fb.AddMessage(userA,client.CreateMessageBuilderWithText("这是userA发的消息").Build(),DateTime.Now);
//转发实体目前只支持实体传入
//当仅有文字时我们可以使用client的CreateMessageBuilderWithText这个方法快速加入文字
```
此外还有一些用于不同情况的重载方法. 比如指定显示的用户昵称, 或使发送者默认为bot号且使消息发送时间为当前时间等重载.

最后修改: 2022-12-19 12:34:01