---
title: "3. Building & Packaging"
subsection: "tutorial"    
tags:
    - tutorial
menu:
    learn-tutorial:
        parent: "getting-started"
        weight: 2
---

SeedStack is built with Maven and provides specific tools for it like BOM dependency management or a plugin for various
tasks. <!--more--> 

## Dependency management

SeedStack comes with full dependency-management so you **never have to specify individual component version**. 

This is done with a [Maven BOM](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#Importing_Dependencies)
imported in the POM of the project:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.seedstack</groupId>
            <artifactId>seedstack-bom</artifactId>
            <version>{{< version g="org.seedstack" >}}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

## Packaging

You can package the application with the following command:

```bash
mvn clean package
```

SeedStack automatically packages the application in a [one-JAR capsule](http://www.capsule.io/). After the build, this 
capsule is located into the `target` directory. 

It is a self-executable JAR you can run with the following command:

```bash
java -jar target/my-web-project-capsule.jar
```

{{< callout ref >}}
Learn more about application packaging by reading about the [Maven package goal]({{< ref "docs/maven-plugin/package.md" >}}). 
{{< /callout >}}

## Now what ?

On this page you have learned:

* How to configure your Java IDE.
* How SeedStack dependency management works.
* How to package a SeedStack application.

If everything looks good, continue to the [next page]({{< ref "logging.md" >}}).

{{< callout tips >}}
If you can't get this to work, check the [troubleshooting page]({{< ref "support/troubleshooting" >}}).
{{< /callout >}}