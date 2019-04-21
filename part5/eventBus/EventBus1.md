## 零、Android消息机制
回顾一下Android消息机制，消息机制中有Handler、Message、MessageQueue、Looper。
下面的网址是之前总结的消息机制：
https://blog.csdn.net/J675620982/article/details/81412918

## 一、EventBus中的概念讲解
### 1、TheadMode线程模式
在订阅者中，每个方法都有一个线程模式，用于确认EventBus调用该方法时执行的线程。在EventBus 3.1.1版本中，共有5个线程模式
1. **POSTING**
该模式下，发布者在哪个线程发布消息，订阅者就在哪个线程调用该订阅方法，避免的线程的切换，符合哪些时间极短的任务。同时也要快速处理完，因为运行的可能是在主线程中。

2. **main**
在Android上，将在UI线程中调用订阅者，如果发布线程是主线程，则直接调用订阅者方法，阻止发布线程，否则事件排队等待传递。使用此模式要避免造成ANR

3. **MAIN_ORDERED**
在UI线程中调用订阅者，该事件将始终排队等待传递。保证调用是非阻塞的。

4. **BACKGROUND**
将在后台线程中调用订阅者。如果发布线程不是主线程，则将在发布线程中直接调用订阅者方法。如果发布线程是主线程，则EventBus使用单个后台线程，该线程将按顺序传递其所有事件。使用此模式的订阅者应尝试快速返回以避免阻止后台线程。

5. **ASYNC**
在Android上，将在后台线程中调用订阅者。如果发布线程不是主线程，则将在发布线程中直接调用订阅者方法。如果发布线程是主线程，则EventBus使用单个后台线程，该线程将按顺序传递其所有事件。使用此模式的订阅者应尝试快速返回以避免阻止后台线程。

### 2、Subscribe注解
```java
@Documented         //注解
@Retention(RetentionPolicy.RUNTIME)     //作用时间：运行时
@Target({ElementType.METHOD})           //作用范围：method
public @interface Subscribe {    
    ThreadMode threadMode() default ThreadMode.POSTING;             //线程模式
    boolean sticky() default false;       //粘性事件
    int priority() default 0;       //优先级
}
```

### 3、SubscriberMethod订阅者方法
由EventBus内部使用和生成的订阅者索引。
主要包含如下成员：
final Method method：方法
final ThreadMode threadMode：方法模式
final Class<?> eventType：事件类型
final int priority：优先级
final boolean sticky：是否粘性
String methodString：方法名string。用于比较

### 4、Subscription
封装了订阅者和SubscriberMethod，主要有如下成员
Object subscriber：订阅者
SubscriberMethod subscriberMethdod：订阅者方法的相关信息
boolean active：是否注册

### 5、PostingThreadState发送事件线程状态的封装类
包含是否主线程、subscription、事件等
· List<Object> eventQueue：事件队列
· isPosting：是否发送中
· isMainThread：是否主线程
· subscription：订阅者和对应subscriberMethod
· Object event：事件
· boolean canceled：取消


## 二、getDefault:获取EventBus单例
过程getDefault -> EventBus() -> EventBus(Default_Builder)
在EventBus(Default_Builder)方法中，主要有如下操作：
1. builder.getLogger获得logger

2. 创建三个键值对
· subscriptionsByEventType：根据事件类型保存subscriptions
· typesBySubscriber：根据订阅者获取事件类型
· stickyEvents：粘性事件

3. 创建三个发布渠道poster：
· mainThreadPoster：主线程的发布者，对应ThreadMode.Main
· backgroundPoster：后台线程的poster，需要处理完所有的pendingPoster，对应ThreadMode.background
· asyncPoster：单独线程的poster，取单个pendingPoster，对应TheadMode.Async

4. 订阅方法寻找器
subscriberMethodFinder = new SubscriberMehodFinder(...)

5. 各种标志位
logSubscriberExceptions：订阅者错误
logNoSubscriberMessages：没有订阅着
sendSubscriberExceptionEvent：发送订阅事件错误
sendNoSubscriberEvent：发送没有订阅者事件
throwSubscriberException：抛出订阅错误
eventInheritance：发送事件是否处理父类、子类的继承关系

6. 线程池executorService

## 三、register(Object subscriber)注册
根据所给的订阅者注册事件，一旦不需要接收事件，需要调用unregister()方法。
所给定的订阅者中必须包含@Subscribe注解所标注的方法，同时方法中需要配置ThreadMode线程模式和priority优先级。

流程：register(Object subscriber) -> subscriber.getClass() -> subscriberMethodFinder.findSubscriberMethods(subscriberClass) -> subscribe(subscriber, subscriberMethod)。
1. **通过反射获取订阅者的对象**
Class<?> subscriberClass = subscriber.getClass()

2. **通过订阅者对象找到该订阅者下的订阅方法集合**
List<SubscriberMethod> subscriberMethods = 
subscriberMethodFinder.findSubscriberMethods(subscriberClass);

3. 在同步代码块中给每个方法做**订阅**
```java
synchronized (this) {   
    for (SubscriberMethod subscriberMethod : subscriberMethods) {                       
        subscribe(subscriber, subscriberMethod);    
    }
}
```

### 1、寻找订阅方法集合
#### SubscriberMethod.findSubscriberMethod(Class<?> subscriberClass)

该方法主要是寻找某一订阅者中的订阅方法。
1. 如果在METHOD.CACHE，方法缓存池中找到该订阅者方法，则返回

2. 判断是否忽略生成的索引，默认为false。如果为true，则调用findUsingReflection(subscriberClass)方法获取，否则调用findUsingInfo(subscriberClass)方法来获取。
3. 如果订阅者方法subscriberMethods为空，则抛出异常，否则添加到方法缓存池METHOD.CACHE中，并返回。

#### FindState（也就是FindInfo）
用来保存已经注解过的方法及其状态，主要包括如下成员：
· List<SubscriberMethod> subscriberMethods：保存所有的订阅方法
· Map<Class, Object> anyMethodByEventType：根据事件类型（key）找到订阅方法（value）
· Map<String, Class> subscriberClassByMethodKey：根据订阅方法（key）找到订阅者（value），其中methodKey代表的是具体的订阅方法
· StringBuilder methodKeyBuilder：构建methodKey

· SubscriberInfo：订阅者信息

#### findUsingInfo(Class<?> subscriberClass)
该方法用于获取订阅方法集合。
流程如下：
1. 获取findStare
2. 初始化findState中的订阅者
3. 循环中获取订阅者信息，当订阅者信息不为空时，获取其订阅方法集合，若该订阅方法符合条件则添加到新的findState的subscriberMethods中；当订阅者信息为空时，则调用findUsingReflectionInSingleClass方法，来判断哪些是订阅者订阅好的方法和事件。
4. 遍历父类
5. 返回和释放资源

### 2、为每个方法做订阅
1. 判断是否有订阅该事件，已经订阅则抛出异常
2. 按照优先级加入到subscriptionsByEventType的中
3. 再添加到typesBySubscriber中
4. 是否为粘性事件，是否需要考虑父类、子类的继承关系，然后调用checkPostStickyEventToSubscription进行事件分发


## 四、post(Object event)发送
发送事件到EventBus总线中，流程如下：
1. 获取postingThreadState和事件队列
2. 判断是否在发送，若是则退出，否则判断是否在主线程、然后循环调用postSingleEvent方法，发送事件
3. 还原postingState中的发送状态等。

### 1. postSingleEvent()方法
1. 判断是否有继承关系，调用postSingleEventForEventType
2. 如果没有找到订阅者，那么就调用NoSubscriberEvent()，发送没有该订阅者事件

### 2. postSingleEventForEventType()方法
主要是调用postToSubscription，进行事件的发送，随后清空postingThreadState中的状态

### 3. postToSubscription()方法
switch-case 判断线程模式threadMode，并调用订阅者中的订阅方法，执行事件。