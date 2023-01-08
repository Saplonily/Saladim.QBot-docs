# Saladim.QBot docs

## SimCommand

SimCommand是一个Saladim.QBot框架官方的一个拓展, 使用它你需要安装额外的包`SaladimQBot.Extensions`, 它主要拓展了对消息的以指令形式解析成对应预定义指令的功能  
注意该包目前因为一些原因暂不支持`.net standard 2.0`, 所以还请升级你的项目至`.net 5+`

> 在该篇文档中指令的含义以及样式:  
> `/echo 114514`  
> `/算 "16 - 9"`  
> `/开始五子棋`  
> `/做旗子 Red Green Blue`  
> 上述中"`/`"定义为指令根前缀, 它是所有指令执行都应该有的内容前缀, 其中`echo`,`算`,`开始五子棋`等空格前面到根前缀的部分我们称作`指令名称`, 其中部分指令还有以空格分割的参数, 我们称作`指令参数`, 如果参数中有空格我们必须使用**双引号**`"`括起来, 比如指令例子的第二项

### 开始使用`SimCommandExecuter`

该类是解析过程的核心类, 它拥有两个构造函数, 首先是接受一个"指令根前缀"字符串这个重载
```cs
SimCommandExecuter simCmd = new("/");
```
这初始化了一个simCmd的执行器, 其中根指令前缀是"`/`", 然后我们需要为其添加几个指令, 首先我们需要一个新类, 然后将其继承于`CommandModule`, 然后写一个方法, 在其之上装饰`CommandAttribute`并传入一个指令名称, 像这样:
```cs
public class TextMiscModule : CommandModule
{
    [Command("hello")]
    public void Hello()
    {

    }
}
```
我们可以在方法里写些东西, 比如回应一个`@执行者 你好!`, 在方法体内我们需使用`Content`来获取指令的一些信息, 比如说执行者`Executor`, 消息窗口`MessageWindow`, 以及消息收到时的`Client`, 就像这样:
```cs
[Command("hello")]
public void Hello()
{
    IMessageEntity entity = Content.Client.CreateMessageBuilder()
        .WithAt(Content.Executor)
        .WithText(" 你好！")
        .Build();
    Content.MessageWindow.SendMessageAsync(entity);
}
```
准备工作做好了后, 我们需要让执行器知道我们有这个模块, 所以我们使用它的`AddModule`方法, 它接收一个`Type`对象:
```cs
simCmd.AddModule(typeof(TextMiscModule));
```
让执行器知道有模块后还不够, 我们得主动去调用它的`MatchAndExecuteAll`方法, 它接收一个`IMessage`实例:
```cs
//在接收到消息事件内, 假设此时msg为该事件的`Message`属性
simCmd.MatchAndExecuteAll(msg);
```

> 你可能会疑惑这里为什么叫`ExecuteAll`, 这是因为执行器会尝试匹配消息的所有文字段并**执行所有匹配的指令**  
> 这既有好处也有坏处, 好处之一就是你可以像正常重载方法一样重载指令  
> 但是坏处就是可能一条消息会触发2条即以上的指令, 这点未来可能会加入指令执行条数限制等  
> 注意只会匹配消息的文本节点, 会忽略所有非文本节点, 即非文本节点**不会**尝试解析成参数

> 默认情况下如果一个指令匹配后会进行反射调用对应模块的空构造函数得到一个新实例, 也就是每条指令执行时的`Module`实例是**不同**的, 之后我们会接受执行器的第二个重载构造器, 它允许我们指定如何获取每次的`Module`实例, 比如指定为使用`IServiceProvider`的`GetRequiredService`方法, 这样我们就可以使用构造器进行依赖注入了

现在编译运行程序, 向群里发一条/hello试试, 你应该会得到`@执行者 你好!`这种格式的回复

### 为指令添加参数

当然, 只有简简单单的无参指令并不是很有趣, 所以我们允许加入指令的参数, 比如说这样:
```cs
[Command("add")]
public void Add(int a, int b)
{
    Content.MessageWindow.SendTextMessageAsync($"{a} + {b} = {a + b}");
}
```
> `SendTextMessageAsync`是发送纯文字消息的简化拓展方法
> 
对函数解析成指令的过程在`AddModule`时反射进行并保存结果

现在我们向bot或其所在群发送类似这样的消息: `/add 114 514`, 那么你应该会收到类似"114 + 514 = 628"的消息

目前来说支持以下一系列的类型作为入参:
- int
- uint
- byte
- char
- long
- short
- ulong
- float
- sbyte
- ushort
- double
- string
- System.Drawing.Color
- System.Numerics.Vector2
- System.Numerics.Vector3

以及对应的一系列数组(以`SimCommandExecuter`的属性`ArraySplitChar`字符进行分割, 默认为逗号`,`)

其中除后三个应该很容易知道是如何解析的, 所以我们暂且介绍下默认实现下后三个类型如何解析:

- System.Drawing.Color
  - 检查是否是`#`开头
    - 首先假设是`#RRGGBB`形式, 解析成功后`A`值为255
    - 否则假设是`#RRGGBBAA`形式, 解析为对应的rgba值
  - 否则检查是否含有半角或全角逗号
    - 假设是`r,g,b`形式, 解析成对应rgba值, 其中`A`为255
    - 假设是`r,g,b,a`形式, 解析成对应rgba值
    - 检查是否含有小数点(以下两种因为是小数所以会将0~1的值映射到0~255上)
      - 假设是`r,g,b`形式, 解析成对应rgba值, 其中`A`为255
      - 假设是`r,g,b,a`形式, 解析成对应rgba值
  - 最后使用`System.Drawing.Color.FromName`检查是否是颜色名称, 解析成响应命名颜色
  - 解析失败
  - 综合来说支持`#`开头的十六进制形式有或没有A, rgb(a)以半角或全角逗号分隔或带有小数点映射0~1到0~255上
    - 例如`#FF667799`, `#BADFF4`, `255,0,255`, `128,129,130,131`, `1.0,0.5,0.2,0.1`

- System.Numerics.Vector2
  - 以半角或全角逗号直接进行分割
    - 分割后长度为2后
      - 检查是否带有半角括号, 是则忽略然后取对应值解析成两个float组装后返回
      - 否则直接取对应值解析成两个float组装后返回
    - 分割长度不为2解析失败
    - 综合来说支持形如`(23,56.0)`, `2,3`的文本

- System.Numerics.Vector3
  - 以半角或全角逗号直接进行分割
    - 分割后长度为3后
      - 检查是否带有半角括号, 是则忽略然后取对应值解析成三个float组装后返回
      - 否则直接取对应值解析成三个float组装后返回
    - 分割长度不为3解析失败
    - 综合来说支持形如`(23,56.0)`, `2,3`的文本

如果你想加入自己自定义的解析格式的话是很容易的, 只需要操作执行器的`CommandParamParsers`这个字典, 其类型为`Dictionary<Type, Func<string, object>>`, 其中`Key`为你想解析的类型, `Value`为对应解析器, 它会传入一个分割后的`string`字符串然后要求返回一个解析后的值, 注意`Type`的类型与返回的object需一直, 否则会出现反射调用方法的错误. 在解析器内部抛出的所有异常均会被认为解析失败且忽略, 解析失败后会认为指令参数输入错误而不执行对应方法.

### 指定模块初始化器

在前面我们介绍过默认的模块初始化器是每次都使用反射调用空构造函数, 为了支持一些依赖注入的需求, 你可以指定模块初始化器, 即使用`SimCommandExecuter`的构造函数的第二个重载, 第二个参数要求传入一个`Func<Type, object?>`, 其中`Type`表示执行器所需要的模块类型, `object?`为对应实例, 如果实例不为`CommandModule`类型会忽略这次指令执行的尝试

> 注意: 当尝试执行的指令名字和**参数的个数**匹配时就会进行模块的实例化操作

通常需要指定模块初始化器的情景是使用依赖注入, 我们在拓展包内为`IServiceCollection`拓展了一个`AddSimCommand`方法, 其原型如下:
```cs
static void AddSimCommand(this IServiceCollection services, Func<IServiceProvider, SimCommandConfig> configProvider, Assembly modulesAsm)
```
大部分情况下下面这种方式已经足够使用了:
```cs
services.AddSimCommand(s => new("/", t => (CommandModule)s.GetRequiredService(t)), typeof(App).Assembly);
```
`SimCommandConfig`的构造器需要接收一个指令根前缀以及模块初始化器, 一般`(CommandModule)s.GetRequiredService(t)`足够使用, 该拓展的最后一个参数为指令模块所在的程序集, 它会反射获取程序集内所有继承于`CommandModule`的类并将其以`AddTransient`的方式加入服务集合中.

### 更加复杂的参数

目前只有一种支持, 即`params`指令, 要使用它我们只需要简单的在参数上加`params`, 例如:
```cs
[Command("adds")] public void Adds(params double[] doubles)
```
上述指令会尝试将所有参数作为`double`解析, 然后放入数组中传递过来, 不过注意目前只支持将`params`参数放入第一个参数的位置, 未来可能会支持`params`位于其他位置的情况
所以这个指令最终的效果是输入类似于`/adds 1 2 3 4 5 6 7.8 2.3 114514`的指令会尝试将之后所有数字解析为`double`放入数组中传递入方法

前面说到指令的参数会以空格分割, 如果参数中含有空格那么必须以双引号包围, 目前还暂不支持只有一个参数的指令直接解析后面所有的可能带空格的文本, 比如:
```cs
[Command("echo")] public void Echo(string s)
```
这个指令遇到`/echo 114514 + 1919810`时并不会触发, 因为解析器认为用户尝试执行一个三参数的echo指令, 这里是没有的, 所以必须以双引号包围: `/echo "114514 + 1919810"`, 因为我们知道这里只会有一个参数, 所以加双引号的方式可能略显繁琐, 如果你的入参是字符串的话你可以使用如上所述的`params`指令, 然后`string.Join(' ',strs)`, 在后期可能会支持这种单参数无需引号的情况

### 模块的`PreCheck`

有时候一个模块你可能仅需要在某些情况下执行里面的指令, 所以你可以选择重写模块的`PreCheck`虚函数, 该虚函数会在参数匹配**但参数没解析成功**的情况下执行, 为`false`时会放弃解析跳出该指令的执行尝试

最后修改: 2023-1-8 21:19:36
