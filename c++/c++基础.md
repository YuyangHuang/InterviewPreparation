# c++


#### Strcpy
  - 源字符串const参数
  - 地址判断非null
  - 链式操作保存首地址

#### 野指针

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

## C++与98版区别
#### C++智能指针 auto
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



