#  深挖iOS底层-类底层（4）  
### 通过LLDB探究类地址：
![-w653](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.003/2020/12/1209/16074348975682.jpg)
经过上述不同打印都可以拿到类的首地址。

继续深入探究首地址：
![-w910](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.003/2020/12/1209/16074349842480.jpg)
通过上述打印，得知类的收地址是2250， 2228是元类。

### 元类
元类的创建都是由编译器自动完成。
* 元类刨根问底
![-w801](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.003/2020/12/1209/16074351430377.jpg)

通过LLDB不断获取首地址得到根元类：NSObject
追溯过程：
类isa 指针 -> 类（LGPerson） ->元类（LGPerson） -> NSObject（根元类）

### 类对象（[object class]）
![-w762](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.003/2020/12/1209/16074353407243.jpg)

* 地址不同？类对象到底有几份？
验证类对象:
![-w869](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.003/2020/12/1209/16074356391146.jpg)
利用各种类的打印，验证类对象在内存中只存在一份  

### isa指针的指向（探究根元类）
探究NSObject过程
![-w592](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.003/2020/12/1209/16074361193941.jpg)
  
### superClass的继承
实例对象之间没有任何关系， 只有类和类之间单纯的继承关系，元类也存在继承，根元类指向NSObject。
最经典的解析图：
![-w556](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.003/2020/12/1209/16074362251703.jpg)
此图完整诠释了，isa指针指向和类的继承关系。

### Class，NSObject，objc_class和底层objc_object的关系
* 通过源码得知objc_object 本质是结构体，也称根对象，所有对象都会满足此继承关系
![-w954](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.003/2020/12/1209/16074363948058.jpg)
  
* objc_class是objc_object类型  
如图
![-w842](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.003/2020/12/1209/16074364408473.jpg)
  
* 所有类，对象，元类 都是以objc_object类型的继承过来，objec底层编译就是objc_object.
![-w617](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.003/2020/12/1209/16074365245195.jpg)

* objc_object结构体包含isa ，objc_class与bjc_object是继承关系因此也有isa，同理所有实例对象，类，元类都继承自objc_object，有isa（isa是Class类型的）  
![-w841](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.003/2020/12/1209/16074365797187.jpg)

* Class底层是 objc_class类型:
![-w392](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.003/2020/12/1209/16074390787451.jpg)
Class是objc_class类型，深入objc_class  

* 探究objc_class的类型
> 老版源码objc_class源码实现（已经弃用）
![-w606](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.003/2020/12/1209/16074799323093.jpg)

> 新版本的objc_class源码实现
![-w705](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.003/2020/12/1209/16074796282566.jpg)

我们发现新版源码结构体继承 objc_object，因为这是c++结构体 并且结构体内有一些函数。

* 举例探究objc_class
![-w660](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.003/2020/12/1209/16074854806285.jpg)
LLDB调试已经帮我们探究类对象的具体结构是什么样子的，类对象的首地址存放isa指针，第二个地址存放的继承的类

### 简单捋下指针和指针的作用
![-w777](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.003/2020/12/1209/16074868630787.jpg)
从上图可以看出a和b就是int类型的指针，都存放着tmp的内存地址，也叫a和b指向tmp的地址，指针的作用就是存地址。

### 对象创建的内存演算
![-w954](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.003/2020/12/1209/16074962787711.jpg)

p1，p2指针：在栈区，都为 1级指针 负责指向 堆区的对应地址

LGPerson：负责给堆区开辟 p1 和 p2指向的内存

堆区的两片内存：存放的是对象的内容，对象的首地址是isa指向的是父
类LGPerson，对象的首地址isa指针存储是LGPerson相关信息。

最左侧的两片内存地址是p1和p2的二级指针，存放的是p1，p2的内存地址。

### 数组指针
连续排列的地址，声明的数组的指针指向首地址，也就是首个个元素的地址。如下图：
![-w943](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.003/2020/12/1209/16074968398689.jpg)
通过lldb控制台看出，通过地址的偏移，依然可以取到元素。
数组内存布局示意图：
![-w571](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.003/2020/12/1209/16074990497351.jpg)

### 探索类（object_class）的内存布局：
![-w648](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.003/2020/12/1209/16075007533810.jpg)
class地址 ：2250 16进制偏移得到2270：32字节转化为16进制 (1-F(0-15))
根据bit.data获取bits内容
![-w628](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.003/2020/12/1209/16075008892474.jpg)
* 根据bits信息中（class_rw_t类）中 methods()，properties()，protocols() 探索属性列表，方法列表，协议列表
![-w607](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.003/2020/12/1209/16075033064679.jpg)


* 我们来通过LLDB完整的演算一下objc_object类型的内存布局
![-w888](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.003/2020/12/1209/16075020167789.jpg)
1. 先获取类对象的地址
2. 偏移32字节后，得到bits 转换成bits类型
3. bits是class_data_bits_t类型深入class_data_bits_t源码
![-w674](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.003/2020/12/1209/16075028622770.jpg)
通过bits调用data()方法 就获取到了class_rw_t类型的变量，通过此变量就能获取到methods()，properties()，protocols() 。
获取方法列表（也可以获取属性列表，协议列表）
举个例子：
OC类的属性和方法：
![-w426](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.003/2020/12/1209/16075061905844.jpg)

* 属性列表推导
![-w657](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.003/2020/12/1209/16075054919215.jpg)

方法列表推导
![-w542](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.003/2020/12/1209/16075063611904.jpg)
* 方法列表中也会同时生属性的set，get方法。
![-w589](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.003/2020/12/1209/16075064380729.jpg)  







