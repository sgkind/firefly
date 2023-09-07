# C++面试知识点

1. 使用ODBC API开发数据库应用程序的步骤是什么?

    连接数据源。分配语句句柄。准备并执行SQL语句。获取结果集。提交事务。断开数据源连接并释放环境句柄。

2. 关键字static的作用是什么?

这个简单的问题很少有人能回答完全。在C语言中，关键字static有三个明显的作用：
1). 在函数体，一个被声明为静态的变量，该变量的内存只被分配一次，在这一函数被调用过程中维持其值不变。
2). 在模块内（但在函数体外），一个被声明为静态的变量可以被模块内所用函数访问，但不能被模块外其它函数访问。它是一个本地的全局变量。
3). 在模块内，一个被声明为静态的函数只可被这一模块内的其它函数调用。那就是，这个函数被限制在声明它的模块的本地范围内使用。
4). 在类中的static成员变量属于整个类所拥有，对类的所有对象只有一份拷贝；
5). 在类中的static成员函数属于整个类所拥有，这个函数不接收this指针，因而只能访问类的static成员变量。 

3. 关键字const是什么含义
只要能说出const意味着“只读”就可以了。
`const int a;` ，a是一个常整型数
`int const a;` ，a是一个常整型数
`const int *a;`  a是一个指向常整型数的指针（也就是，整型数是不可修改的，但指针可以）
`int * const a;`  a是一个指向整型数的常指针（也就是说，指针指向的整型数是可以修改的，但指针是不可修改的）
`int const * a const;` 一个意味着a是一个指向常整型数的常指针（也就是说，指针指向的整型数是不可修改的，同时指针也是不可修改的）

1. 关键字const的作用是为给读你代码的人传达非常有用的信息，实际上，声明一个参数为常量是为了告诉了用户这个参数的应用目的。如果你曾花很多时间清理其它人留下的垃圾，你就会很快学会感谢这点多余的信息。（当然，懂得用const的程序员很少会留下的垃圾让别人来清理的。）
2. 通过给优化器一些附加的信息，使用关键字const也许能产生更紧凑的代码。
3. 合理地使用关键字const可以使编译器很自然地保护那些不希望被改变的参数，防止其被无意的代码修改。简而言之，这样可以减少bug的出现。
作用：
  * 欲阻止一个变量被改变，可以使用const关键字。在定义该const变量时，通常需要对它进行初始化，因为以后就没有机会再去改变它了；
  * 对指针来说，可以指定指针本身为const，也可以指定指针所指的数据为const，或二者同时指定为const；
  * 在一个函数声明中，const可以修饰形参，表明它是一个输入参数，在函数内部不能改变其值；
  * 对于类的成员函数，若指定其为const类型，则表明其是一个常函数，不能修改类的成员变量；
  * 对于类的成员函数，有时候必须指定其返回值为const类型，以使得其返回值不为“左值”。例如：
const classA operator*(const classA& a1,const classA& a2);
operator*的返回结果必须是一个const对象。如果不是，这样的变态代码也不会编译出错：
classA a, b, c;
(a * b) = c; // 对a*b的结果赋值
操作(a * b) = c显然不符合编程者的初衷，也没有任何意义。

4. 关键字volatile有什么含意 并给出三个不同的例子。
一个定义为volatile的变量是说这变量可能会被意想不到地改变，这样，编译器就不会去假设这个变量的值了。精确地说就是，优化器在用到这个变量时必须每次都小心地重新读取这个变量的值，而不是使用保存在寄存器里的备份。下面是volatile变量的几个例子：
1. 并行设备的硬件寄存器（如：状态寄存器）
2. 一个中断服务子程序中会访问到的非自动变量(Non-automatic variables)
3. 多线程应用中被几个任务共享的变量

5. 分别给出BOOL，int，float，指针变量 与“零值”比较的 if 语句（假设变量名为var）
解答：
BOOL型变量：if(!var)
int型变量： if(var==0)
float型变量：const float EPSINON = 0.00001;
if ((x >= – EPSINON) && (x <= EPSINON)
指针变量： if(var==NULL)
数组名
1、数组名作为函数形参时，沦为普通指针。仅仅只是一个指针；在失去其内涵的同时，它还失去了其常量特性，可以作自增、自减等操作，可以被修改。
void Func ( char str[100] )
{
sizeof( str ) = ?   //答案4
} 
2、数组名指代一种数据结构，这种数据结构就是数组；
例如：
char str[10];
cout ＜＜ sizeof(str) ＜＜ endl;
输出结果为10，str指代数据结构char[10]。
3、数组名可以转换为指向其指代实体的指针，而且是一个指针常量，不能作自增、自减等操作，不能被修改；
char str[10];
str++; //编译出错，提示str不是左值
写一个“标准”宏MIN，这个宏输入两个参数并返回较小的一个
#define MIN(A,B) ((A) <= (B) ? (A) : (B))
为什么标准头文件都有类似以下的结构? 
#ifndef __INCvxWorksh
#define __INCvxWorksh
#ifdef __cplusplus   //防止被重复引用
extern “C” {
#endif
#ifdef __cplusplus
}
#endif
#endif /* __INCvxWorksh */

作为一种面向对象的语言，C++支持函数重载，而过程式语言C则不支持。函数被C++编译后在symbol库中的名字与C语言的不同。例如，假设某个函数的原型为： void foo(int x, int y);该函数被C编译器编译后在symbol库中的名字为_foo，而C++编译器则会产生像_foo_int_int之类的名字。_foo_int_int这样的名字包含了函数名和函数参数数量及类型信息，C++就是考这种机制来实现函数重载的。为了实现C和C++的混合编程，C++提供了C连接交换指定符号extern “C”来解决名字匹配问题，函数声明前加上extern “C”后，则编译器就会按照C语言的方式将该函数编译为_foo，这样C语言中就可以调用C++的函数了。
编写类String的构造函数、析构函数和赋值函数，已知类String的原型为：
```
class String
{
 public:
  String(const char *str = NULL); // 普通构造函数
  String(const String &other); // 拷贝构造函数
  ~ String(void); // 析构函数
  String & operate =(const String &other); // 赋值函数
 private:
  char *m_data; // 用于保存字符串
};
```
解答：
```
//普通构造函数
String::String(const char *str) {
  if(str==NULL) {
    m_data = new char[1]; // 得分点：对空字符串自动申请存放结束标志’\0′的空
    //加分点：对m_data加NULL 判断
   *m_data = ‘\0′;
  } else {
    int length = strlen(str);
    m_data = new char[length+1]; // 若能加 NULL 判断则更好
    strcpy(m_data, str);
  }
}
// String的析构函数
String::~String(void) {
  delete [] m_data; // 或delete m_data;
}

//拷贝构造函数
String::String(const String &other) { // 得分点：输入参数为const型
  int length = strlen(other.m_data);
  m_data = new char[length+1]; //加分点：对m_data加NULL 判断
  strcpy(m_data, other.m_data);
}
//赋值函数
String & String::operate =(const String &other) { // 得分点：输入参数为const型
  if(this == &other) //得分点：检查自赋值
    return *this;
  delete [] m_data; //得分点：释放原有的内存资源
  int length = strlen( other.m_data );
  m_data = new char[length+1]; //加分点：对m_data加NULL 判断
  strcpy( m_data, other.m_data );
  return *this; //得分点：返回本对象的引用
}
```

区别
引用与指针有什么区别?
1) 引用必须被初始化，指针不必。
2) 引用初始化以后不能被改变，指针可以改变所指的对象。
3) 不存在指向空值的引用，但是存在指向空值的指针。

全局变量和局部变量在内存中是否有区别? 答 、全局变量储存在静态数据区，局部变量在堆栈中。

类成员函数的重载、覆盖和隐藏区别?
a.	成员函数被重载的特征：
（1）相同的范围（在同一个类中）；（2）函数名字相同；（3）参数不同；（4）virtual 关键字可有可无。
b.覆盖是指派生类函数覆盖基类函数，特征是：
（1）不同的范围（分别位于派生类与基类）；（2）函数名字相同；（3）参数相同；（4）基类函数必须有virtual 关键字。
c.“隐藏”是指派生类的函数屏蔽了与其同名的基类函数，规则如下：
（1）如果派生类的函数与基类的函数同名，但是参数不同。此时，不论有无virtual关键字，基类的函数将被隐藏（注意别与重载混淆）。
（2）如果派生类的函数与基类的函数同名，并且参数也相同，但是基类函数没有virtual 关键字。此时，基类的函数被隐藏（注意别与覆盖混淆）

Struct与class的区别
struct 的成员默认是公有的，而类的成员默认是私有的。struct 和 class 在其他方面是功能相当的。
从感情上讲，大多数的开发者感到类和结构有很大的差别。感觉上结构仅仅象一堆缺乏封装和功能的开放的内存位，而类就象活的并且可靠的社会成员，它有智能服务，有牢固的封装屏障和一个良好定义的接口。既然大多数人都这么认为，那么只有在你的类有很少的方法并且有公有数据（这种事情在良好设计的系统中是存在的!）时，你也许应该使用 struct 关键字，否则，你应该使用 class 关键字。

枚举与#define的区别
1），#define 宏常量是在预编译阶段进行简单替换。枚举常量则是在编译的时候确定其值。
2），一般在编译器里，可以调试枚举常量，但是不能调试宏常量。
3），枚举可以一次定义大量相关的常量，而#define 宏一次只能定义一个。

exit与return的区别
①exit（）函数在退出程序后会将控制权交回给操作系统
②当通过return语句从一般函数返回时控制权将交给调用该函数的函数
③在main()函数中使用return语句返回后，控制权将交给操作系统，因此在主函数中return语句的功能与exit（）函数功能相同。

指针常量
const char*, char const*, char*const的区别问题几乎是C++面试中每次都会有的题目。
把一个声明从右向左读。
char * const cp; ( * 读成 pointer to )
cp is a const pointer to char
const char * p;
p is a pointer to const char;
char const * p;
同上因为C++里面没有const*的运算符，所以const只能属于前面的类型。

链表和数组
二者都属于一种数据结构
从逻辑结构来看
1. 数组必须事先定义固定的长度（元素个数），不能适应数据动态地增减的情况。当数据增加时，可能超出原先定义的元素个数；当数据减少时，造成内存浪费；数组可以根据下标直接存取。
2. 链表动态地进行存储分配，可以适应数据动态地增减的情况，且可以方便地插入、删除数据项。（数组中插入、删除数据项时，需要移动其它数据项，非常繁琐）链表必须根据next指针找到下一个元素
从内存存储来看
1. (静态)数组从栈中分配空间, 对于程序员方便快速,但是自由度小
2. 链表从堆中分配空间, 自由度大但是申请管理比较麻烦 
从上面的比较可以看出，如果需要快速访问数据，很少或不插入和删除元素，就应该用数组；相反， 如果需要经常插入和删除元素就需要用链表数据结构了。
Sizeof和strlen
① sizeof是运算符，计算数据所占的内存空间；strlen（）是一个函数，计算字符数组的字符数；
② sizeof可以用类型作参数；strlen（）只能用char*作参数，必须是以’/0’结束
③ 数组做sizeof的参数不退化,传递给strlen就退化为指针了;
④ sizeof操作符的结果类型是size_t，它在头文件中typedef为unsigned int类型。该类型保证能容纳实现建立的最大对象的字节大小


/为了实现链式操作，将目的地址返回，加3分！ 
char * strcpy( char *strDest, const char *strSrc ) 
{ 
assert( (strDest != NULL) && (strSrc != NULL) ); 
char *address = strDest; 
while( (*strDest++ = * strSrc++) != ‘\0’ ); 
return address; 
} 

int strlen( const char *str ) //输入参数const 
{ 
assert( strt != NULL ); //断言字符串地址非0 
int len; 
while( (*str++) != ‘\0′ ) 
{ 
len++; 
} 
return len; 
} 
