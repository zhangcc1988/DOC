### 类

```c++
#include <iostream>
using namespace std;

class A
{
private:                 //  访问修饰符（private/public/protected）
    string name;         //  变量
public:
    void print();        //  成员函数
    A(string nameStr);   //  构造
};  // 分号

void A::print(){
    cout << name << endl;
}

A::A(string n){
    name = n;
}

int main()
{
    A a("小明");
    a.print();
    return 0;
}

```

##### 成员函数

- 类内

- 类外

  ```c++
  class A
  {
  private:
      string name;
  public:
      void print(); // 类外
      A(string nameStr);
  
      void syaHi(){ // 类内
          cout << "hello:" << name << endl;
      }
  };
  
  void A::print(){  // 类外（范围解析运算符 :: 定义该函数）
      cout << name << endl;
  }
  ```

##### 构造函数

```c++
class A
{
private:
    string name = "小刚"; // 默认值 
public:   
    A(string nameStr);  // 1 
    A(){};              // 2
    void print(){
         cout << "hello" << name << endl;
    }
};

A::A(string n){         // 1
    name = n;
}

int main()
{
    // A a = A();
    // A a;            // 1

    // A a = A("小明");
    // A a ("小明");    // 2
    // A a {"小明"};    // 2
    
    // a.print();
    return 0;
}

// 1,2:简写
```

##### explicit 构造函数

当构造函数只有一个参数时，会进行自动隐式转换；explicit可以杜绝隐式转换

隐式转换：A a = 1; 一个参数 就可以返回一个对象

```c++
class A
{
private:
    int age;
public:
    explicit A(int a){
        age = a;
    }
};


int main()
{
    A a = 1;  // 错了  explicit 不允许隐式转换
    A a(1);   // 正确
    
    return 0;
}
```



##### 拷贝构造

拷贝构造函数通常用于：

- 赋值
- 传参
- 返回值 

```c++
class A
{
public:
    A(const A &a){        // 定义
        name = a.name;
        cout << "run" << endl;
    }

    A(string n){
        name = n;
    }

    string get(){
        return this ->name;
    }
private:
    string name;
};

int main()
{
    A a("jon");
    A a1 = a;          // 执行
    cout << a1.get() << endl;
    return 0;
}
```



##### 析构函数

析构函数的名称与类的名称是完全相同的，只是在前面加了个波浪号（~）作为前缀，它不会返回任何值，也不能带有任何参数。析构函数有助于在跳出程序（比如关闭文件、释放内存等）前释放资源。

```c++
class A
{
public:
    ~A(){
        cout << "~A run" << endl;  
    }
};

int main()
{
    A a;
    return 0;
}
```



###### virtual 析构

**virtual析构函数好处：当父类指针指向子类对象时，执行释放操作，子类对象也会被释放掉**

1：没有virtual，delete f 不会释放 son 对象

```c++
class Father
{
public:
    virtual ~Father()            // 1
    {
        cout << "~Father run" << endl;
    }
};

class Son : public Father
{
public:
    ~Son()
    {
        cout << "~son run" << endl;
    }
};

int main()
{
    Father * f = new Son();
    delete f;
    return 0;
}

```



##### 类内初始化

```c++
class A
{
private:
    int age;
    string name;
public:
    explicit A():age{12},name{"jon"} // 给初值 
    {
    }
    void getAge(){
        cout << name << age << endl;
    }
};

int main()
{
    A a;
    a.getAge();
    return 0;
}
```

##### class 与 struct

```c++
struct A
{
  // 默认是public
};

等同于:

class A
{
	public:
}

```

##### 继承

- 子类可以访问父类中所有的非私有成员（public，protected）
- main方法中子类不可以访问父类的protected成员 
- 类的继承方式（public，protected，private），默认是 protected;

```c++
class A{};
class B : public A{};
```



######  多继承

```c++
class A{};
class B{};
class C : public A, public B{};
```

###### 虚继承

- 1：虚继承 
- 如果没有虚继承，2会不知道是从B,还是C中获得变量a;(从A中继承)

```c++
class A
{
public:
    int a = 1;
};

class B : virtual public A{}; // 1
class C : virtual public A{}; // 1
class D : public B, public C
{
    public:
    void print(){
        cout << a << endl;    // 2
    }
};

int main()
{
    D d;
    d.print();
    return 0;
}
```



##### 内部类

定义嵌套类的初衷是建立仅供某个类的成员函数使用的类类型。目的在于隐藏类名，减少全局的标识符，从而限制用户能否使用该类建立对象。这样可以提高类的抽象能力，并且强调了两个类（外围类和嵌套类）之间的主从关系。

```c++
#include <iostream>
#include <string>

using namespace std;

class A
{
public:
    class B
    {
    public:
        B(const char *str)
        {
            cout << "construct B" << str << endl;
        }
        void pirntB();
    };
    A() : b(" in class A")
    {
        cout << "construct A" << endl;
    }
    void printA();
    B b;
};

void A::B::pirntB()
{
    cout << "fn for B" << endl;
}

void A::printA()
{
    cout << "fn for A" << endl;
}

int main()
{
    A a;
    a.b.pirntB();
    a.printA();

    A::B b(" create!!!");
    return 0;
}
```

- 从作用域的角度来看，嵌套类与外围类是两个完全独立的类，只是主从关系，二者不能相互访问，也不存在友元关系。

##### 局部类

在一个函数体内定义的类称为局部类。局部类可以定义自己的数据成员和函数成员。它也是一种作用域受限的类。

```c++
#include <iostream>
#include <string>

using namespace std;

int global_v = 100;

void func()
{
    static const int part_v = 10;
    class A
    {
    public:
        int num;
        A(int a)
        {
            num = a;
        }
        void printA()
        {
            cout << "global value:" << global_v << endl;
            cout << num << endl;
        }
    };

    A a(part_v);
    a.printA();
}

int main()
{
    func();
    return 0;
}
```

- 局部类只能在定义它函数内部使用，在其他地方不能使用；
- 局部类的所有成员函数都必须定义在类体内，因此在结构上不是特别灵活；
- 在局部类的成员函数中，可以访问上级作用域的所有变量，如函数局部变量、全局变量等；
- 局部类中不能定义静态数据成员，因为这种数据成员的初始化无法完成，静态成员数据的定义和初始化必须放在全局作用域。



##### 抽象类（接口）

```c++
class A
{
public:
    virtual void print() = 0; // 纯虚函数
};
```

```c++
class A
{
public:
    virtual void print() = 0;
    int getAge(){
        cout << age << endl;
        return age;
    }
protected:
    int age = 10;
};

class B : public A
{
    public:
    void print(){
        cout << "hello" << endl;
    }
};

int main()
{
    A * a = new B;
    a->print();
    a->getAge();
    return 0;
}
```

- 如果类中至少有一个函数被声明为***纯虚函数***，则这个类就是***抽象类***



##### 重载，重写，隐藏

###### 重载：

- 同一个类中参数列（参数的类型，个数，顺序不同）的同名函数，与返回类型无关；

```c++
class A
{
public:
    void say(){};
    void say(string str){};
};
```

```c++
struct A
{
    void operator()()    // （）操作符重载
    {
        cout << "hi\n";
    }

    void operator+(const A & a){   // + 操作符重载
        cout << "++" << endl;
    }
};

int main()
{
    A a;
    a();
    A b;
    a + b;
    return 0;
}
```





###### 重写：

- 1：父类方法必需是virtual修饰（虚函数）；
- 2：子类方法必需与父类方法一致（函数名，参数列表，返回值类型）；

```c++
class A
{
public:
    virtual void say()               //  1
    {
        cout << "A say" << endl;
    };
};

class B : public A
{
public:
    void say()                      // 2
    {
        cout << "B say" << endl;
    }
};

int main()
{
    A *a = new B;
    a->say();
    return 0;
}
```

###### 隐藏

```c++
class A
{
public:
    void say(string name)                  // 1
    {
        cout << "A say" << name << endl;
    };
};

class B : public A
{
public:
    void say()                             // 2
    {
        cout << "B say" << endl;
    }
};

// 只要同名函数，不管参数列表是否相同，基类函数都会被隐藏；(B 类的对象无法调用 A:say(string name))
```



##### static

- 在修饰变量的时候，static 修饰的静态局部变量只执行初始化一次，而且延长了局部变量的生命周期，直到程序运行结束以后才释放。
- static 修饰全局变量/函数的时候，这个全局变量只能在本文件中访问，不能在其它文件中访问，即便是 extern 外部声明也不可以。

```c++
class A
{
private:
    int age = 1;

public:
    static int a;
    static void say()
    {
        cout << "hello" << endl;
    };
};

int A::a = 1;

int main()
{
    cout << A::a << endl;
    A::say();
    return 0;
}
```



##### virtual

- 1：虚函数
- 2：纯虚函数
- 3：虚继承

```c++
class A
{
private:
    int age = 1;

public:
    virtual void say(){};                      // 1
    virtual void say(string str) = 0;          // 2
};

class B : virtual public A                     // 3
{

public:
    void say(string str)
    {
        cout << str << endl;
    };
};

int main()
{
    B b;
    b.say("jon");
    return 0;
}

```



##### const

```c++
class A
{
private:
    const int age = 8; 				// 1
    mutable string name;            // 4
    mutable int score;              // 4
	
public:    
    void setName(string n) const;
    void say(const string n);
    void cgScore(string n, int s) const;  // 4
};


void A::setName(string n) const     // 2
{
    name = n;
};

void A::say(const string n){        // 3
    n = "jon";                      
}

void A::cgScore(string n, int s) const    // 4
{
    name = n;
    score = s;
    cout << name << ":" << score << endl;
}

int main()
{
    A a;
    a.cgScore("jon", 100);
    return 0;
}

- 1：修饰变量
- 2：修饰成员函数（函数不能改变成员变量的值）
- 3：const 修饰的参数 在函数内不可以被改变
- 4：mutable 可以修改 const 修饰的方法或变量
```

##### constexpr

- 编译期执行



##### using

- using  可以打开父类中的隐藏函数
- using 可以用于定义别名，typedef 功能类似

```c++
class B1
{
public:
    void f(int i)
    {
        cout << "B1 f" << endl;
    }
};

class B2
{
public:
    void f(double i)
    {
        cout << "B2 f" << endl;
    }
};

class D : public B1, public B2
{
private:
    int age;
public:
    using B1::f;
    using B2::f;
    void f(char c)
    {
        cout << "D f" << endl;
    }
    
    using m_int = int;  // 1
    typedef int n_int;  // 2
    n_int getAge() const
    {
        return age;
    }
    void setAge(m_int a)
    {
        age = a;
    }
};

int main()
{
    D d;
    d.f(1);
    d.f(1.1);
    d.f('d');
    return 0;
}
```



##### friend

```c++
class A
{
    
private:
    int age;
    friend class B;// 申明B是A的友元类（这里放在private,protected都行）

public:
    friend void printAge(A b); // 申明友元函数
};

void printAge(A a)
{
    a.age = 10;  //  因为是友元函数可以访问A的private变量
    cout << a.age << endl;
}


class B : public A
{

public:
    void findA(A &a);
};

void B::findA(A &a)
{
    a.age = 15;   //  因为是B是A的友元类，可以访问A的所有成员 
    cout << a.age << endl;
}

int main()
{
    A a;
    printAge(a);

    B b;
    b.findA(a);

    return 0;
}

// 类A的友员函数，友员类可以访问类A的所有成员
```



##### final（同java）

```c++
class A final{};

class B : public A{};
```

- final 修饰的类A 不可以被继承	

```c++
class A
{
public:
    virtual void print() final{}; //1
};

class B : public A
{
    void print() override{};
};
```

- 1：final 修饰的方法不可以被重写

##### delete

```c++
class A
{
public:
    A(const A& a) = delete;
    A(){};
};

// delete 方法禁用（拷贝，赋值禁用）
```

##### default

```c++
class Fruit
{
public:
    Fruit() = default;
    Fruit(int n1) : a1(n1) {}

private:
    int a1;
};

// default 函数，编译器将为default函数自动生成"函数体"
```



### 运算符重载

```
函数类型 operator 运算符名称 (形参表列)
{
	对运算符的重载处理
}
```

### 模板

模板定义以关键字template开始，后跟以个**模板参数列表**，这是一个逗号分隔的一个或多个模板参数的列表，用尖括号括起来。

模板参数表示在类或函数定义中用到的**类型或值**，在使用模板时，我们显式或隐式指定模板实参，将其绑定到模板参数上。

```c++
template<typename T,typename M>   // typename,class 都行
```

```c++
template<typename T>                                  // 函数模板
bool compare(const T& value1, const T& value2)
{
    return value1 == value2;
}
```

```c++
template<typename T>                                  // 类模板
class Printer
{
public:
    Printer<T>(T value):value_(value) {};
private:
    T value_;
};
```

```c++
template<int N,int M>              // 1                   
int compare(const char (&value1)[N], const char(&value2)[M])
{
    return strcmp(value1,value2);
}

 
int main()
{
    cout << compare("11", "112") << endl;
    system("pause");
}

//1 非类型模板参数(非类型模板参数的模板实参必须是常量表达式:整型或者是一个指向对象或函数类型的指针或引用)

```

```c++
template<typename T,typename F=less<int>> // 1
bool compare(const T & value1, const T & value2, F func = F())
{
    return func(value1, value2);
}

template<typename T=int> // 2
class MyClass {
public:
    T value = {};
};

// 1,2 为函数和类模板提供默认实参
```

```c++
template <typename T1,typename T2,typename T3>
T1 getResult(T2 value1, T3 value2)
{
    return value1+value2;
}

int main()
{
    cout <<getResult<int>(1,2)<< endl;
    cout << getResult<int>(1.0, 2.2) << endl;
    cout << getResult<string>(string("hello "), string("world!")) << endl;
    system("pause");
}

// 指定显式模板实参(可以只指定前面的一部分实参，后面的可以让编译器推断。)
```



### 基础部份

##### 字符串

```c++
const char *c = "hello";
char c_list[] = "hello";

 while (*c != '\0')
 {
     cout << *c << endl;
     c ++;
 }
```

- 字符串以“\0”结尾

##### 引用

- 左值**可以取地址、位于等号左边**；而右值**没法取地址，位于等号右边**。

  - int a = 5;
  - a可以通过 & 取地址，位于等号左边，所以a是左值。
  - 5位于等号右边，5没法通过 & 取地址，所以5是个右值。

- ###### **左值引用**

  - ```c++
    int a = 5;
    int &ref_a = a; // 左值引用指向左值，编译通过
    
    const int &ref_a = 5;  // const左值引用是可以指向右值的：
    const int & ref_a = a; // 也可以指向左值
    ```

- ###### **右值引用**

  - ```c++
    int &&ref_a_right = 5; // ok
    // 右值引用的标志是&&
    // 可以指向右值，不能指向左值：
    ```

- ##### **std::move**

  - ```c++
    // 形参是个右值引用
    void change(int&& right_value) {
        right_value = 8;
    }
    
    int main() {
        int a = 5; // a是个左值
        int &ref_a_left = a; // ref_a_left是个左值引用
        int &&ref_a_right = std::move(a); // ref_a_right是个右值引用
     
        change(a); // 编译不过，a是左值，change参数要求右值
        change(ref_a_left); // 编译不过，左值引用ref_a_left本身也是个左值
        change(ref_a_right); // 编译不过，右值引用ref_a_right本身也是个左值
         
        change(std::move(a)); // 编译通过
        change(std::move(ref_a_right)); // 编译通过
        change(std::move(ref_a_left)); // 编译通过
     
    }
    ```

- ##### **完美转发 std::forward**

  - ```c++
    void change2(int&& ref_r) {
        ref_r = 1;
    }
     
    void change3(int& ref_l) {
        ref_l = 1;    
    }
    
    
    int main() {
       int && ref_r = 0;
       change3(ref_r); 
        // ok，change3的入参是左值引用，需要接左值，ref_r是左值，编译通过
       change2(std::forward<int &&>(ref_r));  
        // ok，std::forward的T是右值引用类型(int &&)，符合条件b，因此u(ref_r)会被转换为右值
       change3(std::forward<int &>(ref_r)); 
        // ok，std::forward的T是左值引用类型(int &)，符合条件a，因此u(ref_r)会被转换为左值
    }
    
    // 左值引用，右值引用都是左值
    // std::forward<T>(u)有两个参数：T与 u。 
    // a. 当T为左值引用类型时，u将被转换为T类型的左值； 
    // b. 否则u将被转换为T类型右值。
    
    ```

    

- https://zhuanlan.zhihu.com/p/335994370

##### 枚举

```c++
enum class Traffic_light : char   // 指定枚举值的基础类型为char，默认int
{
    red,
    green,
    yellow
};

void print(Traffic_light t)
{
    printf("%d\n", t);
}

int main()
{
    Traffic_light l = Traffic_light::green;
    print(l);
    return 0;
}
```

##### 函数指针

```c++
void print(string s)
{
    cout << s << endl;
}

int main()
{
    void (*fnP)(string) = print; // 该函数接收一个string参数 且无返回
    //void (*fnP)(string) = &print;
    //fnP("hello");
    //(*fnP)("hello");
    
    return 0;
}
```

### 异常处理

##### try catch

```c++
    try
    {
        throw "ooooops";
    }
    catch(const char * str)          // 1
    {
        std::cerr << str << '\n';
    }
    catch(...){
        cout << "error" << endl;     // 2
    }
```

- 1：异常类型也能是基本类型，例如int，char*
- 2：使用catch(...)能捕获所有异常

```c++
   try
    {
        throw std::exception();
    }
    catch(const std::exception& e)
    {
        std::cerr << e.what() << '\n';
    }
```

- 3：捕获标准异常
- 4：C++标准只要求try catch捕获throw出来的异常，并不要求捕获系统异常(如被0除，段错误，CPU异常等)。



##### noexcept

```c++
void print() noexcept(false)
{
    throw "error";
};
```

- 该关键字告诉编译器，函数中不会发生异常,这有利于编译器对程序做更多的优化。
- noexcept(false)：true 不会抛异常；false：可能抛异常

### C++ lambda表达式

https://paul.pub/cpp-lambda-function-bind/

[ capture-list ] { body }
[ capture-list ] ( params ) { body }
[ capture-list ] ( params ) -> ret { body }
[ capture-list ] ( params ) mutable exception attribute -> ret { body }

- **capture-list** 是需要捕获的变量列表，用逗号分隔。
- **params** 是参数列表。
- **ret** 指明了lambda表达式的返回值。通过return语句，如果编译器能够推断出返回值的类型。或者表达式没有返回值，“-> ret”可以省略。
- **body** 函数体。
- **mutable** 当捕获列表是以复制（见下文）的形式捕获时，默认这些复制的值是const的，除非指定了mutable。
- **exception** 提供了异常的说明。
- **attribute** 对于attribute的描述可以参见这里：http://en.cppreference.com/w/cpp/language/attributes，这里不多说明。



```c++
int main(){
    vector<int> numbers { 1, 2, 3, 4, 5, 10, 15, 20, 25, 35, 45, 50 };
    int sum = 0;
    std::for_each(numbers.begin(), numbers.end(), [&sum] (const int& i) { sum += i;}); // 求和
    cout<<sum<<endl;
    return 0;
}
```

lambda表达式中的捕获列表的语法，它可能是以下几种情况中的一种：

- [] 不捕获任何变量
- [&] 以引用的方式捕获所有变量
- [=] 以复制的方式捕获所有变量
- [=, &foo] 以引用的方式捕获foo变量，但是以复制的方式捕获其他变量
- [bar] 以复制的方式捕获bar变量，不再捕获任何其他变量
- [this] 捕获this指针

##### std::function

std::function的语法是这样：
template <class Ret, class... Args> class function<Ret(Args...)>;

ret：返回值；

args：参数表；

```c++
void printNumber(vector<int>& number, function<bool (int)> filter) {
	for (const int& i : number) {
	    if (filter(i)) {
	        cout<<i<<endl;
	    }
	}
}

void main(){
    printNumber(numbers, [] (int i){ return i % 5 == 0;});
    return 0;
}
```

##### std::bind

std::bind的语法是这样的：

template <class Fn, class... Args> bind (Fn&& fn, Args&&... args);
template <class Ret, class Fn, class... Args> bind (Fn&& fn, Args&&... args);

```c++
#include <iostream>
#include <vector>
#include <functional>
using namespace std;

bool isBetween(int i, int min, int max)
{
    return i >= min && i <= max;
}

void printNumber(vector<int> &number, function<bool(int)> filter)
{
    for (const int &i : number)
    {
        if (filter(i))
        {
            cout << i << endl;
        }
    }
}

int main()
{
    vector<int> numbers{1, 2, 3, 4, 5, 10, 15, 20, 25, 35, 45, 50};
    function<bool(int)> filter = bind(isBetween, placeholders::_1, 20, 40); // _1： 为占位符
    
    // 占位符的数量可以是任意多的，像这样：
	// std::placeholders::_1, std::placeholders::_2, …, std::placeholders::_N。
    // 具体看filter（） 的入参
    
    printNumber(numbers, filter);

    return 0;
}
```



### 智能指针 

- shared_ptr：允许多个智能指针指向同一个对象
- unique_ptr：独占所指向的对象
- weak_ptr：一种弱引用，指向shared_ptr所管理的对象



shared_ptr:

```c++
void testFn()
{
    shared_ptr<A> ptr = make_shared<A>(11);    // 使用make_shared初始化。（age） 对应A的构造函数参数 
    // shared_ptr<A> ptr(new A);               // new返回的指针来初始化智能指针，接受指针参数的构造函数是explicit的

    cout << ptr.get()->getAge() << endl;       // get 返回 A* 
    cout << ptr.use_count() << endl;           // 记录有多少个其它shared_ptr指向相同的对象
    ptr.reset(new A(18));                      // 指向新的对象，删除旧的
    
    shared_ptr<A> ptr2 = ptr;
    cout << ptr.unique() << endl;              //是否唯一指针指向同一对（1,0）
}
```

unique_ptr:

```c++
 // auto obj = unique_ptr<A>(new A(1));      
    A * a = new A(11);
    unique_ptr<A> obj(a);                   // new返回的指针来初始化智能指针
    
    cout << obj.get()->getAge() << endl;
```

### stl

##### vector(类似ArrayList)

```c++
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

bool cp(const int &a, const int &b)
{
    return a > b;
}

int main()
{

    vector<int> vList{1, 2, 3, 12, 45, 33, 23, 10};

      // sort(vList.begin(), vList.end(), [](int a, int b)
    //      { return a > b; });
   sort(vList.begin(), vList.end(), cp);

    for (int i : vList)
    {
        cout << i << endl;
    }
    
    return 0;
}
```

#####  list(双向链表)

```c++
#include <iostream>
#include <list>
#include <algorithm>

using namespace std;

int main()
{

    list<int> vList{3, 1, 2, 4};

    vList.push_back(12);
    vList.push_front(7);
    
    vList.sort([](int a, int b){return a > b;});
    

    for (int i : vList)
    {
        cout << i << endl;
    }

    return 0;
}
```

##### set

```c++
#include <iostream>
#include <set>
#include <algorithm>
#include <string>

using namespace std;

bool cp(int a, int b)
{
    return a > b;
}

int main()
{
    
    //set<int> s = {11, 33, 44, 3, 5, 76, 9};
    //s.insert(12);
    
    set<int, decltype(cp) *> s2(cp);
    s2.insert(1);
    s2.insert(100);

    for (int i : s2)
    {
        cout << i << endl;
    }
    return 0;
}
```

##### map

```c++
#include <iostream>
#include <map>
#include <algorithm>
#include <string>

using namespace std;

int main()
{

    map<string, int> m;
    m["小明"] = 10;
    m.insert(pair<string,int>("小刚",12));

    map<string,int> m2 = {{"小乔",14},{"大智",15}};
    m.insert(m2.begin(),m2.end());
    
    for (auto a : m)
    {
        cout << a.first << ":" << a.second << endl;
    }

    return 0;
}
```

##### queue

```c++
#include <iostream>
#include <algorithm>
#include <string>

#include <queue>

using namespace std;

int main()
{
    int array[]{1, 2, 3, 4, 5};
    priority_queue<int> pq(array, array + 5);
    pq.push(8);

   while (!pq.empty())
   {
        cout << pq.top() << endl;
        pq
   }

    return 0;
}
```

##### stack

```c++
#include <iostream>
#include <algorithm>
#include <string>

#include <stack>

using namespace std;

struct People
{
    string name;
    int age;

    People(string p_name, int p_age) : name(move(p_name)), age(move(p_age))
    {
        cout << "constructed \n";
    }

    People(const People &p) : name(move(p.name)), age(move(p.age))
    {
        cout << "copy constructed \n";
    }

    People(const People &&p) : name(move(p.name)), age(move(p.age))
    {
        cout << "move constructed \n";
    }
};

int main()
{
    stack<People> s_array;
    s_array.push(People("小刚", 10)); // 构造，移动构造

    cout << "-------------------------------------- \n";

    People p{"小花", 8};
    s_array.push(p);  // 构造，拷贝构造

    cout << "-------------------------------------- \n";

    s_array.emplace("小黑", 11);   // 构造

    cout << "-------------------------------------- \n";

    while (!s_array.empty())
    {
       cout << s_array.top().name << ":" << s_array.top().age << endl;
       s_array.pop();
    }

    return 0;
}


打印：
constructed
move constructed
--------------------------------------
constructed
copy constructed
--------------------------------------
constructed
--------------------------------------
小黑:11
小花:8
小刚:10    
```

### 并发

https://paul.pub/cpp-concurrency/#id-future

##### 创建线程

```c++
#include <iostream>
#include <thread> 

using namespace std; 

void hello() { 
  cout << "Hello World from new thread." << endl;
}

int main() {
  thread t(hello); 
  t.join(); 

  return 0;
}

```

##### 锁

```c++
#include <iostream>
#include <thread>
#include <mutex>

using namespace std;

static mutex mt;
static timed_mutex tmt;
static recursive_mutex rmt;


void fn(int *num)
{
    mt.lock();      // 1
    for (size_t i = 0; i < 100000; i++)
    {
        (*num)++;
    }
    mt.unlock();   // 2
}

int main()
{
    int num = 0;

    thread t1(fn, &num);
    thread t2(fn, &num);

    t1.join();
    t2.join();

    this_thread::sleep_for(chrono::seconds(3));

    cout << num << endl;
    return 0;
}

// 1,2 加去锁
```

- mutex

  - 调用方线程从它成功调用 lock 或者 try_lock 开始，到它调用 unlock 为止，占用该 mutex
  - 调用线程占用 mutex，所有其它线程试图要求 mutex 的所有权，如果请求线程调用 lock(),则将阻塞;如果请求线程调用 try_lock() 则返回 false。

- time_mutex

  - 接受一个时间范围，也就是在这个时间段范围内没有获取锁，则该线程被阻塞。

- recursive_mutex

  - 可重入锁

- lock_guard：在对象的构造函数中，调用 mutex.lock()，然后在析构函数中，调用 unlock(）；

- unique_lock：与 std::lock_guard() 类似，但是提供了更好的上锁机制和解锁控制

  

包含在(std_mutex_h,mutex)文件中

```c++
void fn(int *num)
{
    lock_guard<mutex> lg(mt);
    for (size_t i = 0; i < 100000; i++)
    {
        (*num)++;
    }
}

// lock_guard  mutex

void fn(int *num)
{
    if (tmt.try_lock_for(chrono::seconds(3)))
    {
        lock_guard<timed_mutex> lg(tmt, adopt_lock);
        for (size_t i = 0; i < 100000; i++)
        {
            (*num)++;
        }
    }
}

// lock_guard time_mutex
```

##### 异步

```c++
#include <iostream>
#include <cmath>
#include <future>

using namespace std;

static const int MAX = 16;
static double sum = 0;

void worker(int min, int max)
{
    for (int i = min; i <= max; i++)
    {
        sum += sqrt(i);
    }
}

int main()
{
    auto f1 = async(worker, 0, MAX);
    cout << "Async task triggered" << endl;
    future_status en = f1.wait_for(chrono::seconds(10));
    cout << (int) en << endl;
    cout << "Async task finish, result: " << sum << endl;

    return 0;
}

// async
```

- async
- packaged_task
- promise与future

