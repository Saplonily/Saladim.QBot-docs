# Session 储存服务

## Session
在这里我们谈论的`Session`不是web那边的`Session`, 我们只是借用它的一个大概的意思: `储存`. 这里的储存像是储存每个群或者用户对应的数据, 每一个`Session`都是"一个群或者用户"和"其对应的数据"的键值对. 这里我们的键是`SessionId`, `SessionId`包含两个`long`字段, 分为`UserId`和`GroupId`.  
呃可能你会疑惑"一个群或者用户"为什么需要两个字段存, 好吧这里我们只是图方便把存群的和存用户的统一起来了, 那么在你只存用户数据而不关心它所在群如何你可以将群id设置为0, 实际上我们内部处理`Session`时对只存群或者只存用户信息的实现就是设置不需要的字段为0(暂时还不需要读懂下面的代码, 只需知道不用的字段是置0的):
```cs
public TSession GetUserSession<TSession>(long userId) where TSession : class, ISession, new()
    => GetSession<TSession>(new SessionId(userId));

public TSession GetGroupSession<TSession>(long groupId) where TSession : class, ISession, new()
    => GetSession<TSession>(new SessionId(0, groupId));

public TSession GetGroupUserSession<TSession>(long groupId, long userId) where TSession : class, ISession, new()
    => GetSession<TSession>(new SessionId(userId, groupId));
```

好了目前为止我们都谈论的是理论上的东西, 回到实际, 数据肯定是多种多样的, 最常见的这里我们实现了两种, 分别是`MemorySession`和`StoreSession`. 前者我们叫"内存Session", 我们通常用它来存储一些运行时的不需要程序关闭时保存的信息, 比如一个q群正在进行的游戏, 一个用户正在进行的交互等. 内存Session的实现和使用都非常简单, 实现上内部就是通过建立一个字典来储存这些, 其类型为`Dictionary<Type, Dictionary<SessionId, ISession>>`. 回到之前的数据类型的谈论, 后者(`StoreSession`)我们叫"储存Session"顾名思义这部分数据是要存入磁盘且应用程序开关时都要读写. 在`Extensions`包里我们默认提供了`SessionSugarStoreService`, 其使用数据库以及[`SqlSugar`](https://www.donet5.com/Home/Doc?typeId=1180)这个[orm](https://cn.bing.com/search?q=orm).

呃好像我们还没说`Session`和`SessionService`的联系, 好吧我觉得你现在应该很迷惑这玩意到底是什么东西然后在骂写文档的这写的什么玩意. 这里大概说个我们对它的定义, `Session`是个实际的储存着数据的类, 而`SessionService`用来管理这些类, 大概应该清除了吧...?

接下来说说具体怎么使用

## 使用Session

### 内存Session

我们先以内存`Session`为例, 使用和操作这些`Session`首先需要一个`SessionService`, 假设你已经在配置依赖注入的地方注入了一个`MemorySessionService`, 当然如果你不使用依赖注入那确保一下这东西你能获取到且是单例模式:
```cs
services.AddSingleton<MemorySessionService>();
```
然后在你需要的地方获取一下:
```cs
var mss = serviceProvider.GetRequiredService<MemorySessionService>();
```
um, 现在假设我们在一个场景, 每个用户都有一个数字, 他们可以用指令加减, 同时这个数字不保证是永久的(就是存在内存里, 开关程序就会丢失). 首先我们需要准备一个`Session`类, 我们假设它叫`PlayNumberSession`, 然后让它实现`ISession`以便`SessionService`能便利的存储它:
```cs
public class PlayNumberSession : ISession
{
    public SessionId SessionId { get; set; }

    public double TheNumber { get; set; }
}
```
假设你现在已经制作了一个指令, 它的作用是让用户的数字+=1, 那么你可以这样增加其Session里的数字:
```cs
long userId = Content.Executor.UserId;
var aNiceSession = mss.GetUserSession<PlayNumberSession>(userId);
aNiceSession.TheNumber += 1;
```
嗯, 没了, 大概就是这样, 在你第一次获取`PlayNumberSession`时如果服务发现没保存它, 那么它会调用默认的空构造函数构建一个, 然后返回, 之后会将这个session引用储存下来, 之后再次获取时都是获取的相同的session, 所以我们才敢获取处理后什么也不干.

在SaladimOffBot里面它是这么用这个内存session的:
```cs
public class LineUpSession : ISession
{
    public SessionId SessionId { get; set; }

    public List<IGroupUser> CurrentWaitings { get; set; } = new();
}
```
这是五子棋游戏的排队session, 它每次获取都是使用群号获取, 所以它是一个群独立的session, 每次当用户执行开始五子棋指令时会获取群有没有人正在排队, 没有则加入, 有则宣布开始游戏, 这是一个典型的使用例子.

## 储存Session

储存Session可能稍微复杂点, 在`Extensions`里的实现是`SqlSugar`这个`orm`, 其余用法与内存session差不多一致, 但是注意我们不能一获取完设置完数据就不管了, 我们需要保存它, 就像这样:
```cs
var s = sessionStoreService.GetUserSession<FiveInARowStoreSession>(u.UserId);
s.WinTimes += 1;
sessionStoreService.SaveSession(s);
```

同时传递过去的Session数据类型必须继承于`SugarStoreSession`, 它内置了一些将`SessionId`转为字符串的功能. 同时, 最简单的声明一个自己的Session类型需要注意几个部分:
- 给类装饰`SugarTable(string)`特性
- 给每个需要储存的属性装饰`SugarColumn`特性
- 当你不需要储存某个属性时使用`SugarColumn(IsIgnore = true)`特性

其余更多关于这些数据库储存orm的部分请见[SqlSugar的官方文档](https://www.donet5.com/Home/Doc?typeId=1180)

因为是数据库储存, 所以这里的实现里的Session服务包含一个返回`ISugarQueryable<TSession>`的`GetQueryable<TSession>`方法. 你可以用它进行linq查询.

`Extensions`里的那个服务实现`SessionSugarStoreService`需要传递一个`SqlSugarScope`对象, 具体也参见[SqlSugar的官方文档](https://www.donet5.com/Home/Doc?typeId=1180)及[SqlSugar文档入门必看部分](https://www.donet5.com/Home/Doc?typeId=1181)

当然你也可以选择参看SaladimOffBot里的一些使用例子, 目前只在储存五子棋胜局的地方用到.

最后修改: 2023-2-3 12:13:27