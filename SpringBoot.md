

### aop

```java
@Component
@Aspect
public class LogComponent {
    @Pointcut("execution(* org.javaboy.aop.service.*.*(..))")
    public void pc1() {
    }

    @Before(value = "pc1()")
    public void before(JoinPoint jp) {
        String name = jp.getSignature().getName();
        System.out.println("before--" + name);
    }

    @After(value = "pc1()")
    public void after(JoinPoint jp) {
        String name = jp.getSignature().getName();
        System.out.println("after--" + name);
    }

    @AfterReturning(value = "pc1()", returning = "result")
    public void afterReturning(JoinPoint jp, Object result) {
        String name = jp.getSignature().getName();
        System.out.println("afterReturning----" + name + "-----" + result);
    }

    @AfterThrowing(value = "pc1()",throwing = "e")
    public void afterThrowing(JoinPoint jp,Exception e) {
        String name = jp.getSignature().getName();
        System.out.println("afterThrowing---"+name+"----"+e.getMessage());
    }

    @Around("pc1()")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        Object proceed = pjp.proceed();
        return "www.javaboy.org";
    }
}
```

