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

**示例**

```
package com.oneconnect.sg.service.impl;

@Service
public class UserManagerServiceImpl implements UserManagerService {

    @Override
    @Transactional(rollbackFor = Exception.class)
    public Long addUser(User user, Boolean reviewFlag) {
        //do something
    }

 	@Override
    public void updateUser(User user) {
    	// do something
    	addUser(user,true);
    }
}
```

**切面类**

```
@Aspect
@Component
public class ValidatorAOP {
    @Pointcut("execution(* com.oneconnect.sg.service..*(..)) and 			   	 @annotation(org.springframework.stereotype.Service)")
    public void controllerMethodPointcut() {
    
    }

    @Around("controllerMethodPointcut()")
    public Object Interceptor(ProceedingJoinPoint pjp) throws Throwable {
       //do something
    }

}
```


**结果**
开发中可能会遇到如上所示的业务代码

我们在updateUser方法中直接调用了addUser方法,此时addUser的切面以及事务都不会生效(单指addUser上的事务及切面,假设updateUser没有事务,实际开发中可能会存在这种情况)

原因是调用addUser是通过this调用的,而不是通过Spring管理的Bean来调用,也就是调用的不是代理之后的方法

**解决方法**
通过注入的Bean来调用方法,可根据具体业务看是否需要采用这种方法

这样事务和切面都会生效

```
public class UserManagerServiceImpl implements UserManagerService {

    @Autowired
    @Lazy
    private UserManagerService userManagerService;

    @Override
    @Transactional(rollbackFor = Exception.class)
    public Long addUser(User user, Boolean reviewFlag) {
        //do something
    }

 	@Override
    public void updateUser(User user) {
    	// do something
    	userManagerService.addUser(user,true);
    }
}
```


原文链接：https://blog.csdn.net/zhuchencn/article/details/104818356