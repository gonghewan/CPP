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
  - 标志位
    - badbit表示发生系统级的错误，如不可恢复的读写错误。通常情况下一旦badbit被置位，流就无法再使用了
    - failbit 表示发生可恢复的错误，如期望读取一个数值，却读出一个字符等错误。这种问题通常是可以修改的，流还可以继续使用
    
