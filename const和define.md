程序运行分为4个步骤























### const、宏

#### const介绍

- C++中的const的目的是通过编译器来保证对象的常量性，强制编译器将所有可能违背const对象的常量性的操作都视为error。

- const修饰变量

  - 用const修饰变量的语义是要求编译器去阻止所有对该变量的赋值行为。因此，必须在const变量初始化时就提供给它初值。

- const修饰指针与引用

  - const修饰引用时，其意义与修饰变量相同。

  - ```C++
    const  int  *p1; // p1 is a non-const pointer and points to a const int
    int  * const  p2; // p2 is a const pointer and points to a non-const int
    const  int  * const  p3; // p3 is a const pointer and points to a const it
    const  int  *pa1[10]; // pa1 is an array and contains 10 non-const pointer point to a const int
    int  * const  pa2[10]; // pa2 is an array and contains 10 const pointer point to a non-const int
    const  int  (* p4)[10]; // p4 is a non-const pointer and points to an array contains 10 const int
    const  int  (*pf)(); // pf is a non-const pointer and points to a function which has no arguments and returns a const int
    ```

    

  - 指针自身为const表示不可对该指针进行赋值，而指向物为const则表示不可对其指向进行赋值。因此可以将引用看成是一个自身为const的指针，而const引用则是const Type * const指针。

  - 指向为const的指针是不可以赋值给指向为非const的指针，const引用也不可以赋值给非const引用，但反过来就没有问题了，这也是为了保证const语义不被破坏。

  - C++类中的this指针就是一个自身为const的指针，而类的const方法中的this指针则是自身和指向都为const的指针。

- 类中的const成员变量

  - 类中的const成员变量可分为两种：非static常量和static常量。

    - 非static常量：类中的非static常量必须在构造函数的初始化列表中进行初始化，因为类中的非static成员是在进入构造函数的函数体之前就要构造完成的，而const常量在构造时就必须初始化，构造后的赋值会被编译器阻止。

      ```c++
      class  B {
      public :
           B(): name( "aaa" ) {
               name = "bbb" ; // !error
           }
      private :
           const  std::string name;
      };
      ```

    - static常量：static常量是在类中直接声明的，但要在类外进行唯一的定义和初始值，常用的方法是在对应的.cpp中包含类的static常量的定义：

      ```c++
      // a.h
      class  A {
           ...
           static  const  std::string name;
      };
       
      // a.cpp
      const  std::string A::name( "aaa" );
      ```

- const修饰函数

  - C++中可以用const去修饰一个类的非static成员函数，其语义是保证该函数所对应的对象本身的const性。在const成员函数中，所有可能违背this指针const性（const成员函数中的this指针是一个双const指针）的操作都会被阻止，如对其它成员变量的赋值以及调用它们的非const方法、调用对象本身的非const方法。但对一个声明为mutable的成员变量所做的任何操作都不会被阻止。这里保证了一定的逻辑常量性。
  - const 成员函数可以使用类中的所有成员变量，但是不能修改它们的值，这种措施主要还是为了保护数据而设置的。const 成员函数也称为常成员函数。常成员函数需要在声明和定义的时候在函数头部的结尾加上 const 关键字。

#### constexpr

- constexpr是C++11中新增的关键字，其语义是“常量表达式”，也就是在编译期可求值的表达式。最基础的常量表达式就是字面值或全局变量/函数的地址或sizeof等关键字返回的结果，而其它常量表达式都是由基础表达式通过各种确定的运算得到的。constexpr值可用于enum、switch、数组长度等场合。

- constexpr所修饰的变量一定是编译期可求值的，所修饰的函数在其所有参数都是constexpr时，一定会返回constexpr。

- constexpr还能用于修饰类的构造函数，即保证如果提供给该构造函数的参数都是constexpr，那么产生的对象中的所有成员都会是constexpr，该对象也就是constexpr对象了，可用于各种只能使用constexpr的场合。注意，constexpr构造函数必须有一个空的函数体，即所有成员变量的初始化都放到初始化列表中。

- 好处：

  - 是一种很强的约束，更好地保证程序的正确语义不被破坏。

  - 编译器可以在编译期对constexpr的代码进行非常大的优化，比如将用到的constexpr表达式都直接替换成最终结果等。
  - 相比宏来说，没有额外的开销，但更安全可靠。

#### define和const的区别

- 参考[const与#define相比，区别和优点超详解总结！](https://blog.csdn.net/weibo1230123/article/details/81981384)

- 区别：
  - 起作用阶段： \#define是在编译的预处理阶段起作用，而const是在编译、运行的时候起作用。
  - 起作用的方式： #define只是简单的字符串替换，没有类型检查。而const有对应的数据类型，是要进行判断的，可以避免一些低级的错误。 
  - 存储方式：#define只是进行展开，有多少地方使用，就替换多少次，它定义的宏常量在内存中有若干个备份；const定义的只读变量在程序运行过程中只有一份备份。
  - 代码调试的方便程度： const常量可以进行调试的，define是不能进行调试的，因为在预编译阶段就已经替换掉了。
- const优点：
  - const常量有数据类型，而宏常量没有数据类型。编译器可以对前者进行类型安全检查。而对后者只进行字符替换，没有类型安全检查，并且在字符替换可能会产生意料不到的错误。
  - 有些集成化的调试工具可以对const常量进行调试，但是不能对宏常量进行调试。
  - const可节省空间，避免不必要的内存分配，提高效率

#### define和constexpr的区别

- constexpr更节省空间

- 宏由预处理器定义，并且每次发生时都被替换为代码。 预处理器很笨，不能理解C++语法或语义。 宏会忽略诸如名称空间，类或功能块之类的作用域，因此您不能在源文件中为其他任何名称使用名称。 

- 唯一喜欢使用宏的时间是当需要预处理器理解其值时，以便在 #if < code>条件，例如

  ```C++
   #define MAX_HEIGHT 720 
   #if MAX_HEIGHT< 256 
  	height_type = unsigned char; 
   #else 
  	height_type = unsigned int; 
   #endif
  ```

  

#### constexpr和const区别

- const 变量的初始化可以延迟到运行时，而 constexpr 变量必须在编译时进行初始化。所有 constexpr 变量均为常量，因此必须使用常量表达式初始化。

#### const引用修改

- const实际上保证的，并不是变量的值不得改动，而是变量指向的那个内存地址所保存的数据不得改动。对于简单类型的数据（数值、字符串、布尔值），值就保存在变量指向的那个内存地址，因此等同于常量。但对于复合类型的数据（主要是对象和数组），变量指向的内存地址，保存的只是一个指向实际数据的指针，`const`只能保证这个指针是固定的（即总是指向另一个固定的地址），至于它指向的数据结构是不是可变的，就完全不能控制了。因此，将一个对象声明为常量必须非常小心。

#### 常量指针

- 常量指针本质是指针，常量修饰它，表示这个指针乃是一个指向常量的指针（变量）。指针指向的对象是常量，那么这个对象不能被更改。
- const int *p; int const *p;
- 常量指针说的是不能通过这个指针改变变量的值，但是还是可以通过其他的引用来改变变量的值的。常量指针可以指向其他的地址。

#### 指针常量

- 指针是形容词，常量是名词。这回是以常量为中心的一个偏正结构短语。那么，指针常量的本质是一个常量，而用指针修饰它，那么说明这个常量的值应该是一个指针。指针常量的值是指针，这个值因为是常量，所以不能被赋值。
- int* const p;









宏难道就不用了嘛？