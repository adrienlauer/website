---
title: "Logging"
subsection: "getting-started"    
tags:
    - tutorial
menu:
    learn-getting-started:
        parent: "basics"
        weight: 3
---

SeedStack logging is based on the [SLF4J](https://www.slf4j.org/) logging facade. The recommended logging implementation
is [Logback](https://logback.qos.ch/). It is already present in the generated projects. 

A logger for the enclosing class can be injected with the {{< java "org.seedstack.seed.Logging" "@" >}} annotation. Edit 
the `HelloResource` class to inject a logger:

```java
package org.generated.project.interfaces.rest;

import javax.ws.rs.GET;
import javax.ws.rs.Path;
import org.seedstack.seed.Logging;
import org.slf4j.Logger;

@Path("hello")
public class HelloResource {
    @Logging
    private Logger logger;

    @GET
    public String hello() {
        logger.info("The hello() method was called");
        return "Hello World!";
    }
}
``` 

## Now what ?

On this page you have learned:

* How to obtain a SL4J logger and log a simple statement.

If everything looks good, continue to the [next page]({{< ref "configuration.md" >}}).

{{% callout tips %}}
If you can't get this to work, check the [troubleshooting page]({{< ref "support/troubleshooting" >}}).
{{% /callout %}}