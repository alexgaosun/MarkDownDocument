# 深挖iOS底层（3）
### Sizeof()
* 定义：打印当前数据类型的占用大小，单位为字节
sizeof(结构体)：结构体元素的类型的总和
sizeof(类)：打印的objc是对象指针的大小，固定8字节
* sizeof(),class_getInstanceSize，malloc_size区别
NSObject内存分配探究：

![NSObject内存分配探究](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.002/2020/12/1208/817C5AB7-1670-4BF8-B45E-7458FF9FEE50.png)
总结：sizeof打印的是指针大小 8字节
class_getInstanceSize：运行时NSObject占用大小 8字节
malloc_size：系统分配给对象的大小  16字节。

### LLDB调试技巧：
* LLDB调试-内存读取

```
p/x中
p是打印
x是读取内存的命令（以16进制显示）

x/4gx中
第一个x是（读取内存），
后面的g代表（每次读取8字节），
x是（16进制显示结果），
4代表（连续打印4段）
```

* LLDB调试-进制转换
![w612](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.002/2020/12/1208/16074232179113.jpg)

### 通过对象地址，探究对象的
![w794](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.002/2020/12/1208/16074238732931.jpg)
NS都为指针类型，可通过当前指针存储的地址访问地址指向的内容。
常量直接打印当前赋值，会出检索对应地址。

### 什么是对象
* clang编译器
![w632](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.002/2020/12/1208/16074270917065.jpg)

打开cpp文件，对比.m文件和cpp文件的对比
![w602](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.002/2020/12/1208/16074272235680.jpg)
对象的本质实际就是结构体，如上图所示。
* 底层编译对象类中第一个变量（isa指针）：
struct NSObject_IMPL NSObjct_IVARS :isa
* 对象属性在底层的创建：生成set和get方法
  ![w926](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.002/2020/12/1208/16074274028251.jpg)

* 探究set方法中 **objc_setProperty**(在objc4源码中可查询实现)
![w1047](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.002/2020/12/1208/16074275311418.jpg)

* 深入reallySetProperty（创建新值，释放旧值）
![w1050](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.002/2020/12/1208/16074285489197.jpg)
总结：所有对象创建属性都会调用objc_setProperty,统一管理，（上下层分离，通过锁进行隔离），利用了适配器原则。

### 联合体位域
* 字节：字节也叫Byte，1Byte=8bit(位)，1024Byte(字节)=1KB，1024KB=1MB，1024MB=1GB，1024GB=1TB。bit是描述电脑数据量的最小单位。
* 结构体（struct）：中所有变量是共存关系，优点是可以共存。缺点是不管用不用都会分配结构体内元素内存。
* 联合体（union）：各变量是互斥关系，缺点是不能共存，优点是精细灵活，也节省内存开销。如下图
![w435](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.002/2020/12/1208/16074286685122.jpg)
这里利用左移运算符（<< ）来控制数值，这样就可采用1个字节控制4个不同变量。如图
![w991](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.002/2020/12/1208/16074287092650.jpg)

* 通过底层2的总结探究到alloc核心方法calloc生成方法需要三步:
```
> 1.告知系统申请多少内存：instanceSize
> 2.开辟内存拿回指针 :calloc
> 3.类和指针绑定:initInstanceIsa
```

### 前两个方法的实现已探究，今天探究initInstanceIsa：
创建了一个[LGPerson alloc] 进入源码调试:
* isa的初始化方法（initInstanceIsa）：
    ![w990](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.002/2020/12/1208/16074291592421.jpg)

* 深入initIsa方法：
![w620](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.002/2020/12/1208/16074291920850.jpg)

* 核心代码依然是isa_t,接下来就要深入isa_t类型，一探究竟。
![w968](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.002/2020/12/1208/16074292711916.jpg)

* 通过上图探究一下：ISA_BITFIELD
![w635](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.002/2020/12/1208/16074293613765.jpg)

##### 解读ISA_BITFIELD所有元素：
* nonpointer:表示是否对isa指针开启指针优化
0:纯isa指针，1:不止是类对象地址,isa中包含了类信息、对象的引用计数等  

* has_ assoc: 关联对象标志位，0没有，1存在

* has_ cxx_ dtor: 该对象是否有C++或者Objc的析构器,如果有析构函数,则需要做析构逻辑，如果没有,则可以更快的释放对象  

* shiftcls:
存储类指针的值。开启指针优化的情况下，在arm64架构中有33位用来存储类指针。

* magic:用于调试器判断当前对象是真的对象还是没有初始化的空间

* weakly_referenced:志对象是否被指向或者曾经指向-一个ARC的弱变量,
没有弱引用的对象可以更快释放。

* deallocating:标志对象是否正在释放内存

* has_ sidetable_ rc:当对象引用技术大于10时，则需要借用该变量存储进位

* extra_rc: 当表示该对象的引用计数值，实际上是引用计数值减1,
例如，如果对象的引用计数为10，那么extra_rc 为9。如果引用计数大于10,
则需要使用到下面的has_sidetable_rc。 

### 初始化isa核心（objc_object::initIsa）
![w1810](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.002/2020/12/1208/16074307600637.jpg)

* newisa.bits赋值后打印内容：
  ![w645](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.002/2020/12/1208/16074308436496.jpg)

* 初始化后 magic 为何等于59？
根据MAGIC_VALUE的默认值
0x001d8000000000001
通过计算器查看
![w633](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.002/2020/12/1208/16074309195127.jpg)
从47位后得到：111011
![w536](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.002/2020/12/1208/16074310610795.jpg)
十进制的59转化为机器码（二进制）即为11101

### 在objc_object::initIsa方法 探究shiftcls赋值演算：
![w1021](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.002/2020/12/1208/16074311397567.jpg)

* 通过objc_object::getIsa()探究isa
![w1031](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.002/2020/12/1208/16074312893285.jpg)

* 深入ISA()
![w642](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.002/2020/12/1208/16074313703487.jpg)

* 通过objc_object::initIsa方法最后一步断点演算ISA()
![w1005](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.002/2020/12/1208/16074314238511.jpg)

### 通过LLDB操作内存位移的演算来探究（shiftcls）isa的信息
![w758](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.002/2020/12/1208/16074315639691.jpg)

* NSObjc类却是Class类型而不是isa_t类型原因：
![w946](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.002/2020/12/1208/16074318209004.jpg)
通过getClass底层:object_getClass会调用objc_object::getIsa() 
![w701](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.002/2020/12/1208/16074320948787.jpg)
ISA()方法进行了强转
![w416](https://cdn.jsdelivr.net/gh/alexgaosun/AGSCDN@1.002/2020/12/1208/16074321399184.jpg)



    











