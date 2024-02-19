# 理论五：接口vs抽象类的区别？如何用普通的类模拟抽象类和接口？

在面向对象编程中，抽象类和接口是两个经常被用到的语法概念，是面向对象四大特性，以 及很多设计模式、设计思想、设计原则编程实现的基础。可以使用接口来实现面 向对象的抽象特性、多态特性和基于接口而非实现的设计原则，使用抽象类来实现面向对象 的继承特性和模板设计模式等等。

C++ 这种编程语言 只支持抽象类，不支持接口；而像 Python 这样的动态编程语言，既不支持抽象类，也不支 持接口。

## 什么是抽象类和接口？区别在哪里？

典型的抽象类的使用场景（模板设计模式）。Logger 是一个记录日志的抽象类，FileLogger 和 MessageQueueLogger 继承 Logger，分别实现两种不同 的日志记录方式：记录日志到文件中和记录日志到消息队列中。FileLogger 和 MessageQueueLogger 两个子类复用了父类 Logger 中的 name、enabled、 minPermittedLevel 属性和 log() 方法，但因为这两个子类写日志的方式不同，它们又各自重写了父类中的 doLog() 方法。

```java
// 抽象类
public abstract class Logger {
    private String name;
    private boolean enabled;
    private Level minPermittedLevel;
    
    public Logger(String name, boolean enabled, Level minPermittedLevel) {
        this.name = name;
        this.enabled = enabled;
        this.minPermittedLevel = minPermittedLevel;
	}

    public void log(Level level, String message) {
        boolean loggable = enabled && (minPermittedLevel.intValue() <= level.intVal
        if (!loggable) return;
        doLog(level, message);
	}
    protected abstract void doLog(Level level, String message);
}
                                       
// 抽象类的子类：输出日志到文件
public class FileLogger extends Logger {
    private Writer fileWriter;
    
    public FileLogger(String name, boolean enabled,
    Level minPermittedLevel, String filepath) {
    super(name, enabled, minPermittedLevel);
    this.fileWriter = new FileWriter(filepath);
	}
    @Override
    public void doLog(Level level, String mesage) {
        // 格式化 level 和 message, 输出到日志文件
        fileWriter.write(...);
    }
}
    // 抽象类的子类: 输出日志到消息中间件 (比如 kafka)
    public class MessageQueueLogger extends Logger {
        private MessageQueueClient msgQueueClient;
        public MessageQueueLogger(String name, boolean enabled,
        Level minPermittedLevel, MessageQueueClient msgQueueClient) {
        super(name, enabled, minPermittedLevel);
        this.msgQueueClient = msgQueueClient;
    }
    @Override
    protected void doLog(Level level, String mesage) {
    // 格式化 level 和 message, 输出到消息中间件
    msgQueueClient.send(...);
    }
}
```

抽象类具有哪些特性。我总结了下面三点。

- 抽象类不允许被实例化，只能被继承。也就是说，你不能 new 一个抽象类的对象出来 （Logger logger = new Logger(…); 会报编译错误）。 
- 抽象类可以包含属性和方法。方法既可以包含代码实现（比如 Logger 中的 log() 方 法），也可以不包含代码实现（比如 Logger 中的 doLog() 方法）。不包含代码实现的 方法叫作抽象方法。 
- 子类继承抽象类，必须实现抽象类中的所有抽象方法。对应到例子代码中就是，所有继 承 Logger 抽象类的子类，都必须重写 doLog() 方法。

```java
// 接口
public interface Filter {
	void doFilter(RpcRequest req) throws RpcException;
}
// 接口实现类：鉴权过滤器
public class AuthencationFilter implements Filter {
    @Override
    public void doFilter(RpcRequest req) throws RpcException {
    	//... 鉴权逻辑..
    }
}
// 接口实现类：限流过滤器
public class RateLimitFilter implements Filter {
    @Override
    public void doFilter(RpcRequest req) throws RpcException {
    	//... 限流逻辑...
    }
}
// 过滤器使用 demo
public class Application {
    // filters.add(new AuthencationFilter());
    // filters.add(new RateLimitFilter());
    private List<Filter> filters = new ArrayList<>();
    public void handleRpcRequest(RpcRequest req) {
    	try {
    		for (Filter filter : fitlers) {
    			filter.doFilter(req);
    		}
    	} catch(RpcException e) {
    	// ... 处理过滤结果...
    	}
    // ... 省略其他处理逻辑...
    }
}

```

通过 Java 中的 interface 关键字定 义了一个 Filter 接口。AuthencationFilter 和 RateLimitFilter 是接口的两个实现类，分别 实现了对 RPC 请求鉴权和限流的过滤功能。

接口都有哪些特性。我也总结了三点。

- 接口不能包含属性（也就是成员变量）。 
- 接口只能声明方法，方法不能包含代码实现。
- 类实现接口的时候，必须实现接口中声明的所有方法。

讲了抽象类和接口的定义，以及各自的语法特性。从语法特性上对比，这两者有比 较大的区别，比如抽象类中可以定义属性、方法的实现，而接口中不能定义属性，方法也不 能包含代码实现等等。除了语法特性，从设计的角度，两者也有比较大的区别。

抽象类实际上就是类，只不过是一种特殊的类，这种类不能被实例化为对象，只能被子类继承。相对于抽象类的 is-a 关系来说，接口表示一种 has-a 关系，表示具有某些功能。

## 抽象类和接口能解决什么编程问题？

抽象类不能实例化，只能被继承。继承能解 决代码复用的问题。所以，抽象类也是为代码复用而生的。多个子类可以继承抽象类中定义的属性和方法，避免在子类中，重复编写相同的代码。

```java
// 父类：非抽象类，就是普通的类. 删除了 log(),doLog()，新增了 isLoggable().
public class Logger {
    private String name;
    private boolean enabled;
    private Level minPermittedLevel;
    public Logger(String name, boolean enabled, Level minPermittedLevel) {
    	//... 构造函数不变，代码省略...
    }
    protected boolean isLoggable() {
        boolean loggable = enabled && (minPermittedLevel.intValue() <= level.intVal
        return loggable;
    }
}
                                       
// 子类：输出日志到文件
public class FileLogger extends Logger {
    private Writer fileWriter;
    
    public FileLogger(String name, boolean enabled,
    Level minPermittedLevel, String filepath) {
    	//... 构造函数不变，代码省略...
    }
    public void log(Level level, String mesage) {
        if (!isLoggable()) return;
        // 格式化 level 和 message, 输出到日志文件
        fileWriter.write(...);
    }
}

```

这个设计思路虽然达到了代码复用的目的，但是无法使用多态特性了。像下面这样编写代 码，就会出现编译错误，因为 Logger 中并没有定义 log() 方法。

```java
Logger logger = new FileLogger("access-log", true, Level.WARN, "/users/wangzhen")
logger.log(Level.ERROR, "This is a test log message.");
```

在 Logger 父类中，定义一个空的 log() 方 法，让子类重写父类的 log() 方法，实现自己的记录日志的逻辑，不就可以了吗？这个设计思路能用，但是，它显然没有之前通过抽象类的实现思路优雅。

在 Logger 中定义一个空的方法，会影响代码的可读性。如果我们不熟悉 Logger 背后 的设计思想，代码注释又不怎么给力，我们在阅读 Logger 代码的时候，就可能对为什 么定义一个空的 log() 方法而感到疑惑，需要查看 Logger、FileLogger、 MessageQueueLogger 之间的继承关系，才能弄明白其设计意图。

当创建一个新的子类继承 Logger 父类的时候，我们有可能会忘记重新实现 log() 方法。 之前基于抽象类的设计思路，编译器会强制要求子类重写 log() 方法，否则会报编译错误。

Logger 可以被实例化，换句话说，我们可以 new 一个 Logger 出来，并且调用空的 log() 方法。这也增加了类被误用的风险。当然，这个问题可以通过设置私有的构造函数 的方式来解决。不过，显然没有通过抽象类来的优雅。

**为什么需要接口？它能够解决什么编程问题？**

抽象类更多的是为了代码复用，而接口就更侧重于解耦。接口是对行为的一种抽象，相当于 一组协议或者契约，你可以联想类比一下 API 接口。调用者只需要关注抽象的接口，不需 要了解具体的实现，具体的实现代码对调用者透明。接口实现了约定和实现相分离，可以降低代码间的耦合性，提高代码的可扩展性。

接口是一个比抽象类应用更加广泛、更加重要的知识点。比如，我们经常提到 的“基于接口而非实现编程”，就是一条几乎天天会用到，并且能极大地提高代码的灵活 性、扩展性的设计思想。

## 如何模拟抽象类和接口两个语法概念？

C++ 只有抽象类，并没有接口，那从代码实现 的角度上来说，是不是就无法实现 Filter 的设计思路了呢？ 实际上，我们可以通过抽象类来模拟接口。

接口的定义：接口中没有成员变量，只有方法声明，没有方法实现，实现 接口的类必须实现接口中的所有方法。只要满足这样几点，从设计的角度上来说，我们就可以把它叫作接口。实际上，要满足接口的这些语法特性并不难。在下面这段 C++ 代码中， 我们就用抽象类模拟了一个接口（下面这段代码实际上是策略模式中的一段代码）。

```c++
class Strategy { // 用抽象类模拟接口
    public:
        ~Strategy();
        virtual void algorithm()=0;
    protected:
    	Strategy();
};
```

抽象类 Strategy 没有定义任何属性，并且所有的方法都声明为 virtual 类型（等同于 Java 中的 abstract 关键字），这样，所有的方法都不能有代码实现，并且所有继承这个抽象类 的子类，都要实现这些方法。从语法特性上来看，这个抽象类就相当于一个接口。

## 如何决定该用抽象类还是接口？

判断的标准很简单。如果我们要表示一种 is-a 的关系，并且是为了解决代码复用的问题，我们就用抽象类；如果我们要表示一种 has-a 关系，并且是为了解决抽象而非代码复用的问题，那我们就可以使用接口。

从类的继承层次上来看，抽象类是一种自下而上的设计思路，先有子类的代码重复，然后再 抽象成上层的父类（也就是抽象类）。而接口正好相反，它是一种自上而下的设计思路。我 们在编程的时候，一般都是先设计接口，再去考虑具体的实现。

## 重点回顾

1. 抽象类和接口的语法特性 抽象类不允许被实例化，只能被继承。它可以包含属性和方法。方法既可以包含代码实现， 也可以不包含代码实现。不包含代码实现的方法叫作抽象方法。子类继承抽象类，必须实现 抽象类中的所有抽象方法。接口不能包含属性，只能声明方法，方法不能包含代码实现。类 实现接口的时候，必须实现接口中声明的所有方法。 
2. 抽象类和接口存在的意义 抽象类是对成员变量和方法的抽象，是一种 is-a 关系，是为了解决代码复用问题。接口仅 仅是对方法的抽象，是一种 has-a 关系，表示具有某一组行为特性，是为了解决解耦问 题，隔离接口和具体的实现，提高代码的扩展性。
3. 抽象类和接口的应用场景区别 什么时候该用抽象类？什么时候该用接口？实际上，判断的标准很简单。如果要表示一种 is-a 的关系，并且是为了解决代码复用问题，我们就用抽象类；如果要表示一种 has-a 关 系，并且是为了解决抽象而非代码复用问题，那我们就用接口。

# 理论六：为什么基于接口而非实现编程？有必要为每个类都定义 接口吗？

## 如何解读原则中的“接口”二字？

“基于接口而非实现编程”这条原则中的“接口”，可以理解 为编程语言中的接口或者抽象类。这条原则能非常有效地提高代码质量，之所以这么说，那是因为，应用这条 原则，可以将接口和实现相分离，封装不稳定的实现，暴露稳定的接口。上游系统面向接口 而非实现编程，不依赖不稳定的实现细节，这样当实现发生变化的时候，上游系统的代码基 本上不需要做改动，以此来降低耦合性，提高扩展性。

“基于接口而非实现编程”这条原则的另一个表述方式，是“基于抽象而非实现编 程”。后者的表述方式其实更能体现这条原则的设计初衷。在软件开发中，最大的挑战之一 就是需求的不断变化，这也是考验代码设计好坏的一个标准。越抽象、越顶层、越脱离具体 某一实现的设计，越能提高代码的灵活性，越能应对未来的需求变化。好的代码设计，不仅 能应对当下的需求，而且在将来需求发生变化的时候，仍然能够在不破坏原有代码设计的情 况下灵活应对。而抽象就是提高代码扩展性、灵活性、可维护性最有效的手段之一。

## 如何将这条原则应用到实战中？

```java
public class AliyunImageStore {
 //... 省略属性、构造函数等...
 
 	public void createBucketIfNotExisting(String bucketName) {
     // ... 创建 bucket 代码逻辑...
     // ... 失败会抛出异常..
 }
 
     public String generateAccessToken() {
     // ... 根据 accesskey/secrectkey 等生成 access token
     }

     public String uploadToAliyun(Image image, String bucketName, String accessTok
     //... 上传图片到阿里云...
     //... 返回图片存储在阿里云上的地址 (url）...
     }

     public Image downloadFromAliyun(String url, String accessToken) {
     //... 从阿里云下载图片...
     }
}
    
// AliyunImageStore 类的使用举例
public class ImageProcessingJob {
     private static final String BUCKET_NAME = "ai_images_bucket";
     //... 省略其他无关代码...

     public void process() {
         Image image = ...; // 处理图片，并封装为 Image 对象
         AliyunImageStore imageStore = new AliyunImageStore(/* 省略参数 */);
         imageStore.createBucketIfNotExisting(BUCKET_NAME);
         String accessToken = imageStore.generateAccessToken();
         imagestore.uploadToAliyun(image, BUCKET_NAME, accessToken);
     }
 
}
```

假设我们的系统中有很多涉及图片处理和存储的业务逻辑。图片经过处理之后被上传到阿里 云上。为了代码复用，我们封装了图片存储相关的代码逻辑，提供了一个统一的 AliyunImageStore 类，供整个系统来使用。

不过，软件开发中唯一不变的就是变化。过了一段时间后，我们自建了私有云，不再将图片 存储到阿里云了，而是将图片存储到自建私有云上。为了满足这样一个需求的变化，我们该如何修改代码呢？

我们需要重新设计实现一个存储图片到私有云的 PrivateImageStore 类，并用它替换掉项 目中所有的 AliyunImageStore 类对象。

首先，AliyunImageStore 类中有些函数命名暴露了实现细节。其次，将图片存储到阿里云的流程，跟存储到私有云的流程，可能并不是完全一致的。

解决这个问题的根本方法就是，在编写代码的时候，要遵 从“基于接口而非实现编程”的原则，具体来讲，我们需要做到下面这 3 点。

- 函数的命名不能暴露任何实现细节。比如，前面提到的 uploadToAliyun() 就不符合要 求，应该改为去掉 aliyun 这样的字眼，改为更加抽象的命名方式，比如：upload()。 
- 封装具体的实现细节。比如，跟阿里云相关的特殊上传（或下载）流程不应该暴露给调用者。我们对上传（或下载）流程进行封装，对外提供一个包裹所有上传（或下载）细节的方法，给调用者使用。 
- 为实现类定义抽象的接口。具体的实现类都依赖统一的接口定义，遵从一致的上传功能 协议。使用者依赖接口，而不是具体的实现类来编程。

按照这个思路，把代码重构一下。

```java
public interface ImageStore {
 String upload(Image image, String bucketName);
 Image download(String url);
}
public class AliyunImageStore implements ImageStore {
 //... 省略属性、构造函数等...
 public String upload(Image image, String bucketName) {
 createBucketIfNotExisting(bucketName);
 String accessToken = generateAccessToken();
 //... 上传图片到阿里云...
 //... 返回图片在阿里云上的地址 (url)...
 }
 public Image download(String url) {
 String accessToken = generateAccessToken();
 //... 从阿里云下载图片...
 }
 private void createBucketIfNotExisting(String bucketName) {
 // ... 创建 bucket...
 // ... 失败会抛出异常..
 }
 private String generateAccessToken() {
 // ... 根据 accesskey/secrectkey 等生成 access token
 }
}
// 上传下载流程改变：私有云不需要支持 access token
public class PrivateImageStore implements ImageStore {
 public String upload(Image image, String bucketName) {
 createBucketIfNotExisting(bucketName);
 //... 上传图片到私有云...
 //... 返回图片的 url...
 }
 public Image download(String url) {
 //... 从私有云下载图片...
 }
private void createBucketIfNotExisting(String bucketName) {
// ... 创建 bucket...
// ... 失败会抛出异常..
}
}
// ImageStore 的使用举例
public class ImageProcessingJob {
private static final String BUCKET_NAME = "ai_images_bucket";
//... 省略其他无关代码...
public void process() {
Image image = ...;// 处理图片，并封装为 Image 对象
ImageStore imageStore = new PrivateImageStore(...);
imagestore.upload(image, BUCKET_NAME);
}
}

```

除此之外，很多人在定义接口的时候，希望通过实现类来反推接口的定义。先把实现类写 好，然后看实现类中有哪些方法，照抄到接口定义中。如果按照这种思考方式，就有可能导 致接口定义不够抽象，依赖具体的实现。这样的接口设计就没有意义了。不过，如果你觉得 这种思考方式更加顺畅，那也没问题，只是将实现类的方法搬移到接口定义中的时候，要有 选择性的搬移，不要将跟具体实现相关的方法搬移到接口中，比如 AliyunImageStore 中 的 generateAccessToken() 方法。 

总结一下，我们在做软件开发的时候，一定要有抽象意识、封装意识、接口意识。在定义接 口的时候，不要暴露任何实现细节。接口的定义只表明做什么，而不是怎么做。而且，在设 计接口的时候，我们要多思考一下，这样的接口设计是否足够通用，是否能够做到在替换具 体的接口实现的时候，不需要任何接口定义的改动。

## 是否需要为每个类定义接口？

这条原则的设计初衷是，将接口和实现相分离，封装不稳定的实现，暴露 稳定的接口。上游系统面向接口而非实现编程，不依赖不稳定的实现细节，这样当实现发生 变化的时候，上游系统的代码基本上不需要做改动，以此来降低代码间的耦合性，提高代码 的扩展性。

如果在我们的业务场景中，某个功能只有一种实现方式，未来也不 可能被其他实现方式替换，那我们就没有必要为其设计接口，也没有必要基于接口编程，直 接使用实现类就可以了。

除此之外，越是不稳定的系统，我们越是要在代码的扩展性、维护性上下功夫。相反，如果 某个系统特别稳定，在开发完之后，基本上不需要做维护，那我们就没有必要为其扩展性， 投入不必要的开发时间。

## 重点回顾

1. “基于接口而非实现编程”，这条原则的另一个表述方式，是“基于抽象而非实现编 程”。后者的表述方式其实更能体现这条原则的设计初衷。我们在做软件开发的时候，一定要有抽象意识、封装意识、接口意识。越抽象、越顶层、越脱离具体某一实现的设计，越能 提高代码的灵活性、扩展性、可维护性。 

2. 我们在定义接口的时候，一方面，命名要足够通用，不能包含跟具体实现相关的字眼； 另一方面，与特定实现有关的方法不要定义在接口中。 
3. 基于接口而非实现编程”这条原则，不仅仅可以指导非常细节的编程开发，还能指导更加上层的架构设计、系统设计等。比如，服务端与客户端之间的“接口”设计、类库的“接口”设计。

# 理论七：为何说要多用组合少用继承？如何决定该用组合还是继承？

在面向对象编程中，有一条非常经典的设计原则，那就是：组合优于继承，多用组合少用继承。为什么不推荐使用继承？组合相比继承有哪些优势？如何判断该用组合还是继承？

## 为什么不推荐使用继承？

继承是面向对象的四大特性之一，用来表示类之间的 is-a 关系，可以解决代码复用的问 题。虽然继承有诸多作用，但继承层次过深、过复杂，也会影响到代码的可维护性。

假设我们要设计一个关于鸟的类。我们将“鸟类”这样一个抽象的事物概念，定义为一个抽 象类 AbstractBird。所有更细分的鸟，比如麻雀、鸽子、乌鸦等，都继承这个抽象类。 我们知道，大部分鸟都会飞，那我们可不可以在 AbstractBird 抽象类中，定义一个 fly() 方 法呢？答案是否定的。尽管大部分鸟都会飞，但也有特例，比如鸵鸟就不会飞。鸵鸟继承具 有 fly() 方法的父类，那鸵鸟就具有“飞”这样的行为，这显然不符合我们对现实世界中事 物的认识。当然，你可能会说，我在鸵鸟这个子类中重写（override）fly() 方法，让它抛 出 UnSupportedMethodException 异常不就可以了吗？

这种设计思路虽然可以解决问题，但不够优美。因为除了鸵鸟之外，不会飞的鸟还有很多， 比如企鹅。对于这些不会飞的鸟来说，我们都需要重写 fly() 方法，抛出异常。这样的设 计，一方面，徒增了编码的工作量；另一方面，也违背了我们之后要讲的最小知识原则 （Least Knowledge Principle，也叫最少知识原则或者迪米特法则），暴露不该暴露的接 口给外部，增加了类使用过程中被误用的概率。

你可能又会说，那我们再通过 AbstractBird 类派生出两个更加细分的抽象类：会飞的鸟类 AbstractFlyableBird 和不会飞的鸟类 AbstractUnFlyableBird，让麻雀、乌鸦这些会飞的 鸟都继承 AbstractFlyableBird，让鸵鸟、企鹅这些不会飞的鸟，都继承 AbstractUnFlyableBird 类，不就可以了吗？

继承关系变成了三层。不过，整体上来讲，目前的继承关系还比较简 单，层次比较浅，也算是一种可以接受的设计思路。我们再继续加点难度。在刚刚这个场景 中，我们只关注“鸟会不会飞”，但如果我们还关注“鸟会不会叫”，那这个时候，我们又 该如何设计类之间的继承关系呢？

是否会飞？是否会叫？两个行为搭配起来会产生四种情况：会飞会叫、不会飞会叫、会飞不 会叫、不会飞不会叫。如果我们还需要考虑“是否会下蛋”这样一个行为，那估计就要组合爆炸了。类的继承层次 会越来越深、继承关系会越来越复杂。而这种层次很深、很复杂的继承关系，一方面，会导 致代码的可读性变差。因为我们要搞清楚某个类具有哪些方法、属性，必须阅读父类的代码、父类的父类的代码……一直追溯到最顶层父类的代码。另一方面，这也破坏了类的封装 特性，将父类的实现细节暴露给了子类。子类的实现依赖父类的实现，两者高度耦合，一旦 父类代码修改，就会影响所有子类的逻辑。 

总之，继承最大的问题就在于：继承层次过深、继承关系过于复杂会影响到代码的可读性和 可维护性。这也是为什么我们不推荐使用继承。

## 组合相比继承有哪些优势？ 

实际上，我们可以利用组合（composition）、接口、委托（delegation）三个技术手 段，一块儿来解决刚刚继承存在的问题。 

我们前面讲到接口的时候说过，接口表示具有某种行为特性。针对“会飞”这样一个行为特 性，我们可以定义一个 Flyable 接口，只让会飞的鸟去实现这个接口。对于会叫、会下蛋这 些行为特性，我们可以类似地定义 Tweetable 接口、EggLayable 接口。

```java
public interface Flyable {
void fly();
}
public interface Tweetable {
void tweet();
}
public interface EggLayable {
void layEgg();
}
public class Ostrich implements Tweetable, EggLayable {// 鸵鸟
//... 省略其他属性和方法...
@Override
public void tweet() { //... }
@Override
public void layEgg() { //... }
}
public class Sparrow impelents Flayable, Tweetable, EggLayable {// 麻雀
//... 省略其他属性和方法...
@Override
public void fly() { //... }
@Override
public void tweet() { //... }
@Override
public void layEgg() { //... }
}
```

不过，我们知道，接口只声明方法，不定义实现。也就是说，每个会下蛋的鸟都要实现一遍 layEgg() 方法，并且实现逻辑是一样的，这就会导致代码重复的问题。那这个问题又该如 何解决呢？ 

我们可以针对三个接口再定义三个实现类，它们分别是：实现了 fly() 方法的 FlyAbility 类、实现了 tweet() 方法的 TweetAbility 类、实现了 layEgg() 方法的 EggLayAbility 类。 然后，通过组合和委托技术来消除代码重复。

```java
public interface Flyable {
void fly()；
}
public class FlyAbility implements Flyable {
@Override
public void fly() { //... }
}
// 省略 Tweetable/TweetAbility/EggLayable/EggLayAbility
public class Ostrich implements Tweetable, EggLayable {// 鸵鸟
private TweetAbility tweetAbility = new TweetAbility(); // 组合
private EggLayAbility eggLayAbility = new EggLayAbility(); // 组合
//... 省略其他属性和方法...
@Override
public void tweet() {
tweetAbility.tweet(); // 委托
}
@Override
public void layEgg() {
eggLayAbility.layEgg(); // 委托
}
}

```

继承主要有三个作用：表示 is-a 关系，支持多态特性，代码复用。而这三个作用 都可以通过其他技术手段来达成。比如 is-a 关系，我们可以通过组合和接口的 has-a 关系 来替代；多态特性我们可以利用接口来实现；代码复用我们可以通过组合和委托来实现。所 以，从理论上讲，通过组合、接口、委托三个技术手段，我们完全可以替换掉继承，在项目 中不用或者少用继承关系，特别是一些复杂的继承关系。

## 如何判断该用组合还是继承？

们鼓励多用组合少用继承，但组合也并不是完美的，继承也并非一无是处。从上面的例子来看，继承改写成组合意味着要做更细粒度的类的拆分。这也就意味着，我们要定义更 多的类和接口。类和接口的增多也就或多或少地增加代码的复杂程度和维护成本。所以，在 实际的项目开发中，我们还是要根据具体的情况，来具体选择该用继承还是组合。

如果类之间的继承结构稳定（不会轻易改变），继承层次比较浅（比如，最多有两层继承关 系），继承关系不复杂，我们就可以大胆地使用继承。反之，系统越不稳定，继承层次很 深，继承关系复杂，我们就尽量使用组合来替代继承。

除此之外，还有一些设计模式会固定使用继承或者组合。比如，装饰者模式（decorator pattern）、策略模式（strategy pattern）、组合模式（composite pattern）等都使用了 组合关系，而模板模式（template pattern）使用了继承关系。

仅仅为了代码复用，生硬地抽象出一个父类出来，会影响到代码的可读性。还有一些特殊的场景要求我们必须使用继承。如果你不能改变一个函数的入参类型，而入参 又非接口，为了支持多态，只能采用继承来实现。比如下面这样一段代码，其中 FeignClient 是一个外部类，我们没有权限去修改这部分代码，但是我们希望能重写这个类 在运行时执行的 encode() 函数。这个时候，我们只能采用继承来实现了。

## 重点回顾

1. 为什么不推荐使用继承？ 继承是面向对象的四大特性之一，用来表示类之间的 is-a 关系，可以解决代码复用的问 题。虽然继承有诸多作用，但继承层次过深、过复杂，也会影响到代码的可维护性。在这种 情况下，我们应该尽量少用，甚至不用继承。 
2. 组合相比继承有哪些优势？ 继承主要有三个作用：表示 is-a 关系，支持多态特性，代码复用。而这三个作用都可以通 过组合、接口、委托三个技术手段来达成。除此之外，利用组合还能解决层次过深、过复杂 的继承关系影响代码可维护性的问题。 
3. 如何判断该用组合还是继承？ 尽管我们鼓励多用组合少用继承，但组合也并不是完美的，继承也并非一无是处。在实际的 项目开发中，我们还是要根据具体的情况，来选择该用继承还是组合。如果类之间的继承结 构稳定，层次比较浅，关系不复杂，我们就可以大胆地使用继承。反之，我们就尽量使用组 合来替代继承。除此之外，还有一些设计模式、特殊的应用场景，会固定使用继承或者组 合。