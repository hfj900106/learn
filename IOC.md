解决对象之间的耦合度过高的问题



package test;

import org.springframework.beans.tests.Person;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.FileSystemXmlApplicationContext;

public class Test {
    public static void main(String[] args) throws ClassNotFoundException {
        ApplicationContext ctx = new FileSystemXmlApplicationContext
                ("spring-beans/src/test/resources/beans.xml");
        System.out.println("number : " + ctx.getBeanDefinitionCount());
        ((Person) ctx.getBean("person")).work();
    }
}


ApplicatContext 的标准实现是 FileSystemXmlApplicationContext。
该类的构造方法中包含了容器的启动，IOC的初始化。


#### 3. 从 FileSystemXmlApplicationContext 开始剖析

从这里开始，我们即将进入复杂的源码。

我们进入 FileSystemXmlApplicationContext 的构造方法：

![](http://upload-images.jianshu.io/upload_images/4236553-68d0fed37d45af94.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到该构造方法被重载了，可以传递 configLocation 数组，也就是说，可以传递过个配置文件的地址。默认刷新为true，parent 容器为null。进入另一个构造器：

![](http://upload-images.jianshu.io/upload_images/4236553-b61ee400629edb3d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

该构造器做了2件事情，一是设置配置文件，二是刷新容器，refresh 方法才是初始化容器的重要方法。该方法是 FileSystemXmlApplicationContext  的父类 AbstractApplicationContext 的方法。
 AbstractApplicationContext.refresh() 方法实现：

/**
     *
     * 1. 构建BeanFactory，以便产生所需要的bean定义实例
     * 2. 注册可能感兴趣的事件
     * 3. 创建bean 实例对象
     * 4. 触发被监听的事件
     *
     */
    @Override
    public void refresh() throws BeansException, IllegalStateException {
        synchronized (this.startupShutdownMonitor) {
            // 为刷新准备应用上下文
            prepareRefresh();
            // 告诉子类刷新内部bean工厂，即在子类中启动refreshBeanFactory()的地方----创建bean工厂，根据配置文件生成bean定义
            ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
            // 在这个上下文中使用bean工厂
            prepareBeanFactory(beanFactory);
            try {
                // 设置BeanFactory的后置处理器
                postProcessBeanFactory(beanFactory);
                // 调用BeanFactory的后处理器，这些后处理器是在Bean定义中向容器注册的
                invokeBeanFactoryPostProcessors(beanFactory);
                // 注册Bean的后处理器，在Bean创建过程中调用
                registerBeanPostProcessors(beanFactory);
                //对上下文的消息源进行初始化
                initMessageSource();
                // 初始化上下文中的事件机制
                initApplicationEventMulticaster();
                // 初始化其他的特殊Bean
                onRefresh();
                // 检查监听Bean并且将这些Bean向容器注册
                registerListeners();
                // 实例化所有的（non-lazy-init）单件
                finishBeanFactoryInitialization(beanFactory);
                //  发布容器事件，结束refresh过程
                finishRefresh();
            }
            catch (BeansException ex) {
                if (logger.isWarnEnabled()) {
                    logger.warn("Exception encountered during context initialization - " +
                            "cancelling refresh attempt: " + ex);
                }
                // 为防止bean资源占用，在异常处理中，销毁已经在前面过程中生成的单件bean
                destroyBeans();
                // 重置“active”标志
                cancelRefresh(ex);
                throw ex;
            }
            finally {
                // Reset common introspection caches in Spring's core, since we
                // might not ever need metadata for singleton beans anymore...
                resetCommonCaches();
            }
        }
    }
