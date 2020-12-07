# C++

## 字符编码

- ASCII

  每一个二进制位（bit）有`0`和`1`两种状态，八个二进制位这被称为一个字节（byte）。一个字节一共可以用来表示256种不同的状态，每一个状态对应一个符号，就是256个符号，从`00000000`到`11111111`。上个世纪60年代，美国制定了一套字符编码，ASCII 码一共规定了128个字符的编码，对英语字符与二进制位之间的关系，做了统一规定。

- 非ASCII编码

  英语用128个符号编码就够了，但是用来表示其他语言，128个符号是不够的。于是利用字节中闲置的最高位编入新的符号。比如，法语中的`é`的编码为130（二进制`10000010`）。这样一来，这些欧洲国家使用的编码体系，可以表示最多256个符号。

  但是，这里又出现了新的问题。不同的国家有不同的字母，因此，哪怕它们都使用256个符号的编码方式，代表的字母却不一样。比如，130在法语编码中代表了`é`，在希伯来语编码中却代表了字母`Gimel` (`ג`)，在俄语编码中又会代表另一个符号。但是不管怎样，所有这些编码方式中，0--127表示的符号是一样的，不一样的只是128--255的这一段。至于亚洲国家的文字使用的符号更多，汉字就多达10万左右。一个字节只能表示256种符号，肯定是不够的，就必须使用多个字节表达一个符号。比如，简体中文常见的编码方式是 GB2312，使用两个字节表示一个汉字，所以理论上最多可以表示 256 x 256 = 65536 个符号。

  中文编码的问题需要专文讨论，这篇笔记不涉及。这里只指出，虽然都是用多个字节表示一个符号，但是GB类的汉字编码与后文的 Unicode 和 UTF-8 是毫无关系的。

- Unicode

  Unicode 是一种所有符号的编码，但它只是一个符号集，只规定了符号的二进制代码，却没有规定这个二进制代码应该如何存储。

- UTF-8

  是一种变长的编码方式。它可以使用1~4个字节表示一个符号，根据不同的符号而变化字节长度。UTF-8 的编码规则很简单，只有二条：

  1）对于单字节的符号，字节的第一位设为`0`，后面7位为这个符号的 Unicode 码。因此对于英语字母，UTF-8 编码和 ASCII 码是相同的。

  2）对于`n`字节的符号（`n > 1`），第一个字节的前`n`位都设为`1`，第`n + 1`位设为`0`，后面字节的前两位一律设为`10`。剩下的没有提及的二进制位，全部为这个符号的 Unicode 码。

  下表总结了编码规则，字母`x`表示可用编码的位。

  > ```
  > Unicode符号范围        |        UTF-8编码方式
  > (十六进制)             |              （二进制）
  > ----------------------+---------------------------------------------
  > 0000 0000-0000 007F   | 0xxxxxxx
  > 0000 0080-0000 07FF   | 110xxxxx 10xxxxxx
  > 0000 0800-0000 FFFF   | 1110xxxx 10xxxxxx 10xxxxxx
  > 0001 0000-0010 FFFF   | 11110xxx 10xxxxxx 10xxxxxx 10xxxxxx
  > ```

  跟据上表，解读 UTF-8 编码非常简单。如果一个字节的第一位是`0`，则这个字节单独就是一个字符；如果第一位是`1`，则连续有多少个`1`，就表示当前字符占用多少个字节。前三个字节`EF BB BF`表示这是UTF-8编码。
  
  下面，还是以汉字`严`为例，演示如何实现 UTF-8 编码。
  
  ```
  严`的 Unicode 是`4E25`（`100111000100101`），根据上表，可以发现`4E25`处在第三行的范围内（`0000 0800 - 0000 FFFF`），因此`严`的 UTF-8 编码需要三个字节，即格式是`1110xxxx 10xxxxxx 10xxxxxx`。然后，从`严`的最后一个二进制位开始，依次从后向前填入格式中的`x`，多出的位补`0`。这样就得到了，`严`的 UTF-8 编码是`11100100 10111000 10100101`，转换成十六进制就是`E4B8A5
  ```

- 大端序和小端序

  Unicode 规范定义，每一个文件的最前面分别加入一个表示编码顺序的字符，这个字符的名字叫做"零宽度非换行空格"（zero width no-break space），用`FEFF`表示。这正好是两个字节，而且`FF`比`FE`大`1`。

  如果一个文本文件的头两个字节是`FE FF`，就表示该文件采用大头方式；如果头两个字节是`FF FE`，就表示该文件采用小头方式。

  ```
  **变量a**
  int a = 0x12345678
  
  **地址的高端与低端**
  0x00000001
  0x00000002
  0x00000003
  0x00000004
  从上倒下，由低到高，地址值小的为低端，地址值大的为高端。
  
  Bit-endian 如此存放(按原来顺序存储)
  0x00000001      -- 12
  0x00000002      -- 34
  0x00000003      -- 56
  0x00000004      -- 78
  
  Little-endian 如此存放(颠倒顺序储存)
  0x00000001      -- 78
  0x00000002      -- 56
  0x00000003      -- 34
  0x00000004      -- 12
  ```

## 变量

### 变量类型

- 在不声明变量类型时，小数默认为double类型。
- 整形数后加f会失效，想声明float必须是`1.f`

### 变量初始化

- 变量初始化值取决于变量类型和变量声明的位置
- built-in类型的变量在函数外声明时初始化为0，在函数内部声明时不做初始化处理，其内存存放为任意值

**在变量未被赋值之前进行copy或其他尝试利用值的行为都会引发错误**

- 自定义类型变量初始化，如果该类型并未定义构造函数则系统调用默认的构造函数`A a=A()`，如果该类型定义了构造函数`A(int aa){xxx}` 则`A a=A()`会因未遵循构造函数的格式而报错

- 定义数组时，如果没有显式提供初始化列表，则数组元素的自动化初始规则同普通变量一样：
  - 函数体外定义的内置类型数组，其元素初始为0
  - 函数体内定义的内置类型数组，其元素无初始化
  - 类类型数组无论在哪里定义，皆调用默认构造函数进行初始化，无默认构造函数则必须提供显示初始化列表
  - 如果定义数组时，仅提供了部分元素的初始列表，其剩下的数组元素，若是类类型则调用默认构造函数进行初始，若是内置类型则初始为0（不论数组定义位置）
  - 对于动态分配的数组，如果数组元素是内置类型，其元素无初始化；如果数组元素是类类型，依然调用默认构造函数进行初始化，也可以在使用跟在数组长度后面的一对空圆括号对数组元素做值初始化

### 变量定义与声明的区别

- 声明：让程序知道有这个变量，声明语句中一旦包含初始化值则extern声明失效，该声明语句即为一个定义语句

- 定义：创建实体，一个变量的定义即包含了声明

变量的声明确定了数据类型和变量名，定义会进行内存的收集分配，变量可以定义一次但可以声明多次，如在一个文件中的变量被多个文件引用时产生多个声明。

```CPP
extern int i;   //declares but does not define i
int j;  //declares and defines j
```

### 变量名

- 使用下划线的变量名不可以在后面紧接大写字符，不可以连续两个下划线，函数外定义的变量名不可用下划线为起始字符

### 引用和指针

- 引用实质上是一个常量指针，其对应的内存中存储的是被引用对象的内存地址，引用只能且必须初始化一次，使用引用只是为变量增加一个新名字，由于指针的错误难以被编译器发现，所以使用不可变的指针更加安全，后续使用引用时就是使用其指向的变量（所有属性和其指向变量相同）
- 指针可以改变（复制，再赋值），可以是未经初始化的空指针

但二者语法使用有区别

```cpp
#include<iostream>
int main(){
        int val=1024;
        int a=1.01;
        int &ref = val;
        val=3;
        std::cout << ref<<std::endl;
        std::cout << a<<std::endl;
        std::cout << "pointer"<<std::endl;
        int *p=&val;
        std::cout<< p << "/"  << *p << std::endl;
        return 0;
}
输出：
3
1
pointer
0x7ffc9d9300a0/3
```
```
#include <iostream>
using namespace std;

int main()
{
    int a=20;
    int *p=&a;
    int ** q=&p;
    cout << typeid(p).name() << p<< endl; //Pi 0x61fe1c
    cout << typeid(*p).name() << *p<<endl; //i 20
    cout << typeid(&p).name() << &p<<endl; //PPi 0x61fe10
    cout << typeid(q).name() << &q<<endl; //PPi 0x61fe08
    return 0;
}
memory: 
0x61fe08: 10 fe 61 00 00 00 00 00|1c fe 61 00 00 00 00 00    
0x61fe18: 10 00 00 00 14 00 00 00|90 19 0a 01 00 00 00 00   
0x61fe28: c7 13 40 00 00 00 00 00|00 00 00 00 00 00 00 00    
0x61fe38: 48 00 00 00 00 00 00 00|70 79 40 00 00 00 00 00
```

- 空指针的声明方法

  ```cpp
  int *p = nullptr;(recommanded)
  int *p = 0; // int null=0,*p=null合法，将0赋值给指针代表空指针 
  int *p = NULL;(old eidtion)
  ```

- 指针可比较

  ```
  ==：
  both null
  指向同一地址
  指向同一对象（指向不同对象的指针可能存储同一地址，如原地址对应的对象从内存中抹去，该地址又存储了新对象）
  ```

- 指针的赋值过程不会默认进行类型转换，如果将int * 指向double会报错

- 一个例子

  ```cpp
  #include<iostream>
  int main(){
          int a=1;
          int b=2;
          constexpr int k = a + b;//编译时就已求出值
          int *const pointer=&a;
          const int *pointer2const=&a;
          int& pointer1=a;
          //const int &pointer1=a;//编译pointer1=b时会报错，因为声明pointer1是int常量的引用，所以不能用pointer1改变其指向的值
          std::cout << *pointer << "---" << pointer1 << std::endl;
          a = a + 2;
          //*pointer2const = *pointer2const + 1;//根据pointer2const的声明，其指向>的是常量，但实际上是否是常量并不能确定，如果是变量仍然能用该变量的其他名字为其更改值，只不过无法通过pointer2const来修改它的值
          std::cout << *pointer << "---" << pointer1 << std::endl;
          pointer1 = b;//实际是a=b，并没有提供重新给引用类型的变量pointer1的方法
          //pointer = &b; //根据pointer的声明，他是一个常量，一经赋值后不能更改
          std::cout << *pointer << "---" << pointer1 << std::endl;
  }
  ```

- Top-Level const  与引用类似，一旦指向一个对象后不可变更，top-level const 指的是指针本身是一个常量，low-level const指的是指针所指对象是一个常量

  ```
  int *const p= &i //top-level const
  const int *p = &i //we can change p, i is a top-level const
  ```

- constexpr

  ```
  C++之父是这样解释的：
  const：承诺不改变这个值，主要用于说明接口，这样变量传递给函数就不担心变量会在函数内被修改了编译器负责确认并执行const的承诺。
  constexpr：在编译时求值，主要用于说明常量，作用是允许数据置于只读内存以及提升性能,但并不能保证它一定在编译时被计算，也可以在运行时被计算。
  值得注意的是，constexpr用于声明指针变量时，与const不同
  const int *p = nullptr; //p是一个指向int常量的指针变量
  constexpr int *p = nullptr; //p是一个指向int的指针常量, constexpr代表top-level const
  无论指针声明是指向常量int还是变量int，在为指针赋值时都可以赋值一个变量int
  ```

### 别名

尽量避免给指针做别名，在涉及const声明时可能会产生与想法不同的结果

```
两种声明方式
typedef double wages;
using SI = Sales_item;
```

### Auto类型

用表达式赋值时可以使用auto声明该变量类型为表达式计算后的类型，它通常会忽略top-level const

```
auto item = val1 + val2
//值得注意的是在一句声明了多个变量的声明语句中使用auto
auto p=3.14,q=3;//error: p and q are different type
const int ci = 0, &cr=ci;
auto b = ci;//b is an int
const auto b = ci; //b is an const int
```

### decltype类型

使用该声明该变量类型为表达式计算后的类型，但不会为该变量赋值为表达式的计算结果

```
decltype(f()) x=y;
examples:
const int ci=0,&cj=ci;
decltype(ci) x=1;
decltype(r+0) b;
decltype((ci)) d;//error: d is a int& and must be initialized
decltype(ci) d;//d is a int
```

### 头文件

**头文件中应该只放变量和函数的声明，而不能放它们的定义**

```
Person.h:

#ifndef PERSON_H
#define PERSON_H
class Person
{
public:
    int age;
    char* name;
    char* sex;
public:
    Person();    //构造函数
    void say();
};
#endif

Person.cpp:
#include <iostream>
#include "Person.h"

using namespace std;

Person::Person()
{
    cout << "创建了一个新的对象！" << endl;
}

void Person::say()
{
    cout << "大家好，我叫" << name << ",性别" << sex << ",今年" << age << "岁." << endl;
}

使用：
#include <iostream>
#include "Person.h"
using namespace std;
int main()
{
    Person xc;
    xc.age = 18;
    xc.name = "流浪";
    xc.sex = "男";
    xc.say();
    return 0;
}
```
## 字符串 数组 

### 字符串声明

```
string s1;//default initialization;s1 is empty string
string s2(s1);//s2 is a copy of s1
string s2 = s1;//equivalent to s2(s1)
string s3("value");//direct initialization; s3 is the copy of the string literal,not including null
string s3 = "value";//copy initialization; equivalent to s3("value")
string s4(10,'c');//direct initialization; s4 :n copies of 'c'
```

 ### 从输入中读取字符串

- cin从第一个非空白字符开始读入，至下一空白字符结束
- getline(cin,str)按行读取但会忽略行尾的\n
- str.empty(),str.size()

### 字符串连接

```c++
string s4 = "hello" + "," //error:no string operand(操作数)
string s5 = "hello" + "," + s1 //error:can't add string literals(复合常量)
string s6 = s1 + "xxx" //ok
string s7 = ("hello"+ ",") //error:can't add string literals
```

### 字符串遍历

```cpp
for (declaration:expression)
	statement
for (auto c: str) //遍历str中的每个char，将它复制给c
	cout << c << endl;
for (auto &c: str) //c引用str中的每个char
    cout << c << endl;
for (decltype(s.size()) index = 0; index != s.size(); ++index) //用decltype声明类型，变量类型并不是int，而是unsigned; index++也可以
    cout << s[index] << endl;

string::size_type n; //声明合法字符串下标类型，保证n >= 0，遍历时只需验证是否超出字符串长度即可
while (cin >> n)
    ...
```

### Vector

```cpp
#include <vector>
using std::vector;
实例化方式：
Vector<T> v1; //default initialization, v1 is empty
Vector<T> v2(v1); //copy v1 to v2
Vector<T> v2 = v1; //equal to v2(v1), copy v1 to v2
Vector<T> v3(n,val); //initailize v3 with n elements whose value = val
Vector<T> v4(n); //initailize v3 with n elements whose value-initialized by T
Vector<T> v5{a,b,c,...}
Vector<T> v6 = {a,b,c,...}
//Especially:
Vector<string> v7{10,"hello"} //v7 has ten elements with value "hello"

比较大小：
a == b //两个vector的大小和每个位置的元素都相同
a <= b //a的长度小于b且a与b在每个位置的元素相同

动态添加元素：
vector<int> ivec;
ivec.push_back(42);
ivec[0] = 0; //error: [] can only refer the element which has existed

遍历：
//一般不使用for循环，动态变化时会出错
//迭代器
for (auto it = s.begin(); it != s.end(); it++)
{
    *it = toupper(*it);
}
```

## 迭代器
- 标准容器迭代器操作
```cpp
*iter	Return a reference to the element denoted by the iter
iter->mem	equals to (*iter).mem
++iter refer to the next element  
--iter refer to the before element
iter1 == iter2	iter1 and iter2 refer to the same element or they are the off-the-end iterator for the same container
```
- vector, string, array类容器
```cpp
iter + n	
iter - n
iter += n
iter -= n
iter1 - iter2
>,>=,<,<=	//One iterator is less than another if it refers to an elemment that appears in the container before the one referred to by the other iterator
```
- 迭代器类型
```
vector<int>::iterator it; // it can read and write vector<int> elements
string::iterator it2;     // it2 can read and write characters in a string
vector<int>::const_iterator it3; // it3 can read but not write elements
string::const_iterator it4;      // it4 can read but not write characters
```
A const_iterator behaves like a const pointer. Like a const pointer, a const_iterator may read but not write the element it denotes; an
object of type iterator can both read and write. If a vector or string is const, we may use only its const_iterator type. With a nonconst vector or string,
we can use either iterator or const_iterator.

The type returned by begin and end depends on whether the object on which they
operator is const. If the object is const, then begin and end return a
const_iterator; if the object is not const, they return iterator:
```
vector<int> v;
const vector<int> cv;
auto it1 = v.begin();  // it1 has type vector<int>::iterator
auto it2 = cv.begin(); // it2 has type vector<int>::const_iterator
```
Regardless of whether the vector (or string) is const, they return a const_iterator.
```
auto it3 = v.cbegin(); // it3 has type vector<int>::const_iterator
```
- 使用迭代器时不应向该迭代器指向的容器中增加元素
## 运算符
### rvalue和lvalue
### 计算顺序
```
int i = f1() * f2(); //如果f1和f2对同一个对象做运算，不能保证结果运算正确，因为f1和f2的运算顺序并不确定

int i = 0;
cout << i << " " << ++i << endl; // output: 0,1 or 1,1

s = f() + g() * h() + j() //无法保证f(),g(),h(),j()的运算顺序，只能保证其结果的运算顺序
```
能够保证运算顺序的操作符有：'&&','||','?:',','
### 数学运算
- 注意运算结果是否超出其类型的限制长度
- 除法
  - 整数相除时，商的小数部分舍弃(向0取整)
  - 早期的版本中负商向上/下取整，新标准中选取向0取整的方法
- 取余
  - 规定 if m%n is nonzeroo, it has the same sign as m. 
  ```
  21 % 6; /* result is 3 */ 21 / 6; /* result is 3 */
  21 % 7; /* result is 0 */ 21 / 7; /* result is 3 */
  -21 % -8; /* result is -5 */ -21 / -8; /* result is 2 */
  21 % -5; /* result is 1 */ 21 / -5; /* result is -4 */
  ```
- bool型true和false在与int类型对象比较时会转换为1/0
   ```
   if (val) { /* ... */ } // true if val is any nonzero value
   if (!val) { /* ... */ } // true if val is zero
   if (val == true) { /* ... */ } // true only if val is equal to 1!
   ```
- 优先级
   ```
    i != j < k   <==>  i != (j < k) 
    *pbeg++      <==>  *(pbeg++)
   ```
- 连续赋值
   ```
   int ival, jval;
   ival = jval = 0; // <==> javal = 0 -> ival = jval
   int ival, *pval; // ival is an int; pval is a pointer to int
   ival = pval = 0; // error: cannot assign the value of a pointer to an int
   string s1, s2;
   s1 = s2 = "OK"; // string literal "OK" converted to string
   ```
- .和->
   ```
   string s1 = "a string", *p = &s1;
   auto n = s1.size(); // run the size member of the string s1
   n = (*p).size(); // run size on the object to which p points, * has a lower precedence then . 
   n = p->size(); // equivalent to (*p).size()  
   ```
- 条件操作符
   ```
   finalgrade = (grade > 90) ? "high pass" : (grade < 60) ? "fail" : "pass";
   cout << ((grade < 60) ? "fail" : "pass"); // prints pass or fail
   cout << (grade < 60) ? "fail" : "pass"; // prints 1 or 0! <==> (cout << (grade < 60) )? "fail" : "pass";
   cout << grade < 60 ? "fail" : "pass"; // error: compares cout to 60 <==> cout << grade; cout < 60 ? "fail" : "pass";
   ```
- sizeof
  The result of sizeof is a constant expression of type size_t. The operator takes one of two forms:
   - sizeof (type)
   - sizeof expr
   ```
   Sales_data data, *p;
   sizeof(Sales_data); // size required to hold an object of type Sales_data
   sizeof data; // size of data's type, i.e., sizeof(Sales_data)
   sizeof p; // size of a pointer
   sizeof *p; // size of the type to which p points, i.e., sizeof(Sales_data)
   sizeof data.revenue; // size of the type of Sales_data's revenue member
   sizeof Sales_data::revenue; // alternative way to get the size of revenue
   ```
   The result of applying sizeof depends in part on the type involved:
   - sizeof char or an expression of type char is guaranteed to be 1.
   - sizeof a reference type returns the size of an object of the referenced type.
   - sizeof a pointer returns the size needed hold a pointer.
   - sizeof a dereferenced pointer returns the size of an object of the type to which the pointer points; the pointer need ==not be valid==.
   - sizeof an array is the size of the entire array. It is equivalent to taking the sizeof the element type times the number of elements in the array. Note that sizeof does      not convert the array to a pointer.
   - sizeof a string or a vector returns only the size of the fixed part of these types; it does not return the size used by the object’s elements.
- comma
  逗号的优先级最低
  ```
  someValue ? ++x, ++y : --x, --y   <==>  (someValue ? ++x, ++y : --x), --y
  ```
- type conversion
  在可以转换的情况下优先考虑不损失精度
  ```
  int ival = 3.541 + 3;  <==>  3.541+3.0---> int(6.541)
  ```
  The compiler automatically converts operands in the following circumstances:
  - In most expressions, values of integral types smaller than int are first promoted to an appropriate larger integral type.
  - In conditions, nonbool expressions are converted to bool.
  - In initializations, the initializer is converted to the type of the variable; in assignments, the right-hand operand is converted to the type of the left-hand.
  - In arithmetic and relational expressions with operands of mixed types, the types are converted to a common type.
  - As we’ll see in Chapter 6, conversions also happen during function calls.
## 四种强制转换
  - static_cast 可以实现 C++内置基本类型的转换；支持子类指针到父类指针的转换，并根据实际情况调整指针的值，反过来也支持，但会给出编译警告，它作用最类似C风格的“强制转换”，一般来说可认为它是安全的；
  ```
  double d = 12.34;
  int a = static_cast<int>(d); // a = 12
  class Base{};
  class Derived :public Base{};
  Derived d;
  Base e = staic_cast<Base>(d);
  ```
  - dynamic_cast 和static_cast是相对的，static_cast是在编译的时候进行转换的。它是动态的在运行时候转换的，而且 只能在继承类对象的指针或引用之间进行转换，在进行转换的时候，会根据当前RTTI （运行时类型识别）判断类型对象之间的转换是否合法，如果合法就转换成功了，返回了指向类的引用或指责，但是如果转换是非法的，则返回NULL或者0。使用dynamic_cast进行转换的时候，基类一定要有虚函数，另外它向上转换的时候是兼容的，向下转换的时候有类型安全检查，比static_cast要安全。
  ```
  class A {
  public:
	virtual void f() {}
  };
  class B : public A
  {};
  void fun(A* pa) {
	// dynamic_cast会先检查是否能转换成功，
	// 能成功则转换，不能则返回0
	cout << "pa" <<' '<< typeid(pa).name()<<pa << endl;
	B* pb1 = static_cast<B*>(pa);
	B* pb2 = dynamic_cast<B*>(pa);
	cout << "pb1" <<' '<<typeid(pb1).name()<<' '<< pb1 << endl;
	cout << "pb2" <<' '<<typeid(pb2).name()<<' '<< pb2 << endl;
	//pa   class A*  00AFFE00
	//pb1  class B*  00AFFE00
	//pb2  class B*  00000000
	}
  int main() {
	A a;
	fun(&a);
	system("pause");
	return 0;
  }
  ```
  - const_cast 目的并不是为了让你去修改一个本身被定义为const的值，因为这样做的后果是无法预期的。const_cast的目的是修改一些指针/引用的权限，如果我们原本无法通过这些指针/引用修改某块内存的值，现在你可以了，但是当你去改变const的值时并不会真正地改变const的值，而且会被提示未定义的动作。如我们可能调用了一个参数不是const的函数，而我们要传进去的实际参数确实const的，但是我们知道这个函数是不会对参数做修改的。于是我们就需要使用const_cast去除const限定，以便函数能够接受这个实际参数。
  ```
  void func(const int& a)//形参为，引用指向const int
  {
	int& b = const_cast<int&>(a);//去掉const限定，因为原本为非常量
	b++;
	return;
  }
  int main()
  {
	int a = 100;
	func(a);
	cout << a << endl;  // 打印101
	return 0;
  }
  ```
  - reinterpret_cast 支持任何转换，但仅仅是如它的名字所描述的那样“重解释”而已，不会对指针的值进行任何调整，用它完全可以做到“指鹿为马”，但很明显，它是最不安全的转换
  ```
  double d = 9.3;
  double *pd = &d;
  int *pi = reinterpret_cast<int*>(pd);
  //上面是将 double * 转换为 int * ，但是不可以用于非指针类型的转换，reinterpret_cast 同时也不能将一个const指针转换为void*指针
  //这里是将一个整形函数转换为函数指针的类型去调用，但是存在着缺陷
  typedef void(*FUNC)();
  int DoSomething(int i){
	cout << "DoSomething" << endl;
	return 0;
  }
  void Test(){
	// reinterpret_cast可以编译 
	// 以FUNC的定义方式去看待DoSomething函数,非常的BUG，下面转换函数指针的代码是不可移植的，C++不保证所有的函数指针都被一样的使用，所以这样用有时会产生不确定的结果
	FUNC f = reinterpret_cast<FUNC>(DoSomething);
	f();
	}
  ```
## 四种跳转语句
- break 只影响最近的循环或switch，终止本层循环
- continue 只影响最近的循环，跳过其后的语句立即开始下次循环
- goto 无条件跳转至同一函数的另一语句
  ```
  goto end;  
  int ix = 10; // error: goto bypasses an initialized variable definition 
  end: 
  // error: code here could use ix but the goto bypassed its declaration  
  ix = 42;
  // backward jump over an initialized variable definition is okay
  begin: 
    int sz = get_size(); 
    if (sz <= 0) { 
        goto begin; 
  }
  ```
## Exception Handling
- exception
  ```
  if (item1.isbn() != item2.isbn()) 
      throw runtime_error("Data must refer to same ISBN");
  ```
- try Block
  ```
  try { 
       program-statements 
  } catch (exception-declaration) {  
       handler-statements 
  } catch (exception-declaration) {  
       handler-statements 
  } // . . .
  ```
## 函数
- argument(实参) 与 parameter(形参)
  实参是函数调用的实际值，是形参的初始值
- local variable 与 static
  ```
  size_t count_calls()
  {
   static size_t ctr = 0; // value will persist across calls
   return ++ctr;
  }
  int main()
  {
   for (size_t i = 0; i != 10; ++i) cout << count_calls() << endl; return 0;
  }
  //This program will print the numbers from 1 through 10 inclusive.
  ```
- CC命令
  ```
  //a.c
  #include <stdio.h>
  #include "b.h"
  main(){
  bb();
  }
  //b.h
  int a;
  void bb(void);

  //b.c
  #include <stdio.h>
  #include "b.h"
  int a=4;
  void bb(){
  printf("das%d\n",a);
  }
  
  cc -c a.c;此命令生成a.o
  cc -c b.c;此命令生成b.o
  ar -crv libb.a b.o;此命令生成静态库文件
  以下六中连接方式都是可以的，最终都生成了同样的可执行文件a：
  cc -o a a.c b.c
  cc -o a a.c b.o
  cc -c a a.o b.c
  cc -o a a.o b.o
  cc -o a a.c libb.a
  cc -o a a.o libb.a
  以上都是将库文件直接复制到程序文件中，链接的文件既可以是.o文件，也可以是源代码文件，但是有一点特殊的是，当我们生成动态库的时候，就只能用源代码文件去生成动态库了，而不能用中间代码.o文件去生成，举个例子：a.c是主程序源代码，a.o是编译后的二进制文件，生成动态库文件libb.so用下面的命令: cc -shared -fPIC -o libb.so a.c, 而不能用cc -shared -fPIC -o libb.so a.o
  ```

## 其它
 - char * 和 char[]的区别
   ```
        #include "stdafx.h"
	#include <string>
	#include <iostream>
	using namespace std;
	int _tmain(int argc, _TCHAR* argv[])
	{
	char *c1 = "abc";
	char c2[] = "abc";
	cout <<" *c1是 ：" << *c1 <<endl; // a 
	cout <<"c2 是："<< c2 << endl; //abc
	cout <<"*c2 是："<< *c2 << endl; //a

	string namee = "xuhaitao";
	const char *c3 = namee.data();
	cout <<"*c3是： "<< *c3 << endl; //x
	const char *c4 = namee.c_str(); 
	cout <<"*c4是： "<<*c4 << endl; //x

	getchar();
	return 0;
	}
   ```
