1. C++里面的内存是如何划分的，不同区域用于存放什么
   * .rodata段：存放只读数据，比如printf语句中的格式字符串和开关语句的跳转表。也就是你所说的常量区。
   * .text段：存放已编译程序的机器代码。
     注意：程序加载运行时，.rodata段和.text段通常合并到一个Segment（Text Segment）中，操作系统将这个Segment的页面只读保护起来，防止意外的改写。
   * .data段：存放已初始化的全局变量。而局部变量在运行时保存在栈中，既不出现在.data段，也不出现在.bss段中。就是你所说的全局区。

      例如：全局作用域中的int ival = 10，static int a = 30，以及局部作用域中的static int b = 30，这3个变量均存放在.data段中。注意，局部作用域中的static变量的生命周期和其余栈变量的生命周期是不同的。

   * .bss段：存放未初始化的全局变量。在目标文件中这个段不占据实际的空间，它仅仅是一个占位符，在加载时这个段用0填充。目标文件区分初始化和未初始化变量是为了空间效率：在目标文件中，未初始化变量不需要占据任何实际的磁盘空间。全局变量如果不初始化则初值为0，同理可以推断，static变量（不管是函数里的还是函数外的）如果不初始化则初值也是0，也分配在.bss段。例如，全局作用域中的int ival; ival显然存放在.bss段
   *  栈：函数的参数和局部变量是分配在栈上（但不包括static声明的变量）。在函数被调用时，栈用来传递参数和返回值。由于栈的后进先出特点，所以栈特别方便用来保存/恢复调用现场。
   * 堆：用于存放进程运行中被动态分配的内存段，它的大小并不固定，可动态扩张或缩减。当进程调用malloc/free等函数分配内存时，新分配的内存就被动态添加到堆上（堆被扩张）/释放的内存从堆中被剔除（堆被缩减）

2. C++里面的vector,map,list的实现方式
  1. vector
      vector是我们用到最多的数据结构，其底层数据结构是数组，由于数组的特点，vector也具有以下特性：
      1、O(1)时间的快速访问；
      2、顺序存储，所以插入到非尾结点位置所需时间复杂度为O(n)，删除也一样；
      3、扩容规则：
                当我们新建一个vector的时候，会首先分配给他一片连续的内存空间，如std::vector<int> vec，当通过push_back向其中增加元素时，如果初始分配空间已满，就会引起vector扩容，其扩容规则在gcc下以2倍方式完成：首先重新申请一个2倍大的内存空间；然后将原空间的内容拷贝过来；最后将原空间内容进行释放，将内存交还给操作系统；
      4、注意事项：
                根据vector的插入和删除特性，以及扩容规则，我们在使用vector的时候要注意，在插入位置和删除位置之后的所有迭代器和指针引用都会失效，同理，扩容之后的所有迭代器指针和引用也都会失效。

  2. map & multimap & unordered_map & unordered_multimap
    a.map与multimap底层数据结构
      map与multimap是STL中的关联容器、提供一对一key-value的数据处理能力； map与multimap的区别在于，multimap允许关键字重复，而map不允许重复. 这两个关联容器的底层数据结构均为红黑树，关于红黑树的理解可以参考教你透彻了解红黑树一文。
    
      根据红黑树的原理，map与multimap可以实现O(lgn)的查找，插入和删除。

    b. unordered_map 与unordered_multimap底层数据结构
        unordered_map与unordered_multimap 对比2.1中的两种map在于其2.1中的两个容器实现了以key为序排列，也就是说map与multimap为有序的。而unordered_map与unordered_multimap中key为无序排列，其底层实现为hash table，因此其查找时间复杂度理论上达到了O(n)，之所以说理论上是因为在理想无碰撞的情况下，而真实情况未必如此。
  
  3. set & multiset & unordered_set & unordered_multiset
      以上四种容器也都是关联容器，set系与map系的区别在于map中存储的是<key-value>，而set可以理解为关键字即值，即只保存关键字的容器。

      3.1 set & multiset底层数据结构
          set与multiset有序存储元素，这两种容器的底层实现与map一样都是红黑树，所以能实现O(lgn)的查找，插入，删除操作。

          set与multiset的区别在于是否允许重复；

      3.2 unordered_set & unordered_multiset
          与unordered_map & unordered_multimap相同，其底层实现为hash table；
  4. priority_queue
      优先级队列相当于一个有权值的单向队列queue，在这个队列中，所有元素是按照优先级排列的。priority_queue根据堆的处理规则来调整元素之间的位置，关于堆的原理，可以参考堆；根据堆的特性，优先级队列实现了取出最大最小元素时间复杂度为O(1),对于插入和删除，其最坏情况为O(lgn)。
  5. list:
     list的底层数据结构为双向链表，特点是支持快速的增删。
  6. queue  
      queue为单向队列，为先入先出原则。
  7. deque
      deque为双向队列，其对比queue可以实现在头尾两端高效的插入和删除操作。