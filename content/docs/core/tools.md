---
title: "Tools"
subsection: "core"
tags:
    - cli
menu:
    docs-core:
        parent: "basics"
        weight: 2
---

SeedStack applications can contain administrative commands named "tools". They execute in the same context as the
application they are embedded in, but are ephemeral commands that exit upon completion.<!--more-->

Tools are executed like a [normal application]({{< ref "docs/core" >}}) but with the `seedstack.tool` system 
property set to name of the tool to execute. With a capsule:

```bash
java [jvm-args] -Dseedstack.tool=toolName -jar app-capsule.jar [tool-args]
```

{{< callout info >}}
Tools are often used 
{{< /callout >}}

embed "tools" which are 

A SeedStack application can be run in "tool mode": instead of using the normal launcher, a special tool launcher
will be used to execute a particular tool


. In this case a special launcher is invoked and will execute the specified
tool. You can run any module in tool mode by specifying the `seedstack.tool` system property. With a capsule:
 
```bash
java [jvm-args] -Dseedstack.tool=toolName -cp ... org.seedstack.seed.core.SeedMain [tool-args]
```

or with a Capsule:




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
