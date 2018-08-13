#Cocoa设计模式

##Ⅰ、一种可控制一切的模式

###一、MVC

1. 减少应用程序内部的耦合，但会增加程序的复杂性。将视图层、逻辑层、数据层解耦，有利于程序的扩展开发、问题调试，提高可读性。


##Ⅱ、基础模式

###一、两阶段创建（[[Class alloc] init]）

1. NSZone概念。两个或者多个对象关联密切，但可能会被分配到相距很远的内存中，甚至一个在内存中另一个在硬盘（虚拟内存）中，它们在频繁切换调用时，内存页面调度频繁，会浪费性能，产生抖动。
  显示的将关联密切的对象创建在同一个NSZone（initWithZone:）对象中，可以解决以上问题。但如果采用的是自动内存管理机制，系统会忽略传过来的指定的分区对象。
2. ▲当一个类提供了多个初始化方法时，它应该指定一个初始化方法。如NSObject指定的初始化方法为init，UIView指定的初始化方法为initWithFrame:。指定的初始化方法通常要接受最多的参数变量，所有其他初始化方法都在实现时调用指定初始化方法。
3. 所有Objective-C方法中隐式的接收到两个参数，一个是self，它是接收消息的对象，另一个是_cmd，用于识别接收到的消息。

###二、模板方法（不要来找我，我会去找你。如-dealloc, -drawRect:）

1.  用处：高度重用的方式来实现一些通用的算法和过程，定制若干步骤。
2.  重写模板方法：
  （1）何时可以调用默认模板方法，如-drawRect:，父类的默认实现中，并未做任何事情，所以随时可以调用默认实现
  （2）何时应该调用模板方法，如-hitTest:，-viewWillApear等方法，如果未调用父类同名模板方法，父类方法就不会被执行，这样是可能出问题的
  （3）何时必须调用模板方法，如-dealloc，在手动内存管理环境下，如果不重写-dealloc方法，势必会造成内存泄露
3.  利用模板方法进行设计的步骤：
  （1）确定一个算法的步骤并用一个或多个方法实现它，将算法设计成对定义好的方法的一系列调用
  （2）指出算法里可定制的步骤，并为每个模板方法提供一个合理的默认实现
  （3）明确描述基类的每一个默认模板方法是否可以、应该或是必须被子类中重写的实现调用
4.  *Cocoa一般不会直接调用-drawRect:方法，而是会发送一个-setNeedDisplay:的消息，这会触发Cocoa在稍后来重绘视图（将包含一次-drwaRect:方法的调用）
5.  ▲Cocoa中其他常见重要的模板方法
  （1）- (void)forwardInvocation:(NSInvocation *)anInvocation;
  （2）- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector;
6.  模板方法的缺点
  （1）带来了父子类关系，即带来了强耦合问题
  （2）Cocoa没有提供区别可以、应该或必须调用默认实现的方案，在使用时可能会产生不必要的问题（可以通过协议@require/@optional进行标志）
  所以，模板方法最好只保留于最成熟稳定的设计中。

###三、动态创建（通过NSClassFromString()方法动态创建对象）

1. 在类之间进行解耦，并把使用哪个类的决定推迟到运行时  （Question：动态创建一个类，该类的生命周期是怎样的？如果重复的创建同一个类，会不会导致崩溃？）
2. 动态创建支持了插件架构的实现（NSBundle包含了代码、图片、IB、文本、XML文件等，使之成为一个程序包，常用于实现插件）

###四、类别（Category）

1. 提供了良好的扩展性、多开发人员开发同一类的协调性，减少了子类的使用，减少了耦合性

###五、匿名类型和异类容器（id和NSArray、NSMutableArray、NSDictionary、NSMutableDictionary、NSSet……）

1. *XNU内核
  （1）内环Mach提供基础服务，如CPU调度、IPC（进程间通信inter-process communication）、虚拟内存
  （2）外环层BSD提供高层服务，如进程管理、网络服务、文件系统
  （3）I/O Kit
2. *指令集
  （1）armv7｜armv7s｜arm64都是ARM处理器的指令集
  （2）i386是针对intel通用微处理器32位处理器，x86_64是针对x86架构的64位处理器
  模拟器32位处理器测试需要i386架构，
  模拟器64位处理器测试需要x86_64架构，
  真机32位处理器需要armv7,或者armv7s架构，
  真机64位处理器需要arm64架构。

###六、枚举器（Iterator）

###七、执行选择器和延迟执行（-(id)performSelector:(SEL)aSelector和-(id)performSelector:(SEL)aSelector withObject:(id)anObject afterDelay:(NSTimeInterval)delay）

1. 指定模式的延迟执行-(id)performSelector:(SEL)aSelector withObject:(id)anObject afterDelay:(NSTimeInterval)delay inModes:(NSArray *)modes; （Question：怎么才算一个RunLoop循环）
2. 消息发送核心方法
  （1）id objc_msgSend(id self, SEL op, ...);
  （2）id objc_msgSendSuper(struct objc_super *super, SEL op, ...);
3. performSelector:方法的实现
- (id)performSelector:(SEL)aSelector
  {
  IMP methodImplementation = [self methodForSelector:aSelector];
  return (*IMP)(self, aSelector);
  }
4. 消息发送的流程
  Test *test = [[Test alloc] init];
  [test performSelector(@selector(xxx))];

+ Test NSObject initialize 
+ Test NSObject alloc 
+ Test NSObject allocWithZone: 
- Test NSObject init 
- Test NSObject performSelector: 
+ Test NSObject resolveInstanceMethod: 
- Test NSObject forwardingTargetForSelector: 
- Test NSObject methodSignatureForSelector: 
- Test NSObject class 
- Test NSObject doesNotRecognizeSelector: 

（1）调用resolveInstanceMethod:方法，允许用户在此时为该Class动态添加实现。如果有实现了，则调用并返回。如果仍没实现，继续下面的动作。
（2）调用forwardingTargetForSelector:方法，尝试找到一个能响应该消息的对象。如果获取到，则直接转发给它。如果返回了nil，继续下面的动作。
（3）调用methodSignatureForSelector:方法，尝试获得一个方法签名。如果获取不到，则直接调用doesNotRecognizeSelector抛出异常。
（4）如果步骤3中获取到了方法签名，调用forwardInvocation:方法，将地步骤3获取到的方法签名包装成Invocation传入，如何处理就在这里面了。

（Todo：验证一下消息查找顺序）
（1）给实例对象消息的过程(调用对象方法)
    根据对象的isA指针去该对象的类方法中查找，如果找到了就执行
    如果没有找到，就去该类的父类类对象中查找
    如果没有找到就一直往上找，直到根类（NSObject）
    如果都没有找到就报错
（2）给类对象发送消息(调用类方法)
    根据类对象的isA指针去元对象中查找，如果找到了就执行
    如果没有找到就去父元对象中查找
    如果如果没有找到就一直往上查找，直到根类（NSOject）
    如果都没有找到就报错

###八、访问器（getter && setter）

1. 普通的getter方法，是按值返回的，获取到的返回值，跟原对象并不是同一个对象，如下面类的方法实现：
  @interface Dog : NSObject

- (NSMutableString *)name;

@end

@interface Dog() {
  NSMutableString *_myName;
}

@end 

@implementation Dog

- (NSMutableString *)name
  {
  return [_myName mutableCopy];
  }

@end

2. 带有get前缀的getter方法，是按引用返回的

###九、归档和解归档（NSKeyedArchiver && NSKeyedUnarchiver）

###十、复制

1. 浅复制
- (id)copy
  {
  return [self retain];
  }

2. 深复制
- (id)deepCopy
  {
  return [[NSKeyedUnarchiver unarchiveObjectWithData:[NSKeyedArchiver archivedDataWithObject:self]] retain];
  }

##Ⅲ、主要通过解耦来变得强大的模式

###一、单例

###二、通知

1. 本质上通知是通过KVO实现的，注册成为通知接收者，即注册成为观察者
2. NSNotificationCenter处理单进程之间的通知，NSNotifacationQueue处理异步通知，NSDistributedNotificationCenter处理单个计算机上不同的进程之间的通知。

###三、委托

1. 委托模式也可以减少子类化（如NSApplication对象，系统可以采取代理模式产生AppDelegate的方式通知开发者Application的生命周期，也可以通过允许开发者子类化NSApplication，重写模板方法的方式达到目的，但是委托模式显然更好一些）

###四、层次结构

###五、插座变量、目标和动作（nib文件）

###六、响应者链

1. 用户操作产生的消息传递链（通过-hitTest:方法查找最合适接收消息的视图，操作系统最先接收到用户消息，转发给Application，再转发给AppDelegate，再转发给Window对象，Window对象开始-hitTest:）
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event {
    // 判断自己能否接收事件
    if (self.userInteractionEnabled == NO || self.alpha <= 0.01 || self.hidden == YES) {
        return nil;
    }

    // 判断点是不是在当前视图上
    if (![self pointInside:point withEvent:event]) {
        return nil;
    }

    // 从后往前遍历自己的子控件，寻找更合适的View
    for (long i = self.subviews.count - 1; i >= 0; i--) {
        // 获取子控件
        UIView *childView = self.subviews[i];
        
        // 将自己坐标系的点转化成子控件坐标系的点
        CGPoint childPoint = [self convertPoint:point toView:childView];
        
        // 递归调用hitTest方法，寻找更加合适的View
        UIView *fitView = [childView hitTest:childPoint withEvent:event];
        if (fitView) {
            return fitView;
        }
    }
    // 没有找到比自己更适合的View
    return self;
    }
2. 响应者链
  当消息传递链找到最适合相应消息的视图之后，由该视图开始判断是否能够响应该事件。
  如果能够响应，则响应时间，该消息被处理，传递停止；
  如果该视图不能响应该事件，则根据视图层次结构往上一层（nextResponder）及传递该事件，直到Window对象。
3. 响应者链的应用
  （1）通过响应者链可以获取当前最上层的ViewController、最上层的NavigationController等
  （2）*通过判断是否有target来响应某事件，控制一些视图的是否可用，如Button的Enable属性（Untested）
  （3）*通过修改响应者链，来达到一些目的（Untested）

###七、联合存储

1. NSMapTable对象，实现通过类别给类添加属性（Untested）
  关于NSPointerArray、NSHashTable、NSMapTable可参考http://www.isaced.com/post-235.html

###八、调用（NSInvocation）

1. 调用用于保存消息的状态、参数和返回值
2. 可以将消息发送者和接收者完全解耦，发送者和接收者可以位于不同的进程中，或者时间上隔离开
3. 调用常用于分布式对象、撤销和重做的支持、预定的周期性事件处理中
  结论：▲调用打包了一个OC消息，使用调用，开发人员可以自由的创建、修改消息，可以捕获消息并转发给其他对象，可以延迟发送消息、重发消息。

###九、原型

###十、▲享元（封装非对象值、减少内存使用量、替代其他对象）

1. 优化性能和存储，但会使编码复杂度提高，目前感觉像是Cell的复用机制
2. 两次执行[NSNumber numberWithInt:99];时，得到很可能是相同的实例

###十一、装饰器

1. 装饰器模式通过复合给【对象】添加了公用的可重用能力，代替通过子类化添加这些能力
2. 可以在运行时添加或配置装饰器

Ⅳ、主要用于隐藏复杂性的模式

Ⅴ、模式应用的实用工具
1. NSProxy代理转发消息，没有父类，尽可能保证所有的消息都通过-forwardInvocation:方法实现

【Question】标志了未处理疑问
【Todo】标志了未处理事情
【Untested】标志了未经试验或未通过代码证实过的点