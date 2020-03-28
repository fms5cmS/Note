# 3.1 引入

问题：

代码混乱：越来越多的非业务需求（日志和验证等）加入后，原有的业务方法急剧膨胀，每个方法在处理核心逻辑的同时还必须兼顾其他多个关注点。

代码分散：以日志需求为例，只是为了满足这个单一需求，就不得不在多个模块（方法）中多次重复相同的日至代码，如果日志需求发生变化，必须修改所有模块。 

使用==动态代理==解决上述问题代理模式的原理：使用一个代理将对象包装起来，然后用该代理对象取代原始对象，任何对原始对象的调用都要通过代理。代理对象决定是否以及何时将方法调用转到原始对象上。 

```java
public class ArithmeticCalculatorLoggingProxy {
    //要代理的对象
    private ArithmeticCalculator target;

    public ArithmeticCalculator getLoggingProxy() {
        ArithmeticCalculator proxy = null;

        //代理对象由哪个类加载器负责加载
        ClassLoader loader = target.getClass().getClassLoader();
        //代理对象的类型，即其中有哪些方法
        Class[] interfaces = new Class[] {ArithmeticCalculator.class};
        //当调用代理对象其中的方法时，该执行的代码
        InvocationHandler h = new InvocationHandler() {
            /**
              * proxy:正在返回的代理对象，一般情况下，在invoke方法中都不使用该对象。eg：调用proxy的toString方法，又会在调用这里的invoke方法，从而一直循环下去，栈溢出
              * method：正在调用的方法
              * args：调用方法时，传入的参数
              */
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                String methodName = method.getName();
                //日志
                System.out.println("The method " + methodName + " begins with " + Arrays.asList(args));
                //执行方法
                Object result = method.invoke(target, args);
                //日志
                System.out.println("The method " + methodName + " ends with " + result);
                return result;
            }
        };
        proxy = (ArithmeticCalculator) Proxy.newProxyInstance(loader, interfaces, h);
        return proxy;
    }
}
```

 **==AOP ：Aspect-Objected Programming 面向切面编程。OOP的补充==** 。切面编程落实到软件工程其实是为了更好地模块化，而不仅仅是为了减少重复代码。

每个事务逻辑位于一个位置，代码不分散，便于维护和升级；业务模块更简洁，只包含核心业务代码。 

术语：

- 切面(Aspect):  ==横切关注点(跨越应用程序多个模块的功能)被模块化的特殊对象== 
- 通知(Advice):  ==切面必须要完成的工作== 
- 目标(Target): ==被通知的对象== 
- 代理(Proxy): ==向目标对象应用通知之后创建的对象== 
- 连接点（Joinpoint）：==程序执行的某个特定位置==：如类某个方法调用前、调用后、方法抛出异常后等。
  - 连接点由两个信息确定：方法表示的程序执行点；相对点表示的方位。
  - eg： ArithmethicCalculator#add() 方法执行前的连接点，
    - 执行点为 ArithmethicCalculator#add()； 
    - 方位为该方法执行前的位置 
- 切点（pointcut）：==AOP 通过切点定位到特定的连接点。==
  - 类比：连接点相当于数据库中的记录，切点相当于查询条件。
  - 一个切点可以匹配多个连接点，切点通过`org.springframework.aop.Pointcut`接口进行描述，它使用类和方法作为连接点的查询条件。
  - 每个类都拥有多个连接点：例如 ArithmethicCalculator 的所有方法实际上都是连接点，即连接点是程序类中客观存在的事务。切点和连接点不是一对一的关系，



# 3.2 注解配置

AspectJ：Java社区最完整最流行的AOP框架

1. 加入jar包

   ```xml
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-aop</artifactId>
       <version>4.3.16.RELEASE</version>
   </dependency>
   <dependency>
       <groupId>org.aspectj</groupId>
       <artifactId>aspectjweaver</artifactId>
       <version>1.9.1</version>
   </dependency>
   ```

2. 基于注解的方式

   1. 把横切关注点的代码抽象到切面的类中 
   2. 在类中声明各种通知（见后面的5种通知）

3. ==将切面类和业务逻辑类（目标方法类）加入容器中==：包扫描或者`@Bean`的方式

4. 告诉Spring谁是切面类：给==切面类加上`@Aspect`==

   - 在切面类上的每个通知方法上标注通知注解，并告诉Spring何时何地运行(切入点表达式)

5. ==开启AspectJ注解==，两种方法：

   1. 使用配置文件（需引入aop-命名空间）：

      ```xml
      <!-- 使AspectJ注解起作用：自动为匹配AspectJ注解的类生成代理对象，此时生成的对象是一个代理对象 -->
      <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
      ```

   2. 在**配置类上**标注`@EnableAspectJAutoProxy` ！！！！！！

      ```java
      @EnableAspectJAutoProxy
      @Configuration
      public class MainConfigOfAOP {
          @Bean
          public MathCalculator mathCalculator(){
              return new MathCalculator();
          }
          @Bean
          public LogAspect logAspect(){
              return new LogAspect();
          }
      }
      ```

- AspectJ支持5种类型的==**通知注解**==：
  - `@Before`：前置通知，在方法执行之前执行
  - `@After`：后置通知（无论是否发生异常），在方法执行之后执行
  - `@AfterRunning`：返回通知，在方法返回结果之后执行
  - `@AfterThrowing`：异常通知，在方法抛出异常后通知
  - `@Around`：环绕通知：围绕着方法执行。手动推进目标方法运行（`joinPoint.procced()`）



- 利用方法签名编写AspectJ切入点表达式，说明通知的方法（以前置通知为例）
  - `@Before("访问权限 返回类型 包名.类名.方法名(参数类型)")`
  - 抽取公共的切入点表达式：
    1. 创建一个空方法如`public void pointCut(){}`，标注`@Pointcut("execution(访问权限 返回类型 包名.类名.方法名(参数类型))")`；如果切入点是当前类中的引用，可以直接写`@Pointcut()`；
    2. 其他通知使用时：`@Before("pointCut()")`直接传入空方法名

| 内容              | 抽象 | 含义                       |
| ----------------- | ---- | -------------------------- |
| 访问权限+返回类型 | *    | 表示所有的返回类型         |
| 类名              | *    | 表示包里面的所有类         |
| 方法名            | *    | 表示类中的所有方法         |
| 参数类型          | ..   | 表示任意个数的任意返回类型 |

- 在通知方法中声明一个类型为`JoinPoint`的参数（必须是第一个参数），就能访问链接细节，如方法名和参数值 

```java
/**
 * 把这个类声明为一个切面：
 * 1.把该类放入IOC容器中 即加入@Component注解 ；
 * 2.再声明为一个切面 即加入@Aspect注解
 * 3.给切面指定优先级，即加入@Order注解，传入的值越小，优先级越高。（这一步可以不写）
 */
@Order(1)
@Aspect
@Component
public class LoggingAspect {
    /**
     * 定义一个方法，用于声明切入点表达式。一般的，该方法中再不需要添入其他代码
     * 使用 @Pointcut 来声明切入点表达式
     * 后面的其他通知直接使用方法名来引用当前的切入点表达式
     */
    @Pointcut("execution(public int aopAnnotation.impl.ArithmeticCalculatorImpl.*(..))")
    public void declareJoinPointExpression() {}

    //前置通知。利用上面的方法来得到切入点表达式
    @Before("declareJoinPointExpression()")
    public void beforeMethod(JoinPoint joinPoint) {
         String methodName = joinPoint.getSignature().getName();
         List<Object> args = Arrays.asList(joinPoint.getArgs());
         System.out.println("The method " + methodName + " begins with " + args);
    }


    /**后置通知，在后置通知中还不能访问目标方法执行的结果 */
    @After("execution(* aopAnnotation.impl.*.*(..))")
    public void afterMethod(JoinPoint joinPoint) {
         String methodName = joinPoint.getSignature().getName();
         System.out.println("The method " + methodName + " ends.");
    }
    /**
     * 返回通知 可以访问到方法的返回值的！
     * @param result 接收方法的返回值，该参数名必须与注解中的returning值相同
     */
    @AfterReturning(value = "execution(public int aopAnnotation.impl.ArithmeticCalculatorImpl.*(..))", returning = "result")
    public void afterReturning(JoinPoint joinPoint, Object result) {
         String methodName = joinPoint.getSignature().getName();
         System.out.println("The method " + methodName + " ends with " + result);
    }
    /**
     * 异常通知，在目标方法出现异常时执行 可以访问到异常对象；且可以指定在出现特定异常时再执行代码
     * @param ex 接收异常对象，该参数名必须与注解中的throwing的值相同
     */
    @AfterThrowing(value = "execution(* aopAnnotation.impl.*.*(..))", throwing = "ex")
    public void afterThrowing(JoinPoint joinPoint, Exception ex) {
         String methodName = joinPoint.getSignature().getName();
         System.out.println("The method " + methodName + " occurs exception: " + ex);
    }
    /**
     * 环绕通知需要携带 ProceedingJoinPoint 类型的参数
     * 环绕通知类似于动态代理的全过程，ProceedingJoinPoint类型的参数可以决定是否执行目标方法 环绕通知必须有返回值，返回值为目标方法的返回值
     * 很少使用！
     */
    @Around("execution(* aopAnnotation.impl.*.div(..))")
    public Object aroundMethod(ProceedingJoinPoint pjp) {
         Object result = null;
         String methodName = pjp.getSignature().getName();
         try {
             // 前置通知
             System.out.println("the method " + methodName + " begins with " + Arrays.asList(pjp.getArgs()));
             // 执行目标方法
             result = pjp.proceed();
             // 返回通知
             System.out.println("the method " + methodName + " ends with " + result);
         } catch (Throwable e) {
             // 异常通知
             System.out.println("the method " + methodName + " occurs exception : " + e);
             throw new RuntimeException(e);
         }finally {
             //后置通知
             System.out.println("the method " + methodName + " ends." );
         }
         return result;
    }
}
```

如果该切面以外的切面想要使用相同的切入点表达式，可以 

```java
@Before("aopAnnotation.impl.LoggingAspect.declareJoinPointExpression()") 
//包名.类名.方法（同一个包下可以不写包名，同一个切面中不写类名）
```



# 3.3 XML配置

```xml
<!-- 配置Bean -->
<bean id="arithmeticCalculator" class="aopXML.impl.ArithmeticCalculatorImpl"></bean>
     
<!-- 配置切面的Bean -->
<bean id="loggingAspect" class="aopXML.impl.LoggingAspect"></bean>
<bean id="vlidationAspect" class="aopXML.impl.VlidationAspect"></bean>
     
<!-- 配置AOP -->
<aop:config>
   <!-- 配置切点表达式 -->
   <aop:pointcut expression="execution(public int aopXML.impl.ArithmeticCalculatorImpl.*(..))" id="pointcut"/>
   <!-- 配置切面及通知 -->
   <aop:aspect ref="loggingAspect" order="2">
        <aop:before method="beforeMethod" pointcut-ref="pointcut"/>
        <aop:after-returning method="afterReturning" pointcut-ref="pointcut" returning="result"/>
        <aop:after-throwing method="afterThrowing" pointcut-ref="pointcut" throwing="ex"/>
        <aop:after method="afterMethod" pointcut-ref="pointcut"/>
         <!--<aop:around method="aroundMethod" pointcut-ref="pointcut" />-->
   </aop:aspect>
   <aop:aspect ref="vlidationAspect" order="1">
        <aop:before method="vlidationArgs" pointcut-ref="pointcut"/>
   </aop:aspect>
</aop:config>
```

