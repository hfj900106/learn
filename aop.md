# learn
read and write
## 事务
### 问题：事务失效
### 场景：
对象O的方法A（没有加事务）调用加方法B（加了事务），结果B方法内出现异常时事务没有回滚
### 解决办法：
1、springboot中默认spring.aop.auto=true不过为了保险可以在启动类加上@EnableAspectJAutoProxy(exposeProxy=true)，这个注解开启AOP代理自动配置，使用cglib进行代理对象的生成
2、方法A中获取对象O的代理对象，用代理对象去调用B方法  
eg:((UserServiceImpl)AopContext.currentProxy()).saveUser(userSave);

##### 代理的方式：jdk动态代理、CGlib动态代理

##### Spring如何选择用JDK还是CGLiB？  
1）当Bean实现接口时，Spring就会用JDK的动态代理。  
2）当Bean没有实现接口时，Spring使用CGlib是实现。  
3）可以强制使用CGlib（在spring配置中加入<aop:aspectj-autoproxy proxy-target-class="true"/>）。  

