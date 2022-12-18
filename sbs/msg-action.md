# Saladim.QBot docs

## 消息构建和消息处理

### 发送构建

一般地, 我们提供了`SendMessageAsync(string rawString)`这个函数给绝大部分的`IMessageWindow`实现类, 通过这个函数我们可以使用格式化编码后的一段字符串来发送消息实体, 在`GoCqHttp`中, 它是以CQ码进行编码的

CQ码的一般形式是:
```
[CQ:at,qq=2748166392]
[CQ:poke]
[CQ:forward,flag=43qF34fqf23gfFds]
[CQ:image,url=http://127.0.0.1:5702]
```
以`[CQ:`开头, 然后描述该CQ码的类别, CQ码的类别有很多, 常用的有`at`@,`poke`戳一戳,`forward`消息组合转发,`image`图片,`record`录音音频等. 在类别描述完之后, 如果类别还不能完全描述这个非文本节点时, 后面会尾随一系列以逗号分隔的参数, 比如`at`节点就需要一个`qq`的参数描述对应用户qq号, 参数的格式为`name=value`形的键值对, 在参数描述完之后, 以`]`符号关闭. 同样的, 在参数的值中含有特殊字符`[],&`时会被进行转义. 转义方式与我们在之前章节讨论的转义是一致的.

幸运的是我们不需要自己去构建一个消息构建器去组装这个复杂的字符串, 在`GoCqHttp`实现中我们提供了`MessageEntityBuilder`, 事实上实现`SaladimQBot.Core.IClient`的时候你也同时必须实现`CreateMessageBuilder()`. 一般可通过一下几个方式实例化一个`builder`:
```cs
//直接使用构造器
var mb = new MessageEntityBuilder(client);
//使用Client提供的方法
var mb = client.CreateMessageBuilder();
```
获取到构建器后我们就可以开始组装这个消息了, 这里我们以如下的消息为例:
```
[CQ:reply,msg]
```
//未完工