# iOS 面试指南
## 准备简历
突出重点，简明扼要的简历是必须的，推荐一个简历模版，有一个免费版，三个收费版，地址：[awesome-resume](https://github.com/resumejob/awesome-resume)。HTML 模版，转 PDF 可以用 Python 脚本、Safari、Chrome插件、Adobe工具等等方式。

## 知识点
### UI
* UITableView 重用池&多线程下的数据源同步
* 事件传递&响应链：[史上最详细的iOS之事件的传递和响应机制](https://www.jianshu.com/p/2e074db792ba)
* 绘制原理&异步绘制
* 卡顿掉帧：[iOS 保持界面流畅的技巧](https://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)
* 离屏渲染
* 图像显示原理

### Runtime
* Runtime 的理解一：（[简书](https://www.jianshu.com/p/a23f0b30baf6) | [Blog](http://zynlo.xyz/2018/06/08/runtime的理解/)）
* Runtime 的理解二：（[简书](https://www.jianshu.com/p/5d87f3e32108) | [Blog](http://zynlo.xyz/2018/06/14/runtime的理解二/)）

### 内存管理
* 修饰关键字
* ARC&MRC：（[简书](https://www.jianshu.com/p/88bc29146363) | [Blog](http://zynlo.xyz/2018/06/27/内存管理的理解/)）
* autoreleasepool
* 循环引用：场景&解决方案
* 数据结构

### Block
* 实现原理
* 内存管理
* 循环引用
* 与函数指针的区别
* __block 在 ARC 和 MRC 下有什么不同

### KVO
* 实现机制
* 使用注意点&第三方实现
* 注册依赖键是什么
* 如何实现 KVO 手动通知

### 多线程
推荐文章：[关于iOS多线程，你看我就够了](https://www.jianshu.com/p/0b0d9b1f1f19)；[多线程的理解（一）](http://zynlo.xyz/2018/07/04/多线程的理解一/)
* 进程&线程
* 进程间通信&线程间通信
* NSThread & GCD & NSOperation
* 多线程同步
* 并行并发&同步异步
* 死锁

### RunLoop
推荐文章：[深入理解RunLoop](https://www.jianshu.com/p/0b0d9b1f1f19)
* 实现原理
* 与线程的关系
* AFNetworking 中如何运用 Runloop
* 利用 runloop 解释页面渲染的过程

### 网络
* Http和Https是什么以及区别
* Https通信过程
* 三次握手和四次挥手以及为什么
* HTTP 请求报文 和 响应报文的结构
* 数据传输的加密过程
* TCP/IP 五层模型
* OSI 七层模型
* 断点续传


### 设计模式
* 六大设计原则
* 常用设计模式
* 如何设计一个图片缓存框架
* 如何设计一个时长统计框架
* 如何实现 App 换肤

### 架构
推荐一个经典系列：[iOS 应用架构谈](https://casatwy.com/iosying-yong-jia-gou-tan-kai-pian.html)
* MVC
* MVVM
* 组件化

### 三方库源码理解
* YYKit
* SDWebImage
* AFNetworking
* MBProgressHUD
* MJRefresh

### 持续集成
* jenkins&叮叮机器人

### 单元测试
* 是什么&如何运用
* NSAssert

### Hybrid
* JavaScriptCore.framework
* ReactNative
* Flutter
* Cordova
* Appcan

### 算法
* 查找两个子视图的共同父试图
* 字符串反转
* 有序数组合并
* 链表反转

### APM（Application Performance Management）
* 内存泄漏检测：[Instrument 检测内存泄漏](http://zynlo.xyz/2018/06/27/Instrument检测内存泄漏/)
* Crash监控&卡顿监控以及底层的实现原理
* 如何优化 App 启动时间
* 如何对 APP 进行内存、电量、网络流量的优化
* 如何降低 App 包大小
* Instruments 进行性能调优(Time Profiler、Zombies、Allocations、Leaks)

## 面试技巧
### 自我介绍
声音平稳清晰，内容简洁大方，突出重点优点，引起面试官注意力和好奇心，是一个好的自我介绍的基本要点。推荐一篇文章：[面试的时候，如何自我介绍？](https://www.zhihu.com/question/19603341)

### 常问开放性问题
* 最有深度的项目，项目遇到的最大问题，解决方案
* 离职的原因
* 和同事领导的关系

## 推荐书籍
* 重构
* 图解HTTP
* 算法图解
* [leetcode](https://leetcode.com)
* 剑指Offer
* 算法导论





