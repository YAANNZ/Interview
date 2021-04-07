
### alloc和alloc init
* alloc（字面意思：分配） 来创建对象并分配空间
* init 进行初始化工作，并非必须，源码如下
```
- (id)init {
    return _objc_rootInit(self);
}

id _objc_rootInit(id obj) {
    return obj;
}
```
* 只 alloc 的情况下，调用方法无警告且不崩溃。

代码示例：
```
Person *p = [Person alloc];
[p testMethodd];
```
* 以下示例验证init的作用，三个对象指向同一块内存，同一个地址，是同一个对象，只是三个指针指向了它。
* 打印了对象、对象内存地址、指向对象的指针的地址。
```
- (void)viewDidLoad {
    [super viewDidLoad];
    
    ZZPerson *p1 = [ZZPerson alloc];
    ZZPerson *p2 = [p1 init];
    ZZPerson *p3 = [p1 init];
    ZZNSLog(@"%@ - %p - %p",p1,p1,&p1);
    ZZNSLog(@"%@ - %p - %p",p2,p2,&p2);
    ZZNSLog(@"%@ - %p - %p",p3,p3,&p3);
}
```
打印结果：
```
<ZZPerson: 0x600003c9cc30> - 0x600003c9cc30 - 0x7ffee24cc0f8
<ZZPerson: 0x600003c9cc30> - 0x600003c9cc30 - 0x7ffee24cc0f0
<ZZPerson: 0x600003c9cc30> - 0x600003c9cc30 - 0x7ffee24cc0e8
```



### alloc init 和 new
* 本质上没有区别，alloc、init可以自定义初始化内容。

内部代码：
```
+ (id)new {
    return [callAlloc(self, false/*checkNil*/) init];
}
```

### 参考
* https://www.jianshu.com/p/427e66c806f0
* https://my.oschina.net/u/4418367/blog/3974523
* https://zhuanlan.zhihu.com/p/338173849
* https://draveness.me/object-init/
