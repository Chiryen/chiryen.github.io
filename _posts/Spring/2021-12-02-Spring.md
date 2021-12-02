# Spring

## IOC和DI的区别

IOC的依赖处理实现方式是包含DI（依赖注入）和依赖查找两种方式。

依赖查找：主动获取、相对繁琐、侵入业务逻辑，可读性好

示例

```
beanFactory.getBean("beanName")
```

依赖注入：被动提供、相对便利、低入侵，可读性一般

示例

```
@Autowired
```



## Spring有什么优势

IOC管理，依赖查找和依赖注入

AOP

事务抽象

事件机制

SPI扩展

强大的第三方整合

易测试性