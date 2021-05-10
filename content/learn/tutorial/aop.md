---
title: "7. AOP"
subsection: "tutorial"    
tags:
    - injection
    - tutorial
menu:
    learn-tutorial:
        parent: "basics"
        weight: 4
---

SeedStack supports Aspect-Oriented Programming (AOP) through method interception.<!--more--> 

{{< callout info >}}
Method interception is useful to add cross-cutting concerns ("aspects") to existing code. For instance, SeedStack 
uses it for transactions, security enforcement or validation.
{{< /callout >}}

## Declaring a method interceptor

In this tutorial, we will add an annotation-based method interceptor that can log a method call. Interceptors can
be triggered by any kind of condition on classes and methods, as long as your express those conditions in predicates
(see below).

Let's define the `@Trace` annotation first. As it is a general API, we will put it in the `org.generated.project` 
package:

```java
package org.generated.project;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(value = RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
public @interface Trace {
    // no parameters
}
```

From there we need to create a class implementing {{< java "org.seedstack.seed.SeedInterceptor" >}}. As it is 
implementation, we can put it inside the `org.generated.project.infrastructure.aop` package:

```java
package org.generated.project.infrastructure.aop;

import java.lang.reflect.Method;
import java.util.function.Predicate;
import org.aopalliance.intercept.MethodInvocation;
import org.generated.project.Trace;
import org.seedstack.seed.Logging;
import org.seedstack.seed.SeedInterceptor;
import org.slf4j.Logger;

public class LoggingTraceInterceptor implements SeedInterceptor {
    @Logging
    private Logger logger;

    @Override
    public Predicate<Class<?>> classPredicate() {
        return c -> true; // any class
    }

    @Override
    public Predicate<Method> methodPredicate() {
        return m -> m.isAnnotationPresent(Trace.class);
    }

    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        Method method = invocation.getMethod();
        logger.info("--> {}({})", method.getName(), invocation.getArguments());
        Object result = invocation.proceed();
        logger.info("<-- {}", method.getName());
        return result;
    }
}
```

By placing the `@Trace` annotation on the `hello()` method of our `HelloResource`, we satisfy the predicates present
in our interceptor and as such mark the method as intercepted:

```java
package org.generated.project.interfaces.rest;

import javax.ws.rs.GET;
import javax.ws.rs.Path;
import org.generated.project.Trace;

@Path("hello")
public class HelloResource {
    @GET
    @Trace
    public String hello() {
        return "Hello World!";
    }
}
```

Upon calling the method from the browser, you should see the following lines in the log:

```none
INFO  2020-02-24 13:12:58,919 XNIO-1 task-1   o.g.p.i.aop.LoggingTraceInterceptor      --> hello([])
INFO  2020-02-24 13:12:58,920 XNIO-1 task-1   o.g.p.i.aop.LoggingTraceInterceptor      <-- hello
```

{{< callout tips >}}
SeedStack does come with a lot of predicates that you can use in your own interceptors:

* Annotations: {{< java "org.seedstack.shed.reflect.AnnotationPredicates" >}}
* Classes: {{< java "org.seedstack.shed.reflect.ClassPredicates" >}}
* Constructors and methods: {{< java "org.seedstack.shed.reflect.ExecutablePredicates" >}}
{{< /callout >}}

## Limitations

Method interception is implemented by dynamic subclassing and overriding of matched methods. It has very little overhead
but has the following limitations:

* Class must be public or package-private,
* Class must be non-final,
* Method must be public, package-private or protected,
* Method must be non-final,
* Instances must be created by SeedStack (interception cannot be used on instances created with `new`).  

{{< callout warning >}}
**Caution:** if the conditions above are not met on a particular method, method interception will NOT occur on it.  
{{< /callout >}}

## Now what ?

On this page you have learned:

* How to declare an interceptor build an in inject a dependency by field injection,
* How to inject a dependency by constructor injection,
* When and how to declare an implementation as Singleton.

If everything looks good, continue to the [next page]({{< ref "testing.md" >}}).

{{< callout tips >}}
If you can't get this to work, check the [troubleshooting page]({{< ref "support/troubleshooting" >}}).
{{< /callout >}}