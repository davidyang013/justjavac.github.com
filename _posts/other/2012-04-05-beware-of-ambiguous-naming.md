---
layout: post
title: 当心那些有歧义的命名
description: 当心那些有歧义的命名
keywords: 技巧,命名
category : other
tags : [命名, 技巧]
---

## 关键点

    “别人还能把这个名字理解成什么意思？”通过不断的问自己这个问题来积极检查每一个命名。

事实上，这种富有创造性的、不断尝试“错误理解”的方法，能够有效的发现歧义的命名，
并修正它们。正如本文中的示例，我们将随时通过“骑驴看唱本 ——边走边瞧”的方式来 探讨所见到名字的误解之处，
然后选取一个更好的名字。

*示例：Filter() *

假设写了一段代码来操作数据库结果的集合：

    results = Database.all_objects.filter("year <= 2011")

那么，results包含什么数据呢？

* 所有满足year<=2011的对象
* 所有<strong>不</strong>满足year<=2011的对象
 
问题的由来是从filter这个有歧义的词开始的，它没有清楚表达它的意思是“选取”还是“剔除”。
因此，应该避免使用filter，它太容易造 成误解！ 
如果这里想要的效果是“选取”，一个更好的名字是select；如果想要的是“剔除”，更好的名字则是exclude。 

## 为布尔值取名 

当为布尔值变量命名或者函数返回布尔值的时候，
要特别注意真和假所表达出来的真实意思，
这里就有一个很危险的例子：

    bool read_password = true;

这句代码意思取决于当时怎么阅读的（没有其他的意思了），显然这里有两种截然不同的理解：

* 需要读密码
* 密码已经被读过了

在这个用例下，做好避免用单词read，可以考虑使用need_password或者user_is_authenticated来代替。 

*通常情况下，添加单词is、has、can或者should可以让布尔值的意思更加清晰易懂。*

比如说有个函数叫 `SpaceLeft()`，乍一看，就会想到这个函数返回的值是数字。
如果需要明确返回值是布尔值，一个更好的名字是 `HasSpaceLeft()`。 

还有，*尽量避免使用反义短句来命名*。例如：

    bool disable_ssl = false;

改成如下代码则更容易理解，同时更契合原意：

    bool use_ssl = true;

## 符合用户期望 

很多名字是带有误导性的，因为对于某个名字，用户自已有一个预想的定义，但是代码的意思可能恰恰不是这个意思。
如此情况下，最好作出“让步”并改 变名字，消除 误导性。 

*示例：`get*()` *

许多程序员都在使用这样的编码规范：某个方法以get开头来表达一个“轻量级的访问器”以返回内部成员。
违反这个规范将很容易误导用户。 

避免下面的例子中java代码段的做法：

    public class StatisticsCollector {

        public void addSample(double x) { ... }

        public double getMean() {
            // Iterate through all samples and return total / num_samples
        }
        
        ...
    }

这里，getMean的实现是枚举过去所有的数据，并计算其平均值。
如果数据量很大的时候，这一步的开销将会是非常大的。

但是，一个不了解情况的 程 序员则会很粗心的调用它并且假设这是一个很廉价的调用。 
因此，这个方法应该改名成类似computeMean()这样的，
看起来这样就是一个代价高昂的操作了（或者，另一个选择就是改写其实现，变成一 个名副其实的轻量级操作）。 

*示例：`list::size()` *

这里讲一个C++标准库里的命名问题。
这段代码导致的结果是，很难定位和修复类似导致服务器龟速运行之类的问题：

    void ShrinkList(list<Node>& list, int max_size) {
        while (list.size() > max_size) {
            FreeNode(list.back());
            list.pop_back();
        }
    }

这样的bug的导致是作者没有意识到`list.size()`是一个`O(n)`复杂度的操作——它挨个计数链表的节点得出总数而不是返回已计算 好的总个数，
这将导致ShrintList是一个O(n2) 的操作。 

从技术角度讲，这段代码没有问题，也能通过所有的单元测试。
但是当调用`ShrintList()`并传入一个包含上亿数量级的list时，它可能将耗费数小时的时间。
 
或许你会认为，这个是调用者的错误使用，他/她没有认真仔细的阅读相关的文档！
确实是这样的，但是，事实上，这里的`list.size()`不是一 个恒准时（constant-time）操作，这太意外了！
其他所有的C++容器类都是恒准时的`size()`方法呀。 

假如把`size()`更名成`countSize()`或者`countElements()`，类似的错误就会大大减少了。
C++标准库的实现者可能想的 是使用一个`size()`方法去和其他的容器匹配，像vector和map，这样API的一致性看起来更好。
正是由于这样的做了，导致程序员容易误 用并认为这是一个很快的操作，和其他的容器一样！
幸运的是，最新的C++标准要求`size()`是O(1)复杂度。