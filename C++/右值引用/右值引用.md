篇幅较长，算是从0开始介绍的，请耐心看~

该篇介绍了左值和右值的区别、左值引用的概念、右值引用的概念、std::move()的本质、移动构造函数、移动复制运算符和RVO。

## 1 左值和右值

首先来介绍一下左值和右值的区别，内容参考于《C++ primer 5th》4.1。

**当一个对象被用作右值的时候，用的是对象的值（内容）；当对象被用作左值的时候，用的是对象的身份（在内存中的位置）。**（受对象用途影响）

原则：在需要右值的地方可以用左值代替，但是不能把右值当成左值使用（对象移动除外）。当一个左值代替右值使用时，实际使用的是它的内容（值）。

在C++中，左值可以位于赋值语句的左侧，但右值不可以。有一种情况下，左值不能位于赋值语句的左侧，即以常量对象为代表的某些左值。例：

```c++
const int MAX_LEN = 10; // MAX_LEN是左值
MAX_LEN = 5;            // 错误：试图向const对象赋值。 左值不能位于赋值语句的左侧。
```

网络上有一种说法，左值位于赋值语句的左侧，右值位于右侧（这句话不是相对于变量说的）。例：

```c++
int a = 1;    // a是左值（在内存中的位置）， 1是右值（内容），不能给1赋值
int y = a;    // 用左值代替右值，把内容赋值给y。a依然是左值，可以取地址。但是在该表达式中，左值代替右值使用，所以赋值语句右边也可以说是右值，只不过是a的内容。
```

用到左值的运算符：

- 赋值运算符需要一个（非常量）左值作为其运算对象，得到的结果也仍然是一个左值。
- 取地址符作用于一个左值运算对象，返回一个指向该运算对象的指针，这个指针是一个右值（无法放到赋值语句的左侧，进行赋值）。
- 内置解引用运算符、下标运算符、迭代器解引用运算符、string和vertor的下标运算符。
- 内置类型和迭代器的递增递减运算符作用于左值运算对象，其前置版本所得的结果也是左值。

总结：常量、有地址的变量一定是左值，临时值是右值。左值可以当成右值用。

## 2 左值引用

左值引用：引用是变量的别名，指向左值。但const左值引用除外，由于const的不可变性，所以const引用可以指向右值，我们经常使用const引用作为函数参数传递。例：

```
int a = 1;    
int &b = a;			// 正确
int &c = 10;		// 错误：10是右值。
const int &d = 10;  // 正确 
```

关于左值引用的更多内容，可以参考我的另一篇文章（建议看完左值引用再来看这篇）：深入理解左值引用 https://zhuanlan.zhihu.com/p/390611356

## 3 右值引用

内容参考于《C++ primer 5th》13.6。

### 3.1 出现

在重新分配内存的过程中，从旧元素将元素拷贝到新内存是不必要的，更好的方式是移动元素。还有一些可以移动但不能拷贝的类，如：IO类和unique_ptr类。索所以，为了支持移动操作，新标准引入了一种新的引用类型——**右值引用**。

### 3.2 概念

右值引用：必须绑定到右值的引用，且**只能绑定到一个将要销毁的对象**。所以，可以自由地将一个右值引用地资源“移动”到另一个对象中。通过&&获得右值引用（也可以说，接管对象的控制权）。例：

```c++
int a = 1;
int &b = a;   			// 正确：左值引用，a是左值
int &&c = a;  			// 错误：右值引用，不能绑定到一个左值上
int &d = a*3; 			// 错误：左值引用，a*3是右值
const int &e = a*3; 	// 正确：左值引用，const引用可以绑定到一个右值上
int &&f = a*3;     		// 正确：右值引用，a*3是右值
int &&g = 10;			// 正确：右值引用，10是右值
```

变量表达式依然是左值，例：

```c++
int &&a = 10;   // 正确：10是右值
int &&b = a; 	// 错误：即使a是右值引用，但a依然是左值，a不是临时对象
```

从上面的例子，可以看到：左值引用有持久的状态；右值要么是字面常量，要么是在表达式求值过程中创建的临时对象。

由于右值引用只能绑定到临时对象（不管编译器怎么做，但这个我们需要遵守），所以：

- 所引用的对象将要被销毁

- 该对象没有其他用户（保证安全，在使用的时候一定要特别确定这一点）

而且，使用右值的代码可以自由地接管所引用的对象的资源。

在移动之后，要谨慎操作原对象，一般不操作，因为我们不确定移动操作做了哪些内容，原对象也是处于一种不确定的状态。

我们一般不会使用const右值引用，当然，编译器也不会报错。（和右值引用的目的冲突）

### 3.3 应用

#### 3.3.1 右值引用绑定到左值上

在左值与右值的区别中，我们知道左值是可以代替右值的。那么右值引用是不是可以引用到“左值”上呢？答案是可以的，新版标准库给我们提供了一个函数——move()，该函数的含义是：告诉编译器，虽然我们有一个左值，但是我们希望可以像右值一样处理。例：

```c++
#include <utility>
int a = 1;
int &&c = a;  			// 错误：右值引用，不能绑定到一个左值上
int &&h = std::move(a); // 正确：使用std::move()把a当成右值处理。
```

#### 3.3.2 std::move()本质

首先，我们来看一下std::move源码：

```c++
// xtr1common文件
	// STRUCT TEMPLATE remove_reference
template<class _Ty>
	struct remove_reference
	{	// remove reference
	using type = _Ty;
	};

// xtr1common文件
template<class _Ty>
	using remove_reference_t = typename remove_reference<_Ty>::type;

// type_traits文件
		// FUNCTION TEMPLATE move
template<class _Ty>
	_NODISCARD constexpr remove_reference_t<_Ty>&&
		move(_Ty&& _Arg) noexcept
	{	// forward _Arg as movable
	return (static_cast<remove_reference_t<_Ty>&&>(_Arg));
	}
```

从源码中，可以得知：std::move既可以传入一个左值也可以传入一个右值，如果是左值（这里，传入的左值的类型是T&而不是T），则将一个左值转换成右值（`_Ty& &&`会被折叠成`_Ty&`， type是T）。其实std::move的作用仅仅是将左值转换成右值，也就是一次类型转换：`static_cast<_Ty&&>(_Arg)`。也就是说，std::move其实不“移动”，只是转换成右值引用。例：

```c++
int v = 5;									// 正确：v是左值
int &&r_ref = 8;							// 正确：re_ref引用右值
int &&r_ref_move = std::move(v); 			// 正确：r_ref_move=5, v是左值，std::move做了一次类型转换。_Ty& &&会被折叠成_Ty&。（赋值之后，v的值是不确定的，这个受移动赋值运算符里的内容影响）
int &&r_ref_move2 = std::move(r_ref_move);	// 正确：r_ref_move2=5, r_ref_move是左值，std::move做了一次类型转换
int &&r_ref_move3 = std::move("hello");		// 正确：可以给一个std::move传递一个右值
v = 9;										// 正确：v、r_ref_move、r_ref_move2=9
r_ref_move = 10;							// 正确：v、r_ref_move、r_ref_move2=10
r_ref_move2 = 11;							// 正确：v、r_ref_move、r_ref_move2=11
```

#### 3.3.3 移动构造函数和移动赋值运算符

接下来，介绍一下移动构造函数和移动赋值运算符，这两个是右值引用的典型例子。

**移动拷贝构造函数**

移动构造函数中的第一个参数是该类类型的一个右值引用，本质是在转移对象的控制权。所以我们需要先更新新对象的指针，然后把原对象中的指针置为nullptr。

下面看一个例子：

```c++
// 不考虑规范，仅仅是一个例子
class MyClass
{
	// 移动构造函数
    // noexcept不抛出异常
	MyClass(MyClass &&c) noexcept;
	// ...
private:
	std::string *p;
};

// 接管c中的内存，不分配任何新内存（与拷贝构造函数不同）
MyClass::MyClass(MyClass &&c) noexcept
	: p(c.p)
{
	// 对c运行析构函数是安全的（确保原对象进入可析构的状态）
	c.p = nullptr;
}
```

**移动赋值运算符**

移动赋值运算符写法如下：

```c++
// 不考虑规范，仅仅是一个例子
class MyClass
{
	// 移动赋值运算符
	MyClass& operator=(MyClass &&c) noexcept;
	// ...
private:
	std::string *p;
};

MyClass& MyClass::operator=(MyClass &&c) noexcept
{
      // 检查自赋值：不能在使用右侧运算对象的资源之前旧释放左侧运算对象的资源（可能是相同的资源）
	if (this != &c) {
        free();  //释放已有元素
		p = c.p;
		c.p = nullptr;
	}
    return *this;
}
```

**关于异常**

不抛出异常的移动构造函数和移动赋值运算符必须标记为noexcept。

但是，移动构造函数也可能出现异常，这个时候就不能声明为noexcept。比如：vector的增长，可能会导致内存的重新分配。使用移动构造函数和拷贝构造函数的结果会不同：
- 如果使用移动构造函数，很有可能移动了部分元素后出现异常，这样会导致——旧空间中的元素已经被改变，新空间中未构造的元素尚不存在。
- 如果使用拷贝构造函数出现异常，则很容易处理。当在新内存中构造元素时，旧元素保持不变。如果此时发生异常，vector可以释放新分配的内存并返回，vector原有的元素不变。
- 在重新分配内存的过程中，必须使用拷贝构造函数而不是移动构造函数。（这就是noexcept的作用，让编译器决定是否调用移动构造函数）

**合成的移动操作**

我们需要注意：如果一个类没有移动操作，类会使用对应的拷贝操作来代替移动操作。编译器可以将一个T&&转换成const T&，然后调用拷贝构造函数。所以，并不是使用了移动就一定可以提升性能。当然，我们可以在自定义类中自己声明定义移动操作。

那么如果我们没有声明定义移动操作，编译器什么时候合成默认的移动函数呢？答案是：一个类没有定义任何自己版本的拷贝控制成员，且类的每个非static数据成员都可以移动时，合成。具体要求如下（忘记出处了，好像是某个翻译过来的......）：

- 如果发生以下情况，编译器将生成移动构造函数（**move** **constructor**）
  - 用户未声明拷贝构造函数（**copy** **constructor**）
  - 用户未声明拷贝赋值运算符（**copy** **assignment operator**）
  - 用户未声明移动赋值运算符（**move** **assignment operator**）
  - 用户未声明析构函数（**destructor**）
  - 该类未被标记为已删除（**delete**）
  - 所有非static成员均为可移动的（**moveable**）
- 如果发生以下情况，编译器将生成移动赋值运算符（**move** **assignment operator**）
  - 用户未声明拷贝构造函数（**copy** **constructor**）
  - 用户未声明拷贝赋值运算符（**copy** **assignment operator**）
  - 用户未声明移动构造函数（**move** **constructor**）
  - 用户未声明析构函数（**destructor**）
  - 该类未被标记为已删除（**delete**）
  - 所有非static成员均为可移动的（**moveable**）

而且，移动操作永远不会隐式定义为删除的函数。但是，我们如果我们使用=default显示地要求编译器生成默认移动操作，且编译器不能移动所有成员，编译器会将移动操作定义为删除的函数（安全）。

需要注意的几点：

- 如果有类成员的移动构造函数或移动赋值运算符被定义为删除的或是不可访问的，则类的移动构造函数或移动赋值运算符被定义为删除的。
- 如果有类的析构函数被定义为删除的或是不可访问的，则类的移动构造函数被定义为删除的。
- 如果有类的成员是const的或是引用的，则类的移动赋值运算符被定义为删除的。

- 定义了一个移动构造函数或移动赋值运算符的类必须也定义自己的拷贝操作。否则，这些成员默认地被定义为删除的。

三/五原则：定义一个类时，建议定义拷贝构造函数、拷贝赋值运算符、析构函数，当需要拷贝资源时，建议也定义移动构造函数、移动赋值运算符。C++并不要求我们定义所有的操作，但是这些操作通常被看成一个整体。

#### 3.3.4 std::move()的一个例子

来源《C++程序设计语言》

来看一下交换函数：

```c++
// 一种比较常规的写法
template<class T>
void swap(T&a, T&b)
{
	T tmp{a};
	a = b;
	b = tmp;
}

// 当遇到string、vector这类类型的交换，第一种方法的拷贝将会造成很大的花费，所以出现下面的一种写法：
template<class T>
void swap(T&a, T&b)
{
	T tmp{static_cast<T&&>(a)};
	a = static_cast<T&&>(b);
	b = static_cast<T&&>(tmp);
}

// 由于move函数的本质是static_cast<T&&>，所以对上面的函数还可以优化一下写法
template<class T>
void swap(T&a, T&b)
{
	T tmp{std::move(a)};
	a = std::move(b);
	b = std::move(tmp);
}
```

在这个例子中，如果类型T存在移动赋值运算符，那么运算性可能会提高。

## 4 补充—协助完成返回值优化（RVO）

来源：《More Effective C++》条款20、《Effective C++》条款21、《C++标准库》3.1.5

例1：

```c++
X foo()
{
    X x;
    ...
    return x;
}
```

对于例1：

- 如果X有一个可取用的copy或move构造函数，编译器可以选择略去其中的copy版本，即RVO。（平常简单的返回std::move()可能会出错，这要看优化方式以及编译器怎么处理了）
- 否则，如果X有一个move构造函数，X就被moved（搬移）。
- 否则，如果X有一个copy构造函数，X就被copied（复制）。
- 否则，报出一个编译器错误。

例2：

```c++
X&& foo()
{
    X x;
    ...
    return std::move(x);
}
```

对于例2，该函数返回的是一个local nonstatic对象，返回右值引用是有风险的。具体看编译器优化。（当然，最好不这样使用。）

例3：

```c++
// 对于返回一个对象的函数进行优化。
// Rational为分数类，numerator是分子，denominator是分母。

// plan 1：返回指针，但是写法很难看（Rational c = *(a*b)），而且可能会导致资源泄露（忘记删除函数返回的指针）。
const Rational* operator*(const Rational& lhs, const Rational& rhs);

// plan 2：必须付出一个构造函数调用的代价，且可能会导致资源泄露
const Rational& operator*(const Rational& lhs, const Rational& rhs)
{
    Rational* result = new Rational(lhs.numerator() * rhs.numerator(), 
									lhs.denominator() * rhs.denominator())
    return *result;
}

// plan 3：返回引用，在函数退出前，result已经被销毁。所以，引用指向一个不再存活的对象，会很危险且不正确。
const Rational& operator*(const Rational& lhs, const Rational& rhs)
{
    Rational result(lhs.numerator() * rhs.numerator(), 
					lhs.denominator() * rhs.denominator())
    return result;  // 局部非静态对象
}

// 所以，如果函数一定得以值方式返回对象，是无法消除的。所以只能尽可能地降低对象返回的成本，而不是想尽办法消除对象本身。

// plan 4：有效率且正确的方法。虽然我们构造了临时对象，但是C++允许编译器将临时对象优化，使它们不存在。编译器优化后，调用operator*时没有任何临时对象被调用出来。只需要一个constructor（用以产生c的代价）。
const Rational operator*(const Rational& lhs, const Rational& rhs)
{
	return Rational(lhs.numerator() * rhs.numerator(), 
					lhs.denominator() * rhs.denominator())
}

// plan 5：最有效率的做法。使用inline消除调用operator*的函数开销。
inline const Rational operator*(const Rational& lhs, const Rational& rhs)
{
	return Rational(lhs.numerator() * rhs.numerator(), 
					lhs.denominator() * rhs.denominator())
}

Rational a = 10;
Rational b(1,2);
Rational c = a*b;
```

## 5 总结

移动并不**移动**，只是转移控制权。

std::move()只是做了一次类型转换，转换成一个右值引用，然后方便后续操作，比如：构造、赋值等。真正的内存管理，是交由移动构造、移动赋值等移动操作处理的。有没有性能优化，要看有没有移动操作以及移动操作的处理。