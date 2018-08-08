Watch Dog监控主线程卡顿的实现：

```objective-c
@interface PingThread : NSThread

@property (nonatomic, assign) double threshold;
@property (nonatomic, assign) BOOL   pingTaskIsRunning;
@property (nonatomic, copy)   handler handler;

@end

@implementation PingThread

- (instancetype)initWithThreshold:(double)threshold handler:(handler)handler
{
    if (self = [super init]) {
        self.pingTaskIsRunning = NO;
        self.threshold = threshold;
        self.handler = handler;
    }

    return self;
}

- (void)main
{
   dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);

    while (!self.cancelled) {
        self.pingTaskIsRunning = YES;
        dispatch_async(dispatch_get_main_queue(), ^{
            self.pingTaskIsRunning = NO;
            dispatch_semaphore_signal(semaphore);
        });
    
        [NSThread sleepForTimeInterval:self.threshold];
        if (self.pingTaskIsRunning) {
            self.handler();
        }
        
        dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
    }
}

@end

@interface BHWatchDog()


@property (nonatomic, assign) double threshold;
@property (nonatomic, strong) PingThread *pingThread;

@end

@implementation BHWatchDog

- (instancetype)initWithThreshold:(double)threshold strictMode:(BOOL)strictMode
{
    self = [self initWIthThreshold:threshold callBack:^() {
        NSString *message = [NSString stringWithFormat:@"👮 Main thread was blocked 👮"];
        if (strictMode) {
            //避免后台切换导致进入断言
            NSAssert([UIApplication sharedApplication].applicationState == UIApplicationStateBackground, message);
        } else {
            BHLog(@"%@", message);
        }
    }];

    return self;
}


- (instancetype)initWIthThreshold:(double)threshold callBack:(watchdogFiredCallBack)callBack
{
    if (self = [self init]) {
        self.threshold = threshold;
        self.pingThread = [[PingThread alloc] initWithThreshold:threshold handler:callBack];
        [self.pingThread start];
    }
   
    return self;
}


- (void)dealloc
{
    [self.pingThread cancel]; // 线程的退出注意用cancel方法
}

@end
```

