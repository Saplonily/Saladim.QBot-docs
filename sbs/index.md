# Saladim.QBot docs

## 目录

> 下述提到的所有'目前'时间均指最后修改时的时间

在开始阅读这些文档之前我们仍先推荐你阅读[快速开始](../fast-start/fast-start.md)


**注:** 无超链接的项还在施工中

- [新建工程并添加包引用](./new-and-add-ref.md)
- [go-cqhttp环境配置](./env-config.md)
- [启动Client, 订阅消息事件](./start-client-and-sub.md)
- [实体,更多事件及可过期类型](./entity-msg-and-expirable.md)
- [消息构建和消息处理](./msg-action.md)
- [消息转发](./forward-msg.md)
- [面向纯接口使用框架](./use-interface.md)
- [拓展: SimCommand](./sim-cmd.md)
- 拓展: Pipeline
- 拓展: StoreSession, MemorySession
- 拓展: MessageCoroutine
- ...

> 注: 因为一些go-cqhttp的问题, 有关临时消息的收发(`PrivateMessage`类(不包括子类`FriendMessage`))的支持可能会相当的烂  
> 所以请务必使用部分方式排除所有除了 `GroupMessage` *或* `FriendMessage` 的消息  
> 在使用纯`SimCommand`环境中我们推荐重写`PreCheck`  
> 在使用`MessagePipeline`环境中我们推荐将日志中间件后的第二项设为排除临时消息中间件

最后修改: 2023-1-6 21:15:08