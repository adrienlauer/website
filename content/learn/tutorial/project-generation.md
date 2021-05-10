---
title: "1. Project generation"
subsection: "tutorial"
tags:
    - tutorial
aliases: 
    - "/learn/tutorial"    
menu:
    learn-tutorial:
        parent: "getting-started"
        weight: 1
---

This page will help you create your first SeedStack application.<!--more--> 

## Use the generator

Using the SeedStack generator is the easiest way to start building a new SeedStack application. It allows you to select
the type of project you want to create and, by answering a few questions, **generate a fully operational application**.

Assuming that you have [Maven installed](https://maven.apache.org/install.html), run the following command:

```bash
mvn -U org.seedstack:seedstack-maven-plugin:generate
```

Choose the `web` project type. Then just press enter each time the generator asks a question. It will use the default
values.

{{< callout ref >}}
For more information about project generation, please read [this guide]({{< ref "learn/guides/generating-a-project" >}}) 
{{< /callout >}}

## Run it from command-line

To run the Web application, use the `watch` goal. It will **watch source folders** and when something change trigger a 
recompile and an application refresh. Run the following command:

```bash
mvn seedstack:watch
```   
    
{{< callout info >}}
You can terminate the application gracefully with `Ctrl+C`.
{{< /callout >}}

{{< callout tips >}}
The `watch` goal supports the [LiveReload](http://livereload.com/) protocol to automatically reload the browser page when
the application change. You must **install and activate a LiveReload extension for you browser** to make it work. 
{{< /callout >}}

{{< callout tips >}}
You can also run the application without hot-reloading with the `run` goal. 
{{< /callout >}}

## Hello World

As the simplest example, the ancient tradition of the "Hello World" sample must be honored. 

You can find a REST resource in the `org.generated.project.interfaces.rest` package, named `HelloResource`. It 
exposes a REST API on `/hello` which returns the "Hello World!" text:

```java
package org.generated.project.interfaces.rest;

import javax.ws.rs.GET;
import javax.ws.rs.Path;

@Path("hello")
public class HelloResource {
    @GET
    public String hello() {
        return "Hello World!";
    }
}
``` 

To test the result, in your favorite browser, go to `http://localhost:8080/hello`. It should display the 
"Hello World!" string.

## What's happening ?

At application startup, SeedStack scans the classpath **searching for specific code patterns it will recognize**. Which
patterns are recognized depends on which SeedStack modules are active in the application. 

As an example, the JAX-RS resource above is recognized by the `seed-rest-jersey2` module which automatically exposes it.

{{< callout info >}}
To be detected by the framework, **application code must be placed in a package that is scanned by SeedStack**. 
This is done in the `application.yaml` [YAML](https://en.wikipedia.org/wiki/YAML) configuration file, at the root of the 
classpath:

```yaml
application:
    basePackages: org.generated.project
```  
{{< /callout >}}

## Modify the application

Now that you have successfully run the application, let's modify it. The "Hello World!" text is generated by a REST 
resource located in the `HelloResource.java` file: 
 
* Change the text returned by the `hello()` method to something else.
* Save the file.
* The application should be refreshed.
* If you enabled the LiveReload extension (see above), the browser page should refresh automatically.

## That's it!

Congratulations! You've just successfully run and modified your first SeedStack application.
{{< img src="mascot/mascot-happy.png" >}}

## Now what ?

On this page you have learned:

* How to create a SeedStack project from scratch using the generator.
* How to run the project in hot-reloading mode.
* What a basic REST resource looks like. 

If everything looks good, continue to the [next page]({{< ref "developing.md" >}}).

{{< callout tips >}}
If you can't get this to work, check the [troubleshooting page]({{< ref "support/troubleshooting" >}}).
{{< /callout >}}