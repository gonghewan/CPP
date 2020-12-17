# Library
## I/O
- 关系
  istream----->ifstream----->istringstream
     
  iostream---->iofstream---->iostringstream
     
  ostream----->ofstream----->ostringstream
     
  streambuf--->filebuf------>stringbuf
  cin : 一个istream对象，从标准输入中读取数据，注意为字节读取
  cout :一个ostream对象，从标准输出中写入数据
  \>\> : 用来从一个istream对象读取输入数据，如cin \>\> x，表示输入数据流到x
  \<\< : 用来向一个ostream对象写入输出数据，如cout \<\< x，表示x输出到ostream对象
  getline(string库中)：从is中读取字符串到str中，其中delim为字符截至标志
  ```
  istream& getline (istream& is, string& str, char delim);
  istream& getline (istream& is, string& str);
  ```
  - 流对象不可复制或为其它流对象赋值
    ```
    ofstream out1, out2;
    out1 = out2; // error: cannot assign stream objects
    ofstream print(ofstream); // error: can't initialize the ofstream parameter
    out2 = print(out2); // error: cannot copy stream objects
    ```
  - 想要知道流对象当前的状态有许多可用的方法，这些方法可对流对象的状态做出改变和判断
    ```
    // remember the current state of cin
    auto old_state = cin.rdstate(); // remember the current state of cin
    cin.clear(); // make cin valid
    process_input(cin); // use cin
    cin.setstate(old_state); // now reset cin to its old state
    ```
    查询流状态的接口：
    s.eof()  若流 s 的eofbit置位，返回true
    s.fail()  若流 s 的failbit 或badbit 置位，返回true
    s.bad() 若流 s 的badbit置位，返回true
    s.good() 若流s处于有效状态 返回true
    s.clear() 将流s中所有的条件状态位复位，将流的状态设置为有效，返回void
    s.clear(flags) 根据给定的flags标志位，将流s的对应条件状态位复位，flags的类型为strm::iostate，返回void
    s.setstate(flags) 根据给定的flags 标志位，将流s中对应的条件状态位置位，flags类型位strm::iostate 返回void
    s.rdstate() 返回流s当前的条件状态，返回值类型为 strm::iostate
  - 标志位
    - badbit表示发生系统级的错误，如不可恢复的读写错误。通常情况下一旦badbit被置位，流就无法再使用了
    - failbit 表示发生可恢复的错误，如期望读取一个数值，却读出一个字符等错误。这种问题通常是可以修改的，流还可以继续使用
    - 当到达文件的结束位置时，eofbit和failbit都会被置位
    - goodbit 被置位表示流未发生错误。如果badbit failbit 和eofbit 任何一个被置位，则检查流状态的条件会失败。
    ```
    For example, the following turns off failbit and badbit but leaves eofbit untouched:
    // turns off failbit and badbit but all other bits unchanged
    cin.clear(cin.rdstate() & ~cin.failbit & ~cin.badbit);
    ```
- 即时输出的情况
  There are several conditions that cause the buffer to be flushed—that is, to be written—to the actual output device or file:
  • The program completes normally. All output buffers are flushed as part of the return from main.
  • At some indeterminate time, the buffer can become full, in which case it will be flushed before writing the next value.
  • We can flush the buffer explicitly using a manipulator such as endl，flush,ends(endl换行并刷新缓冲区；ends插入一个空字符并刷新缓冲区；flush仅刷新缓冲区).
  • We can use the unitbuf manipulator to set the stream’s internal state to empty the buffer after each output operation. By default, unitbuf is set for cerr, so that writes to cerr are flushed immediately.
  • An output stream might be tied to another stream. In this case, the buffer of the tied stream is flushed whenever the tied stream is read or written. By default, cin and cerr are both tied to cout. Hence, reading cin or writing to cerr flushes the buffer in cout.
- The unitbuf Manipulator
  If we want to flush after every output, we can use the unitbuf manipulator. This manipulator tells the stream to do a flush after every subsequent write. The nounitbuf manipulator restores the stream to use normal, system-managed buffer flushing:
  ```
  cout << unitbuf; // all writes will be flushed immediately
  // any output is flushed immediately, no buffering
  cout << nounitbuf; // returns to normal buffering
  ```
- <font color='red'>在程序崩溃时buffer是不会会刷新的，所以通过io调试时很有可能得到意想不到的结果</font>

- 将cin和cout绑定，库中默认如此定义，在任何的cin前都会flush buffer，我们也可以指定/更改绑定的对象：
  ```
  cin.tie(&cout); // illustration only: the library ties cin and cout for us 
  // old_tie points to the stream (if any) currently tied to cin 
  ostream *old_tie = cin.tie(nullptr); // cin is no longer tied 
  // ties cin and cerr; not a good idea because cin should be tied to cout
  cin.tie(&cerr); // reading cin flushes cerr, not cout
  cin.tie(old_tie); // reestablish normal tie between cin and cout
  ```
- 文件流
  - 需要包含的头文件： <fstream> 
  - 实际上，文件类型可以分为两种： 文本文件和二进制文件。文本文件保存的是可读的字符， 而二进制文件保存的只是二进制数据。二进制模式可以操作图像等文件，文本模式只能读写文本文件，否则会报错。
  - 创建一个文件流，定义一个文件流对象，并将对象和文件关联起来。使用流类型的open函数进行关联文件
     - 创建一个只读文件流：
      ```
      ifstream input(filename); //创建一个ifstream 文件流，并打开给定的文件
      ofstream output;        //创建一个ofstream 文件流，没有关联任何文件
      ```
     创建文件流对象时，如果提供了文件名，open函数会自动被调用。要使用is_open()对文件是否打开进行验证。如果没有提供文件名，需要手动调用open去关联一个文件。如果open失败了，failbit会被置位，所以open之后尽量去检测一下open是否成功。close成员函数可以关闭一个文件的关联，close之后，可以重新打开关联一个新的文件。
    - 手动设置open,我们先看一下open成员函数的原型：
      ```
      public member function 
      void open ( const char * filename,ios_base::openmode mode = ios_base::in | ios_base::out );
      void open(const wchar_t *_Filename,ios_base::openmode mode= ios_base::in | ios_base::out,int prot = ios_base::_Openprot)；
      参数：
      filename    操作文件名  
      mode        打开文件的方式
      prot        打开文件的属性         //基本很少用到，在查看资料时，发现有两种方式
      ```
      打开文件的模式 在ios类(所以流式I/O的基类)中定义，有如下几种方式：
      mode类型             描述          
      ios::in      为输入(读)而打开文件     
      ios::out     为输出(写)而打开文件
      ios::ate     初始位置：文件尾
      ios::app     所有输出附加在文件末尾
      ios::trunc   如果文件已存在则先删除该文件
      ios::binary  二进制方式
      这些模式是能够进行组合使用的，以“或”运算（“|”）的方式：
      ```
      ofstream out;                                               //定义一个写文件流
      out.open("Hello.txt", ios::in|ios::out|ios::binary)         //根据自己需要进行相应的选取
      ```
    - 对文件流进行读或写，无论是默认自动调用open还是手动调用open，当我们open完成之后，就开始对文件进行操作了。
      可以使用 << 进行对文件进行写入
      可以使用 >> 对文件进行读，遇到空格即结束
      使用getline成员函数读，遇到换行符即结束
      同时，我们结合流状态接口可以判断文件是否可读可写，是否读到文件结尾等
- stringstream
    - 在sstream头文件中定义了三个类型来支持内存的IO，通过ostringstream可以向string写入数据，通过istringstream从string读取数据，通过stringstream向string读写数据
    - stringstream特有的操作：
        - sstream strm	strm是一个未绑定的stringstream对象。
        - sstream strm(s)	strm是一个stringstream对象,保存string s的拷贝。此构造函数是explicit 
        - strm.str()	返回strm保存的字符串
        - strm.str(s)	将strm保存的字符串拷贝到string s中，返回void
      当对整行文本并处理行内的某个单词进行处理，此时可以使用string流。
      istringstream将string转换为int(通常可是直接使用to_string()和stoi(),stol(),stof(),stod()等实现string和数值类型的相互转换)
      ```
      string s = "123";
      istringstream isToi(s);//将s绑定到istringstream中
      int a;
      isToi >> a;//string输出为int
      cout << a << endl;
      ```
      读取某文件中人名和电话号码：
      Morgan 1646464648 1164986463
      drew 15465463132
      ```
      struct PersonInfo{
      string name;
      vector<string> phones;//多个电话号码
      }
      //line保存一行的信息
      string line,word;
      vector<PersonInfo> people;
      //读取一行
      while(getline(cin,line)){
          PersonInfo info;
          istringstream iss(line);//一行的字符串绑定istringstream
          iss >> info.name;//分离出姓名，以空格分离
          while(iss >> word)info.phones.push_back(word);
          people.push_back(info);
      }
      ```
      输出上面的信息，但是不能输出有无效号码的人
      ```
      for(const auto &info : people){
            ostringstream formatted,badNums;//保存格式化信息和无效信息
            for(const auto &num : info.phones){
              if(!vaild(num))badNums << " " << num;//如果号码无效则保存在内存badNums中
              else formatted << " " << num;//否则保存号码到formatted
            }
            if(badNums.str().empty())//没有无效号码
                cout << info.name << " " << formatted.str() << endl;
            else
                cerr << "input error:" << info.name << " " << badNums.str() << endl;
        }
        ```
## 容器
  - 顺序容器
    - vector
      - 初始化   
        最常用的有:1. 默认初始化，这里vector为空；2.copy初始化，用另一个vector初始化该vector 3.列表初始化。很少在初始化vector的时候去设定它的size，因为vector的push_bask非常高效。
        ```
        // T 表示实例化类模板时使用的类型
        vector<T> v1             　  　// 默认初始化, 此时v1为空
        vector<T> v1(v2)              // 执行的copy初始化，此时v1与v2的内容相同
        vector<T> v1 = v2      　　   // 与上面相同，都会执行copy构造函数
        vector<T> v1(n)           　  // 此时v1的size大小为n ,它里面的值是根据T的类型进行默认初始化的
        vector<T> v1(n, a)        　  // v1的初始化为n个值为a的元素
        vector<T> v1{a, b, c}    　 　// 列表初始化，v1内现在的元素就是a, b, c （这是c++11标准新入的）
        vector<T> v1 = {a, b, c} 　　 // 与上面相同
        ```
      - 常用操作
        ```
        //属性操作
        v1.size()　　　　　　//v1内已经存放的元素的数目
        v1.capacity()　　　　// v1现有的在存储容量（不再一次进行扩张内存空间的前提下）
        v1.empty()　　　　　// 判断v1是否为空
        v1.max_size()　　　 // 返回vector可以存放的最大元素个数，一般这个数很大，因为vector可以不断调整容量大小。
        v1.shrink_to_fit()　　// 该函数会把v1的capacity()的大小压缩到size()大小，即释放多余的内存空间。
        //访问操作：访问操作都会返回引用，通过它，我们可以修改vector中的值。
        v1[n]　　　　　　　　// 通过下标进行访问vector中的元素的引用 （下标一定要存在 ，否则未定义，软件直接崩了）
        v1.at(n)　　　　　　　// 与上面类似，返回下标为n的元素的引用，不同的是，如果下标不存在，它会抛出out_of_range的异常。它是安全的，建议使用它。
        v1.front()　　　　　　// 返回vector中头部的元素的引用（使用时，一定要进行非空判断）
        v1.back()　　　　　　// 返回vector中尾部的元素 引用（使用时，一定要进行非空判断）
        //添加操作
        v1.push_back(a)　　　　　　　　//在迭代器的尾部添加一个元素
        //v1.push_front(a)　　　　　　　　// vector不支持这个操作
        v1.insert(iter,  a)　　　　　　　// 将元素a 插入到迭代器指定的位置的前面，返回新插入元素的迭代器（在c++11标准之前的版本，返回void)
        v1.insert(iter, iter1, iter2)　　//把迭代器[iterator1, iterator2]对应的元素插入到迭代器iterator之前的位置，返回新插入的第一个元素的迭代器（在c++11标准之前的版本， 返回空）。
        在c++11标准中，引入了emplac_front()、 emplace()、emplace_back(), 它们分别与push_front()、insert()、 push_back()相对应，用法与完成的动作作完全相同，但是实现不一样。 push_front()、insert()各push_back()是对元素使用copy操作来完成的，而emplac_front()、 emplace()和emplace_back()是对元素使用构造来完成的，后者的效率更高，避免了不必要的操作。因此，在以后更后推荐使用它们。
        //删除操作
        v1.erase(iterator)　　　　　// 删除人人迭代器指定的元素，返回被删除元素之后的元素的迭代器。（效率很低，最好别用）
        v1.pop_front() 　　　　　　//vector不支持这个操作
        v1.pop_back()　　　　　　//删除vector尾部的元素 ， 返回void类型 (使用前，一定要记得非空判断）
        v1.clear()　　　　　　　　 //清空所有元素
        //替换操作
        v1.assign({初始化列表})　　　　// 相当于赋值操作
        v1.assign(n, T)　　　　　　　　// 此操作与初始化时的操作类似，用n个T类型的元素对v1进行赋值
        v1.assign(iter1, iter2)　　　// 使用迭代器[iter1, iter2]区间内的元素进行赋值（该迭代器别指向自身就可以），另外，只要迭代器指的元素类型相同即可（存放元素的容器不同，例如：可以用list容器内的值对vector容器进行assign操作，而用 "=" 绝对做不到的。
        v1.swap(v2)　　　　　　// 交换v1与v2中的元素。  swap操作速度很快，因为它是通过改变v1与v2两个容器内的数据结构（可能是类似指针之类的与v1和v2的绑定）完成的，不会对容器内的每一个元素进行交换。 这样做，不仅速度快，并且指向原容器的迭代器、引用以及指针等仍然有效，因为原始的数据没有变。c++ primer中建议使用非成员版本的swap()函数，它在范型编程中很重要。
        ```
      - 小结
         1. vector容器最重要的特性是：它在一段连续的内存空间中存储元素, 在常量时间内对vector容器进行随机访问，高效的在vector的尾部进行添加与删除操作，在中间或头部添加与删除元素的效率很低。
         2. 对vector进行增加与删除元素的操作，都会使迭代器、指针、引用失效（可能有时候它们仍然有效，但不能作这样假设）。所以使用vector的迭代器、引用和指针时，尽量避免增加与删除元素的操作。
         3. 对于vector的迭代器，它除了可以进行++iter与--iter的操作之外，还可以进行算术运算，例如：iter + n、::difference_type a = iter1 - iter2 //它的返回类型为 ::difference_type,例如vector<int>::difference_type （另一个也支持迭代器算术运算的容器为string)
    - string容器
      string与vector类似，但是string不是一种类模板，而就是一种类型，因为它专门用于存放字符的（存放的元素类型已经明确），所以没有设计为类模板。它的所有特性与vector相同，包括存储在连续的空间／快速随机访问／高效在尾部插入与删除／低效在中间插入与删除等， string的迭代器也支持算术运算。可以把string类型看作为vector<char>类型，vector的所有特性都适合于string类型。
      - 特性
        - 初始化
          ```
          string a("xiaoming")
          string a = "xiaoming"
          ```
        - string中包含的专有的操作（相对于vector来说）
          ```
          string的添加与替换：
          在string中，增加了append(）与 replace()函数
          str.append(args)　　　　// 在尾部添加一个字符或一个字符
          str.replace(pos, args)　　　　// 在尾部添加一个字符或一个字符 ,它的重载函数很多,共16个
          string的访问子字符串
          str.substr(_pos, n)　　//该函数可以获得原字符串中的部分字符, 从pos开始的n个字符，当_pos超过范围时，会抛出out_of_range的异常
          
          str的搜索操作：
          str.find(args)　　//查找args 第一次出现的位置
          str.rfind(args)　　//查找args最后一次出现的位置
          str.find_first_of(args)　　　//搜索的是字符， 第一个是args里的字符的位置
          str.find_last_of(args)　　　// 搜索的是字符， 最后一个是args里的字符的位置
          str.find_first_not_of()　　// 搜索的是字符，第一个不是args里的字符的位置
          str.find_last_not_of()　　// 搜索的是字符， 最后一个不是args里的字符的位置
          
          str的大小操作：
          str.length()　　　// 该函数与str.size()函数完成一样，只是名字不同而已罢了。只所以这样搞的原因，可能开发人员感觉length更适合字串符，size更适合容器吧
          
          c字符串的转换函数
          1. 由数值转换为字符串：
          to_string(val): 
          2. 由字符串转换为数值：（要转换的string的第一个非空白符必须是数值中可能出现的字符，处理直到不可能转换为数值的字符为止）
          stoi(str, pos, base)  　　// 字符串转换为整型，其中str表示字符串，pos用于表示转换后截断的第一个非数值字符的下标, base表数值的基数，默认为10，即10进制数。
          stol(str, pos, base)　　　　// 转换为long
          stoul(str, pos, base)　　　　// 转换为 unsigned long
          stoll(str, pos, base)　　　　// 转换为 long long
          stoull(str, pos, base)　　　// 转换为unsigned long long
          stof(str, pos)　　　　　　// 转换为float
          stod(str, pos,)　　　　　// 转换为double
          stold(str, pos,)　　　　　// 转换为long double
          
          对字符的操作：（在cctype头文件中，并不属于string头文件的范围，但是关系很紧密的），以下内容来自:c++ primer 第五版p82, 只写出部分常用来的(字母：alpha, 数字：number或digit)
          isalnum(c)　　// 当为字母或数字时为真
          isalpha(c)　　// 当为字母时为真
          isdigit(c)　　// 当为数字时真
          islower(c)　　// 当为小写字母时为真
          issupper(c)　　// 当为大写字母时为真
          isspace(c)　　// 当为空格时为真
          tolower(c)　　// 转换为小写字母， 当本身为小写字母时，原样输出
          toupper(c)　　// 转换为大写字母， 当本身为大写字母时，原样输出
          ```
    - list容器
      与vector和string相比，list内部的实现为一个双向链表，它的元素不是存储在连续的内存空间中，而是非连续的，这就决定了它不能在常量时间内完成对元素的随机访问，只能从头到尾的遍历一遍。因为它是用双向链表实现的，所以，它的一大特性就是它的迭代器永远不会变为无效（除非这段空间不存在了），即无论增加、删除操作，都不会破坏迭代器。与向量(vectors)相比, 它允许快速的插入和删除，但是随机访问比较慢。list不支持随机存取，要访问第n个元素，必须先遍历前n-1个元素才能访问第n个元素。因此，list没有下标[]操作，也没有at()接口。list的迭代器没有重载+n操作，但是重载了++, --。
      list与vector的差别：
      1. list支持push_front()、pop_front()操作
      2. list不支持vector中的随机访问操作，即使用v1.at( )和v1[ ] 操作
      3. list的删除与增加元素的操作不会破坏迭代器，而 vector与string 会使迭代器失效
      4. list 内部增加了一个sort()的方法，用于实现排序
      5. list增加了一个类似insert()的函数，为splice()：该函数可以实现在常数时间内把一个list插入到另一个list内，与insert()的区别在于insert是进行copy, 而splice()直接操作的链表的指针指向，它有好几个重载函数
      6. list的去重复函数unique()：函数的作用是去除连续重复的元素，参数即可以为空，也可以传入一个二元谓词，用于确定相等的比较算法，配合上sort()函数使用
      7. list的合并函数merge(): 该函数就是合并两个list，它在合并过程中会在两个链表之间进行来回的比较，如果原来的两个list是有顺序的，合并之后的结果也是有序的，如果合并之前是无序的，合并之后也是无序的
    - forward list 单向链表
      略
    - deque 是一个双端的数组，而vector 是单端的
      - 特点
        deque在接口上和vector非常相似，在许多操作的地方可以直接替换。deque可以随机存取元素（支持索引值直接存取，用[]操作符或at()方法）。deque头部和尾部添加或移除元素都非常快速, 但是在中部安插元素或移除元素比较费时。使用时，包含头文件：#include <deque>。
      - 常见操作
        ```
        // 下标方式获取里面的值或赋值
        d1.at(0) = 0;
        int d = d1[1];
        // 头部增加
        d1.push_front(1);
        // 尾部增加
        d1.push_back(2);
        // 头部删除
        d1.pop_front();
        // 尾部删去
        d1.pop_back();
        // 获取第一个元素
        d1.front();
        // 获取最后一个元素
        d1.back();
        // 使用迭代器方式输出
        deque<int>::iterator it = d1.begin();
        for (; it != d1.end(); it++) {
          cout << *it << " ";
        }
        // 常量迭代器
        deque<int>::const_iterator cit = d1.cbegin();
        for( ; cit!=d1.cend(); cit++){
          cout<<*cit;
          cout<<" ";
        }
        // 逆转的迭代器
        deque<int>::reverse_iterator rit=d1.rbegin();
        for(; rit!=d1.rend(); ++rit){
          cout<<*rit;
          cout<<" ";
        }
        // 第一个迭代器
        d1.begin();
        // 最后一个迭代的下一个迭代器
        d1.end();
        // 把d1的全部元素都赋值给d2
        d2.assign(d1.begin(), d1.end());
        // 给d2赋值6个666
        d2.assign(6, 666);
        // 调用赋值运算符重载 把d1的全部元素都赋值给d2
        d2 = d1;
        // 两个容器互换元素
        d2.swap(d1);
        // 获取容器元素的个数
        d1.size();
        // 设置容器的长度，如果短了则删去，多了则用0赋值
        d1.resize(5);
        // 设置容器的长度，多出来的部分用5赋值
        d1.resize(15, 5);
        // 在d1的开头位置插入一个5
        d1.insert(d1.begin(), 5);
        // 在d1的第二个位置开始插入两个888
        d1.insert(d1.begin() + 1, 2, 888);
        // 在第一的开头位置插入d2的所有元素
        d1.insert(d1.begin(), d2.begin(), d2.end());
        // 删除第二个元素
        d1.erase(d1.begin() + 1);
        // 删除一个区间的元素
        d1.erase(d1.begin() + 1, d1.begin() + 3);
        // 删除所有的元素
        d1.clear();
        // 典型的删除容器里的某个元素，vector也一样
        for (deque<int>::iterator it = d1.begin(); it != d1.end();) {
        if (*it == 4) {
            it = d1.erase(it);
          } else {
            it++;
          }
        }
        ```
    - array 
      与 vector 类型相似，数组也可以保存某种类型的一组对象；而它们的区别在于，数组的长度是固定的。数组一经创建，就不允许添加新的元素。指针则可以像迭代器一样用于遍历和检查数组中的元素。设计良好的程序只有在强调速度时才在类实现的内部使用数组和指针。
      ```
      //size是array的类型的组成部分，即array的类型由其元素的类型和其size共同决定
      array<int, 10>::size_type i; // array type includes element type and size
      array<int>::size_type j; // error: array<int> is not a type
      ```
      It is worth noting that although we cannot copy or assign objects of built-in array types, there is no such restriction on array:
      int digs[10] = {0,1,2,3,4,5,6,7,8,9};
      int cpy[10] = digs; // error: no copy or assignment for built-in arrays
      array<int, 10> digits = {0,1,2,3,4,5,6,7,8,9};
      array<int, 10> copy = digits; // ok: so long as array types match
    - 迭代器
      - 种类
        ```
        list<string> a = {"Milton", "Shakespeare", "Austen"};
        auto it1 = a.begin(); // list<string>::iterator
        auto it2 = a.rbegin(); // list<string>::reverse_iterator
        auto it3 = a.cbegin(); // list<string>::const_iterator
        auto it4 = a.crbegin();// list<string>::const_reverse_iterator
        ```
    - swap函数
      ```
      vector<string> svec1(10); // vector with ten elements
      vector<string> svec2(24); // vector with 24 elements
      swap(svec1, svec2);
      ```
      After the swap, svec1 contains 24 string elements and svec2 contains ten. With the exception of arrays, swapping two containers is guaranteed to be fast—the elements themselves are not swapped; internal data structures are swapped. For example, had iter denoted the string at position svec1 [3] before the swap, it will denote the element at position svec2[3] after the swap. Differently from the containers, a call to swap on a string may invalidate iterators, references and pointers.In the new library, the containers offer both a member and nonmember version of swap. Earlier versions of the library defined only the member version of swap. The nonmember swap is of most  importance in generic programs. As a matter of habit, it is best to use the nonmember version of swap.
      
