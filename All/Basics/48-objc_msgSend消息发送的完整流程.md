
### 一、消息发送
1. 首先判断消息接收者receiver是否是nil，如果为nil调用LNilOrTagged、LReturnZero返回0；
2. receiver不为nil，调用CacheLookup查找方法实现IMP缓存；如果找到缓存直接调用TailCallCachedImp调用IMP；
3. 没有缓存，则调用宏MethodTableLookup，调用_class_lookupMethodAndLoadCache3函数 --> lookUpImpOrForward函数；在当前class中查找方法实现：调用getMethodNoSuper_nolock --> search_method_list函数，**通过传入的SEL类型参数selector从方法列表中查找方法实现imp；如果能查找到则缓存imp并调用；**
4. 如果当前class方法列表中没有找到，则递归向父类class查找；（同样的流程：先查找缓存，再从方法列表中查找）

### 二、动态方法解析
* `resolveInstanceMethod`
* `resolveClassMethod`

示例：
```
@implementation Dog

- (void)bark {
    NSLog(@"wang wang");
}

+ (BOOL)resolveInstanceMethod:(SEL)sel {
    if (sel == @selector(mew)) {
        Method method = class_getInstanceMethod(self, @selector(bark));
        class_addMethod(self, sel, method_getImplementation(method), method_getTypeEncoding(method));
        return YES; // YES动态添加了方法  NO没有添加
    }
    return [super resolveInstanceMethod:sel];
}

@end
```


### 三、备用接受者
* `forwardingTargetForSelector`
```
#pragma mark - 备用接受者
- (void)testForwardingReceiver
{
    [self performSelector:@selector(justTestForwarding:) withObject:@"testStr"];
}

- (id)forwardingTargetForSelector:(SEL)aSelector
{
    if (aSelector == @selector(justTestForwarding:))
    {
        return [[DITForwardingTest alloc] init];
    }
    
    return [super forwardingTargetForSelector:aSelector];
}
```

### 四、消息转发
* `-methodSignatureForSelector: `
* `-forwardInvocation:`
```
#pragma mark - 完整消息转发
- (void)testMethodSignature
{
    [self performSelector:@selector(justTestMethodSignature:) withObject:@"testMethodSignatureStr"];
}

- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector
{
    if (aSelector == @selector(justTestMethodSignature:))
    {
        return [NSMethodSignature signatureWithObjCTypes:"v@:"];
    }
    
    return [super methodSignatureForSelector:aSelector];
}

- (void)forwardInvocation:(NSInvocation *)anInvocation
{
    SEL sel = anInvocation.selector;
    DITForwardingTest *methodSignatureTest = [[DITForwardingTest alloc] init];
    
    if ([methodSignatureTest respondsToSelector:sel])
    {
        [anInvocation invokeWithTarget:methodSignatureTest];
    }
    else
    {
        [anInvocation doesNotRecognizeSelector:sel];
//        [super forwardInvocation:anInvocation];
    }
}
```
