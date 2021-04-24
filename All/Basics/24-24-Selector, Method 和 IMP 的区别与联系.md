### IMP、SEL、Method的区别
* IMP：是方法的实现，即：一段c函数
* SEL：是方法名
* Method：是objc_method类型指针，它是一个结构体，如下：
```
struct objc_method {
    SEL _Nonnull method_name                                 OBJC2_UNAVAILABLE;
    char * _Nullable method_types                            OBJC2_UNAVAILABLE;
    IMP _Nonnull method_imp                                  OBJC2_UNAVAILABLE;
} 
```

### 使用场景：

* 实现类的swizzle的时候会用到，通过class_getInstanceMethod(class, SEL)来获取类的方法Method，其中用到了SEL作为方法名

* 调用method_exchangeImplementations(Method1, Method2)进行方法交换

* 我们还可以给类动态添加方法，此时我们需要调用class_addMethod(Class, SEL, IMP, types)，该方法需要我们传递一个方法的实现函数IMP，例如：
```
static void funcName(id receiver, SEL cmd, 方法参数...) {
   // 方法具体的实现   
}
```
函数第一个参数：方法接收者，第二个参数：调用的方法名SEL，方法对应的参数，这个顺序是固定的。


### 补充
Objective-C方法是由一个selector(SEL)，和一个implement(IMP)组成的。selector相当于门牌号，而Implement才是真正的住户（函数实现）；和现实生活一样，门牌可以随便发（@selector(XXX)），但是不一定都找得到住户；而方法调用，其实都是转换为objc_msgSend函数的调用；Objective-C中的方法调用就是消息发送：给receiver（方法调用者）发送了一条消息（selector方法名）;



* 参考：https://www.cnblogs.com/zbblog/articles/12419312.html
* 参考：https://www.jianshu.com/p/84d1771e9792
