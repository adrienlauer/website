---
title: "Configuration"
subsection: "getting-started"    
tags:
    - tutorial
menu:
    learn-getting-started:
        parent: "basics"
        weight: 4
---

The [SeedStack configuration system]({{< ref "docs/core/configuration.md" >}}) is a **global tree structure**, aggregated 
from several sources. The most important configuration source is the `application.yaml` file, located at the root of the classpath.

There are 3 special top-level nodes in the configuration tree:

* `env` contains all [environment variables]({{< ref "docs/core/configuration.md#environment-variables" >}}) as children nodes,
* `sys` contains all [Java system properties]({{< ref "docs/core/configuration.md#system-properties" >}}) as children nodes,
* `classes` can contain a hierarchy of nodes for [attaching metadata to classes]({{< ref "docs/core/configuration.md#class-configuration" >}}).

{{% callout info %}}
Besides these special nodes, each SeedStack module reserves a top-level node for its configuration. You can find
the name of those nodes in each module documentation. 
{{% /callout %}}

To access configuration, you have to [map]({{< ref "docs/core/configuration.md#mapping" >}}) a node in the tree to a Java object:

* Some types like primary types, arrays, lists, sets, etc... can be mapped directly.
* Other types are mapped by recursively matching their attributes to children tree nodes.

For example, write the following YAML in the `application.yaml` file:

```yaml
myConfig:
    names: ["Roger", "Anna"]
```  

Then create a simple `MyConfig` class in the `application` package:

```java
package org.generated.project.application;

import java.util.List;
import org.seedstack.coffig.Config;

@Config("myConfig")
public class MyConfig {
    private List<String> names;
    
    public List<String> getNames() {
        return names;
    }
}
``` 

{{% callout tips %}}
The {{< java "org.seedstack.coffig.Config" "@" >}} annotation correlates this class to the `myConfig` node in 
the tree.
{{% /callout %}}

Edit the `HelloResource` class to inject a fully mapped `MyConfig` POJO by using the 
{{< java "org.seedstack.seed.Configuration" "@" >}} annotation:

```java
package org.generated.project.interfaces.rest;

import javax.ws.rs.GET;
import javax.ws.rs.Path;
import org.generated.project.application.MyConfig;
import org.seedstack.seed.Configuration;

@Path("hello")
public class HelloResource {
    @Configuration
    private MyConfig myConfig;

    @GET
    public String hello() {
        return "Hello " + myConfig.getNames().get(0) + "!";
    }
}
``` 

## Now what ?

On this page you have learned:

* How SeedStack configuration basics work.

If everything looks good, continue to the [next page]({{< ref "dependency-injection.md" >}}).

{{% callout tips %}}
If you can't get this to work, check the [troubleshooting page]({{< ref "support/troubleshooting" >}}).
{{% /callout %}}