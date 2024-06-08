# 设计模式 - 观察者模式(Observer Pattern)


观察者模式是常见的设计模式之一.

<!--more-->

为了便于理解, 考虑一种常见的场景: 被观察者是报刊, 观察者是订阅报刊的人. 一旦有新闻, 订阅者都可以收到报刊的通知.

## 定义消息, 观察者和被观察者
### 消息与EventArgs
第一步是定义消息的格式, 以及观察者和被观察者的行为.
```c#
public class NotifyEventArgs : EventArgs
{
    public string Info { get; set; } = string.Empty;
    public DateTime Time { get; set; }
}
```
首先定义消息, 这里继承了`EventArgs`. C#在语言上就提供了对消息的的支持, 其中
1. [`EventArgs`](https://docs.microsoft.com/en-us/dotnet/api/system.eventargs?view=net-6.0) 抽象了消息. 为了简单起见, 这里消息就只包含内容和发出的时间两部分.
2. [`EventHandler`](https://docs.microsoft.com/en-us/dotnet/api/system.eventhandler?view=net-6.0) 抽象了一个`delegate`, 也就是处理消息的逻辑, 即观察者收到消息之后需要做的事情.

### 定义观察者
```c#
public interface IObserver
{
    public void NotifyEventHandler(object? notifier, NotifyEventArgs args);
}
```
观察者只需要做一件事, 就是收到消息之后做他想做的事情. 这里函数签名与`EventHandler`一致, 被观察者会把它自身传给第一个参数供观察者使用, 第二个参数是`NotifyEventArgs`, 就是前面定义的消息内容.

### 定义被观察者
```c#
public interface IObservable
{
    public void Register(IObserver observer);
    public void OnNotified(NotifyEventArgs args);
}
```
被观察者需要做两件事:
1. `Register`: 订阅观察者 
2. 需要通知观察者的时候, 调用`OnNotified`, 并且传入消息

## 实现观察者
```c#
public class SubscriberAlice : IObserver
{
    public string Name = "Alice";

    public void NotifyEventHandler(object? notifier, NotifyEventArgs args)
    {
        Console.WriteLine($"{this.Name} received {args.Info} from {notifier?.ToString()} at {args.Time}.");
    }
}

public class SubscriberBob : IObserver
{
    public string Name = "Bob";

    public void NotifyEventHandler(object? notifier, NotifyEventArgs args)
    {
        Console.WriteLine($"{this.Name} received {args.Info} from {notifier?.ToString()} at {args.Time}.");
    }
}
```
这里构造了两个观察者, `Alice`和`Bob`, 他们做的事情就是把收到的消息打印出来. 利用`notifier`和`args`, 我们可以看到消息内容, 时间和发送消息的对象.

## 实现被观察者
```c#
public class Observable : IObservable
{
    internal event EventHandler<NotifyEventArgs> NotifyEventHandlers = delegate {};

    public virtual void OnNotified(NotifyEventArgs args) => NotifyEventHandlers(this, args);

    public void Register(IObserver observer)
    {
        this.NotifyEventHandlers += observer.NotifyEventHandler;
    }

    public void Publish()
    {
        var args = new NotifyEventArgs 
        {
            Info = $"[Breaking News!]", 
            Time = DateTime.Now 
        };

        OnNotified(args);
    }

    public override string ToString() => "New York Times";
}
```
1. `NotifyEventHandlers` 可以视为一个集合, 保存了所有观察者注册的行为(`delegate`), 在需要发送消息的时候, 所有行为都会被依次调用, 这样观察者们会被依次通知.
2. `OnNotified`调用`NotifyEventHandlers`里注册的行为.
3. `Register`将观察者的行为添加到`NotifyEventHandlers`中.
4. `Publish`构造`EventArgs`并触发`OnNotified`, 发送消息给所有观察者.

## 启动程序
```c#
class Program
{
    static void Main(string[] args)
    {
        // 初始化被观察者
        var notifier = new Observable();
        
        // 初始化观察者
        var alice = new SubscriberAlice();
        var bob = new SubscriberBob();

        // 注册观察者
        notifier.Register(alice);
        notifier.Register(bob);

        // 通知观察者
        notifier.Publish();
    }
}
```

程序输出
```txt
Alice received [Breaking News!] from New York Times at 8/30/2022 3:37:14 PM.
Bob received [Breaking News!] from New York Times at 8/30/2022 3:37:14 PM.
```

### Reference
 - [Observer Design Pattern | Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/standard/events/observer-design-pattern)
 - [How to: Raise and Consume Events | Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/standard/events/how-to-raise-and-consume-events)

