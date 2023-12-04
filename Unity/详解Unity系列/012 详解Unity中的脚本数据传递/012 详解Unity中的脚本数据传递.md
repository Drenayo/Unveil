## 前言
脚本之间如何进行数据传递，应该是一个很让新手困扰的问题，本篇就来总结一下Unity中，脚本的数据传递和事件通知的一些方法技巧，方便新手学习。

//@[TOC](目录)

> 前排提醒：本文仅代表个人观点，以供交流学习，若有不同意见请评论留言，笔者一定好好学习，天天向上。
> 阅读此文章时，若有不理解的地方，推荐观看本文列出的参考资料来对照阅读。

**Unity版本[2019.4.10f1] 梦小天幼 & 禁止转载**

---
## 一、数据传递
> 多脚本之间，数据传递的技巧方法。

### 1.定义静态字段
> 定义静态字段，一般适用于一些固定的常量，或者实例之间共享的变量。可以很方便的**通过类名来获取该值**。但很限制使用场合。
``` C Shapr
    public static int number = 999;
    public static readonly int NUMBER = 888;
```

### 2.定义公开属性、Get方法
> 定义公开属性或者其Get方法，如果是跨脚本传输数据的话，需要获得该脚本的实例对象才能去访问公开变量，至于如何获取该实例对象请看脚本A。
``` C Shapr
脚本A
    //1.通过拖拽赋值
    public 脚本B b;
    private void Start()
    {
        //2.游戏运行时查找
        b = GameObject.Find("对象名");
    }
```

``` C Shapr
脚本B
    //定义私有变量
    private int number;

    //定义该变量的公开属性
    public int Number{get{return number;}}
    
    //定义该变量的公开方法
    public int GetNumber()
    {
        return number;
    }
```
### 3.PlayerPrefs
> PlayerPrefs本质上是用于数据本地持久化保存和读取的一个Unity内置静态类，但也可以用于脚本通讯。使用方法非常简单。
> 原理：以Key—Value的形式将数据保存在本地，然后在代码中可以写入、读取、更新数据。
``` C Shapr
    //存储整型数据
    PlayerPrefs.SetInt("intKey",999); 

    //取出key为"intKey"的整型数据
    int intVal = PlayerPrefs.GetInt("intKey"); 

    //查找是否存在key为"intKey"的数据
    bool exist = PlayerPrefs.HasKey("intKey");
```
> 可以跨脚本通讯，也可以跨场景通讯。

### 4.单例模式
> 一旦使用单例模式，数据就可以很方便的被调用，无需先获取实例，因为类的内部已经写好了获取方法，但是首先要保证，这个类是独一份的，才适用于这种模式

```C Shapr
    class Test
    {
        private int number = 889;

        Test static test = null;
        public static Test GetInstance()
        {
            if(test == null)
                Test test = new Test();
            return test;
        }
    }
```
---

## 二、消息通知
> 多脚本之间，事情通知

### 1.SendMessage
> 这个方法是由Unity提供的，**可用于游戏对象自身的脚本之间的通知**、父级对子级的通知，子级对父级的通知。但**并不支持两个游戏对象之间的消息通知**。

| 方法                                         | 说明                                                           |
| -------------------------------------------- | -------------------------------------------------------------- |
| SendMessage("接收函数"，需传递的参数)        | 发送给自身的所有脚本                                           |
| SendMessageUpwards("接收函数"，需传递的参数) | 发送给自身的所有脚本以及自身父物体，父、父物体等身上的所有脚本 |
| BroadcastMessage("接收函数"，需传递的参数)   | 发送给自身的所有脚本以及自身子物体、子、子物体等身上的所有脚本 |

**例子 :** 需要注意脚本A和脚本B被挂载到了同一个游戏对象下，不是同一个游戏对象，无效。
``` C Shapr
脚本A
    //当运行游戏，开始执行Start方法时，则向“OnTest”发送一个空（null）的消息并执行它
    private void Start()
    {
        SendMessage("OnTest", null);
    }
```

``` C Shapr
脚本B
    // 执行方
    public void OnTest()
    {
        Debug.Log("由脚本A触发此函数");
    }
```

### 2.定义委托（事件处理机制）
> 通过C#的委托特性，来协调各个脚本之间的通讯和数据传输。

> 这里本来应该举个小栗子，但是小栗子不能让大家直观感受到委托的优点，于是乎我就把Unity常用的一种事件管理器的代码贴在下面，有兴趣的可以看看，如果不是很懂，可以去搜索“Unity事件管理器”等关键词。我就偷个懒，不做过多介绍了。

```CSharp
using UnityEngine;
using System.Collections.Generic;
public class EventCenter
{
    private EventCenter() { }
    private static EventCenter eventCenter= null;
    public static EventCenter GetInstance()
    {
        if (eventCenter == null)
            eventCenter = new EventCenter();
        return eventCenter;
    }
    public delegate void processEvent(Object obj, int param1, int param2); 
    //把委托当成一个指针来理解
    private Dictionary<string, processEvent> eventMap = new Dictionary<string, processEvent>(); 
    //那么这里存贮的就是一个字符串，对应一个函数地址
    //声明一个自定义委托，使用字典键值对存贮，

    //注册
    public void Regist(string name, processEvent func)
    {
        if (eventMap.ContainsKey(name))
            eventMap[name] += func;
        else
            eventMap[name] = func;
    }
    //注销
    public void UnRegist(string name, processEvent func)
    {
        if (eventMap.ContainsKey(name))
            eventMap[name] -= func;
    }
    //触发
    public void Trigger(string name, Object obj, int param1, int param2)
    {
        if (eventMap.ContainsKey(name))
            eventMap[name].Invoke(obj, param1, param2);
    }

    //+++运行逻辑：当触发 触发函数时，判断字典内是否有对应注册好的字符串，若有，则执行字符串对应的函数地址的函数，则完成一次触发
}

/*
    //注册方面
    private void Start()
    {
        EventCenter.GetInstance().Regist("LookItem", OnLookItem);
    }
    private void OnLookItem(Object obj, int param1, int param2)
    {
        Debug.Log(obj.name + "被发现了");
    }


    //触发方面：
    EventCenter.GetInstance().Trigger("LookItem", hit.collider.gameObject, 0, 0);

*/

```

---

## 三、总结和参考资料

### 1.总结
 声明一个静态字段的可用范围较小，但使用通过类名获取，很方便
 声明一个公开属性或Get方法，较为便捷，但还需要获取其实例，才能使用
 PlayerPrpfs技术则是全局通用，很容易理解。
 单例模式是一种设计模式，但是它也可以很方便的访问数据，因为它不需要获取当前实例（所以才叫做单例）。
 SendMessage虽然用于消息通知，但是通知的同时也可以传递参数，但仅能传递同对象脚本，不常用。
 定义委托则是全局通用，通过事件处理机制来协调各个脚本之间的数据传递和消息通知。

### 2.参考
[1].[C# 中的委托和事件(详解)](https://www.cnblogs.com/SkySoot/archive/2012/04/05/2433639.html#3579502)
[2].[Unity开发入门指南（17）：脚本通讯](https://zhuanlan.zhihu.com/p/365040692)
[3].[官方API SendMessage](https://docs.unity3d.com/cn/2020.2/ScriptReference/GameObject.SendMessage.html)
[4].梦小天幼的聪明脑瓜



