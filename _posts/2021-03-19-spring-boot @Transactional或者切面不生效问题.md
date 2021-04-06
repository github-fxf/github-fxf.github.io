---
layout:     post
title:      spring-boot @Transactional或者切面不生效问题记录
subtitle:   
date:       2021-03-19
author:     fxf
header-img: img/post-web.jpg
catalog: true
tags:
    - spring-boot
---

Spring的声明式事务和切面都是通过aop进行动态代理实现的
所以直接通过this来调用方法的话,将不会触发事务和切面

起因
考虑如下一个例子:
```
@Target(value = {ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface MyMonitor {
}
@Component
@Aspect
public class MyAopAdviseDefine {
    private Logger logger = LoggerFactory.getLogger(getClass());
 
    @Pointcut("@annotation(com.xys.demo4.MyMonitor)")
    public void pointcut() {
    }
 
    // 定义 advise
    @Before("pointcut()")
    public void logMethodInvokeParam(JoinPoint joinPoint) {
        logger.info("---Before method {} invoke, param: {}---", joinPoint.getSignature().toShortString(), joinPoint.getArgs());
    }
}
@Service
public class SomeService {
    private Logger logger = LoggerFactory.getLogger(getClass());
 
    public void hello(String someParam) {
        logger.info("---SomeService: hello invoked, param: {}---", someParam);
        test();
    }
 
    @MyMonitor
    public void test() {
        logger.info("---SomeService: test invoked---");
    }
}
@EnableAspectJAutoProxy(proxyTargetClass = true)
@SpringBootAppliMyion
public class MyAopDemo {
    @Autowired
    SomeService someService;
 
    public static void main(String[] args) {
        SpringAppliMyion.run(MyAopDemo.class, args);
    }
 
    @PostConstruct
    public void aopTest() {
        someService.hello("abc");
    }
}
```
在这个例子中, 我们定义了一个注解 MyMonitor, 这个是一个方法注解, 我们的期望是当有此注解的方法被调用时, 需要执行指定的切面逻辑, 即执行 MyAopAdviseDefine.logMethodInvokeParam 方法.

在 SomeService 类中, 方法 test() 被 MyMonitor 所注解, 因此调用 test() 方法时, 应该会触发 logMethodInvokeParam 方法的调用. 不过有一点我们需要注意到, 我们在 MyAopDemo 测试例子中, 并没有直接调用 SomeService.test() 方法, 而是调用了 SomeService.hello() 方法, 在 hello 方法中, 调用了同一个类内部的 SomeService.test() 方法. 按理说, test() 方法被调用时, 会触发 AOP 逻辑, 但是在这个例子中, 我们并没有如愿地看到 MyAopAdviseDefine.logMethodInvokeParam 方法的调用, 这是为什么呢?

这是由于 Spring AOP (包括动态代理和 CGLIB 的 AOP) 的限制导致的. Spring AOP 并不是扩展了一个类(目标对象), 而是使用了一个代理对象来包装目标对象, 并拦截目标对象的方法调用. 这样的实现带来的影响是: 在目标对象中调用自己类内部实现的方法时, 这些调用并不会转发到代理对象中, 甚至代理对象都不知道有此调用的存在.

即考虑到上面的代码中, 我们在 MyAopDemo.aopTest() 中, 调用了 someService.hello("abc"), 这里的 someService bean 其实是 Spring AOP 所自动实例化的一个代理对象, 当调用 hello() 方法时, 先进入到此代理对象的同名方法中, 然后在代理对象中执行 AOP 逻辑(因为 hello 方法并没有注入 AOP 横切逻辑, 因此调用它不会有额外的事情发生), 当代理对象中执行完毕横切逻辑后, 才将调用请求转发到目标对象的 hello() 方法上. 因此当代码执行到 hello() 方法内部时, 此时的 this 其实就不是代理对象了, 而是目标对象, 因此再调用 SomeService.test() 自然就没有 AOP 效果了.

简单来说, 在 MyAopDemo 中所看到的 someService 这个 bean 和在 SomeService.hello() 方法内部上下文中的 this 其实代表的不是同一个对象(可以通过分别打印两者的 hashCode 以验证), 前者是 Spring AOP 所生成的代理对象, 而后者才是真正的目标对象(SomeService 实例).

解决
弄懂了上面的分析, 那么解决这个问题就十分简单了. 既然 test() 方法调用没有触发 AOP 逻辑的原因是因为我们以目标对象的身份(target object) 来调用的, 那么解决的关键自然就是以代理对象(proxied object)的身份来调用 test() 方法.
因此针对于上面的例子, 我们进行如下修改即可:

```
@Service
public class SomeService {
    private Logger logger = LoggerFactory.getLogger(getClass());
 
    @Autowired
    private SomeService self;
 
    public void hello(String someParam) {
        logger.info("---SomeService: hello invoked, param: {}---", someParam);
        self.test();
    }
 
    @CatMonitor
    public void test() {
        logger.info("---SomeService: test invoked---");
    }
}
```
上面展示的代码中, 我们使用了一种很 subtle 的方式, 即将 SomeService bean 注入到 self 字段中(这里再次强调的是, SomeService bean 实际上是一个代理对象, 它和 this 引用所指向的对象并不是同一个对象), 因此我们在 hello 方法调用中, 使用 self.test() 的方式来调用 test() 方法, 这样就会触发 AOP 逻辑了.

Spring AOP 导致的 @Transactional 不生效的问题
这个问题同样地会影响到 @Transactional 注解的使用, 因为 @Transactional 注解本质上也是由 AOP 所实现的.

例如我在 stackoverflow 上看到的一个类似的问题: Spring @Transaction method call by the method within the same class, does not work?
这里也记录下来以作参考.

那个哥们遇到的问题如下:
```
public class UserService {
 
    @Transactional
    public boolean addUser(String userName, String password) {
        try {
            // call DAO layer and adds to database.
        } catch (Throwable e) {
            TransactionAspectSupport.currentTransactionStatus()
                    .setRollbackOnly();
 
        }
    }
 
    public boolean addUsers(List<User> users) {
        for (User user : users) {
            addUser(user.getUserName, user.getPassword);
        }
    } 
}
```
他在 addUser 方法上使用 @Transactional 来使用事务功能, 然后他在外部服务中, 通过调用 addUsers 方法批量添加用户. 经过了上面的分析后, 现在我们就可知道其实这里添加注解是不会启动事务功能的, 因为 AOP 逻辑整个都没生效嘛.

解决这个问题的方法有两个, 一个是使用 AspectJ 模式的事务实现:

<tx:annotation-driven mode="aspectj"/>
另一个就是和我们刚才在上面的例子中的解决方式一样:
```
public class UserService {
    private UserService self;
 
    public void setSelf(UserService self) {
        this.self = self;
    }
 
    @Transactional
    public boolean addUser(String userName, String password) {
        try {
        // call DAO layer and adds to database.
        } catch (Throwable e) {
            TransactionAspectSupport.currentTransactionStatus()
                .setRollbackOnly();
 
        }
    }
 
    public boolean addUsers(List<User> users) {
        for (User user : users) {
            self.addUser(user.getUserName, user.getPassword);
        }
    } 
}
```
