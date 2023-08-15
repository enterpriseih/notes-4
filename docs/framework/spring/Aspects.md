https://www.cnblogs.com/unknows/p/10441164.html

https://www.cnblogs.com/ooo0/p/11018752.html

# AspectUtils

xml配置

```xml
<!-- 切面 -->
<bean id="aspectUtils" class="com.yienx.utils.AspectUtils"/>

<aop:config>
    <!--log 记录日志的一个切面-->
    <aop:aspect id="logAroundAspect" ref="aspectUtils">
        <aop:pointcut id="logBusiness"
                      expression="execution(* com.yienx.controller..*(..))
                       || execution(* com.yienx.rpc.impl..*(..))
                       || execution(* com.yienx.api..*(..))
                       "/>
        <!-- 使用切面aspectUtils里的doLOg方法对切点logBusiness进行增强 -->
        <aop:around pointcut-ref="logBusiness" method="doLog"/>
    </aop:aspect>
</aop:config>

```



aspectUtil

```java
// 举例，输出执行时间
public Object doTime(ProceedingJoinPoint point) {
    String className = point.getTarget().getClass().getName();
    String methodName = point.getSignature().getName();
    long start = System.currentTimeMillis();
    Object result = null;// 方法执行结果
    try {
        result = point.proceed();// 执行目标对象的业务方法
    } catch (Throwable e) {
        // logger.error("AspectUtils.doTime error", e);
        System.out.println("AspectUtils.doTime error" + e);
    } finally {
        // logger.error("执行" + className + "." + methodName + "耗时：" + (System.currentTimeMillis()-start) + "ms");
        System.out.println("执行" + className + "." + methodName + "耗时：" + (System.currentTimeMillis()-start) + "ms");
    }
    return result;
}
```





类型匹配模式：
1、`*`：匹配任何数量字符；比如模式 (*,String) 匹配了一个接受两个参数的方法，第一个可以是任意类型，第二个则必须是String类型
2、`..`：匹配任何数量字符的重复，如在类型模式中匹配任何数量子包；而在方法参数模式中匹配任何数量参数，可以使零到多个。
3、 `+`：匹配指定类型的子类型；仅能作为后缀放在类型模式后边。

参数匹配模式：

`()`匹配了一个不接受任何参数的方法，

`(..)`匹配了一个接受任意数量参数的方法（零或者更多）。 

`(*)`匹配了一个接受一个任何类型的参数的方法。 

`(*,String)`匹配了一个接受两个参数的方法，第一个可以是任意类型， 第二个则必须是String类型。
