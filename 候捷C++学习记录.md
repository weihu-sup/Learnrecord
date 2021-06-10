#C++面向对象程序设计（上）
##1、文件与类的声明

* **头文件防卫式声明**
> 
	#ifndef _COMPLEX_
	#define _COMPLEX_
	...
	#endif

* **构造函数 初始化参数**
>
	class complex
	{
	public:
		complex (double r = 0, double i = 0)
			: re (r) , im(i) //默认初始化为0 
		{ }
	private:
		double re, im;
	}
	complex c1; //使用默认初始值
	complex c2(2.1,3.4); //传入初始值
##2、构造函数
* inline(内联)函数

在class本体内定义（定义和声明的区别）完成，便成为inline候选人。  
在外部定义就不是inline候选人。  
比较简单的函数才会**有可能**成为inline内联函数。  
即使声明为inline函数也有可能不会被编译器编译成内联函数。  
最终函数是否变成inline函数**由编译器决定**。  
内联函数执行效率更高，会很好，哈哈哈。  
函数太复杂编译器没有能力做成inline函数。
>
	class complex
	{
	public:
		complex (double r = 0, double i = 0)
			: re (r) , im(i) //默认初始化为0 
		{ }  //这里用构造函数特有的赋值方法更好 效率更好
	  //complex() : re(0), im(0) {} //会报错 因为上一个构造参数有默认参数
		
>
	complex& operator += (const complex&); //只是声明
	double real () const { return re;} //有可能成为内联函数
	double imag () const { return im;} //有可能成为内联函数  
>
	private:
		double re, im;
		friend complex& _doapl (complex* , const complex&);
	};

>
	complex c1; //使用默认初始值
	complex c2(2.1,3.4); //传入初始值

>
	inline double  //有可能成为内联函数
	imag(const complex& x)
	{
		return x.imag();
	}
注释：class有一个经典的分类： 1、带指针 2、不带指针


* 函数重载

>    
	complex(double r = 0, double i = 0) : re(r) , im(i){}
	complex(): re(0),im(0){} 
	//这两个构造函数不能同时存在 因为都有默认参数，编译器不知道调用哪一个
##3、参数传递与返回值
* 构造函数放在private区域 （即不允许被外界构造对象）

经典例子

这是一个设计模式 最简单的设计模式 Singleton(单例模式)
>
	class A{
	public:
	  static A& getInstance();
	  setup() { ... }
	private:
	  A();
	  A(const A& rhs);
	  ...
	};
>
	A& A::getInstance()
	{	
	  static A a;
	  return a;
	}
* 常量成员函数


	  double real () const { return re;} //有可能成为内联函数
	  double imag () const { return im;} //有可能成为内联函数  
加了const 是常量成员函数     
什么函数要加const   
**不改变数据内容的函数**，如例子real() 和 imag() 函数功能是读即返回变量值，不是写变量值  
设计接口函数时就要考虑函数功能，是否需要加const

倘若设计者忘了加const  

	double real() {return re;}  //此处的声明定义表示 data有可能改变
	double imag() {return im;}
使用者做如下调用会报错：
	
	const complex c1(2,1); //这里表明我的complex 的data是常量不会变
	cout << c1.real(); //调用是不变的 但是函数声明却表明data有可能改变 冲突矛盾
	cout << c1.imag();
* 参数传递（分两类 pass by value . pass by reference(to const)）  
尽量不要传value 传引用或指针    
	
	complex& operator += (**const complex&**);//这里的函数参数传递的是 const 引用（reference(to const)） 表明我传给你的是你不能修改的
* 返回值传递：（return by value vs. return by reference(to const)）  
* 友元  
友元函数可以直接拿自己的数据（private），不需要返回函数来操作 
* 相同类（class）的各个对象互为友元
>
	class complex
	{
	public:
		complex (double r = 0, double i = 0)
			: re (r) , im(i) //默认初始化为0 
		{}
>
	int func(const complex& param)
	{ return parm.re + parm.im;} //这个函数的 param 直接拿了data数据，没有通过函数返回是不是就破坏了封装性？ 其实并没有，因为这个函数调用肯定是同类的不同对象，互为友元
 
>
	private:
		double re, im;
	};
>
	complex c1(2,1);
	complex c2;
	c2.func(c1); //c1 c2 是相同类的不同对象， c1 c2 互为友元这里可以用
* class body外的各种定义（definitions）  
什么情况下可以pass by reference (传)  
什么情况下可以return by reference (返回)  
如果在函数内部的局部声明的空间内存不能return by reference（除此外其他情况都可以）  
	
	inline complex& _doapl(complex* ths, const complex& r) //第一参数将会被改动 第二参数不会被改动
	{
 		ths->re += r.re;   
		ths->im += r.im;
		return * ths;
	}
##4、操作符重载与临时对象
* operator overloading(操作符重载-1，成员函数)  **this**
>
	inline complex& complex::operator += (const complex& r)
	{
		return _doapl(this, r);
	}
>
	inline complex& complex::operator += (this, const complex& r) //不能把this参数写明 是隐藏的 默认
	{
		return _doapl(this, r);
	}
    //任何成员函数都有一个隐藏的this 指针 this指向调用者 谁调用函数就指向谁
* return by reference语法分析  
**传递者**无需知道**接收者**是以**reference形式**接收  （reference）  
reference（引用）和point（指针）区别 ：
如果用Point来传 传的人必须知道是point 接收和传递都为point

	inline complex& _doapl(complex* this, const complex& r)
	{	
 		...
		return *this;
	}

	inline complex& complex::operator += (const complex& r)//可以用& 来接收，因为有this存放
	{
		return _doapl(this, r);
	}
	//+=的操作符重载返回值不能是void 因为 c2 += c1没问题 但是c3 += c2 += c1 就会有问题（c2 += c1 需要返回一个complex）
	
	inline complex operator + (const complex& x, const complex& y)
	{
		return complex (real(x) + real(y) , imag(x) + imag(y));//typename() 创建临时对象
	}
	//上面这个函数决不可以 return by reference, 因为, 它返回的必定是个local object
	//这个函数需要创建temp object（临时对象） 退出函数就不存在了
类名称加小括号 是创建临时对象；特殊语法 很少人用 但是标准库使用很多 
##5、三大函数：拷贝构造，拷贝赋值，析构
只要类带有指针 不能用编译器自带的那一套 需要自己写拷贝构造 拷贝赋值  
如果不带有指针，可以用编译器自带的拷贝构造 拷贝赋值 设计者不用管

	class String
	{
	public:
		String(const char* cstr = 0);
		String(const String& str); //拷贝构造 （注意参数 是自己）
		String& operator = (const String& str); //拷贝复制 （注意参数 是自己）
		~String();  //析构函数
		inline char* get_c_str() const { return m_data;}
	private:
		char* m_data;
	}

构造函数：
  
	inline String::String(const char* cstr = 0)
	{
		if(cstr) {
			m_data = new char[strlen(cstr) + 1];//要加结束符号
			strcpy(m_data, cstr);
		}
		else{
        	m_data = new char[1];
			*m_data = '\0';
		}
	}
析构函数：  

	inline String::~String()
	{
		delete[] m_data;	
	}
* **class with pointer members 必须有 copy ctor（拷贝构造）和 copy op =(拷贝复制)**  

拷贝构造：(copy ctor)

	inline String::String(const String& str)  //深拷贝
	{
		m_data = new char[ strlen(str.m_data) + 1]; //拷贝数据 原有指针不变
		strcpy(m_data, str.m_data);
	}
浅拷贝 只拷贝了指针 造成内存泄漏（如 s1 = s2, s2指针和s1指针指向同一个地址，但s2原本指向的地址空间就没有指针指向了，造成内存泄漏）

拷贝赋值：

	inline String& String：：operator=(const String& str)
	{
		if(this == &str)   //**检测自我赋值**  必不可少 
			return *this;

		deleta[] m_data; //第一步先清空自己
		m_data = new char[ strlen(str.m_data) + 1];//第二步重新创造自己
		strcpy(m_data, str.m_data);//第三步拷贝
		return *this;
	}

注意：  

	String s2(s1); 与 String s2 = s1; 同样调用拷贝构造
	s2 = s1; 调用拷贝赋值函数

##6、堆 栈 与 内存管理
 **栈**，是存在于某作用域的一块内存空间。例如当你调用某个函数，函数本身即会形成一个栈用来放置它所接收的参数，以及返回地址。  
在函数本体内声明的任何变量，其所使用的内存块都取自上述栈。

**堆**，是指由**操作系统**提供的一块全局内存空间，程序可以动态分配，从某中获取若干块。

	class complex{};

	{
	  complex c1(1,2):
	}
c1是所谓的栈对象，其生命在作用域结束之后结束。这种作用域内的对象，有称为自动对象，因为他会被【自动】清理。  

	static complex c2(1,2):
c2便是所谓静态对象， 其生命在作用域结束之后仍然存在，直到整个程序结束。

**new:先分配memory,再调用ctor**
new 动作在编译器内分为三个动作：  
1、分配内存 （其内部实际调用malloc(n)）
2、转型  
3、构造函数  
**delete: 先调用dtor，再释放内存**  
delete 在编译器内分为两步：  
1、析构函数  
2、释放内存   （其内部调用free(ps)）

##7、组合与继承

类的关系（掌握三种就可以了）：继承 复合 委托 

类关系： Composition（复合）关系下的构造和析构 （has a的关系）  
Container 有一个（包含） Component  
**构造由内而外**（Container的构造函数首先调用Component的默认构造函数然后才执行自己）  
**析构由外而内**（Container 的析构函数首先执行自己，然后才调用Comnonent的析构函数）

* 委托（Delegation ）  
	class String  
	{  
	  public:  
	  private:  
		StringRep * rep;  
	}  
	class StringRep  
	{  
	}  
* 继承（Inheritance）   is a  
**构造由内而外**（子类的构造函数首先调用父类的默认构造函数然后才执行自己）  
**析构由外而内**（子类的析构函数首先执行自己，然后才调用父类的析构函数）  
**父类的析构函数必须是virtual，否则会出现 undefined behavior**  

##8、虚函数与多态  

**non-virtual函数：你不希望derived class(子类)重新定义（复写）它**  
**virtual函数：你希望derived class重新定义（复写）它，而且你对它已有默认定义**
**pure virtual函数：你希望derived class 一定要重新定义（复习）它，你对它没有默认定义**  




 


#C++面向对象程序设计（下）




