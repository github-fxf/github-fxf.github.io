---
layout:     post
title:      spring-boot自定义注解，解析自定义注解的属性值，配置文件的参数值
subtitle:   
date:       2021-03-14
author:     fxf
header-img: img/post-web.jpg
catalog: true
tags:
    - spring-boot
---

# spring-boot自定义注解，解析自定义注解的属性值，配置文件的参数值

## 1.依赖

```
`<dependency>`
   `<groupId>org.springframework.boot</groupId>`
   `<artifactId>spring-boot-starter-aop</artifactId>`
`</dependency>
```

## 2.配置文件

```
test.key=testValue
```

## 3.注解

```
package com.qin.annotation;

import java.lang.annotation.*;

@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface AuthDemo {
    String value() default "";
}
```

## 4.切面AOP

```
package com.qin.aspectj;
import com.qin.annotation.AuthDemo;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.context.EnvironmentAware;
import org.springframework.core.env.Environment;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class AuthDemoAspect implements EnvironmentAware {
    Environment environment;

    @Override
    public void setEnvironment(Environment environment) {
        this.environment = environment;
    }

    @Pointcut("@annotation(com.qin.annotation.AuthDemo)")
    public void myPointCut() {

    }

    @Before(value = "myPointCut()")
    public void check(){
        System.out.println("check");
    }

    @After(value = "myPointCut()")
    public void bye(){
        System.out.println("bye");
    }

    /**
     *配置文件配置
     *test.key=testValue
     */
    @Around("myPointCut() && @annotation(authDemo)")
    public void around(ProceedingJoinPoint joinPoint, AuthDemo authDemo){
        //解析配置文件参数代码:${test.key}
        String s = environment.resolvePlaceholders(authDemo.value());
        System.out.println("未解析的值:"+authDemo.value());//${test.key}
        System.out.println("解析后的值:"+ s);//testValue
        //目标方法返回对象
        Object proceed = null;
        try {
             //执行目标方法:可修改目标方法的参数
             proceed = joinPoint.proceed(new Object[]{s});
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
        return ;
    }

}
```

## 5.测试

```
package com.qin.controller.MyResolveController;

import com.qin.annotation.AuthDemo;
import org.springframework.web.bind.annotation.*;

/**
 *@author WZB
 *@date 2020/03/18
 *@desciption 注解属性获取配置文件参数测试
 *
 */
@RestController
@RequestMapping(value = "/api/")
public class MyResolveController{

    @PostMapping(value = "/test")
    @AuthDemo(value = "${test.key}")
    public CommonResponse test(String param) {
        CommonResponse response=new CommonResponse();
        System.out.println(param);
        return response;
    }

}
```


原文链接：https://blog.csdn.net/qq_42513284/article/details/109034771