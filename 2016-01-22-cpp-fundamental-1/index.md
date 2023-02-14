# 利用链表ADT学习C++面向对象(1) - 基础知识


some cpp fundamentals

<!--more-->

## 类的基本语法
一个具有最基础功能的链表接口如下：

```cpp
template <typename T>
class LinkedList
{
protected:
    void init() {}      //私有初始化函数
    struct Node{}       //链节点
    int ListSize;       //链长度
    Node* head;         //头指针
    Node* locate(int index);                      //简易迭代器
    Node* begin_itr() {return locate(0);}         //返回头节点指针, 内联
    Node* end_itr() {return locate(ListSize - 1);}//返回尾节点指针, 内联
public:
    LinkedList() {}                                   //构造函数
    LinkedList(const LinkedList & rhs) {}             //拷贝构造函数
    ~LinkedList()                                     //析构函数
    LinkedList& operator=(LinkedList& rhs);           //操作符重载

    int size() const {return ListSize;}               //链长度, 内联
    T& index(int index) {return locate(index)->data;} //索引函数, 内联
    T& begin() {return begin_itr()->next->data;}      //取头元素, 内联
    T& end() {return end_itr()->data;}                //取尾元素, 内联
    void insert(int index, const T& rhs);      //插入
    void erase(int index);                     //删除
    bool isempty() const {}                    //判空
    void clear() {}                            //清空
};
```

 * 基本结构：如以上代码所示，类的成员由数据和方法组成
 * 数据封装：public和private/protected表示了类的可见性，标示为private/protected的成员只能由类本身的方法所访问， 而public可以被任何类的任何方法访问
 * const变量和方法：`int size() const {}`中的const限制了该方法的权限，使得它只能访问类数据成员而不能修改它们.同理,19行的size()使得例如`LinkedList A.size() = x;`这样的代码非法，即size只可访问不可修改

## 链节点Node

```cpp
struct Node
{
    T data;
    Node* next;
    Node(const T& d=T(), Node* n=NULL)
        :data(d), next(n) {}
};
```

 * Node结构由数据(类型)T， 次节点指针next以及重载的Node的构造函数组成
 * 所谓构造函数，是指在实例化一个class或structure的时候自动加载的函数，包含了初始化的各项默认参数(第6行中的T()和NULL即为默认参数)，同时亦可通过重载手动指定参数，见下
 * 第7行是一个通过初始化列表(initializer list)来手动指定初始化参数的方法，其将Node初始化时传入的参数d和n传给成员数据data和next

## 初始化函数init

```cpp
void init()
{
    head = new Node;
    head->next = NULL;
    ListSize = 1;
}
```

 * 在LinkedList实例化之后自动赋予内部变量默认值， 不过由于它是私有变量所以需要在外部利用构造函数调用它，见构造函数和析构函数部分

## 构造函数,复制构造函数,析构函数和operator=

```cpp
explicit LinkedList() {init();}
LinkedList(LinkedList& rhs);
~LinkedList()
{
    cout << "LinkedList:Destructor...";
    clear();
    delete head;
}
LinkedList& operator=(LinkedList& rhs);
```

 * 构造函数的函数名和类名一致，而析构函数只是在类名之前加上一个～符号。其中第二个构造函数称为复制构造函数, 是为了手动指定从另一个链表初始化的方法, 两者都可视为对默认隐式构造函数的重载
 * explicit参数是为了禁止隐式类型转换带来不必要的麻烦, 复制构造函数没有使用是因为为了下面实现A b = a这样的句法. 如果加上explici则该句式是违法的
 * 重载的析构函数在main执行完毕之后自动调用，这里利用输出字符串来判断其是否成功启动了
 * 关于重载默认构造函数的重要性, 特别是返回指针类成员时的必要性, 用以下实例来说明

```cpp
//一个暴露默认构造函数缺陷的例子
class IntCell
{
public:
    explicit IntCell(int initialValue = 0)
        {storedValue = new int(initialValue);}
    int read() const {return *storedValue;}
    void write(int x) {*storedValue = x;}
private:
    int* storedValue;
};


int f()
{
    IntCell a(2);
    IntCell b = a;
    IntCell c;

    c = b;
    a.write(4);
    cout << a.read( ) << endl << b.read( ) << endl << c.read( ) << endl;

    return 0;
}
```

 * 对以上代码, 该实现将会输出三个4, 而我们想要的结果仅仅是a为4. 这是因为内部数据成员是一个int指针, 而默认赋值过程使得b,c都获得了指向同一个int的指针, 故改变该int导致所有指向该数的指针值对应变化. 要解决此问题, 需要重载operator=以及复制构造函数, 见下
 * 在初始化时赋值调用的是复制构造函数, 而在调用赋值操作符的时候重载的=函数

 ```cpp
//改进默认构造函数的手段
explicit IntCell(const IntCell& rhs) //对应IntCell b = a或IntCell b(a)
    {storedValue = new int(*rhs.storedValue);}
IntCell & operator= ( const IntCell & rhs )  // 对应 b = a
{
    if (this != &rhs) {*storedValue = *rhs.storedValue;}
    return *this;
}
 ```

 * 因此重载的复制构造函数和operator=应该实现深复制, 见下

 ```cpp
//deep copy, discard explicit or assignment upon initializion is unable
LinkedList(LinkedList& rhs) 
{
    init();
    for (int i = 0; i < rhs.size() - 1; ++i) {insert(i, rhs.index(i+1));}
}

LinkedList& operator=(LinkedList& rhs)
{
    if (this != &rhs)
    {
        for (int i = 0; i < rhs.size() - 1; ++i)
            {insert(i, rhs.index(i+1));}
    }
    return *this;
}
 ```

 * 之所以复制构造函数和operator=的形参都没有加const, 是因为使用到的insert并不是const函数, 直接使用会导致违法, 想要合法必须再定义一个const的insert函数,为了简单起见这里没有应用.

## locate函数的实现:类和接口的分离

```cpp
template <typename T>
typename LinkedList<T>::Node*
LinkedList<T>::locate(int index)
{	
    if (index < 0) {cout << "error:index < 0" <<endl; return locate(0);}
if (index > ListSize - 1) {cout << "error:index overflow" <<endl; return locate(ListSize - 1    );}
    Node* itr = head;
    for (int i = 0; i < index; ++i) {itr = itr->next;}
    return itr;
}
```

 * locate函数返回Node指针
 * 模板类主要应用在链表数据节点的数据类型上。此外，实现在类的声明之外, 需要在返回值类型和函数类型之前加上域解析符，表示它们是类LinkedList的成员
 * 对输入的index做出边界溢出判定，若超过最后一个则返回链尾，若小于0则返回头节点
 * 迭代器一般封装在private/protected域之内, 而给用户访问/修改元素需要另外设计接口, 即本例的index, begin和end方法. 完整的public迭代器的实现比较复杂, 需要对权限作严格限制, 见[带迭代器的单链表](https://github.com/shawnau/DataStructure/blob/master/List_ADT/Linked_List/Linked_List.h)

## 其他方法的实现

```cpp
//从index后插入一个节点
template <typename T>
void LinkedList<T>::insert(int index, const T& rhs) 
{
    Node *p = locate(index); 
    Node *newNode = new Node(rhs, p->next);
    p->next = newNode;
    ListSize++;
}
//删除index后一个节点
template <typename T>
void LinkedList<T>::erase(int index) 
{
    Node *p = locate(index);
    locate(index - 1)->next = p->next;
    delete p;
    ListSize--;
}
```

## 测试文件

```cpp
//这是一个类实例
#include <string>
#define CLASS_EXAMPLE_H
#ifdef CLASS_EXAMPLE_H
using namespace std;

class Player
{
private:
    int rank;
    string name;
public:
    Player(int r = 0, const string & n = "#name") : rank(r), name(n) {}
    int getrank() const {return rank;}
    string getname() const {return name;}
};
#endif
```

 * 以上用于测试的类实例代表一个游戏玩家，包括了玩家的name和排名rank，并且提供了getrank()和getname()方法用于取得名字和排名。

 ```cpp
//测试文件
#include <iostream>
#include "../Class_Example.h"
#include "LinkedList_simplified.h"
using namespace std;

int main(int argc, char const *argv[])
{
    const int listcap = 5;
    Player playerlist[listcap] = 
    {
        Player(1, "Cookiezi"),
        Player(2, "WhiteWolf"),
        Player(3, "rrtyui"),
        Player(4, "powergame"),
        Player(5, "SiLviA"),
    };

    LinkedList<Player> players;
    cout << "-----------insert test-----------" <<endl;
    for (int i = 0; i < listcap; ++i)
    {
        players.insert(players.size() - 1, playerlist[i]);
    }
    for (int i = 0; i < listcap; ++i)
    {
        cout << players.index(i+1).getname() << ", " << endl;
    }
    cout << "-----------copy test-----------" <<endl;
    LinkedList<Player> players_copy_1 = players;
    for (int i = 0; i < listcap; ++i)
    {
        cout << players_copy_1.index(i+1).getname() << ", " << endl;
    }
    cout << "-----------operator= test-----------" <<endl;
    LinkedList<Player> players_copy_2;
    players_copy_2 = players_copy_1;
    for (int i = 0; i < listcap; ++i)
    {
        cout << players_copy_2.index(i+1).getname() << ", " << endl;
    }
    return 0;
}
 ```

 * 将24行的`players.size() - 1`改成0，即可从链头插入，那么输出序列将是逆置的

## 输出结果

```
-----------insert test-----------
Cookiezi,
WhiteWolf,
rrtyui,
powergame,
SiLviA,
-----------copy test-----------
Cookiezi,
WhiteWolf,
rrtyui,
powergame,
SiLviA,
-----------operator= test-----------
Cookiezi,
WhiteWolf,
rrtyui,
powergame,
SiLviA,
LinkedList:Destructor...LinkedList:Destructor...LinkedList:Destructor...
Process finished with exit code 0
```


