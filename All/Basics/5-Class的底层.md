
### 对象
OC中的对象指向的是一个objc_object指针类型，typedef struct objc_object *id;从它的结构体中可以看出，它包括一个isa指针，指向的是这个对象的类对象,一个对象实例就是通过这个isa找到它自己的Class，而这个Class中存储的就是这个实例的方法列表、属性列表、成员变量列表等相关信息的。
```
// Represents an instance of a class.
struct objc_object {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;
};
```
### 类
在OC中的类是用Class来表示的，实际上它指向的是一个objc_class的指针类型，typedef struct objc_class *Class;对应的结构体如下：
```
struct objc_class {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
    Class _Nullable super_class                              OBJC2_UNAVAILABLE;
    const char * _Nonnull name                               OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;
    struct objc_ivar_list * _Nullable ivars                  OBJC2_UNAVAILABLE;
    struct objc_method_list * _Nullable * _Nullable methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache * _Nonnull cache                       OBJC2_UNAVAILABLE;
    struct objc_protocol_list * _Nullable protocols          OBJC2_UNAVAILABLE;
#endif

}
```
从结构体中定义的变量可知，OC的Class类型包括如下数据（即：元数据metadata）：super_class（父类类对象）；name（类对象的名称）；version、info（版本和相关信息）；instance_size（实例内存大小）；ivars（实例变量列表）；methodLists（方法列表）；cache（缓存）；protocols（实现的协议列表）;

当然也包括一个isa指针，这说明Class也是一个对象类型，所以我们称之为类对象，这里的isa指向的是元类对象（metaclass），元类中保存了创建类对象（Class）的类方法的全部信息。

以下图中可以清楚的了解到OC对象、类、元类之间的关系
从图中可知，最终的基类（NSObject）的元类对象isa指向的是自己本身，从而形成一个闭环。

元类（Meta Class）：是一个类对象的类，即：Class的类，这里保存了类方法等相关信息。

我们再看一下类对象中存储的方法、属性、成员变量等信息的结构体

**objc_ivar_list**：存储了类的成员变量，可以通过object_getIvar或class_copyIvarList获取；另外这两个方法是用来获取类的属性列表的class_getProperty和class_copyPropertyList，属性和成员变量是有区别的。
```
struct objc_ivar {
    char * _Nullable ivar_name                               OBJC2_UNAVAILABLE;
    char * _Nullable ivar_type                               OBJC2_UNAVAILABLE;
    int ivar_offset                                          OBJC2_UNAVAILABLE;
#ifdef __LP64__
    int space                                                OBJC2_UNAVAILABLE;
#endif
}                                                            OBJC2_UNAVAILABLE;

struct objc_ivar_list {
    int ivar_count                                           OBJC2_UNAVAILABLE;
#ifdef __LP64__
    int space                                                OBJC2_UNAVAILABLE;
#endif
    /* variable length structure */
    struct objc_ivar ivar_list[1]                            OBJC2_UNAVAILABLE;
}
```
**objc_method_list**：存储了类的方法列表，可以通过class_copyMethodList获取。

结构体如下：
```
struct objc_method {
    SEL _Nonnull method_name                                 OBJC2_UNAVAILABLE;
    char * _Nullable method_types                            OBJC2_UNAVAILABLE;
    IMP _Nonnull method_imp                                  OBJC2_UNAVAILABLE;
}                                                            OBJC2_UNAVAILABLE;

struct objc_method_list {
    struct objc_method_list * _Nullable obsolete             OBJC2_UNAVAILABLE;

    int method_count                                         OBJC2_UNAVAILABLE;
#ifdef __LP64__
    int space                                                OBJC2_UNAVAILABLE;
#endif
    /* variable length structure */
    struct objc_method method_list[1]                        OBJC2_UNAVAILABLE;
}
```
**objc_protocol_list**：储存了类的协议列表，可以通过class_copyProtocolList获取。

结构体如下：
```
struct objc_protocol_list {
    struct objc_protocol_list * _Nullable next;
    long count;
    __unsafe_unretained Protocol * _Nullable list[1];
};
```

### 面试题
1. 为什么要设计metaclass

metaclass代表的是类对象的对象，它存储了类的类方法，它的目的是将实例和类的相关方法列表以及构建信息区分开来，方便各司其职，符合单一职责设计原则。使用class_copyMethodList获取类的方法时，获取的只是实例方法，如果要获取类方法需要传入元类。`Class metaClass = object_getClass([Person class]);`[可以参考](https://www.jianshu.com/p/b30c2580d977)

其实这里涉及到了关于面向对象设计的一些东西，具体可以参考[这篇文章](https://www.leewong.cn/2018/05/02/why-metaclass/)

2. class_copyIvarList & class_copyPropertyList区别

class_copyIvarList：获取的是类的成员变量列表和属性列表，即：@interface{中声明的变量}以及通过@property声明的属性

class_copyPropertyList：获取的是类的属性列表，即：通过@property声明的属性

3. class_rw_t 和 class_ro_t 的区别

class_rw_t：代表的是可读写的内存区，这块区域中存储的数据是可以更改的。

class_ro_t：代表的是只读的内存区，这块区域中存储的数据是不可以更改的。

OC对象中存储的属性、方法、遵循的协议数据其实被存储在这两块儿内存区域的，而我们通过runtime动态修改类的方法时，是修改在class_rw_t区域中存储的方法列表。

参考[这篇文章](https://blog.csdn.net/fishmai/article/details/71157861)

4. category如何被加载的,两个category的load方法的加载顺序，两个category的同名方法的加载顺序

category的加载是在运行时发生的，加载过程是，把category的实例方法、属性、协议添加到类对象上。把category的类方法、属性、协议添加到metaclass上。

category的load方法执行顺序是根据类的编译顺序决定的，即：xcode中的Build Phases中的Compile Sources中的文件从上到下的顺序加载的。

category并不会替换掉同名的方法的，也就是说如果 category 和原来类都有 methodA，那么 category 附加完成之后，类的方法列表里会有两个 methodA，并且category添加的methodA会排在原有类的methodA的前面，因此如果存在category的同名方法，那么在调用的时候，则会先找到最后一个编译的 category 里的对应方法。

参考[这篇文章](https://www.jianshu.com/p/40e28c9f9da5)

5. category & extension区别，能给NSObject添加Extension吗，结果如何？

 category：分类

给类添加新的方法
不能给类添加成员变量
通过@property定义的变量，只能生成对应的getter和setter的方法声明，但是不能实现getter和setter方法，同时也不能生成带下划线的成员属性
是运行期决定的
注意：为什么不能添加属性，原因就是category是运行期决定的，在运行期类的内存布局已经确定，如果添加实例变量会破坏类的内存布局，会产生意想不到的错误。

extension：扩展

可以给类添加成员变量，但是是私有的
可以給类添加方法，但是是私有的
添加的属性和方法是类的一部分，在编译期就决定的。在编译器和头文件的@interface和实现文件里的@implement一起形成了一个完整的类。
伴随着类的产生而产生，也随着类的消失而消失
必须有类的源码才可以给类添加extension，所以对于系统一些类，如nsstring，就无法添加类扩展
不能给NSObject添加Extension，因为在extension中添加的方法或属性必须在源类的文件的.m文件中实现才可以，即：你必须有一个类的源码才能添加一个类的extension。

6. class、objc_getClass、object_getclass 方法有什么区别?

* objc_getClass：参数是类名的字符串，返回的就是这个类的类对象；

* object_getClass：参数是id类型，它返回的是这个id的isa指针所指向的Class，如果传参是Class，则返回该Class的metaClass

* [obj class]：则分两种情况：一是当obj为实例对象时，[obj class]中class是实例方法：- (Class)class，返回的obj对象中的isa指针；二是当obj为类对象（包括元类和根类以及根元类）时，调用的是类方法：+ (Class)class，返回的结果为其本身。

### 参考
[iOS 进阶之 Class 底层原理](https://juejin.cn/post/6931611248941334535)
