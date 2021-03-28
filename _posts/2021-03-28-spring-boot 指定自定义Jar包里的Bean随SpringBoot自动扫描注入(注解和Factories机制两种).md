---
layout:     post
title:      spring-boot 指定自定义Jar包里的Bean随SpringBoot自动扫描注入(注解和Factories机制两种)
subtitle:   
date:       2021-03-19
author:     fxf
header-img: img/post-web.jpg
catalog: true
tags:
    - spring-boot
---

当自定义一些Jar包提供给别人使用时，别人的SpringBoot添加maven jar包依赖启动后却无法注入我们的Bean。比如我在Jar包里加了一个切面代理，却没有被注入到SpringBoot中。

这里介绍两种办法：

1. 第一种是通过自定义注解的形式把Bean注入：

	- 自定义注解

		```
		@Retention(RetentionPolicy.RUNTIME)
		@Target(ElementType.TYPE)
		@Documented
		@Import({IBasePointAspectConfig.class})
		@Deprecated
		public @interface EnableIbasePointAspect {
		}
		```

		

	- 扫描注入bean

		```
		@Configuration
		@ComponentScan(basePackages = "com.test") //你需要注入的Bean所在的包
		@Deprecated
		public class IBasePointAspectConfig {
		}
		```

		

	- SpringBoot启动类上添加自定义注解

		```
		@SpringBootApplication
		@EnableIbasePointAspect
		 public class DemoApplication {
		    public static void main(String[] args) {
		        SpringApplication.run(DemoApplication.class, args);
		    }
		}
		```

这种写法的好处在于别人可以灵活的选择注入或者不注入你的Bean;缺点就是想注入得在启动类上加注解，哈哈哈...

2、Factories机制注入Bean
       直接在你自定义Jar包所在的SpringBoot项目的 resources----> META-INF下新增 spring.factories 文件，内容：

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.test.ActuatorConfig
```

其中com.test.ActuatorConfig  是一个具体的类
在ActuatorConfig类中扫描注入Bean

```
@ComponentScan(basePackages = {"com.test", "com.hiber"}) //你需要注入的Bean所在的包,是个数组可以多个
public class ActuatorConfig {
}
```


这样，别人在SpringBoot中添加你的Maven依赖后启动时就可以把切面代理之类的注入到他项目中去了。

至于底层机制自行查资料吧.~~~(*^_^*)
————————————————
版权声明：本文为CSDN博主「北京--小乌龟」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u012246511/article/details/107468783