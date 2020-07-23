# C/C++

#### C和 C++ 区别
- 设计思想：C++面向对象，C面向过程
- 语法：
  - C++封装、继承、多态
  - C++相比C，增加许多类型安全的功能，如强制类型转换
  - C++支持范式编程，比如模板类、函数模板等


#### Strcpy
  - 源字符串const参数
  - 地址判断非null
  - 链式操作保存首地址

#### 野指针
- 野指针就是指向一个已删除的对象或者未申请访问受限内存区域的指针
- 成因
  - 指针变量未初始化
  - 指针释放后之后未置空
  - 指针操作超越变量作用域

#### Nginx负载均衡原理/机制

1. 轮询（默认）

2. 指定权重

3. IP绑定 ip_hash  
每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。

4. fair（第三方）  
按后端服务器的响应时间来分配请求，响应时间短的优先分配。

5. url_hash（第三方）  
按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。

### raii (?)

## C++ 11与98版区别
#### C++智能指针 
##### auto_ptr
######  98版有，C++11弃用
- **原理**：
  - 智能指针本质上是一个类，可以通过将普通指针作为参数传入智能指针的构造函数实现绑定。
  - 通过运算符重载使得其使用方式与指针相同。
- **优势**：
  - 由于auto指针本身是一个类，所以创建时在栈上，故而创建出来的对象在出作用域的时候（函数/程序结束）会自己消亡，所以在类中写好析构函数就可以完成**智能的内存回收**。

- **劣势**：

  - 赋值时（`operator =`），赋值右边的变量内部指针会被release，若再在后面调用`_Right.func()`会导致crash（零指针） ：（附源码部分）
    ```
    问题代码：
    void TestAutoPtr2()
    {
      auto_ptr<Test> my_auto1 (new Test(1));
      if(my_auto1.get())
      {
          (*my_auto1).info_extend = "Test";    //先赋予一个初始值作为区别两个指针的标志
          
          auto_ptr<Test> my_auto2;
          my_auto2 = my_auto1;
          my_auto2->PrintSomething();            //打印发现成功转移
          my_auto1->PringSomething();            //**crash**
      }
    }

    原因：
    _Myt& operator=(_Myt& _Right) _THROW0()
		{	
      // assign compatible _Right (assume pointer)
      reset(_Right.release());
      return (*this);
    }

    _Ty *release() _THROW0()
		{	
      // return wrapped pointer and give up ownership
      _Ty *_Tmp = _Myptr;
      _Myptr = 0;   //**此时会置零**
      return (_Tmp);
		}

    ```
  - 一点源代码拓展：
  C++中的`explicit`关键字只能用于修饰只有一个参数的类构造函数, 它的作用是表明该构造函数是显示的, 而非隐式的, 跟它相对应的另一个关键字是implicit, 意思是隐藏的,类构造函数默认情况下即声明为`implicit`(隐式)。  
  `class auto_ptr`中，显示构造函数只有`explicit auto_ptr(_Ty *_Ptr = 0) _THROW0() : _Myptr(_Ptr)`.

##### unique_ptr（替换auto_ptr）
######  C++11支持

- unique_ptr实现独占式拥有或严格拥有概念，保证同一时间内只有一个智能指针可以指向该对象。它对于避免资源泄露(例如“以new创建对象后因为发生异常而忘记调用delete”)特别有用。
  ```
  unique_ptr<string> p3 (new string ("auto"));   //#4
  unique_ptr<string> p4；                       //#5
  p4 = p3;//此时会报错！！
  ```
  - 编译器认为p4=p3非法，避免了p3不再指向有效数据的问题。因此，unique_ptr比auto_ptr更安全。

- 另外unique_ptr还有更聪明的地方：当程序试图将一个 unique_ptr 赋值给另一个时，如果源 unique_ptr 是个临时右值，编译器允许这么做；如果源 unique_ptr 将存在一段时间，编译器将禁止这么做，比如：
  ```
  unique_ptr<string> pu1(new string ("hello world"));
  unique_ptr<string> pu2;
  pu2 = pu1;                                      // #1 not allowed
  unique_ptr<string> pu3;
  pu3 = unique_ptr<string>(new string ("You"));   // #2 allowed
  ```
  - 其中#1留下悬挂的unique_ptr(pu1)，这可能导致危害。而#2不会留下悬挂的unique_ptr，因为它调用 unique_ptr 的构造函数，该构造函数创建的临时对象在其所有权让给 pu3 后就会被销毁。这种随情况而已的行为表明，unique_ptr 优于允许两种赋值的auto_ptr。

#### shared_ptr
######  C++11支持

- shared_ptr实现共享式拥有概念。多个智能指针可以指向相同对象，该对象和其相关资源会在“最后一个引用被销毁”时候释放。从名字share就可以看出了资源可以被多个指针共享，它使用计数机制来表明资源被几个指针共享。可以通过成员函数use_count()来查看资源的所有者个数。除了可以通过new来构造，还可以通过传入auto_ptr, unique_ptr,weak_ptr来构造。当我们调用release()时，当前指针会释放资源所有权，计数减一。当计数等于0时，资源会被释放。

- shared_ptr 是为了解决 auto_ptr 在对象所有权上的局限性(auto_ptr 是独占的), 在使用引用计数的机制上提供了可以共享所有权的智能指针。

- 成员函数：
  ```
  use_count 返回引用计数的个数
  unique 返回是否是独占所有权( use_count 为 1)
  swap 交换两个 shared_ptr 对象(即交换所拥有的对象)
  reset 放弃内部对象的所有权或拥有对象的变更, 会引起原有对象的引用计数的减少
  get 返回内部对象(指针), 由于已经重载了()方法, 因此和直接使用对象是一样的.如 shared_ptr<int> sp(new int(1)); sp 与 sp.get()是等价的
  ```

#### weak_ptr
######  C++11支持

- weak_ptr 是一种不控制对象生命周期的智能指针, 它指向一个 shared_ptr 管理的对象. 进行该对象的内存管理的是那个强引用的 shared_ptr. 
- weak_ptr只是提供了对管理对象的一个访问手段。weak_ptr 设计的目的是为配合 shared_ptr 而引入的一种智能指针来协助 shared_ptr 工作, **它只可以从一个 shared_ptr 或另一个 weak_ptr 对象构造**, 它的构造和析构不会引起引用记数的增加或减少。weak_ptr是用来解决shared_ptr相互引用时的死锁问题,如果说两个shared_ptr相互引用,那么这两个指针的引用计数永远不可能下降为0,资源永远不会释放。它是对对象的一种弱引用，不会增加对象的引用计数，和shared_ptr之间可以相互转化，shared_ptr可以直接赋值给它，它可以通过调用lock函数来获得shared_ptr。

#### const / static
- const: 
  - 变量值无法被改变
  - 若在函数声明中，则形参不可在函数体中被改变，防止篡改
  - 若在类的成员函数（在函数最后加const），则类的成员变量不可被修改（相当于const类的this指针）
  - 对指针来说，可以指定指针本身为const，也可以指定指针所指的数据为const，或二者同时指定为const
  - 成员函数返回值为const，防止函数放在“左值”。

- static:
  - 函数内：内存只被分配一次，下次call时依然用的是同一个变量
  - 模块内static变量：该模块都可使用，模块外不可
  - 模块内static函数：只可被这一模块内的其他函数调用，使用范围被限制在声明它的模块内
  - 类的static变量：所有类的对象共享一个变量
  - 类的static函数：改函数不接受this指针，故而只能访问类的static变量

#### 宏的使用
- 对于参数和整个宏，要用括号括起来
- 防止宏的副作用，对于参数，实行一比一代入，设计++操作很可能会自增多次
- 不能有`;`

#### C++中四种类型转换

- const_cast：用于将const变量转为非const
- static_cast：用于各种隐式转换，比如非const转const，void*转指针等, static_cast能用于多态向上转化，如果向下转能成功但是不安全，结果未知；
- dynamic_cast：用于动态类型转换。只能用于含有虚函数的类，用于类层次间的向上和向下转化。只能转指针或引用。向下转化时，如果是非法的对于指针返回NULL，对于引用抛异常。要深入了解内部转换的原理。
  - 向上转换：指的是子类向基类的转换
  - 向下转换：指的是基类向子类的转换
  - 它通过判断在执行到该语句的时候变量的运行时类型和要转换的类型是否相同来判断是否能够进行向下转换。

- reinterpret_cast：几乎什么都可以转，比如将int转指针，可能会出问题，尽量少用；

- **为什么不使用C的强制转换？**
  - C的强制转换表面上看起来功能强大什么都能转，但是转化不够明确，不能进行错误检查，容易出错。

## 指针和数组
 | 指针 | 数组 | 
 | --- | --- | 
 | 存数据的地址 | 保存数据 | 
 | 间接访问数据，首先获得指针的内容，然后将其作为地址，从该地址中提取数据 | 直接访问数据 | 
 | 通常用于动态的数据结构 | 通常用于固定数目且数据类型相同的元素 | 
 | 通常用于动态的数据结构 | 通常用于固定数目且数据类型相的元素 | 
 | 通过Malloc分配内存，free释放内存 | 隐式的分配和删除 | 
 | 通常指向匿名数据，操作匿名函数 | 自身即为数据名 | 