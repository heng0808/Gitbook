Effective Objective-C 2.0这本本应该早早就看，最近面试了几家公司，发现很多问题都是出自这里。这本书对于翻译方法稍有欠缺，内容也没那么深入，但是对于日场开发避免一些坑还是有很好的帮助的。
## 第一章 熟悉Objective-C
### 1、了解Objective-C的起源
源至Smalltalk

消息机制
- 消息结构语言由运行环境决定
- 函数调用语言由编译器决定

### 2、在类文件头文件中尽量少引用其他文件
`@class`：向前声明

将引入文件尽量延后，如果都在头文件中引入，将增加编译时间

`向前声明`也可以解决循环引用的问

### 3、多用字面量方法，少用与之等价的方法
1. 缩减代码长度，使之更为易读
2. 字面量方法实际上使一种语法糖

> 语法糖也称为`语法糖衣`，是计算机中与另一套语法相同的一种写法

```objective-c
NSObject *obj1 = [NSObject new];
NSObject *obj2 = nil;
NSArray *arr1 = [NSArray arrayWithObjects:obj1, obj2, nil];
NSArray *arr2 = @[obj1, obj2];
NSDictionary *dic = @{
    @"obj1": obj1,
    @"obj2": obj2
};
```
> 数组第一种写法不会崩溃，会依次插入变量，直到第一个变量为nil   
> 数组第二个会报异常   
> 字典字面量方法会报异常

```objective-c
NSString *str = @"123";
NSNumber *intNum = @1;
NSNumber *floatNum = @2.3f;
NSNumber *doubleNum = @2.345679;
```

### 4、多用类常量，少用#define
`全局符号表`
- 不要使用预处理指令定义常量。这样定义出来的常量不含有类型信息，编译器只是会在编译前据此执行查找与替换操作。即使有人重新定义了常量值，编译器也不会产生警告信息
- 在实现文件中使用`static const`来定义“只在编译单元内可见的产量”。由于此类常量不在全局符号表中，所以无须为其名称加前缀。
- 在头文件中使用extern来声明全局常量，并在相关实现文件中定义其值。这种常量要出现在全局符号表中，所以其名称应加以区隔，通常用与之香港的类名称做前缀。

### 5、用枚举表示选项、状态、状态码
实现枚举所用的类型取决于编译器

Foundation定义了一些辅助宏，能添加类型，并具有向后兼容的能力

在C++编译模式中`NS_OPTION`比`NS_ENUM`在宏定义中具有显示转换类型的能力

## 第二章 对象、消息、运行期
### 6、理解属性这一概念
`noatomic`不能使用`同步锁`

### 7、在对象内部尽量直接使用实例变量
- 在对象内部读取数据时，应该直接通过实例变量来读，而写入数据时，这应该通过属性来写
- 初始化方法以及dealloc方法中，总是应该直接通过实例变量来读写数据
- 有时会使用惰性初始化技术来配置某分数据，这种情况下，要通过属性来读取数据

### 8、理解对象“等同性”这一概念
NSObject协议中有个两个判断对象等同的关键方法：
```objective-c
- (BOOL)isEqual:(id)object;
- (NSUInteger)hash;
```
### 9、以“类簇模式”隐藏实现细节
`工厂模式`是创建类簇的方式之一

### 10、在既有对象中关联对象存储自定义属性
- 可以通过`关联对象`机制来把两个对象连起来
- 定义关联对象时可指定内存管理语义，用以模仿定义属性时所采用的`拥有关系`与`非拥有关系`
- 只有在其他做法不可行时才应选用关联对象，因为这种做法通常会引入难以查找的bug

### 11、理解objc_msgSend的作用
- 消息由接受者、选择子以及参数构成。给某个对象“发送消息”也就相当于在该对象上“调用方法”
- 发给某个对象的全部消息要由“动态消息派发系统”来处理，该系统会查出对应的方法，并执行其代码

### 12、理解消息转发机制
![image](https://raw.githubusercontent.com/SirHeng/Necessary/master/Images/消息转发.png)

### 理解“类对象”的用意
`isMemberOfClass`能判断对象是否为某个特定类的实例

`isKindOfClass`能判断对象使否为某个类或者该类的派生类的实例

- 尽量使用类型信息查询方法来确认对象类型，而不要直接比较类对象，因为某些对象可能实现了消息转发

### 13、用`方法调配技术`调试`黑盒方法`
- 使用另一份实现来替换原有的实现，这道工序叫做`方法调配`，开发者常用此技术向原有的方法添加新功能
- 一般来说，只有在方法调试的时候才用此技术替换原有方法，此技术不宜滥用

### 14、理解`类对象`的用意
- 每个实例都有一个指向`Class`的对象指针，用以表明其类型，而这些`Class`对象则构成了类的继承关系
- 如果对象类型无法在编译期确定，那么就应该使用类型信息查询方法类探知
- 尽量使用类型信息查询方法来确定对象类型，而不要直接比较类对象，因为某些对象可能实现了消息转发功能

## 第三章 接口与Api设计
### 15、用前缀避免命名空间冲突
- 选择与你公司、应用程序或者二进制皆有关联之名称作类名的前缀，并在所有代码中均使用这一前缀
- 若自己所开发的程序库中用到了第三方库，则应为其中的名称加上前缀

### 16、提供全能初始化方法
- 在类中提供一个全能的初始化方法，并于文档里指名。其他的初始化方法均应调用此方法
- 若全能初始化方法与超类不同，则需要覆写超类中的对应方法
- 如果超类的初始化方法不适用于子类，那么应该覆写这个超类方法，并在其中抛出异常

### 17、实现`description`方法
- 实现`description`方法返回一个有意义的字符串，用以描述实例
- 若想在调试时打印出更消息的对象描述信息，则应实现`debugDescription`

### 18、尽量使用不可变对象
- 尽量创建不可变的对象
- 若某属性仅可用于对象的内部修改
- 则应该在“class-continuation分类”中将其由`readonly`属性扩展为`readwrite`属性
- 不要把不可变的`collection`作为属性公开，而应该提供相关方法，以此修改对象中的可变`collection`

### 19、使用清晰而协调的命名方式
- 起名时应遵循标准的Objective-C命名规范，这样创建出来的接口更容易为开发者所理解
- 方法名要言简意赅，从左至右读起来要像个日常用语中的句子才好
- 方法名里不要使用缩略后的类型名称
- 给方法起名时的第一要务就是确保其风格与自己的代码或所要集成的框架相符

### 20、为私有方法名加前缀
- 给私有方法的名称加上前缀，这样可以很容易地将其同公共方法区分开
- 不要单用一个下划线做私有方法的前缀，因为这种做法是预留给苹果公司用的

### 21、理解Objective-C错误模型
- 只有发生了可使整个应用程序崩溃的严重错误时，才应使用异常
- 在错误不那么严重的情况下，可以指派`委托方法`来处理错误，也可以把错误信息放在NSError对象里，经由“输出参数”返回给调用者

### 22、理解NSCopying协议
- 若想令自己所写的对象具有拷贝功能，则需实现NSCopying协议
- 如果自定义的对象分为可变版本与不可变版本，那么就要同时实现NSCopying与NSMutableCopying协议
- 复制对象时需要决定采用浅拷贝还是深拷贝，一般情况下应该尽量执行浅浅拷贝
- 如果你所写的对象需要深拷贝，那么可以考虑新增一个专门执行浅拷贝的方法

## 第四章 协议与分类
### 23、通过委托与数据源协议进行对象间通信
- 委托模式为对象提供了一套接口，使其可由此将相关时间告知其他对象
- 将委托对象应该支持的接口定义成协议，在协议中把可能需要处理的事件定义成方法
- 当某对象需要从另外一个对象中获取数据时，可以使用委托模式。这种情景下，该模式亦称“数据源协议”
- 若有必要，则可实现环游位段的结构体，将委托对象是否能响应相关协议方法这一信息缓存至其中

### 24、将类的实现代码分散到便于管理的各个分类中
各个分类代码都实现在主类中

- 使用分类机制把类的实现代码划分成易于管理的小块
- 将应该视为“私有”的方法归入名叫`Private`的分类中，以隐藏实现细节

### 25、总是为第三方分类的名称加前缀
- 向第三方类中添加分类时，总应给其名称加上你专用的前缀
- 向第三方类中添加分类时，总应给其中的方法加上你专用的前缀

### 26、勿在分类中声明属性
- 把封装数据所用的全部属性都定义在主接口里
- 在“class-continuation分类”之外的其他分类中，可以定义存取方法，但尽量不要定义属性

### 27、使用`class-continuation`隐藏实现细节
- 通过“class-continuation分类”向类中新增加实例变量
- 如果某属性在主接口中声明为“只读”，而分类的内部定义又要用设置方法修改此属性，那么就在“class-continuation分类”中将其扩展为“可读写”
- 把私有方法的原型声明在“class-continuation分类”里面
- 若想使类所遵循的协议不为人所知，则可于“class-continuation分类”中声明

### 28、通过协议提供匿名对象
- 协议可在某种程度上提供匿名类型。具体的对象类型可以淡化成遵从某协议的id类型，协议里规定了对象所应实现的方法
- 使用匿名对象来隐藏类型名称（或类名）
- 如果具体类型不重要，重要的使对象能够响应（定义在协议里的）特定方法，那么可使用匿名对象来表示

## 内存管理
### 29、理解引用计数
- Retain 递增引用计数
- Release 递减引用计数 
- autorelease 待稍后清理“自动释放池”时，在递增引用计数

`retainCount`查看引用计数，此方法不太有用，即便在调试时也如此

> 官方文档：调试内存管理问题时调用该方法是没有任何意义的。因为保不准Cocoa framework中的其它对象会retain我们的目标对象，还有对于autorelease pool中的延迟释放对象，调用这个方法也得不到有用的信息。

对象创建出来，引用计数至少为`1`

当对象的引用计数为`0`的时候，对象就内存就回收了（deallocated），系统将对象占用的内容标记为`reuse`，所有指向该内存的指针都会无效。

如果调用release后，基于某些原因，其引用计数降至0，那么对象所占内存也许会回收，这样的话，再使用该对象可能就将使程序崩溃了。这里只说“可能”，而没说“一定” ，因为对象所占用的内存再“接触分配”之后，只是放回“可用内存池”。如果使用该对象的时尚未覆写该对象内存，那么该对象仍然有效，这时程序不会崩溃。由此可见：过早的释放对象而导致的bug很难调试

为避免这种情况，一般调用了release后都会清空指针。这样能保证不会出现可能指向无效对象的指针，这种指针通常被称为“悬挂指针”。比方说，可以这样编写代码防止此情况发生：
```objective-c
NSNumber *number = [[NSNumber alloc] init];
[arrary addObject:number];
[number release];
number = nil;
```

#### 属性存取方法的内存管理
```objective-c
- (void)setFoo:(id)foo {
    [foo retain];
    [_foo release];
    _foo = foo;
}
```

#### 自动释放池
在Objective-C的引用计架构中，自动释放池时一项重要的特性。调用release会立刻递减对象的引用计数（而且还有可能令系统回收此对象），然而有时候可以不掉用他，改为`autorelease`，此方法会在稍候递减计数，通常时下一次“事件循环”时递减，不过也可能执行的更早（参见第34条）
```objective-c
- (NSString *)stringValue {
    NSString *str = [[NSString alloc] initWithFormat:@"I am this: %@", self];
    // return str; 不能这么写，因为alloc使内存引用计数加一
    // [str release] 也不能直接release，因为返回的要用
    return [str autorelease]; // 这样就会再下一次事件循环中引用计数减一，对外部没有影响
}
```
如果按“引用树”回溯，那么最终会发现一个根对象，在mac os中就是NSApplication对象；在iOS中则试UIApplication。两者都是应用程序启动时创的单例。

### 30、以ARC简化引用计数
若方法名以下列词语开头，则其返回的对象归调用者所有：
- alloc
- new
- copy
- mutableCopy
归调用者所有的意思时：调用上述四种方法的那段代码负责释放方法所返回的对象。

若方法名不以上述四个词语开头，则表示其返回的对象并不归调用者所有。在这种情况下，返回的对象会自动释放，所以其值再跨越方法调用边界后依然有效。要想使对象多存活一段时间，必须令调用者保留它才行
```objective-c
+ (EOCPerson*)newPerson {
    EOCPerson *person = [[EOCPerson alloc] init]; 
    return person;
    /**
    * The method name begins with 'new', and since 'person' * already has an unbalanced +1 retain count from the
    * 'alloc', no retains, releases, or autoreleases are
    * required when returning.
    */
}

+ (EOCPerson*)somePerson {
    EOCPerson *person = [[EOCPerson alloc] init]; 
    return person;
    /**
    * The method name does not begin with one of the "owning"
    * prefixes, therefore ARC will add an autorelease when
    * returning 'person'.
    * The equivalent manual reference counting statement is:
    * return [person autorelease];
    */
}

- (void)doSomething {
    EOCPerson *personOne = [EOCPerson newPerson]; // ...
    EOCPerson *personTwo = [EOCPerson somePerson]; // ...
    /**
    * At this point, 'personOne' and 'personTwo' go out of 
    * scope, therefore ARC needs to clean them up as required. 
    * - 'personOne' was returned as owned by this block of
    * code, so it needs to be released.
    * - 'personTwo' was returned not owned by this block of
    * code, so it does not need to be released.
    * The equivalent manual reference counting cleanup code * is:
    * [personOne release];
    */
}
```

ARC可以在运行期间检测到多余操作，例如：
```objective-c
EOCPerson *tmp = [EOCPerson personWithName:@"Bob Smith"]; 、/_myPerson = [tmp retain]; // add by arc
```
上面代码，arc中会加上retain，所以内部autorelease和外部retain都可以去掉，改为以下调用：
```objective-c
// Within EOCPerson class
+ (EOCPerson*)personWithName:(NSString*)name { 
    EOCPerson *person = [[EOCPerson alloc] init]; person.name = name;
    objc_autoreleaseReturnValue(person);
}
// Code using EOCPerson class
EOCPerson *tmp = [EOCPerson personWithName:@"Matt Galloway"];_myPerson = objc_retainAutoreleasedReturnValue(tmp);
```

#### ARC如何清理实例变量
.cxx_destruc

### 31、在dealloc方法中只释放引用并接触监听
- 在dealloc方法里，应该做的事情就是释放指向其他对象的引用，并取消原来的订阅“键值观测”（KVO）或者NSNotificationCenter等通知，不要做其他的事情。
- 如果对象持有文件描述符等系统资源，那么应该专门编写一个方法来释放此种资源。这样的类要和其他使用者约定：用完资源后必须调用`close`方法
- 执行异步任务的方法不应该在dealloc里调用：只能在正常状态下执行的那些方法也不应该在dealloc里调用，因为此时对象已处于正在回收的状态了

### 32、编写“异常安全代码”时留意内存管理问题
- 捕获异常时，一定要注意将try块内所创立的对象清理干净
- 在默认情况下，ARC不生成安全处理异常的清理代码。开启编译器标志后，可以生成这种代码，不过会导致应用程序变大，而且会降低运行效率

### 33、以弱引用避免保留链
- 将某些引用设为`weak`，可避免出现保留环
- `weak`引用可以自动清理，也可以不自动清空。自动情况时随着ARC而引入的新特性，由运行期系统来实现。在具备自动情况功能的弱引用是哪个，可以随意读取其数据，因为这种引用不会指向已经回收过的对象

### 34、以“自动释放池块”降低内存峰值
```objective-c
int main(int argc, char *argv[]) { 
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, @"EOCAppDelegate");
    } 
}
```
从技术角度，不是非得有个“自动释放块”才行。因为块的末尾恰好时应用程序的终止处，而此时操作系统会把程序所占用的全部内存逗释放掉。虽说如此，但是如果不写这个块的话，那么由于UIApplication函数所自动释放的那些对象，就没有自动释放池可以容纳了，于是系统会发出警告信息来表明这一情况。所以说，这个池可以理解成最外围捕捉全部自动释放对象所有地

- 自动释放池排布在栈中，对象收到autorelease消息后，系统将其放入最顶端的池里
- 合理运用自动释放池，可以降低应用程序的内存峰值
- `autoreleasepool`这种新式写法能创建更为轻便的自动释放池

### 35、用`僵尸对象`调试内存问题
`僵尸对象`是这个非常方便的功能。启用这项调试功能之后，运行期系统会把所有已经回收的实例转化成特殊的“僵尸对象”，而不会真正回收它们。这种对象所在的核心内存无法重用，因此不可能遭到覆写。僵尸对象收到消息后，会抛出异常，其中准确说明了发送过来的消息，并描述了回收之前的那个对象。僵尸对象时调试内存管理问题的最佳方式。

- 系统会修改对象的`isa`的指针，令其指向特殊的僵尸类，从而使该对象变为僵尸对象。僵尸类能够响应所有的选择器，响应的方式为：打印一条包含该消息内存及其接收者的消息，然后终止应用程序

### 36、不要使用retainCount
`retainCount`此方法之所以无用，其首要原因在于：它返回的保留计数只是某个给定时间点上的值。该方法并未考虑到系统会稍候把自动释放池情况，因而不会将后续的释放操作从返回值里减去，这样的话，此值就未必能真实的反应实际的引用计数。

其次`retainCount`可能永远不返回0，因为有时系统会优化对象的释放行为，在引用计数还是1的时候就把它回收了。只有在系统不打算这么优化时，计数值才会递减至0.因此，引用计数永远都不会完全为零。

#### 标签指针（tagged pointer）
在 2013 年 9 月，苹果推出了 iPhone5s ，与此同时，iPhone5s 配备了首个采用 64 位架构的 A7 双核处理器，为了节省内存和提高执行效率，苹果提出了Tagged Pointer的概念。对于 64 位程序，引入 Tagged Pointer 后，相关逻辑能减少一半的内存占用，以及 3 倍的访问速度提升，100 倍的创建、销毁速度提升。

![image](https://raw.githubusercontent.com/SirHeng/Necessary/master/Images/标签指针.jpg)

![image](https://raw.githubusercontent.com/SirHeng/Necessary/master/Images/标签指针1.jpg)

## 第六章 块与大中枢派发
### 37、理解“块”这一概念
![image](https://raw.githubusercontent.com/SirHeng/Necessary/master/Images/块.png)

#### 全局block、栈block、堆block

### 为常用的块类型创建typedef


__NSCFConstantString