# What is AOP
面向对象编程解决了业务模块的封装复用的问题，但是对于某些模块，其本身并不独属于摸个业务模块，而是根据不同的情况，贯穿于某几个或全部的模块之间的。例如登录验证，其只开放几个可以不用登录的接口给用户使用（一般登录使用拦截器实现，但是其切面思想是一致的）；再比如性能统计，其需要记录每个业务模块的调用，并且监控器调用时间。可以看到，这些横贯于每个业务模块的模块，如果使用面向对象的方式，那么就需要在已封装的每个模块中添加相应的重复代码，对于这种情况，面向切面编程就可以派上用场了。

面向切面编程，指的是将一定的切面逻辑按照一定的方式编织到指定的业务模块中，从而将这些业务模块的调用包裹起来

# Element in AOP
 
AOP： consist of pointcut and advice。由切点和通知组成，一般单独成一个类。Pointcut和Advice共同定义了关于切面的全部内容，它是什么时候，在何时和何处完成功能。

1. join point(连接点) : 所有我们能够将通知应用到的地方都是连接点，在Spring中，我们可以认为连接点就是所有的方法（除构造函数外），连接点没有什么实际意义，这个概念的提出只是为了更好的说明切点。【这表示你的应用程序中可以插入AOP方面的一点。也可以说，这是应用程序中使用Spring AOP框架采取操作的实际位置。】
2. advice（通知）: 就是我们想要额外实现的功能，在上面的例子中，实现了性能监控的方法就是通知。【这是在方法执行之前或之后采取的实际操作。 这是在Spring AOP框架的程序执行期间调用的实际代码片段。】
3. pointcut（切点）: 在连接点的基础上，来定义切入点，比如在我们上面的场景中，要对controller层的所有接口完成性能监控，那么就是说所有controller中的方法就是我们的切点（service层，dao层的就是普通的连接点，没有什么作用）。【这是一组一个或多个切入点，在切点应该执行Advice。 您可以使用表达式或模式指定切入点。】
4. introduction（引入）: 我们可以让代理类实现目标类没有实现的额外的接口以及持有新的字段。【引用允许我们向现有的类添加新的方法或者属性】
5. target object(目标对象): 引入中所提到的目标类，也就是要被通知的对象，也就是真正的业务逻辑，他可以在毫不知情的情况下，被织入切面，而自己专注于业务本身的逻辑。
6. proxy object(代理对象): 将切面织入目标对象后所得到的就是代理对象。代理对象是正在具备通知所定义的功能，并且被引入了的对象。
7. weaving(织入): 把切面应用到目标对象来创建新的代理对象的过程。切面的织入有三种方式
  + 编译时织入
  + 类加载时期织入
  + 运行时织入
我们通常使用的SpringAOP都是运行时期织入，另外Spring中也提供了一个Load Time Weaving
（LTW，加载时期织入）的功能，此功能使用较少，有兴趣的同学可以参考一下两个链接：
![image](https://user-images.githubusercontent.com/11779503/216490400-c95f472f-acdb-4dc4-af90-763ae5ffaa3f.png)

https://www.cnblogs.com/wade-luffy/p/6073702.html
https://docs.spring.io/spring/docs/5.1.14.BUILD-SNAPSHOT/spring-framework-reference/core.html#aop-aj-ltw


## Advice
### type of advice
+ before: 方法调用前被增强
+ after： 方法被调用后被增强
+ after-returning: 方法被成功执行之后被增强
+ after-throwing：在方法抛出指定异常之后执行
+ around： 在方法调用前后执行

# How to use

1. first step, enable aspect by using @Aspect in class

***Note: @Aspect必须修饰在spring管理的bean上，否则无法生效***

2. define where to cut in

following are some express :
+ execution：匹配方法签名:"execution(* testExecution(..))"表示的是匹配名为testExecution的方法，*代表任意返回值，(..)表示零个或多个任意参数。
```
    // 指定的方法
    @Pointcut("execution(* testExecution(..))")
    public void anyTestMethod() {}
```
+ within：指定所在类或所在包下面的方法（Spring AOP 独有）
```
    // service 层
    // ".." 代表包及其子包
    @Pointcut("within(ric.study.demo.aop.svc..*)")
    public void inSvcLayer() {}
```
+ @annotation：方法上具有特定的注解
```
    // 指定注解
    @Pointcut("@annotation(ric.study.demo.aop.HaveAop)")
    public void withAnnotation() {}
```
+ bean(idOrNameOfBean)：匹配 bean 的名字（Spring AOP 独有）
```
    // controller 层
    @Pointcut("bean(testController)")
    public void inControllerLayer() {}
```
3. config advice

```
@Before("...")
public void logArgs(JoinPoint joinPoint) {
    System.out.println("方法执行前，打印入参：" + Arrays.toString(joinPoint.getArgs()));
}
```




# Reference
https://blog.csdn.net/qq_41907991/article/details/105500421
https://juejin.cn/post/6844903987062243341#heading-5


