---
title: "Startup & Shutdown"
subsection: "core"
aliases: 
    - /docs/seed/manual/running
    - /docs/seed    
    - /docs/core
tags:
    - cli
menu:
    docs-core:
        parent: "basics"
        weight: 1
---

The main entry-point of SeedStack is the {{< java "org.seedstack.seed.core.SeedMain" >}} class. It can be invoked in 
various ways like:

* Directly from command-line,
* By a SeedStack all-in-one capsule,
* By your development environment (IDE).

This entry-point searches for a **launcher** in the classpath and delegates the effective startup and shutdown of the 
application to it. Exactly one launcher must be present in the classpath. SeedStack base framework provides two launchers:

* A command-line launcher, for [command-line applications]({{< ref "docs/core/cli" >}}).
* An Undertow launcher, for [Web applications]({{< ref "docs/core/web" >}}) embedding the [Undertow Web server](http://undertow.io/).

{{< callout info >}}
Applications are not always executed through the main entry-point:

* [WAR-packaged Web applications]({{< ref "docs/core/web#in-a-container" >}}) are directly launched by the servlet container. 
* In a similar way, integration tests are directly launched by the testing framework.
{{< /callout >}} 

## Startup

### Capsule

SeedStack applications are often packaged as an all-in-one executable JAR named a "Capsule". They can be launched by
simply invoking:

```bash
java [jvm-args] -jar app-capsule.jar [app-args]
```

{{< callout info >}}
Packaging applications in a capsule is done with the [package goal]({{< ref "docs/maven-plugin/package.md" >}}) of the 
SeedStack maven plugin. Every project created with the [generator]({{< ref "learn/tutorial/project-generation.md" >}})
is already configured to be packaged in a capsule.
{{< /callout >}}

### Plain command-line

Without a capsule, SeedStack applications are run like any other Java process, by setting the classpath manually using
the `-cp` argument:

```bash
java [jvm-args] -cp ... org.seedstack.seed.core.SeedMain [app-args]
```

## Shutdown 
    
To shutdown a standalone application, you simply have to gracefully stop the JVM. You can do this on any operating system 
this by hitting `CTRL+C` if the JVM is a foreground process. You can also do this on UNIX systems if the JVM is a 
background process by issuing a `SIGINT` signal to the JVM process:
 
```bash
kill -2 pid
```
    
Before the JVM stops, the shutdown logic of the SeedStack application will be invoked.
     
{{< callout warning >}}
Warning! If you abruptly terminate or kill the JVM process, the application will NOT gracefully shutdown.  
{{< /callout >}}    

## Tool mode

SeedStack application can embed "tools" which are 

A SeedStack application can be run in "tool mode": instead of using the normal launcher, a special tool launcher
will be used to execute a particular tool


. In this case a special launcher is invoked and will execute the specified
tool. You can run any module in tool mode by specifying the `seedstack.tool` system property. With a capsule:
 
```bash
java [jvm-args] -Dseedstack.tool=toolName -cp ... org.seedstack.seed.core.SeedMain [tool-args]
```

or with a Capsule:

```bash
java [jvm-args] -Dseedstack.tool=toolName -jar app-capsule.jar [tool-args]
```


{{< callout tips >}}
The [tool goal]({{< ref "docs/maven-plugin/tool.md" >}}) of the SeedStack Maven plugin can run a tool directly from Maven.

You can also run a tool from your IDE by executing the {{< java "org.seedstack.seed.core.SeedMain" >}} main class with
the `-Dseedstack.tool=<toolName>` JVM argument. You can use the program arguments to specify the arguments of the tool.
{{< /callout >}}

## Development mode

In development, you have the ability to run a standalone application directly from Maven with the [run goal]({{< ref "docs/maven-plugin/run.md" >}}) 
of the SeedStack Maven plugin:

```bash
mvn [-Dargs="app-args"] seedstack:run
```

This will automatically build the classpath from the project dependencies and create an isolated classloader to run the
application. 

## Lifecycle listener

You can implement the {{< java "org.seedstack.seed.LifecycleListener" >}} interface to be notified about the application
lifecycle events:

```java
public class MyLifecycleListener implements LifecycleListener {
    @Logging
    private Logger logger;

    @Override
    public void started() {
        logger.info("Application has been started");
    }

    @Override
    public void stopping() {
        logger.info("Application is about to be stopped");
    }
}
```

Such classes are injectable and can be used to implement initialization and shutdown logic.
