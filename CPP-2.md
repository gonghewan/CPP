# Library
## I/O
- 关系
  istream----->ifstream----->istringstream
     |
  iostream---->iofstream---->iostringstream
     | 
  ostream----->ofstream----->ostringstream
     
  streambuf--->filebuf------>stringbuf
  cin : 一个istream对象，从标准输入中读取数据，注意为字节读取
  cout :一个ostream对象，从标准输出中写入数据
  >> : 用来从一个istream对象读取输入数据，如cin >> x，表示输入数据流到x
  << : 用来向一个ostream对象写入输出数据，如cout << x，表示x输出到ostream对象
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
