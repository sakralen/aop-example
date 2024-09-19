```java
@Aspect
@Component
public class LoggingAspect {

    private static final Logger log = LoggerFactory.getLogger(LoggingAspect.class);

    @Pointcut("target(org.springframework.data.repository.Repository)")
    public void isRepository() {
    }

    @Pointcut("@within(org.springframework.stereotype.Service)")
    public void isService() {
    }

    @Pointcut("@within(org.springframework.web.bind.annotation.RestController)")
    public void isRestController() {
    }

    @Before(value = "isRepository()" +
            "|| isService()" +
            "|| isRestController()")
    public void addLoggingToMethodCall(JoinPoint joinPoint) {
        StringBuilder sb = new StringBuilder();

        // Формирование возвращаемого значения, имени вызывающего класса и названия метода
        sb.append("Invoked method ")
                .append(getMethodName(joinPoint))
                .append("(");

        MethodSignature signature = (MethodSignature) joinPoint.getSignature();

        // Получение информации о названиях, типах и значениях параметров
        String[] parameterNames = signature.getParameterNames();
        Class<?>[] parameterTypes = signature.getParameterTypes();
        Object[] args = joinPoint.getArgs();

        // Формирование подстроки с параметрами метода
        for (int i = 0; i < parameterNames.length; i++) {
            sb.append(parameterTypes[i].getSimpleName())
                    .append(" ")
                    .append(parameterNames[i])
                    .append(": ")
                    .append(args[i] != null ? args[i].toString() : "null");

            if (i < parameterNames.length - 1) {
                sb.append(", ");
            }
        }
        sb.append(")");

        log.info(sb.toString());
    }

    @AfterReturning(value = "isRepository()" +
            "|| isService()" +
            "|| isRestController()",
            returning = "result")
    public void addLoggingToMethodReturn(JoinPoint joinPoint, Object result) {
        String sb = "Returned from method " +
                getMethodName(joinPoint) +
                ": " +
                (result != null ? result.toString() : "null");

        log.info(sb);
    }

    @AfterThrowing(value = "isRepository()" +
            "|| isService()" +
            "|| isRestController()",
            throwing = "throwable")
    public void addLoggingToMethodThrow(JoinPoint joinPoint, Throwable throwable) {
        String sb = "Thrown from method " +
                getMethodName(joinPoint) +
                ": " +
                throwable;

        log.info(sb);
    }

    private static String getMethodName(JoinPoint joinPoint) {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();

        return signature.getReturnType().getSimpleName() +
                " " +
                signature.getDeclaringType().getSimpleName() +
                "." +
                signature.getMethod().getName();
    }

}
```
