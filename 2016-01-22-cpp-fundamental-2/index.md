# 利用链表ADT学习C++面向对象(2) - 栈和队列继承链表


some cpp fundamentals

<!--more-->

## 栈

```cpp
#include <iostream>
#include "LinkedList_simplified.h"
using namespace std;
template <typename T>
class xStack : public LinkedList<T> //don't miss <T> here
{
public:
    T& gettop() {return LinkedList<T>::begin();}
    void push(T rhs) {LinkedList<T>::insert(0, rhs);}
    bool pop(T& popout)
    {
        if (LinkedList<T>::size() == 1)
        {
            cout << "error:Already Empty" << endl;
            return false;
        } else
        {
            popout = LinkedList<T>::begin();
            LinkedList<T>::erase(1);
            return true;
        }
    }
};
```

 * 栈的实现是通过对单链表的继承完成的, 被继承的类称为基类. 注意5行的基类是一个模版类
 * 在继承类中调用基类的成员最好统一加上域解析符以避免混淆
 * public继承表明基类的public成员在派生类中也是public成员, 故还可以不用域解析符而通过this指针来调用基类方法,见queue的实现

## 队列

```cpp
#include <iostream>
#include "LinkedList_simplified.h"
using namespace std;
template <typename T>
class xQueue : public LinkedList<T> //don't miss <T> here
{
public:
    void inQueue(T rhs) {this->insert(this->size()-1, rhs);}
    bool deQueue(T& out)
    {
        if (this->begin_itr()->next == NULL)
        {
            cout << "error:Already Empty" << endl;
            return false;
        } else
        {
            out    = this->begin_itr()->next->data;
            this->erase(1);
            return true;
        }
    }
};
```

 * queue和stack的实现差别仅仅在于push操作和inQueue操作的插入位置不同, 一个在链表头一个在链表尾. 而deQueue操作和pop一致
 * 两者都利用了链表提供的简易迭代器, 这正是为何将其设定为protected的原因.

