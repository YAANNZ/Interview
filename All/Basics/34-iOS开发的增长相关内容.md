### 埋点的分类
* 代码埋点
* 可视化埋点
* 无埋点

### 代码埋点
由开发人员在触发事件的具体方法里，植入多行代码把需要的数据存下来，然后根据上报策略把前一个时间段收集的数据上传到后台。
* 优点：数据收集精准，可以详细的采集任一点的数据，产品、运营工作量少，对照业务映射表就可以还原出相关业务场景、数据精细无须大量的加工和处理。
* 缺点：侵入业务代码，采集点变更灵活性差需要发版本，开发工作量大、前期需要和运营、产品指定的好业务标识，以便产品和运营进行数据统计分析。

### 可视化埋点
根据标识来识别每一个事件， 针对指定的事件进行取参埋点。而事件的标识与参数信息都写在配置表中，通过动态下发配置表来实现埋点统计。

* 优点：数据量相对准确、后期数据分析成本低。
* 缺点：前期控件的唯一识别、定位都需要额外开发；可视化平台的开发成本较高；对于额外需求的分析可能会比较困难。

### 无埋点
无埋点并不是不需要埋点，更准确的说应该是“全埋”， 前端的任意一个事件都被绑定一个标识，所有的事件都别记录下来。 通过定期上传记录文件，配合文件解析，解析出来我们想要的数据， 并生成可视化报告供专业人员分析 ， 因此实现“无埋点”统计。
* 缺点：前期开发统计基础信息的技术产品成本较高、后期数据分析数据量很大、分析成本较高（大量数据传统的关系型数据库压力大）
* 优点：开发人员工作量小、数据全面、无遗漏、产品和运营按需分析、支持动态页面的统计分析

### 可视化埋点示例
* UIViewController
* UIControl
* UITablview(collectionView与tableView基本相同)
* UITapGesture
1. UIViewController PV统计 <br>
页面的统计较为简单，利用Method Swizzing hook 系统的viewDidLoad， 直接通过页面名称即可锁定页面的展示代码如下：
```
@implementation UIViewController (Analysis)

+(void)load
{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{

        SEL originalDidLoadSelector = @selector(viewDidLoad);
        SEL swizzingDidLoadSelector = @selector(user_viewDidLoad);
        [MethodSwizzingTool swizzingForClass:[self class] originalSel:originalDidLoadSelector swizzingSel:swizzingDidLoadSelector];

    });
}

-(void)user_viewDidLoad
{
    [self user_viewDidLoad];

   //从配置表中取参数的过程 1 固定参数  2 业务参数（此处参数被target持有）
    NSString * identifier = [NSString stringWithFormat:@"%@", [self class]];
    NSDictionary * dic = [[[DataContainer dataInstance].data objectForKey:@"PAGEPV"] objectForKey:identifier];
    if (dic) {
        NSString * pageid = dic[@"userDefined"][@"pageid"];
        NSString * pagename = dic[@"userDefined"][@"pagename"];
        NSDictionary * pagePara = dic[@"pagePara"];

        __block NSMutableDictionary * uploadDic = [NSMutableDictionary dictionaryWithCapacity:0];
        [pagePara enumerateKeysAndObjectsUsingBlock:^(id  _Nonnull key, id  _Nonnull obj, BOOL * _Nonnull stop) {

            id value = [CaptureTool captureVarforInstance:self withPara:obj];
            if (value && key) {
                [uploadDic setObject:value forKey:key];
            }
        }];

        NSLog(@"\n 事件唯一标识为：%@ \n  pageid === %@,\n  pagename === %@,\n pagepara === %@ \n", [self class], pageid, pagename, uploadDic);
    }
}
```
2. UIControl 点击统计 <br>
主要通过hook sendAction:to:forEvent: 来实现, 其唯一标识符我们用 targetname/selector/tag来标记，具体代码如下：
```
@implementation UIControl (Analysis)

+(void)load
{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        SEL originalSelector = @selector(sendAction:to:forEvent:);
        SEL swizzingSelector = @selector(user_sendAction:to:forEvent:);
        [MethodSwizzingTool swizzingForClass:[self class] originalSel:originalSelector swizzingSel:swizzingSelector];
    });
}

-(void)user_sendAction:(SEL)action to:(id)target forEvent:(UIEvent *)event
{
    [self user_sendAction:action to:target forEvent:event];

    NSString * identifier = [NSString stringWithFormat:@"%@/%@/%ld", [target class], NSStringFromSelector(action),self.tag];
    NSDictionary * dic = [[[DataContainer dataInstance].data objectForKey:@"ACTION"] objectForKey:identifier];
    if (dic) {

        NSString * eventid = dic[@"userDefined"][@"eventid"];
        NSString * targetname = dic[@"userDefined"][@"target"];
        NSString * pageid = dic[@"userDefined"][@"pageid"];
        NSString * pagename = dic[@"userDefined"][@"pagename"];
        NSDictionary * pagePara = dic[@"pagePara"];
        __block NSMutableDictionary * uploadDic = [NSMutableDictionary dictionaryWithCapacity:0];
        [pagePara enumerateKeysAndObjectsUsingBlock:^(id  _Nonnull key, id  _Nonnull obj, BOOL * _Nonnull stop) {

            id value = [CaptureTool captureVarforInstance:target withPara:obj];
            if (value && key) {
                [uploadDic setObject:value forKey:key];
            }
        }];

        NSLog(@" \n  唯一标识符为 : %@, \n event id === %@,\n  target === %@, \n  pageid === %@,\n  pagename === %@,\n pagepara === %@ \n", identifier, eventid, targetname, pageid, pagename, uploadDic);
    }
}
```

3. TableView (CollectionView) 的点击统计<br>
tablview的唯一标识， 我们使用 delegate.class/tableview.class/tableview.tag的组合来唯一锁定。 主要是通过hook setDelegate 方法， 在设置代理的时候再去交互 didSelect 方法来实现， 具体的原理是 具体代码如下：
```
@implementation UITableView (Analysis)

+(void)load
{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{

        SEL originalAppearSelector = @selector(setDelegate:);
        SEL swizzingAppearSelector = @selector(user_setDelegate:);
        [MethodSwizzingTool swizzingForClass:[self class] originalSel:originalAppearSelector swizzingSel:swizzingAppearSelector];
    });
}

-(void)user_setDelegate:(id<UITableViewDelegate>)delegate
{
    [self user_setDelegate:delegate];

    SEL sel = @selector(tableView:didSelectRowAtIndexPath:);

    SEL sel_ =  NSSelectorFromString([NSString stringWithFormat:@"%@/%@/%ld", NSStringFromClass([delegate class]), NSStringFromClass([self class]),self.tag]);

    //因为 tableView:didSelectRowAtIndexPath:方法是optional的，所以没有实现的时候直接return
    if (![self isContainSel:sel inClass:[delegate class]]) {

        return;
    }

    BOOL addsuccess = class_addMethod([delegate class],
                                      sel_,
                                      method_getImplementation(class_getInstanceMethod([self class], @selector(user_tableView:didSelectRowAtIndexPath:))),
                                      nil);

    //如果添加成功了就直接交换实现， 如果没有添加成功，说明之前已经添加过并交换过实现了
    if (addsuccess) {
        Method selMethod = class_getInstanceMethod([delegate class], sel);
        Method sel_Method = class_getInstanceMethod([delegate class], sel_);
        method_exchangeImplementations(selMethod, sel_Method);
    }
}

//判断页面是否实现了某个sel
- (BOOL)isContainSel:(SEL)sel inClass:(Class)class {
    unsigned int count;

    Method *methodList = class_copyMethodList(class,&count);
    for (int i = 0; i < count; i++) {
        Method method = methodList[i];
        NSString *tempMethodString = [NSString stringWithUTF8String:sel_getName(method_getName(method))];
        if ([tempMethodString isEqualToString:NSStringFromSelector(sel)]) {
            return YES;
        }
    }
    return NO;
}

// 由于我们交换了方法， 所以在tableview的 didselected 被调用的时候， 实质调用的是以下方法：
-(void)user_tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{

    SEL sel = NSSelectorFromString([NSString stringWithFormat:@"%@/%@/%ld", NSStringFromClass([self class]),  NSStringFromClass([tableView class]), tableView.tag]);
    if ([self respondsToSelector:sel]) {
        IMP imp = [self methodForSelector:sel];
        void (*func)(id, SEL,id,id) = (void *)imp;
        func(self, sel,tableView,indexPath);
    }

    NSString * identifier = [NSString stringWithFormat:@"%@/%@/%ld", [self class],[tableView class], tableView.tag];
    NSDictionary * dic = [[[DataContainer dataInstance].data objectForKey:@"TABLEVIEW"] objectForKey:identifier];
    if (dic) {

        NSString * eventid = dic[@"userDefined"][@"eventid"];
        NSString * targetname = dic[@"userDefined"][@"target"];
        NSString * pageid = dic[@"userDefined"][@"pageid"];
        NSString * pagename = dic[@"userDefined"][@"pagename"];
        NSDictionary * pagePara = dic[@"pagePara"];

        UITableViewCell * cell = [tableView cellForRowAtIndexPath:indexPath];
        __block NSMutableDictionary * uploadDic = [NSMutableDictionary dictionaryWithCapacity:0];
        [pagePara enumerateKeysAndObjectsUsingBlock:^(id  _Nonnull key, id  _Nonnull obj, BOOL * _Nonnull stop) {
            NSInteger containIn = [obj[@"containIn"] integerValue];
            id instance = containIn == 0 ? self : cell;
            id value = [CaptureTool captureVarforInstance:instance withPara:obj];
            if (value && key) {
                [uploadDic setObject:value forKey:key];
            }
        }];

        NSLog(@"\n event id === %@,\n  target === %@, \n  pageid === %@,\n  pagename === %@,\n pagepara === %@ \n", eventid, targetname, pageid, pagename, uploadDic);
    }

}

@end
```
4. gesture方式添加的的点击统计。
gesture的事件，是通过 hook initWithTarget:action:方法来实现的， 事件的唯一标识依然是target.class/actionname来锁定的， 代码如下：
```

@implementation UIGestureRecognizer (Analysis)

+ (void)load
{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{

        [MethodSwizzingTool swizzingForClass:[self class] originalSel:@selector(initWithTarget:action:) swizzingSel:@selector(vi_initWithTarget:action:)];
    });
}

- (instancetype)vi_initWithTarget:(nullable id)target action:(nullable SEL)action
{
    UIGestureRecognizer *selfGestureRecognizer = [self vi_initWithTarget:target action:action];

    if (!target || !action) {
        return selfGestureRecognizer;
    }

    if ([target isKindOfClass:[UIScrollView class]]) {
        return selfGestureRecognizer;
    }

    Class class = [target class];

    SEL originalSEL = action;

    NSString * sel_name = [NSString stringWithFormat:@"%s/%@", class_getName([target class]),NSStringFromSelector(action)];
    SEL swizzledSEL =  NSSelectorFromString(sel_name);

    //给原对象添加一共名字为 “sel_name”的方法，并将方法的实现指向本类中的 responseUser_gesture：方法的实现
    BOOL isAddMethod = class_addMethod(class,
                                       swizzledSEL,
                                       method_getImplementation(class_getInstanceMethod([self class], @selector(responseUser_gesture:))),
                                       nil);

    if (isAddMethod) {
        [MethodSwizzingTool swizzingForClass:class originalSel:originalSEL swizzingSel:swizzledSEL];
    }

    //将gesture的对应的sel存储到 methodName属性中，主要是方便 responseUser_gesture： 方法中取出来
    self.methodName = NSStringFromSelector(action);
    return selfGestureRecognizer;
}

-(void)responseUser_gesture:(UIGestureRecognizer *)gesture
{

    NSString * identifier = [NSString stringWithFormat:@"%s/%@", class_getName([self class]),gesture.methodName];

    //调用原方法
    SEL sel = NSSelectorFromString(identifier);
    if ([self respondsToSelector:sel]) {
        IMP imp = [self methodForSelector:sel];
        void (*func)(id, SEL,id) = (void *)imp;
        func(self, sel,gesture);
    }

    //处理业务，上报埋点
    NSDictionary * dic = [[[DataContainer dataInstance].data objectForKey:@"GESTURE"] objectForKey:identifier];
    if (dic) {

        NSString * eventid = dic[@"userDefined"][@"eventid"];
        NSString * targetname = dic[@"userDefined"][@"target"];
        NSString * pageid = dic[@"userDefined"][@"pageid"];
        NSString * pagename = dic[@"userDefined"][@"pagename"];
        NSDictionary * pagePara = dic[@"pagePara"];

        __block NSMutableDictionary * uploadDic = [NSMutableDictionary dictionaryWithCapacity:0];
        [pagePara enumerateKeysAndObjectsUsingBlock:^(id  _Nonnull key, id  _Nonnull obj, BOOL * _Nonnull stop) {
            id value = [CaptureTool captureVarforInstance:self withPara:obj];
            if (value && key) {
                [uploadDic setObject:value forKey:key];
            }
        }];

        NSLog(@"\n event id === %@,\n  target === %@, \n  pageid === %@,\n  pagename === %@,\n pagepara === %@ \n", eventid, targetname, pageid, pagename, uploadDic);

    }
}
```
#### 配置表结构
首先那， 配置表是一个json数据。 针对不同的场景 （UIControl , 页面PV， Tabeview, Gesture）都做了区分， 用不同的key区别。 对于 "固定参数" ， 我们之间写到配置表中，而对于业务参数， 我们之间写清楚参数在业务内的名字， 以及上传时的 keyName， 参数的持有者。 通过Runtime + KVC来取值。 配置表可以是这个样子：（仅供参考）
>说明： json最外层有四个Key, 分别为 ACTION PAGEPV TABLEVIEW GESTURE, 分别对应 UIControl的点击， 页面PV， tableview cell点击， Gesture 单击事件的参数。 每个key对应的value为json格式，Json中的keys， 即为唯一标识符。 标识符下的json有两个key ： userDefine指的 固定数据， 即直接取值进行上报。 而pagePara为业务参数。 pagePara对应的value也是一个json， json的keys， 即上报的keys， value内的json包含三个参数： propertyName 为属性名字， containIn 参数只有0 ，1 两种情况， 其实这个参数主要是为tabview cell的点击取参做区别的，因为点击cell的时候， 上报的参数可能是被target持有，又或者是被cell本身持有 。 当containIn = 0的时候， 取参数时就从target中取值，= 1的时候就从cell中取值。 propertyPath 是一般备选项， 因为有时候从instace内递归取值的时候，可能会出现在不同的层级有相同的属性名字， 此时 propertyPath就派上用处了。 例如有属性 self.age 和 self.person.age ， 其实如果需要self.person.age， 就把 propertyPath的值设为 person/age， 接着在取值的时候就会按照指定路径进行取值。
```
{
    "ACTION": {
        "ViewController/jumpSecond": {
            "userDefined": {
                "eventid": "201803074|93",
                "target": "",
                "pageid": "234",
                "pagename": "button点击，跳转至下一个页面"
            },
            "pagePara": {
                "testKey9": {
                    "propertyName": "testPara",
                    "propertyPath":"",
                    "containIn": "0"
                }
            }
        }
    },

    "PAGEPV": {
        "ViewController": {
            "userDefined": {
                "pageid": "234",
                "pagename": "XXX 页面展示了"
            },
            "pagePara": {
                "testKey10": {
                    "propertyName": "testPara",
                    "propertyPath":"",
                    "containIn": "0"
                }
            }
        }
    },
    "TABLEVIEW": {
        "ViewController/UITableView/0":{
            "userDefined": {
                "eventid": "201803074|93",
                "target": "",
                "pageid": "234",
                "pagename": "tableview 被点击"
            },
            "pagePara": {
                "user_grade": {
                    "propertyName": "grade",
                    "propertyPath":"",
                    "containIn": "1"
                }
            }
        }
    },

    "GESTURE": {
        "ViewController/controllerclicked:":{
            "userDefined": {
                "eventid": "201803074|93",
                "target": "",
                "pageid": "123",
                "pagename": "手势响应"
            },
            "pagePara": {
                "testKey1": {
                    "propertyName": "testPara",
                    "propertyPath":"",
                    "containIn": "0"
                }
            }
        }
    }
}
```
#### 取参方法
```
@implementation CaptureTool

+(id)captureVarforInstance:(id)instance varName:(NSString *)varName
{
    id value = [instance valueForKey:varName];

    unsigned int count;
    objc_property_t *properties = class_copyPropertyList([instance class], &count);

    if (!value) {
        NSMutableArray * varNameArray = [NSMutableArray arrayWithCapacity:0];
        for (int i = 0; i < count; i++) {
            objc_property_t property = properties[i];
            NSString* propertyAttributes = [NSString stringWithUTF8String:property_getAttributes(property)];
            NSArray* splitPropertyAttributes = [propertyAttributes componentsSeparatedByString:@"\""];
            if (splitPropertyAttributes.count < 2) {
                continue;
            }
            NSString * className = [splitPropertyAttributes objectAtIndex:1];
            Class cls = NSClassFromString(className);
            NSBundle *bundle2 = [NSBundle bundleForClass:cls];
            if (bundle2 == [NSBundle mainBundle]) {
//                NSLog(@"自定义的类----- %@", className);
                const char * name = property_getName(property);
                NSString * varname = [[NSString alloc] initWithCString:name encoding:NSUTF8StringEncoding];
                [varNameArray addObject:varname];
            } else {
//                NSLog(@"系统的类");
            }
        }

        for (NSString * name in varNameArray) {
            id newValue = [instance valueForKey:name];
            if (newValue) {
                value = [newValue valueForKey:varName];
                if (value) {
                    return value;
                }else{
                    value = [[self class] captureVarforInstance:newValue varName:varName];
                }
            }
        }
    }
    return value;
}

+(id)captureVarforInstance:(id)instance withPara:(NSDictionary *)para
{
    NSString * properyName = para[@"propertyName"];
    NSString * propertyPath = para[@"propertyPath"];
    if (propertyPath.length > 0) {
        NSArray * keysArray = [propertyPath componentsSeparatedByString:@"/"];

        return [[self class] captureVarforInstance:instance withKeys:keysArray];
    }
    return [[self class] captureVarforInstance:instance varName:properyName];
}

+(id)captureVarforInstance:(id)instance withKeys:(NSArray *)keyArray
{
    id result = [instance valueForKey:keyArray[0]];

    if (keyArray.count > 1 && result) {
        int i = 1;
        while (i < keyArray.count && result) {
            result = [result valueForKey:keyArray[i]];
            i++;
        }
    }
    return result;
}
@end

```


### 利用三方库埋点
专门用来hook的三方库：[Aspects](https://github.com/steipete/Aspects)
#### 主要方法：
Aspects为NSObject类提供了如下的方法：
```
 + (id<AspectToken>)aspect_hookSelector:(SEL)selector
                  withOptions:(AspectOptions)options
                   usingBlock:(id)block
                        error:(NSError **)error;

 - (id<AspectToken>)aspect_hookSelector:(SEL)selector
                  withOptions:(AspectOptions)options
                   usingBlock:(id)block
                        error:(NSError **)error;
```
这两个方法，都可以用来修改原方法。
返回值是一个id<AspectToken>对象，可以用来注销在方法中所做的修改：
```
id<AspectToken> aspect = ...;
[aspect remove];
```


### AOP（面向切面编程）
AOP为Aspect Oriented Programming的缩写，意为：面向切面编程，通过预编译方式和运行期间动态代理实现程序功能的统一维护的一种技术。
在iOS中，利用runtime进行动态替换方法的操作即可以称做面向切面编程。


### 参考
https://www.jianshu.com/p/21b9f99c574e
https://www.jianshu.com/p/d39cf79370db
https://juejin.cn/post/6844903811786473479#heading-16
https://www.jianshu.com/p/10d342ae6ccd
