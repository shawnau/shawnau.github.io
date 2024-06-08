# 设计模式 - 单例与抽象工厂模式


<!--more-->

最近遇到一个需要修改别人写的单例的问题. 单例是这样写的:

```csharp
// Bad code! Do not use!
public sealed class Singleton
{
    private static Singleton instance = null;
    private static readonly object padlock = new object();

    Singleton()
    {
        // do tons of things
    }

    public static Singleton Instance
    {
        get
        {
            if (instance == null)
            {
                lock (padlock)
                {
                    if (instance == null)
                    {
                        instance = new Singleton();
                    }
                }
            }
            return instance;
        }
    }
}
```
