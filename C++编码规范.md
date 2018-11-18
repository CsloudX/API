# C++编码规范

## 概述

在一个团队中，一致的编码规范可以是团队成员间协同更加高效，易于掌握的API可以让使用者快速使用，接口健壮可以使功能更加灵活。纵观业界知名框架，如Qt、.NET、Unity等都完美达到了这些要求。本文借鉴了《Qt API设计原则》、《Google C++编程规范》和《华为C++编程规范》，提取共性，为C++开发者提供一些通用的指导原则。

[TOC]

## 基本原则

API的使用情景主要在以下情景：

* 计算机用来编译目标程序。
* 开发者用来完成自身业务逻辑。
* 使用者调用完成业务逻辑。
* 维护者理解修改以完成业务逻辑。

对于计算机来说，它只关心代码是否存在语法错误，其他如可维护性、可读性等特性，是开发者、使用者和维护者关心的事情。Matthias 在[关于API设计](https://doc.qt.io/archives/qq/qq13-apis.html)的文章提出过自己的观点：API应该极简且完备、语义清晰简单、符合直觉、易于记忆和引导使用者写出可读代码。代码应当是为开发者服务的，因此应当将人放在第一，机器第二。可读性优先，优秀的代码完全能做到自我解释。代码只会写一遍，但是阅读、修改、理解却会很多次，可能是3天后，也可能是3年后。

优秀的程序代码不会因为程序功能越来越多而变得越来越复杂、越来越难以维护。因为在庞大的系统都可以划分为子系统，子系统又可以继续划分子系统。在每一个系统层次中，都是逻辑清晰、组织合理且明确的。相反，由于后期的维护重构，应当是组织越来越清晰，越来越合理。

优秀的代码应当具有如下原则：

###极简

极简指每个class的public成员（包括成员函数和成员变量）尽可能少，public的class也尽可能少。这样的API更易理解、记忆、调试和变更。能够定义成private的成员就不要定义为public，能够暴露的class就不要暴露出来。

例如我们需要实现一个解析XML的类Xml，我们可能还需要实现一些辅助类如XmlParser（语法解析）、XmlReader（读取）、XmlWriter（写入）等、除了辅助类，还有很多处理业务逻辑的成员函数。对于使用者来说，他们只关心读取一个xml文件，获取其中的数据、修改其中数据、写入磁盘等操作，其他的他们并不关心。对于如何实现的，是Xml类提供者考虑的，使用者并不关心。因此，我们只需要暴露一个Xml类，Xml类成员暴露需要使用到的接口即可。伪代码如下：

~~~C++	
class Xml
{
public:
  bool load(const string& pathfile);
  bool save(const string& pathfile);
  
  string nodeKey(const string& path) const;
  string nodeValue(const string& path) const;
  
  void setNodeKey(const string& path, const string& keyValue);
  //...
}
~~~

使用者可以像如下使用代码：

~~~c++
Xml xml;
if(!xml.load("test.xml")){
  //...
  return;
}

string value = xml.nodeValue("root/item1");
xml.setNodeKey("root/item1","item2");

if(!xml.save("test2.xml")){
  //...
  return;
}
~~~

如何在后期发现Xml类存在bug或者需要为Xml调整功能，使用者的代码将不会受到影响。

###完备

完备指一个功能模块所期望功能应该是完整的。如果一个功能放在了错误的模块中，那么用户可能不能在第一时间找到这个API，也是违背了完备性的。

~~~c++
class FileSystem
{
	int calcCircleArea(double radius);	// error，应当放置MathSystem中
}
~~~



###语义清晰简单

就像其他的设计一样，我们应当遵守最少意外原则。好的API可以让常见的事情完成得更简单，并有可能完成不常见的事的可能性。例如我们的Xml类，我们可以这样使用：

~~~c++
// 常见操作
XXml xml;
xml.setNodeValue("root/item1","100")
xml.save("test.xml");	
~~~

也可以这样使用

~~~c++
// 不常见操作
XXml xml;
xml.setVersion("1.0");
xml.setEncoding("ISO-8859-1");
xml.setNodeValue("root/item1","100")
xml.save("test.xml");	// 常见操作，自动解码
~~~

###符合直觉

就像世间万物一样，API也应该符合直觉。对于什么是符合直觉的什么不符合，不同经验和背景的人会有不同的看法。API符合直觉的测试方法：经验不很丰富的用户不用阅读API文档就能搞懂API，而且程序员不用了解API就能看明白使用API的代码。程序代码执行逻辑就是我们看到的代码。

###易于记忆

命名约定应该具有一致性和精确性。使用易于识别的模式和概念，除了十分明确的缩写单词，其他情节应当避免用缩写。如count不要写成cnt，dialog不要写成dlg。

###引导API使用者写出可读代码

代码只写一次，却要多次的阅读（还有调试、修改和维护）。写出可读性好的代码有时候要花费更多的时间，但对于产品的整个生命周期来说是节省了时间的。

因此，检验代码设计得十分优秀的准则就是看其可读性，本文也是围绕这个主题展开的。

## 命名

###清晰明了，驼峰式命名

命名应当清晰明了一眼就能看出所表示的含义，去除不必要的前缀（类名前面的C、整型全面的i、槽函数前面的slot等），不要使用匈牙利命名方式，指针不用前缀p。匈牙利命名法则最先由微软推广，到目前为止，没有任何一家IT公司在推广采用（包括微软）。匈牙利命名法则写出的代码冗余太多且难以阅读。微软的C#、Google的C++命名规则、Qt等没有采用匈牙利命名法则。

命名也是一门艺术，牛头不对马嘴的命名一定要避免。如下是一些糟糕的命名：

~~~c++
void resetProject();	// 其实这是一个加载工程的函数

Point size;
size.x;	// 这个x到底表示宽度还是坐标x呢？

Sutdent s12345;
~~~

### 不要随意缩写，禁止拼音命名

如果单词缩写，就不能让代码第一眼就反应含义。并且，在不同的人缩写的词语也会不同，缩写虽然在编写代码是可以少写几个字母，但是从长远来考虑，缩写带来的回报是负的，因为用户必须记住缩写的含义。因此，除非是十分明确的缩写，否则就不要进行单词缩写。也禁止拼音，那样会让代码含义很难理解。

~~~c++
// bad
int stuCnt;		
Node jieDian;

// good
int studentCount；
Node node;
~~~



### 类、结构体、枚举命名

驼峰式命名且首字母大写，不要采用前缀（如类的C前缀、结构体的S前缀、枚举的E前缀等）。名称必须清晰明了。类、结构体通常表示的某种对象，因此必定为名词性的名称，如果不是，请考虑类设计或类名是否合理。

例外：接口类采用I开头，并且以Interface结束，IXXXInterface的形式。

~~~c++
class EnglishTeacher{}；

typedef struct{
	string 	name;
	int 	age;
}Student;

enum Color{Red,Blue,Green};

class INetworkInterface{};
~~~

另外需要注意的枚举值命名不要过于通用。

~~~c++
// bad
enum CaseSensitivity{Insensitive, Sensitive};
str.indexOf("abc",Insensitive);

// ok
enum CaseSensitivity{CaseInsensitive, CaseSensitive};
str.indexOf("abc",CaseInsensitive);
~~~



### 变量命名

变量命名同样采用驼峰式命名且首字母小写，变量通常也是表示某个对象，因此名称也通常是名词（表示循环的`i`、`j`、`k`等除外）。类成员变量命名在前面加`m_`，方便区分函数参数。匈牙利前缀增加编码工作量、不利于阅读且难于修改，因此禁止添加匈牙利式的前缀。

~~~c++
// 匈牙利式
class CTeacher
{
public:
  	void setName(string sName)
    {
      	m_sName = sName;
    }
private:
  	string 	m_sName;
  	int		m_iAge;
	list<CStudent> m_lstoStudents;
};

// 推荐方式
class Teacher
{
public:
  	void setName(string name)
    {
      	m_name = name;
    }
private:
  	string 	m_name;
  	int		m_age;
  	list<Student> m_students;
}
~~~

对于const变量，单词全部采用大写（尽量用const代替宏），单词间使用下划线隔开。

~~~c++
const double PI = 3.1415926;
const string DEFAULT_NAME = "Untitled";

double area = PI*radius*radius;
doc.setTitle(DEFAULT_NAME);
~~~



### 函数命名

####驼峰式命名，首字母小写

函数命名采用驼峰式命名且首字母小写。函数名必须简单直接，不隐藏设计者意图。对于表示执行某种功能的函数，采用动词+修饰性词组方式，如`setName(const string& name)`，对于表示属性的函数，直接返回属性名称即可，如`name()`，对于表示某种对象某些状态的函数，需要在函数名称上明确表示出来，如`isTooOld()`。以下为使用效果：

~~~c++
// 不好的代码
CStudent oStudent;
string sDefaultName = "Tom";
oStudent.mvSetName(sDefaultName);
if(oStudent.mvGetName()=="Joe" && oStudent.mbIsTooOld())
{
  	...
}

// 好的代码
Student student;
string defaultName = "Tom";
student.setName(defaultName);
if(student.name()=="Joe" && student.isTooOld())
{
  	...
}
~~~

#### 表示属性的函数不要get

很多时候，如果函数返回属性值，不使用get可以让代码更接近于自然语言，因此不要get。如下：

~~~c++
if(student.getName()=="Jone"){
  	...
}

if(student.name()=="Jone"){
  	...
}
~~~

注：get可以在以下情况使用

~~~c++
string name;
student.getName(&name);
~~~

#### Qt的信号函数不使用signal，槽函数不使用slot

在信号函数中需要存在signal字段，说明信号函数命名不直观。

槽函数和成员函数本来就没啥区别，因此不需要添加slot字段。

~~~c++
// bad
connect(lineEdit,SIGNAL(signalTextChanged(string)),widget,SLOT(slotSetText(string)));
widget->slotSetText("Untitled");

// good
connect(lineEdit,SIGNAL(textChanged(string)),widget,SLOT(setText(string)));
widget->setText("Untitled");
~~~



####正反义命名

对于函数，很多时候都是有正反义的接口。例如，add/delete、open/close、load/save等。

### 宏命名

宏名称全部采用大写，单词间采用下划线。

~~~c++
#define MAX(a,b) a>b?a:b

// 头文件define
#ifndef STUDENT_H
#define STUDENT_H

class Student
{
};

#endif // STUDENT_H

~~~



### 文件命名

h文件和cpp文件分别对应类的定义和实现，文件名采用和类名保持一致。

### 相似的类应当有相似的API、相似的功能应当有相似的接口

在合适的时候，通过继承完全能达到此效果。在另一些时候，我们需要在类设计的时候注意命名以达到此效果。

例如我们有一个Xml类Json类，它们都有保存和读取文件的接口。对于保存，我们可能定义为save、saveToFile、saveToDesk等等。对于这类功能接口，我们应当统一为saveToFile，在以后我们想将配置文件从xml迁移为Json，改动的代码将更少，并且在团队内部，也能达到一致性。

## 语言特性

### 基于属性的API

~~~c++
Timer timer;
timer.setInterval(1000);
timer.setSingleShot(true);
timer.start();
~~~

属性是对象状态的一部分，在逻辑合理的情况下，用户应当可以以任何顺序来设置属性。例如，上面的代码应当完全可以写成：

~~~c++
Timer timer;
timer.setSingleShot(true);
timer.setInterval(1000);
timer.start();
~~~

在某些情况下，getter方法返回的结果与所设置的值不同。例如，虽然调用了`widget->setEnabled(true)`，但如果它的弗雷widget任处于disabled状态，那么`widget->isEnabled()`任然返回false。这样是OK的，因为这是符合逻辑的，但是需要注意的是这些情况必须在文档或注释中说明清楚。

### 参数传递约定

参数传递共有3种方式：值传递、引用传递和指针。

对于入参，参数传递规则如下：

* 基本类型如int、double等全部采用值传递。
* 结构体、类对象等采用引用传递以提升效率。

对于出参，参数传递规则如下：

* 全部采用指针传递。

~~~c++
void getHsv(int* h, int* s, int* v)const;
void getHsv(int& h, int& s, int& v)const;
~~~

很多C++书籍会推荐采用引用，因为引用比指针更加安全和优雅，与此相反，在传递出参时，指针可以使用户代码可读性更好，这也是我们追求的。

~~~c++
color.getHsv(&h,&s,&v);
color.getHsv(h,s,v);
~~~

上面第一行代码可以清晰地表示h、s、v参数在函数调用时会被修改。

另外避免出参的方式是使用返回值，代码如下：

~~~c++
struct Hsv{int hue, saturation, value};
Hsv getHsv()const;
~~~

### 函数参数不要过多

一个常见的误解时：实现需要些的代码越少，API就设计得越好。应当记住：代码只会写一次，却要阅读理解修改很多次，可读性才是检验API设计好不好的唯一准则。例如：

~~~c++
Slider* slider = new Slider(1,12,3,10,Qt::Vertical,0,"volume");
~~~

相比如下代码，这段代码阅读起来十分困难（写起来其实也困难）：

~~~c++
QSlider* slider = new QSlider(Qt::Vertical);
slider->setRange(1,12);
slider->setPageStep(3);
slider->setValue(10);
slider->setTitle("volume");
~~~

### bool参数陷阱

bool参数有时会带来无法阅读的代码。在Qt3中，`repaint()`函数有一个bool类型的可选参数表示是否擦除背景。可以写出如下代码：

~~~c++
widget->repaint(false);
~~~

初学者可能这样理解：不需要绘制。显然这个API设计得不好。更好的API应该废弃掉这个bool参数，增加含义明确的API。

~~~c++
widget->repaint();	// 绘制所有，包括背景
widget->repaintWithoutErasing();	// 明确接口
~~~

例外一种比较好的解决方案是使用枚举替代bool参数。

~~~c++
str.replace("user", false);	// false表示大小写不敏感
str.replase("user", CaseInsensitive);
~~~



### 内联函数

十分简单的函数可以定义为内联函数。原则如下：

* 内联函数的函数体代码不超出10行。
* 存在递归的函数不能定义为内联函数。

### 用内联函数代替宏函数

宏展开后可能与设计者本意违背，且宏函数无法调试。除了头文件的防止多重包含的宏，其他地方都不应当出现宏。宏函数也意味着这是一个全局函数，与面向对象的思想违背。因此，请重新考虑，使用内联函数来替代宏函数。

~~~c++
#define MAX(a,b) a>b?a:b;
#define max(int a, int b){return a>b? a:b;}

int n = MAX(a+b,a-b);	// error! int n = a+b>a-b?a+b:a-b;
int n = max(a+b,a-b);	// ok
~~~

### 使用括号来明确执行顺序

对于运算符比较多的公式，采用括号来明确执行次序。

~~~c++
int n = a+b>a-b?a+b:a-b;		// bad
int n = (a+b)>(a-b)? a+b:a-b;	// good
~~~

### 谁申请、谁释放

如果在一个函数中new处一个对象，则在此函数中必须delete这个对象。如果一个类会申请资源，那么这个类同样承担在释放资源的责任。

~~~c++
// bad code
Student* student = new Student;	
...
insertStudentToDbAndDelete(student);

// good
Student* student = new Student;	
...
insertStudentToDb(student);
delete student;
student = nullptr;
~~~

有些时候我们可能会有如下代码：

~~~c++
mutex.lock();
if（cond1){
  	...
  	mutex.unlock();
  	return;
}
if（cond2){
  	...
  	mutex.unlock();
  	return;
}
mutex.unlock();
~~~

对于这种类似代码，很容易因为函数提前退出尔忘记解锁。可以巧用类的自动析构来避免这种情况。

~~~c++
class MutexLocker
{
public:
  	MutexLocker(Mutex* mutex):m_mutex(mutex){m_mutex->lock();}
  	~MutexLocker(){m_mutex->unlock();}
private:
  	Mutex* m_mutex;
};

// 修改代码如下
MutexLocker locker(&mutex);
if（cond1){
  	...
  	return;
}
if（cond2){
  	...
  	return;
}
~~~

类似的方案如C++的智能指针、资源的自动释放、状态的还原等。

### 禁止使用goto语句

goto语句强行跳转到指定代码，打乱正常逻辑，不利于理解，禁止采用。

### 巧用PrivateData实现数据隐藏

如果我们需要设计一个类如下，我们的代码可能如下：

~~~c++
class T1
{
public:
  	// 对外接口
  	void func1();
  	void func2();
private:
  	void func3();	// 内部辅助函数
private:
  	int m_n1;	// 内部数据
};
~~~

由于某种原因，我们需要隐藏我们的实现，即不想让用户看到我们的内部数据和内部辅助函数。可选的方案是提供一个接口类，让T1继承这个接口。这里再提供一种可选方案：PrivateData类。

~~~c++
// T1.h
class T1Private;
class T1
{
public:
  	// 对外接口
  	void func1();
  	void func2();
private:
  	T1Private* m_data;	// 内部数据
};

// T1.cpp
class T1Private
{
  	int m_n1;
  	void func3();
  	friend class T1;
};
~~~

**注意:**使用PrivateData类应当注意拷贝构造函数和赋值构造函数。

## 工程管理

### 及时清理废弃代码

在功能调整、bug修复等过程中，某些代码会被屏蔽、某些功能模块会被剔除。如果我们不及时清理，后期工程将变得十分臃肿，并且时间越久，清理起来越困难。如果新人接手，废弃的代码还会添加新人的学习代码。过多屏蔽的代码会影响阅读。废弃的模块会增加理解系统的难度。因此，一定要及时清理废弃的代码。

### 提炼重复代码到函数

如果相似或重复的代码出现，请停下来思考。是否逻辑存在关联，是否应当把代码提炼成独立的函数。如果的确是业务逻辑一致，一定要提炼成一个函数。这样不止减少代码量，降低程序复杂度，对后期维护也十分有利。例如：如果重复的代码出现3处，后期发现这段代码存在bug，如果没有提炼成一个函数，修改的地方就有3处、也或者30处。我们不能保证没处都已修正，同样就无法保证bug已修复。

### 函数体不要太长

函数体太长意味着逻辑太多，不利于阅读和理解。应当才分成多个函数重新组织。

### 避免嵌套太深

嵌套是指if、while、switch等语句的大括号。如果嵌套过深，也会造成理解困难。嵌套不要超过4层。

### 入参检查合法性

在函数体其实位置检查入参的合法性（如果需要）。入参的检查有两种，一种是程序不应该出现不合法的情况，如果出现，一定是程序bug。这种参数请使用断言，这样在后期正式版本可以提升效率。

~~~c++
assert(a!=0);
~~~

例外一种是正常情况可能发生参数不合法，如用户输入的。这种情况请使用if条件判断。

~~~c++
if(a==0){
  	return false;
}
~~~

### 检查函数返回值

如果函数有返回值，除非是否确定可以忽略返回值，否则一定要检查函数返回值。软件有很大一部分bug都是没有检查返回值引起的。

### 尽量使用const

成员函数、变量等如果确定不会修改数据，尽可能使用const修饰，这样可以避免一些潜在由于数据修改导致的bug。将这些检查交给机器。

### 禁止使用全局变量、函数和枚举

全局变量使用单例模式替代，全局函数使用类静态函数替代。不应当有全局枚举，枚举应当存在于某个类中。禁止使用全局变量、函数和枚举可以让程序模块更加清晰，防止蜘蛛网的产生。

### UI采用工具布局

采用工具布局UI，不要手动编写布局代码。工具布局UI方便直观、容易修改。手动编写代码也不能提升我们的编程技术，也很不直观，代码写出后必须调试才能看到效果。例如，假设一个我们的UI采用3x3的GridLayout，如果后期我们需要在中间增加一个控件，如果采用代码布局，修改的代码将会很多，并且容易出错。如果采用工具布局则完全没有任何问题，将简单繁琐且没技术含量的事情交接机器，我们需要处理的是业务逻辑。

~~~c++
layout->addWidget(w1,0,0);
layout->addWidget(w2,0,1);
layout->addWidget(w3,0,2);
layout->addWidget(w4,1,0); // 需要在此插入控件，后面的所有代码都需要调整！
layout->addWidget(w5,1,1);
layout->addWidget(w6,1,2);
layout->addWidget(w7,2,0);
layout->addWidget(w8,2,1);
layout->addWidget(w9,2,2);
~~~

### 不要编写聪明的函数

编写聪明的函数指设计者揣测使用者的意图来编写函数。这样的函数往往是违背**函数简单直接，不隐藏设计者意图**的原则的。例如，我们需要实现一个现实文本的Label，它提供一个设置文本的方法`setText(const string& text)`，如设计者自作聪明，让`setText(const string& text)`可以自动解析html。这样，用户在使用的时候就会出现问题，如——为何现实的文本和获取到的文本不一致？遇到这样情况，请直接提供一个`setHtml(const string& html)`方法。或者提供一个`setEnableHtml(bool enable)`方法，而不是让`setText(const string& text)`变得强大。

### 一个函数就完成一个功能

一个函数功能太多，不但难以理解，后期维护也是否困难。例如一个窗口类提供一个如下函数：

~~~c++
void createAndShowWindowIfNoWindow(); // 如果没有窗口在创建并显示一个窗口
~~~

以上代码如果不看函数说明是很难理解的，并且容易误解。这样的代码应当属于业务逻辑。改写代码如下：

```c++
if(isNoWindow())
{
  	createWindow();
  	showWindow();
}
```

这样代码就达到自我解释并且不再有歧义。

另外一种一个函数完成多个功能的反面教材是Win32API，这也是大家觉得Win32API不好学的原因。同一个函数参数太多，并且组合不同，功能也不同。

### 一个变量只有一个含义

不要使用同一个变量表示多个含义，否则代码不清晰且难以维护。

~~~c++
// bad 
int count = teacherCount();
...
count = studentCount();
...

// good
int teacherCount = teacherCount();
...
int studentCount = studentCount();
...
~~~

### 逻辑块代码靠近，逻辑块间空行隔开

就像文章分段落一样，逻辑代码块应当组织在一块，包括变量的定义和初始化。逻辑块之间使用空行隔开，强调逻辑不一样。

~~~c++
//bad
int studentCount=0;
int tacherCount=0;
teacherLabel.setText("Teacher");
studentCount = GLOBAL->studentCount();
teacherCount = teacherCountSpinBox.value();
studentCountSpinBox.setValue(student);
GLOBAL->setTeacherCount(teacherCount);

// good
teacherLabel.setText("Teacher");
int teacherCount = teacherCountSpinBox.value();
GLOBAL->setTeacherCount(teacherCount);

int studentCount = GLOBAL->studentCount();
studentCountSpinBox.setValue(student);
~~~



## 文件组织

###一个类对应一个h文件和一个cpp文件

一个类对应一个h文件和一个cpp文件。h文件存放定义，cpp存放实现代码。头文件使用`define`宏防止多重包含。不允许使用hpp等后缀文件名。

例子：

```c++
// Student.h
#ifndef STUDENT_H
#define STUDENT_H

class Student
{
public:
  	Student(const string& name, int age);
  
  	void setName(const string& name);
  
private:
  	string m_name;
  	int m_age;
};

#endif //STUDENT_H
```

```c++
// Student.cpp
#include "Student.h"

Student::Student(const string& name, int age)
  	:m_name(name),m_age(age)
{
}

void Student::setName(const string& name)
{
  	m_name=name;
}
```

###尽量减少头文件的include

头文件对外就是接口文件，在头文件中include太多，如果某个依赖的头文件发生改变，在所有相关的文件将重新编译，减少头文件的include，可以提升编译效率。

并且，include了使用者不需要的头文件，也可能造成使用者的困惑及使用困难。例如，假设需要提供一个软件保存功能的类。前期我们使用`QXml`，并且我们在头文件中`#include <QXml>`，那么使用者在自己的工程也必须添加Qt的xml模块。后期由于某种原因，我们保存不再使用xml，改为sql。并且我们修改头文件`#include <QSql>`，这时候使用者也必须再次修改自己的工程，为工程添加Qt的sql模块。站在使用者的角度，其实并不关系保存功能使用的xml还是sql。因此，解决问题的方案也很简单，移动不合理的include到cpp文件即可。

### 警惕循环依赖

循环依赖表示头文件a包含头文件b，头文件b又包含头文件a。如果出现循环依赖，请慎重分析设计是否合理。

##排版

### 注释

优秀的代码自我注释。在通常的情形下，如果代码需要注释才能说清，那么代码逻辑和命名可能处理得不好，这时候应该重新代码而不是添加注释，注释是救不了糟糕的代码的。因此，通常情况下，类、函数、变量和逻辑代码都不需要注释。以下情况才需要注释：

* 函数参数传入有限制条件。
* 函数返回异常情况。如果某个返回int的函数返回-1表示失败。
* 指示代码的实现算法。

注释添加在代码块的上方或右方。这样可以让IDE为我们自动提示注释。

`/*...*/`如果嵌套容易出现问题，因此注释因全部采用`//`，IDE可以为我们完成所有工作。

### 缩进

tab缩进全部采用4个空格的宽度。`{}`必须单独起行以提升代码可读性。

```c++
// bad
void func(){
  	int a=0;
  	int b=0;
  	if(cond1){
        for(int i=0;i<10;i++){
          	code1;
            a=1;
          	b=1;
        }
    	
    }else if{
      	a=2;
      	b=2;
      	code1;
    }
}

// ok
void func()
{
  	int a=0;
  	int b=0;
  	if(cond1)
    {
    	for(int i=0;i<10;i++)
        {
          	code1;
            a=1;
          	b=1;
        }
    }
  	else if
    {
      	a=2;
      	b=2;
      	code1;
    }
}
```

