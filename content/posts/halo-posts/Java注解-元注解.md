---
title: Java注解-元注解
id: 33
date: 2024-09-27 19:10:52
auther: admin
cover: 
excerpt: 元注解@Retentionpublic @interface Retention {    /**     * Returns the retention policy.     * @return the retention policy     */    RetentionPolicy val
permalink: /?p=33
categories:
 - java
tags: 
 - java
 - 注解
---

## 元注解
### @Retention
```java
public @interface Retention {
    /**
     * Returns the retention policy.
     * @return the retention policy
     */
    RetentionPolicy value();
}
```
此注解表明了注解的生命周期
```java
public enum RetentionPolicy {
    /**
     * Annotations are to be discarded by the compiler.
     */
    SOURCE,

    /**
     * Annotations are to be recorded in the class file by the compiler
     * but need not be retained by the VM at run time.  This is the default
     * behavior.
     */
    CLASS,

    /**
     * Annotations are to be recorded in the class file by the compiler and
     * retained by the VM at run time, so they may be read reflectively.
     *
     * @see java.lang.reflect.AnnotatedElement
     */
    RUNTIME
}
```
有三种，分别是源代码、字节码、和运行时
### @Target
```java
public @interface Target {
    /**
     * Returns an array of the kinds of elements an annotation type
     * can be applied to.
     * @return an array of the kinds of elements an annotation type
     * can be applied to
     */
    ElementType[] value();
}
```
表明该注解可以标注在什么位置
```java
public enum ElementType {
    /** Class, interface (including annotation type), or enum declaration */
    TYPE,

    /** Field declaration (includes enum constants) */
    FIELD,

    /** Method declaration */
    METHOD,

    /** Formal parameter declaration */
    PARAMETER,

    /** Constructor declaration */
    CONSTRUCTOR,

    /** Local variable declaration */
    LOCAL_VARIABLE,

    /** Annotation type declaration */
    ANNOTATION_TYPE,

    /** Package declaration */
    PACKAGE,

    /**
     * Type parameter declaration
     *
     * @since 1.8
     */
    TYPE_PARAMETER,

    /**
     * Use of a type
     *
     * @since 1.8
     */
    TYPE_USE
}
```
### @Documented
它的作用是能够将注解中的元素包含到 Javadoc 中去。
### @Inherited
@Inherited注解了的注解修饰了一个父类，如果他的子类没有被其他注解修饰，则它的子类也继承了父类的注解。
### @Repeatable
这个元注解修饰的注解可以同时作用一个对象多次，但是每次作用注解又可以代表不同的含义。
#### 示例
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface LogType {
    Log [] value();
}

```
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Repeatable(LogType.class)
public @interface Log {

    String value() default "";

}
```
```java
@Service
public class UserService {

    @Autowired
    private UserManager userManager;


    @Log(value = "记录空间")
    @Log(value = "记录时间")
    public String getUser() {
        return userManager.getUser();
    }

}
```
```java
@Around("log()")
    public Object aroundAdvice(ProceedingJoinPoint proceedingJoinPoint){
        try {
            logger.info("around通知");
            Log[] logs = getMethod(proceedingJoinPoint).getAnnotationsByType(Log.class);
            Stream.of(logs).forEach(x->{
                logger.info(x.value());
            });
            return proceedingJoinPoint.proceed();
        }catch (Throwable e){
            logger.info("方法运行出现异常，{},{}",e,e.getMessage());
         return null;
        }
    }

    protected Method getMethod(ProceedingJoinPoint pjp) {
        Method method = null;
        try {
            MethodSignature ms = (MethodSignature) pjp.getSignature();
            method = pjp.getTarget()
                    .getClass()
                    .getMethod(ms.getName(), ms.getParameterTypes());
        } catch (NoSuchMethodException e) {
            //ignore
        }
        return method;
    }
```

