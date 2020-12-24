### list
 - list 双向链表，会在结尾元素后添加一个空元素
   
   eg: | 1 | 2 | 3 |
    
   实际上是 | 1 -> | 2 -> | 3 ->| XX ->|
      
             ^                    ^
              
             |                    |
              
           list.begin()         list.end()     返回的是iterator，*iter获取内容
             ^              ^
              
             |              |
               
           list.front()   list.back()        返回的是引用
   在一些编译器中可能在输出list.end()的解引用时输出的是list的最后一个有内容的元素（例子中的“3”），有的可能输出空
   值得注意的一点：
      ```
      if (!c.empty()) {
         c.front() = 42; // assigns 42 to the first element in c
         auto &v = c.back(); // get a reference to the last element
         v = 1024; // changes the element in c
         auto v2 = c.back(); // v2 is not a reference; it's a copy of c.back()
         v2 = 0; // no change to the element in c
      }
      ```
### vector
 - 随机存取
   vector.at(n)要比vector[n]安全
 - 有增删的遍历
   错误做法：
   ```
    // disaster: the behavior of this loop is undefined
    auto begin = v.begin(),
    end = v.end(); // bad idea, saving the value of the end iterator
    while (begin != end) {
    // do some processing
    // insert the new value and reassign begin, which otherwise would be invalid
    ++begin; // advance begin because we want to insert after this element
    begin = v.insert(begin, 42); // insert the new value
    ++begin; // advance begin past the element we just added
    }
    ```
    正确做法：
    ```
    // safer: recalculate end on each trip whenever the loop adds/erases elements
    while (begin != v.end()) {
    // do some processing
    ++begin; // advance begin because we want to insert after this element
    begin = v.insert(begin, 42); // insert the new value
    ++begin; // advance begin past the element we just added
    }
    ```
