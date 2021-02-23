### KVO和KVC的底层原理
#### KVO
* 参考：https://www.jianshu.com/p/6a3ebc4b8106
* 全称为Key-Value-Observe，原理如下：
> 系统创建派生类，原类的isa指向派生类（基于runtime替换，isa-swizzling），派生类重写监听属性的setter方法，在调用原类相应属性的setter方法调用前后分别发出willChangeValueForKey:和didChangeValueForKey：的通知。
* 为什么说KVO基于KVC实现的

> 当使用KVO观察某个类属性时，会为该类创建一个子类，子类重写setter方法时，跟KVCset时的搜索顺序是一样的，都是先搜索set<Key>,然后在搜_set<Key>。

> 在为observe的change字典里的old和new赋值时，用到了KVC的valueForKey:

> 也许是苹果在KVO文档里的这句话。为了理解KVO，你首先要理解KVC：
Important: In order to understand key-value observing, you must first understand key-value coding.


#### KVC
* 全称为：Key-Value Coding，常用方法如下：
```
- (void)setValue:(id)value forKeyPath:(NSString *)keyPath; <br>
- (void)setValue:(id)value forKey:(NSString *)key; <br>
- (id)valueForKeyPath:(NSString *)keyPath; <br>
- (id)valueForKey:(NSString *)key; 
```
* 原理
> * 设值 <br>
> 当调用setValue：属性值 forKey：@”name“的代码时，底层的执行机制如下： <br>
> 程序优先调用set<Key>:属性值方法，代码通过setter方法完成设置。注意，这里的<key>是指成员变量名，首字母大小写要符合KVC的命名规则，下同
如果没有找到setName：方法，KVC机制会检查+ (BOOL)accessInstanceVariablesDirectly方法有没有返回YES，默认该方法会返回YES，如果你重写了该方法让其返回NO的话，那么在这一步KVC会执行setValue：forUndefinedKey：方法，不过一般开发者不会这么做。所以KVC机制会搜索该类里面有没有名为_<key>的成员变量，无论该变量是在类接口处定义，还是在类实现处定义，也无论用了什么样的访问修饰符，只在存在以_<key>命名的变量，KVC都可以对该成员变量赋值。
如果该类即没有set<key>：方法，也没有_<key>成员变量，KVC机制会搜索_is<Key>的成员变量。
和上面一样，如果该类即没有set<Key>：方法，也没有_<key>和_is<Key>成员变量，KVC机制再会继续搜索<key>和is<Key>的成员变量。再给它们赋值。
如果上面列出的方法或者成员变量都不存在，系统将会执行该对象的setValue：forUndefinedKey：方法，默认是抛出异常。

> * 取值 <br>
> 当调用valueForKey：@”name“的代码时，KVC对key的搜索方式不同于setValue：属性值 forKey：@”name“，其搜索方式如下： <br>
首先按get<Key>,<key>,is<Key>的顺序方法查找getter方法，找到的话会直接调用。如果是BOOL或者Int等值类型， 会将其包装成一个NSNumber对象。
如果上面的getter没有找到，KVC则会查找countOf<Key>,objectIn<Key>AtIndex或<Key>AtIndexes格式的方法。如果countOf<Key>方法和另外两个方法中的一个被找到，那么就会返回一个可以响应NSArray所有方法的代理集合(它是NSKeyValueArray，是NSArray的子类)，调用这个代理集合的方法，或者说给这个代理集合发送属于NSArray的方法，就会以countOf<Key>,objectIn<Key>AtIndex或<Key>AtIndexes这几个方法组合的形式调用。还有一个可选的get<Key>:range:方法。所以你想重新定义KVC的一些功能，你可以添加这些方法，需要注意的是你的方法名要符合KVC的标准命名方法，包括方法签名。
如果上面的方法没有找到，那么会同时查找countOf<Key>，enumeratorOf<Key>,memberOf<Key>格式的方法。如果这三个方法都找到，那么就返回一个可以响应NSSet所的方法的代理集合，和上面一样，给这个代理集合发NSSet的消息，就会以countOf<Key>，enumeratorOf<Key>,memberOf<Key>组合的形式调用。
可能上面的两条查找方案对读者不好理解，简单来说就是如果你在自己的类自定义了KVC的实现，并且实现了上面的方法，那么恭喜你，你可以将返回的对象当数组(NSArray)用了，详情见下面的示例代码
如果还没有找到，再检查类方法+ (BOOL)accessInstanceVariablesDirectly,如果返回YES(默认行为)，那么和先前的设值一样，会按_<key>,_is<Key>,<key>,is<Key>的顺序搜索成员变量名，这里不推荐这么做，因为这样直接访问实例变量破坏了封装性，使代码更脆弱。如果重写了类方法+ (BOOL)accessInstanceVariablesDirectly返回NO的话，那么会直接调用valueForUndefinedKey:
还没有找到的话，调用valueForUndefinedKey:



* 注意事项：
> * value的值必须是id,也就是说不能传基本数据类型，必须是指针类型的变量，所以使用基本数据类型的时候，要装箱成为NSNumber类型。
> * keyPath方法是集成了key的所有功能，也就是说对一个对象的一般属性进行赋值、取值，两个方法是通用的，都可以实现。但是对对象中的对象的属性进行赋值，只有keyPath能够实现，例如person对象有一个car对象的price属性，就需要keyPath:@"car.price"。keyPath就是用“.”把key的路径拼接起来。
> * 继承了NSObject的类型都可以使用KVC。

* 用途
> * 访问修改私有变量，例如：UIPageControl修改圆点为图片；在ios 13之后，UITextField禁止通过KVC修改属性（字体颜色、大小等）
> * xib设置圆角、颜色
> * 字典转模型,setValuesForKeysWithDictionary:

* 参考：https://www.jianshu.com/p/9839ecd75377
* 参考：https://www.jianshu.com/p/45cbd324ea65




