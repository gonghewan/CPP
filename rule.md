- 定义数组大小时使用size_t类型

  ```cpp
  for (decltype(s.size()) index = 0; index != s.size(); ++index) //用decltype声明类型，变量类型并不是int，而是unsigned; index++也可以
      cout << s[index] << endl;
  ```

- 引用才能改变对象
  ```cpp
  for (auto c: str) //遍历str中的每个char，将它复制给c
    cout << c << endl;
  for (auto &c: str) //c引用str中的每个char
      cout << c << endl;
  ```
- int j; j就是一个int型的引用，j的内容其实是变量的地址，在运行时是按照地址读值
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
