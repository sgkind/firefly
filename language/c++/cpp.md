C++知识点
===============

* 拷贝构造函数

   拷贝构造函数被用来“从同型对象初始化自我对象”；
   copy assignment操作符被用来“从另一个同型对象拷贝其值到自我对象”。
   ```cpp
   Widget w1;      // 调用default构造函数
   Widget w2(w1);  // 调用copy构造函数
   Widget w3 = w2; // 调用copy构造函数
   w1 = w2;        // 调用copy assignment操作符
   ```
   pass by value意味着调用copy构造函数
* 目前C++支持的编程形式：过程形式、面向对象形式、函数形式、泛型形式（即Template）
  和元编程形式。
* 尽量以const、enum和inline替换#define -> 宁可编译器替换预处理器。
* 类内常量要被声明为satic。

    如果一个常量是class专属常量又是static且为整数类型，要在实现文件中提供定义式，
    并且此定义式不提供初值。
* 如果关键字const出现在*左边，表示被指物是常量，如果出现在*右边，表示指针自身是
  常量。如果出现在*两边，表示被指物和指针两者都是常量。
* 基类更早于派生类被初始化，类的成员变量总是以其声明次序被初始化。
* 编译器产生的析构函数是个no-virtual，除非这个类的基类自身声明为virtual析构函数.
* 当derived class对象经由一个base class指针被删除，而该base class带有一个non-
  virtual析构函数，其结果未定义——实际执行时发生的是对象的derived成分没被销毁.
* 任何class只要带一个virtual函数，都几乎确定应该也有一个virtual析构函数.
* 析构函数绝对不要抛出异常。
* 绝不在析构函数和析构函数中调用virtual函数.
* 在类内声明泛化copy构造函数（是个member template）并不会阻止编译器生成它们自己
  的copy构造函数（一个non-template), 所以如果你想要控制copy构造的方方面面，你必
  须同时声明泛化copy构造函数和“正常的”copy构造函数。
* 当我们编写一个class template，而它所提供之“与此template相关的”函数支持“所有参数
  之隐式类型转化”时，请将那些函数定义为“class template内部的friend函数”。
* 如果operator new接受的参数除了一定会有的那个size_t之外还有其他，这便是个所谓的
  placement new。因此，上述的operator new是个placement版本。众多placement new版本
  中特别有用的一个是“接受一个指针指向对象该构造函数之处”。






### 待解决问题

-[ ]函数模板与类模板的区别
-[ ]4个cast函数
-[ ]智能指针的相关知识
