


=======================================
Spring AOP
=======================================


Spring AOP
=========================================
Spring AOP可以有如下几种实现形式：

1.经典的基于代理的AOP；
2.@AspectJ注解驱动的切面；
3.纯POJO切面；
4.注入式AspectJ切面。


在base这个项目中，我使用aspectj进行aop代码的插入，这个配置起来比较方便，而且效率也很高。

配置文件：

:: 

    <!--启动Spring对@AspectJ注解的支持 -->
    <aop:aspectj-autoproxy/>

这样就开启spring的aspectj的功能，我们就可以使用代码实现AOP编程了。

代码实例：

::

    @Component
    @Aspect
    public class NovaAspectAdvice {
        @Before(value ="execution(* com.neunn.cloud.*.*(..))")
        public void doBefore(JoinPoint jp) {
            
        }
        @After(value ="execution(* com.neunn.cloud.*.*(..))")
        public void doAfter(JoinPoint jp) {
            
        }
        @AfterReturning(value = "execution(* com.neunn.*.create*(..))", returning = "result")
        public void doAfter(JoinPoint jp, Object result) {
            
        }
        @Around(value = "execution(* com.neunn.*.create*(..))")
        public void doAround(ProceedingJoinPoint pjp) throws Throwable {

        }
        @AfterThrowing(value = "execution(* com.neunn.*.create*(..))", throwing = "e")
        public void doThrow(JoinPoint jp, Throwable e) {

        }
    }

这里简单解释下，aspectj只是其中一种实现方法，包括五种方法，before，after，afterreturn，around，afterthrow分别作用在截获方法的开始，之后，返回后，整个执行过程，抛出异常后。
网上有个参考资料把after return中的参数写成了String类型，导致我开始运行的时候怎么也截获不到AfterReturning方法之内，差点就换其他方式进行截获了。

这里around方式没有执行成功，返回的对象jersyclient解析不了，暂时还未解决这个问题。


最佳实践
=======================================
Spring Boot配置ehcache
---------------------------------------
在开发中有一条token需要集群中各个点进行缓存共享，这里需要使用ehcache的远程对象调用（或者广播）等方式进行配置。
在开发中发现ehcache怎么也不生效，原本以为是applicationContext搞的鬼，比如没有进行annotation的配置。

之后我又重新搭建了一套工程，使用springboot进行配置，但不是web工程，后来查看资料，普通工程和web工程的上下文使用
的是不同的类，这是由Spring Boot自动识别并配置的，也可以通过手动setApplicatonContex或者setWebEnviroment方式进行
配置，但这样配置后，自己启动一会就莫名退出了，原因待查。

通过将好用的配置不断进行对比，关键的方式是在与缓存的调用方式：

.. code:: java

    @Service
    public class SemanticService{

        @Cacheable("tokenCache")
        public String getToken(){
            ......
        }
        public void searchIndex(){
            String token = this.getToken();//在自己的其他方法调用getToken方法
            ......
        }
    }
    //上层类方法
    @Autowired
    private SemanticService semanticService;
    public void semanticService(){
        String token = semanticService.getToken();//在上层调用者上调用getToken方法
        ...
    }

那么上述两种方法都能走cache吗？

回答是前者不能调用cache，后者可以。其实大家都很熟悉Spring的调用栈，并不是像我们平时new一个对象然后进行调用，
而是生成一个动态代理对象，通过反射的方式进行调用，如果这期间方法签名上有任何AOP的配置，在反射调用前会检查
Annotation的配置，然后进行Annotation的调用，再执行该方法。但如果你在自己的方法中调用getToken，这样就跳过了
Spring的AOP，就相当于我们自己简单调用了自己一下，跳过AOP就会导致Cache的注解失效了。


