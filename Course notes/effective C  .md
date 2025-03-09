## Item 1: 将 C++ 视为语言联合体

 C++ 已经成为一个 multiparadigm programming language（多范式的编程语言），一个囊括了 procedural（过程化），object-oriented（面向对象），functional（函数化），generic（泛型）以及 metaprogramming（元编程）特性的联合体。
 可以简单的分为四个子语言，C，Object-Oriented C++，Template C++，STL。

## Item 2: 用 consts, enums 和 inlines 取代 \#define
作为一个 language constant，AspectRatio 被 compilers明确识别并确实加入 symbol table。另外，对于 floating point constant来说，使用 constant比使用/#define 能产生更小的代码。这是因为 preprocessor盲目地用 1.653 置换 macro name（宏名字）ASPECT_RATIO，导致你的 object code中存在多个 1.653 的拷贝，如果使用 constant，就绝不会产生多于一个的拷贝。
此外，还有两个值得注意的情况：
- 除了 pointer（指针）指向的目标是常量外，pointer（指针）本身被声明为 const 更加重要。
```C++
const char * const authorName = "Scott Meyers";//√
const std::string authorName("Scott Meyers");//√√
```

- 第二个特殊情况涉及到 class-specific constants（类属（类内部专用的）常量）。为了将一个 constant（常量）的作用范围限制在一个 class（类）内，你必须将它作为一个类的 member（成员），而且为了确保它最多只有一个 constant（常量）拷贝，你还必须把它声明为一个 static member（静态成员）。
```C++
class GamePlayer {
private:
  static const int NumTurns = 5;      // constant declaration
  int scores[NumTurns];               // use of constant
  ...
};
```

## Item 3: 只要可能就用 const
对于 pointers（指针），你可以指定这个 pointer（指针）本身是 const，或者它所指向的数据是 const，或者两者都是，或者都不是：
```C++
char greeting[] = "Hello";

char *p = greeting; // non-const pointer,
                    // non-const data
const char *p = greeting; // non-const pointer,
                          // const data
char * const p = greeting; // const pointer,
                           // non-const data
const char * const p = greeting; // const pointer,
                                 // const data
```

## Item 4: 确保 objects（对象）在使用前被初始化

一个 array（数组）（来自 C++ 的 C 部分）不能确保它的元素被初始化，但是一个 vector（来自 C++ 的 STL 部分）就能够确保。
处理这种事情的表面不确定状态的最好方法就是总是在使用之前初始化你的对象。对于 built-in types（内建类型）的 non-member objects（非成员对象），需要你手动做这件事。例如：
```C++
int x = 0;                                // manual initialization of an int
const char * text = "A C-style string";   // manual initialization of a
                                          // pointer (see also Item 3)

double d;                                 // "initialization" by reading from
std::cin >> d;                            // an input stream
```
除此之外的几乎全部情况，initialization（初始化）的重任就落到了 constructors（构造函数）的身上。这里的规则很简单：确保 all constructors（所有的构造函数）都初始化了 object（对象）中的每一样东西。

 - assignment（赋值）的写法：
```C++
class PhoneNumber { ... };

class ABEntry {                         // ABEntry = "Address Book Entry"
public:
  ABEntry(const std::string& name, const std::string& address,
          const std::list<PhoneNumber>& phones);
private:
  std::string theName;
  std::string theAddress;
  std::list<PhoneNumber> thePhones;
  int num TimesConsulted;
};

ABEntry::ABEntry(const std::string& name, const std::string& address,
                 const std::list<PhoneNumber>& phones)
{
  theName = name;                       // these are all assignments,
  theAddress = address;                 // not initializations
  thePhones = phones;
  numTimesConsulted = 0;
}
```
 - initialization（初始化）的写法：member initialization list（成员初始化列表）
```C++
ABEntry::ABEntry(const std::string& name, const std::string& address,
                 const std::list<PhoneNumber>& phones)

: theName(name),
  theAddress(address),                  // these are now all initializations
  thePhones(phones),
  numTimesConsulted(0)

{}                                      // the ctor body is now empty
```
这个 constructor（构造函数）的最终结果和前面那个相同，但是通常它有更高的效率。assignment-based（基于赋值）的版本会首先调用 default constructors（缺省构造函数）初始化 theName，theAddress 和 thePhones，然而很快又在 default-constructed（缺省构造）的值之上赋予新值。那些 default constructions（缺省构造函数）所做的工作被浪费了。
## Item 5: 了解 C++ 为你偷偷地加上和调用了什么函数
编译器可以隐式生成一个 class（类）的 default constructor（缺省构造函数），copy constructor（拷贝构造函数），copy assignment operator（拷贝赋值运算符）和 destructor（析构函数）。
## Item 6: 如果你不想使用 compiler-generated functions（编译器生成函数），就明确拒绝
于 copy constructor（拷贝构造函数）和 copy assignment operator（拷贝赋值运算符）就像 Item 5 中指出的，如果你不声明它们，而有人又想调用它们，编译器就会替你声明它们。
这就限制了你。如果你不声明 copy constructor（拷贝构造函数）或 copy assignment operator（拷贝赋值运算符），编译器也可以替你生成它们。你的 class（类）还是会支持 copying（拷贝）。另一方面，如果你声明了这些函数，你的 class（类）依然会支持 copying（拷贝）。而我们此时的目的却是 prevent copying（防止拷贝）。
```C++
class HomeForSale {
public:
  ...

private:
  ...
  HomeForSale(const HomeForSale&);            // declarations only（防止拷贝）
  HomeForSale& operator=(const HomeForSale&);
};
```
- 为了拒绝编译器自动提供的机能，将相应的 member functions（成员函数）声明为 private，而且不要给出 implementations（实现）。使用一个类似 Uncopyable 的 base class（基类）是方法之一。
## Item 7: 在 polymorphic base classes（多态基类）中将 destructors（析构函数）声明为 virtual（虚拟）
 C++ 规定：当一个 derived class object（派生类对象）通过使用一个 pointer to a base class with a non-virtual destructor（指向带有非虚拟析构函数的基类的指针）被删除，则结果是未定义的。运行时比较典型的后果是 derived part of the object（这个对象的派生部分）不会被析构。
 消除这个问题很简单：给 base class（基类）一个 virtual destructor（虚拟析构函数）。于是，删除一个 derived class object（派生类对象）的时候就有了你所期望的行为。将析构 entire object（整个对象），包括全部的 derived class parts（派生类构件）
```C++
class TimeKeeper {
public:
  TimeKeeper();
  virtual ~TimeKeeper();
...
};

TimeKeeper *ptk = getTimeKeeper();
...

delete ptk;                             // now behaves correctly
```

- polymorphic base classes（多态基类）应该声明 virtual destructor（虚拟析构函数）。如果一个 class（类）有任何 virtual functions（虚拟函数），它就应该有一个 virtual destructor（虚拟析构函数）。
- 不是设计用来作为 base classes（基类）或不是设计用于 polymorphically（多态）的 classes（类）就不应该声明 virtual destructor（虚拟析构函数）。
## Item 8: 防止因为 exceptions（异常）而离开 destructors（析构函数）
- destructor（析构函数）应该永不引发 exceptions（异常）。如果 destructor（析构函数）调用了可能抛出异常的函数，destructor（析构函数）应该捕捉所有异常，然后抑制它们或者终止程序。
- 如果 class（类）客户需要能对一个操作抛出的 exceptions（异常）做出回应，则那个 class（类）应该提供一个常规的函数（也就是说，non-destructor（非析构函数））来完成这个操作。

## Item 9: 绝不要在 construction（构造）或 destruction（析构）期间调用 virtual functions（虚拟函数）

- 在 construction（构造）或 destruction（析构）期间不要调用 virtual functions（虚拟函数），因为这样的调用不会转到比当前执行的 constructor（构造函数）或 destructor（析构函数）所属的 class（类）更深层的 derived class（派生类）。
## Item 10: 让 assignment operators（赋值运算符）返回一个 reference to \*this（引向 \*this 的引用）
关于 assignments（赋值）的一件有意思的事情是你可以把它们穿成一串。
```C++
int x, y, z;
x = y = z = 15; // chain of assignments
x = (y = (z = 15));//the same output
```
这里的实现方法是让 assignment（赋值）返回一个 reference to its left-hand argument（引向它的左侧参数的引用），而且这就是当你为你的 classes（类）实现 assignment operators（赋值运算符）时应该遵守的惯例：
```C++
class Widget {
public:
  ...
Widget& operator=(const Widget& rhs)   // return type is a reference to
{                                      // the current class
  ...
  return *this;                        // return the left-hand object
  }
  ...
};
```
- 让 assignment operators（赋值运算符）返回一个 reference to \*this（引向 \*this 的引用）。

## Item 11: 在 operator= 中处理 assignment to self（自赋值）
下面是一个表面上看似合理 operator= 的实现，但如果出现 assignment to self（自赋值）则是不安全的。（它也不是 exception-safe（异常安全）的，但我们要过一会儿才会涉及到它。）
```C++
Widget&
Widget::operator=(const Widget& rhs)              // unsafe impl. of operator=
{
  delete pb;                                      // stop using current bitmap
  pb = new Bitmap(*rhs.pb);                       // start using a copy of rhs's bitmap

  return *this;                                   // see Item 10
}
```
防止这个错误的传统方法是在 operator= 的开始处通过 identity test（一致性检测）来阻止 assignment to self（自赋值）：
```C++
Widget& Widget::operator=(const Widget& rhs)
{
  Bitmap *pOrig = pb;               // remember original pb
  pb = new Bitmap(*rhs.pb);         // make pb point to a copy of *pb
  delete pOrig;                     // delete the original pb

  return *this;
}
```
- 当一个 object（对象）被赋值给自己的时候，确保 operator= 是行为良好的。技巧包括比较 source（源）和 target objects（目标对象）的地址，关注语句顺序，和 copy-and-swap。
- 如果两个或更多 objects（对象）相同，确保任何操作多于一个 object（对象）的函数行为正确。
## Item 12: 拷贝一个对象的所有组成部分
当你声明了你自己的拷贝函数，你就是在告诉编译器你不喜欢缺省实现中的某些东西。编译器对此好像怒发冲冠，而且它们会用一种古怪的方式报复：当你的实现存在一些几乎可以确定错误时，它偏偏不告诉你。
- 拷贝函数应该保证拷贝一个对象的所有数据成员以及所有的基类部分。
- 不要试图依据一个拷贝函数实现另一个。作为代替，将通用功能放入第三个供双方调用的函数。

## Item 13: 使用对象管理资源
为了确保 createInvestment 返回的资源总能被释放，我们需要将那些资源放入一个类中，这个类的析构函数在控制流程离开 f 的时候会自动释放资源。实际上，这只是本 Item 介绍的观念的一半：将资源放到一个对象的内部，我们可以依赖 C++ 的自动地调用析构函数来确保资源被释放。
- 为了防止资源泄漏，使用 RAII ( Resource Acquisition Is Initialization)对象，在 RAII 对象的构造函数中获得资源并在析构函数中释放它们。
- 两个通用的 RAII 是 tr1::shared_ptr 和 auto_ptr。tr1::shared_ptr 通常是更好的选择，因为它的拷贝时的行为是符合直觉的。拷贝一个 auto_ptr 是将它置为空。

## Item 14: 谨慎考虑资源管理类的拷贝行为
禁止拷贝。在很多情况下，允许 RAII 被拷贝是没有意义的。这对于像 Lock 这样类很可能是正确的，因为同步的基本要素的“副本”很少有什么意义。当拷贝对一个 RAII 类没有什么意义的时候，你应该禁止它。Item 6 解释了如何做到这一点。声明拷贝操作为私有。
- 拷贝一个 RAII 对象必须拷贝它所管理的资源，所以资源的拷贝行为决定了 RAII 对象的拷贝行为。
- 普通的 RAII 类的拷贝行为不接受拷贝和进行引用计数，但是其它行为是有可能的。

## Item 15: 在资源管理类中准备访问裸资源（raw resources）

- API 经常需要访问裸资源，所以每一个 RAII 类都应该提供取得它所管理的资源的方法。
- 访问可以通过显式转换或者隐式转换进行。通常，显式转换更安全，而隐式转换对客户来说更方便。

## Item 16: 使用相同形式的 new 和 delete

- 如果你在 new 表达式中使用了 []，你必须在对应的 delete 表达式中使用 []。如果你在 new 表达式中没有使用 []，你也不必在对应的 delete 表达式中不使用 []。

## Item 17: 在一个独立的语句中将 new 出来的对象存入智能指针
- 在一个独立的语句中将 new 出来的对象存入智能指针。如果疏忽了这一点，当异常发生时，可能引起微妙的资源泄漏。//自动delete的指针

## Item 18: 使接口易于正确使用，而难以错误使用

- 好的接口易于正确使用，而难以错误使用。你应该在你的所有接口中为这个特性努力。
- 使易于正确使用的方法包括在接口和行为兼容性上与内建类型保持一致。
- 预防错误的方法包括创建新的类型，限定类型的操作，约束对象的值，以及消除客户的资源管理职责。
- tr1::shared_ptr 支持自定义 deleter。这可以防止 cross-DLL 问题，能用于自动解锁互斥体（参见 Item 14）等。

## Item 19: 视类设计为类型设计

- 你的新类型的对象应该如何创建和销毁？如何做这些将影响到你的类的构造函数和析构函数，以及内存分配和回收的函数（operator new，operator new[]，operator delete，和 operator delete[] ——参见 Chapter 8）的设计，除非你不写它们。
- 对象的初始化和对象的赋值应该有什么不同？这个问题的答案决定了你的构造函数和你的赋值运算符的行为和它们之间的不同。这对于不混淆初始化和赋值是很重要的，因为它们相当于不同的函数调用（参见 Item 4）。
- 以值传递（passed by value）对于你的新类型的对象意味着什么？记住，拷贝构造函数定义了一个新类型的传值（pass-by-value）如何实现。
- 你的新类型的合法值的限定条件是什么？通常，对于一个类的数据成员来说，仅有某些值的组合是合法的。那些组合决定了你的类必须维持的不变量。这些不变量决定了你必须在成员函数内部进行错误检查，特别是你的构造函数，赋值运算符，以及 "setter" 函数。它可能也会影响你的函数抛出的异常，以及你的函数的异常规范（exception specification）（你用到它的可能性很小）。
- 你的新类型是否适合放进一个继承图表中？如果你从已经存在的类继承，你将被那些类的设计所约束，特别是它们的函数是 virtual 还是 non-virtual（参见 Item 34 和 36）。如果你希望允许其他类继承你的类，将影响到你是否将函数声明为 virtual，特别是你的析构函数（参见 Item 7）。
- 你的新类型允许哪种类型转换？你的类型身处其它类型的海洋中，所以是否要在你的类型和其它类型之间有一些转换？如果你希望允许 T1 类型的对象隐式转型为 T2 类型的对象，你就要么在 T1 类中写一个类型转换函数（例如，operator T2），要么在 T2 类中写一个非显式的构造函数，而且它们都要能够以单一参数调用。如果你希望仅仅允许显示转换，你就要写执行这个转换的函数，而且你还需要避免使它们的类型转换运算符或非显式构造函数能够以一个参数调用。（作为一个既允许隐式转换又允许显式转换的例子，参见 Item 15。）
- 对于新类型哪些运算符和函数有意义？这个问题的答案决定你应该为你的类声明哪些函数。其中一些是成员函数，另一些不是（参见 Item 23、24 和 46）。
- 哪些标准函数不应该被接受？你需要将那些都声明为 private（参见 Item 6）。
- 你的新类型中哪些成员可以被访问？这个问题的可以帮助你决定哪些成员是 public，哪些是 protected，以及哪些是 private。它也可以帮助你决定哪些类和／或函数应该是友元，以及一个类嵌套在另一个类内部是否有意义。
- 什么是你的新类型的 "undeclared interface"？它对于性能考虑，异常安全（exception safety）（参见 Item 29），以及资源使用（例如，锁和动态内存）提供哪种保证？你在这些领域提供的保证将强制影响你的类的实现。
- 你的新类型有多大程度的通用性？也许你并非真的要定义一个新的类型。也许你要定义一个整个的类型家族。如果是这样，你不需要定义一个新的类，而是需要定义一个新的类模板。
- 一个新的类型真的是你所需要的吗？是否你可以仅仅定义一个新的继承类，以便让你可以为一个已存在的类增加一些功能，也许通过简单地定义一个或更多非成员函数或模板能更好地达成你的目标。

## Item 20: 用 pass-by-reference-to-const（传引用给 const）取代 pass-by-value（传值）

- 用传引用给 const 取代传值。典型情况下它更高效而且可以避免切断问题。
- 这条规则并不适用于内建类型(eg: int)及 STL 中的迭代器和函数对象类型。对于它们，传值通常更合适。
## Item 21: 当你必须返回一个对象时不要试图返回一个引用

- 绝不要返回一个局部栈对象的指针或引用，绝不要返回一个被分配的堆对象的引用，如果存在需要一个以上这样的对象的可能性时，绝不要返回一个局部 static 对象的指针或引用。（Item 4 提供的一个返回一个局部 static 的设计的例子是合理的，至少在单线程的环境中是这样。）

## Item 22: 将数据成员声明为 private

假设我们有一个 public 数据成员，随后我们消除了它。有多少代码会被破坏呢？所有使用了它的客户代码，其数量通常大得难以置信。从而 public 数据成员就是完全未封装的。但是，假设我们有一个 protected 数据成员，随后我们消除了它。现在有多少代码会被破坏呢？所有使用了它的派生类，典型情况下，代码的数量还是大得难以置信。从而 protected 数据成员就像 public 数据成员一样没有封装，因为在这两种情况下，如果数据成员发生变化，被破坏的客户代码的数量都大得难以置信。这并不符合直觉，但是富有经验的库实现者会告诉你，这是千真万确的。一旦你声明一个数据成员为 public 或 protected，而且客户开始使用它，就很难再改变与这个数据成员有关的任何事情。有太多的代码不得不被重写，重测试，重文档化，或重编译。从封装的观点来看，实际只有两个访问层次：private（提供了封装）与所有例外（没有提供封装）。
- 声明数据成员为 private。它为客户提供了访问数据的语法层上的一致，提供条分缕析的访问控制，允许不变量被强制，而且为类的作者提供了实现上的弹性。

## Item 23: 用非成员非友元函数取代成员函数

- 用非成员非友元函数取代成员函数。这样做可以提高封装性，包装弹性，和机能扩充性。
## Item 24: 当类型转换应该用于所有参数时，声明为非成员函数
- 如果你需要在一个函数的所有参数（包括被 this 指针所指向的那个）上使用类型转换，这个函数必须是一个非成员。

## Item 25: 考虑支持不抛异常的 swap
- 如果 std::swap 对于你的类型来说是低效的，请提供一个 swap 成员函数。并确保你的 swap 不会抛出异常。
- 如果你提供一个成员 swap，请同时提供一个调用成员 swap 的非成员 swap。对于类（非模板），还要特化 std::swap。
- 调用 swap 时，请为 std::swap 使用一个 using declaration，然后在调用 swap 时不使用任何 namespace 限定条件。
- 为用户定义类型完全地特化 std 模板没有什么问题，但是绝不要试图往 std 中加入任何全新的东西。

## Item 26: 只要有可能就推迟变量定义

- 只要有可能就推迟变量定义。这样可以增加程序的清晰度并提高程序的性能。

## Item 27: 将强制转型减到最少

- 避免强制转型的随时应用，特别是在性能敏感的代码中应用 dynamic_casts，如果一个设计需要强制转型，设法开发一个没有强制转型的侯选方案。
- 如果必须要强制转型，设法将它隐藏在一个函数中。客户可以用调用那个函数来代替在他们自己的代码中加入强制转型。
- 尽量用 C++ 风格的强制转型替换旧风格的强制转型。它们更容易被注意到，而且他们做的事情也更加明确
```c++
const_cast<T>(expression)
dynamic_cast<T>(expression)
reinterpret_cast<T>(expression)
static_cast<T>(expression)
```

## Item 28: 避免返回对象内部构件的“句柄”
- 避免返回对象内部构件的句柄（引用，指针，或迭代器）。这样会提高封装性，帮助 const 成员函数产生 cosnt 效果，并将空悬句柄产生的可能性降到最低

## Item 29: 争取异常安全（exception-safe）的代码
- 即使当异常被抛出时，异常安全的函数不会泄露资源，也不允许数据结构被恶化。这样的函数提供基本的，强力的，或者不抛出保证。
- 强力保证经常可以通过 copy-and-swap 被实现，但是强力保证并非对所有函数都可用。
- 一个函数通常能提供的保证不会强于他所调用的函数中最弱的保证。

## Item 30: 理解 inline 化的介入和排除
- 将大部分 inline 限制在小的，调用频繁的函数上。这使得程序调试和二进制升级更加容易，最小化潜在的代码膨胀，并最大化提高程序速度的几率。
- 不要仅仅因为函数模板出现在头文件中，就将它声明为 inline。

## Item 31: 最小化文件之间的编译依赖
- 最小化编译依赖后面的一般想法是用对声明的依赖取代对定义的依赖。基于此想法的两个方法是 Handle 类和 Interface 类。
- 库头文件应该以完整并且只有声明的形式存在。无论是否包含模板都适用于这一点。

## Item 32: 确保 public inheritance 模拟 "is-a"
- public inheritance 意味着 "is-a"。适用于 base classes 的每一件事也适用于 derived classes，因为每一个 derived class object 都是一个 base class object。

## Item 33: 避免覆盖（hiding）“通过继承得到的名字”
