title: "C++程序与内存"
date: 2014-05-22 18:31:55
tags:
id: 260
categories: 编程技术
tags: C++
---

这里所指的程序是又叫源代码指的电脑中要做某件事情的步骤.当然还有个应用程序的东西，它是程序代码通过编译器根据不同设置编译出来的一个文件.内存值的是电脑的运行内存就是RAM,比如你的电脑有2G内存那么这个电脑的运行内存就是2G.那么程序与内存到底是个什么关系呢？比如有人说这个应用程序占了我电脑50M内存，其实这就是说电脑的内存容量被消耗了5OM.这里从程序员也就是应用功能实现者的角度来说程序与内存的关系.

<!--more-->其实程序在内存中存储的格式是通过二进制0或1的位(bit)来存储的,一般的机器是通过8位bit为一个字节(byte),在C++中基础类型(bool、char、short、int、long、float、double)都是通过最小的占用空间来分配类型存储空间的，具体的分配数值根据各个编译器的情况来决定的。比如：char 占用1字节的空间在内存中就是8位，一般int 占用4个字节就是在内存中就是32位。一般基础的容器类型像数组和自定义类型的占用空间大小都是根绝元素具体类型大小的一个总和，可以通过sizeof()方法来获取类型具体占用空间的大小。对象类型分配的字节空间在内存中都有一个16进制的地址符来对应的.具体有多少个地址符这个根据机器的内存大小来分配的.

在程序运行时内存会给程序分配三个空间:堆，栈，静态内存,下边就说明它们是什么？
堆：就是自由存储的空间程序通过某种方式需要放共享对象的地方，比如：new实例化一个类时内存分配的空间，在不要使用该对象时,堆使用的空间编译器不会自动回收需要显示在程序中释放和销毁对象占用的空间,不过现在大部分语言如Java、ActionScrtip的编译器都有自动回收机制，所以就不需要担心自己在有时候不会释放未使用对象而造成的内存泄漏（程序运行时机器的内存一直增加并且未把不使用的对象释放掉），结果就会使应用崩溃并不能正常使用.但是在C++中 必须显示调用delete来释放存储在堆中的变量所占用的空间.
栈：只在程序运行块时才存在,但栈内存会在不使用块的时候被由编译器自动回收。
静态内存：的里面存储的时全局和局部的静态变量或方法（static）和常量（const）等，这部分占用空间会一直运行到程序结束时才会被释放回收。

还有就是不得不说下指针和引用了，指针就是存放变量在内存中地址的变量，而引用则时变量的另外一个名字,定义引用时程序把引用和初始值就绑定了,定义引用必须被初始化给一个变量并且类型必须和变量类型一致,引用不是一个对象不能定义引用的引用,引用初始化值后不可以更改.而指针则可以指向不同的对象但必须类型必须和初始化值对象的类型一致。

那么堆栈还有静态空间在具体程序中是什么样子呢?用代码来说明
<pre class="lang:c++ decode:true">#include &lt;iostream&gt;
#include &lt;string&gt;

using namespace std;

static int count=1; //存储在静态内存里,在程序运行结束后释放

class Person{
    string name;
    int age;
public:
    Person(string nVal,int aVal):name(nVal),age(aVal){};
    std::string getName();
    int getAge();
};

string Person::getName()
{
    return name;
}

int Person::getAge()
{
    return age;
}

void sayHello(Person p)
{
    Person p1("T1",28);//运行到这里的时候存放在栈内存中,当程序运行sayHello函数结束时被释放
    cout&lt;&lt;"Name:"&lt;&lt;p1.getName()&lt;&lt;"Age:"&lt;&lt;p1.getAge()&lt;&lt;endl;
}

Person getPerson(string nameVal,int ageVal)
{
  Person result(nameVal,ageVal);//在函数体运行结束时释放
  return result;
}
int main()
{
    Person p2("T2",12);//这里的p2变量生命周期是main函数结束
    sayHello(p2);
    /**这里其实时给P3拷贝了一个和函数体执行时相同的数据的Person类实例,
    如果这里返回的是函数的临时变量的引用或指针,那么会出现未定义**/
    Person p3 = getPerson();

    auto p = new Person('P',1);//存放在堆内存中
    delete p;//释放了堆内存中p所指向Peson对象所占用的空间
    p= nullptr;//置空了Person类型指针值
    return 0;
}</pre>