---
layout:     post
title:      "Qtforeach关键字的脑洞"
subtitle:   "bad Qt foreach"
date:       2018-03-25 12:00:00
author:     "Wjl"
header-img: "img/post_msfr.png"
catalog: true
tags:
    - 原创
    - Qt
    - 低级失误
---

> 狗年过的还真是怠惰的呢....

今天我们的主角是qt 的foreach 关键字;
曾经我一直想当然的以为QT_FOREACH = BOOST_FOREACH;

直到我遇见这样一个问题：
通过list中存储的数据去服务下载对应的资源
```C++
A实现：
    QList<PackageData*>::iterator it;
    for(it = m_pDataList->begin(); it != m_pDataList->end(); ++it) {
        CommDownLoader *handler = new CommDownLoader(m_loggedData.ticket, (*it)->id,                                        m_loginData->serverAddr, (*it)->softwareName);
        QThreadPool::globalInstance()->start(handler);
        connect(handler, &CommDownLoader::missionComplete, this, &Communicator::onDownLoadMissionComplete);
    }

B实现：
    foreach(auto pData, m_pDataList) {
        CommDownLoader *handler = new CommDownLoader(m_loggedData.ticket, pData->id, m_loginData->serverAddr, pData->softwareName);
        QThreadPool::globalInstance()->start(handler);
        connect(handler, &CommDownLoader::missionComplete, this, &Communicator::onDownLoadMissionComplete);
    }
```

A实现可以完成功能,当使用B实现时编译器报错信息如下：
```C
../../../Qt5.6.3/5.6.3/gcc_64/include/QtCore/qglobal.h:957:34: error: 'class QForeachContainer<QList<_PackageData*>*>' has no member named 'i'
     for (variable = *_container_.i; _container_.control; _container_.control = 0)
../../../Qt5.6.3/5.6.3/gcc_64/include/QtCore/qglobal.h:1011:21: note: in expansion of macro 'Q_FOREACH'
 #    define foreach Q_FOREACH
                     ^~~~~~~~~
communication/communicator.cpp:135:5: note: in expansion of macro 'foreach'
     foreach(auto pData, m_pDataList) {
     ^~~~~~~
../../../Qt5.6.3/5.6.3/gcc_64/include/QtCore/qglobal.h:955:58: error: 'class QForeachContainer<QList<_PackageData*>*>' has no member named 'e'
      _container_.control && _container_.i != _container_.e;         \
                                                          ^
```
在最开始懵比了一下后，
emmmm,这是一个const_iterator问题,我容器内保存的指针是非const的,好吧，我真的从来没注意过，这个东西如果编译正常就会向下面这样的结果
```cpp
    char c = 'a';
    const char* pc = &c;
    c = 'b';
```
这肯定不是我们想看到的。

其实这也没啥大惊小怪的,语法糖而以,有最好,没有就换种方式,继续干活....  
晚上的doctime时，想到这个事,我就想了解一下.  
翻了翻文档.看见这样一句.  

> Qt automatically takes a copy of the container when it enters a foreach loop. If you modify the container as you are iterating, that won't affect the loop. (If you don't modify the container, the copy still takes place, but thanks to implicit sharing copying a container is very fast.) Similarly, declaring the variable to be a non-const reference, in order to modify the current item in the list will not work either.
  
 自动获取容器的副本 如果更改容器也不会影响循环,如果不修改我们还是会拷贝一份：）   
 然后在补充一句隐式共享非常的快 blah blah blah~~~
 声明该值为非const的引用,如果命令它去修改当前item ,也不会工作....

然后是社区一片吐槽,5.7之后Qt 又推出了`QT_NO_FOREACH`宏...
八卦之魂觉醒了..
继续深入了解;

qt中 foreach(关键字) = Q_FOREACH(variable, container)
将宏展开
```cpp
#  define Q_FOREACH(variable, container)                                \
for (QForeachContainer<QT_FOREACH_DECLTYPE(container)> _container_((container)); \
     _container_.control && _container_.i != _container_.e;         \
     ++_container_.i, _container_.control ^= 1)                     \
    for (variable = *_container_.i; _container_.control; _container_.control = 0)
```
QForeachContainer实现如下：
```cpp
template <typename T>
class QForeachContainer {
    QForeachContainer &operator=(const QForeachContainer &) Q_DECL_EQ_DELETE;
public:
    inline QForeachContainer(const T& t) : c(t), i(c.begin()), e(c.end()), control(1) { }
    const T c;
    typename T::const_iterator i, e;
    int control;
};
```
实际上是把容器复制到一个名为QForeachContainer对象中，然后遍历它来工作的。
这层拷贝看来是低效的.

来看这两个例子
测试1
```cpp
    QList<int> list;
    int i1 = 0;
    int i2 = 1;
    list.append(i1);
    list.append(i2);
    foreach(auto ix, list)
        list.append(ix);
    foreach(auto iy, list)
        qDebug() << iy;
```
最终输出 0 1 0 1;

这行代码
测试2
```cpp
    QList<int> list;
    int i1 = 0;
    int i2 = 1;
    list.append(i1);
    list.append(i2);
    const QList<int> lst = list;
    foreach(auto ix, lst)
        list.append(ix);

    foreach(auto iy, lst)
        qDebug() << iy;

```
可以编译并运行,不报任何错误.

最终输出 0 1;

迭代器遍历容器时(begin,end),会导致数据产生分离

`循环体中修改容器本身是一件很危险的事情`,天知道它在什么情况下就爆炸了..

测试1假如容器是不可变长容器?

测试2对无定义行为不加任何编译限制,会对将来运行中的程序产生怎样的影响?


