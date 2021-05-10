---
title: "Domain module"
type: "home"
zones:
    - "GettingStarted"
sections:
    - "GettingStartedProjectTemplates"
tags:
    - onboarding
    - domain-driven design
menu:
    GettingStartedProjectTemplates:
        weight: 60
---

A reusable JAR designed to contain one or more business domain(s) based on the [business framework]({{< ref "docs/business" >}}).<!--more--> 

# Creation

You need to have [Apache Maven 3.1+](https://maven.apache.org/) installed. 
To create a reusable domain project from scratch, execute the following command:

```bash
mvn -U org.seedstack:seedstack-maven-plugin:generate -Dtype=domain
```

{{< callout info >}}
This will invoke the generate goal of the SeedStack maven plugin [generate goal]({{< ref "docs/maven-plugin/manual/generate.md" >}}) which will:

* Discover the latest version of the [SeedStack reference distribution]({{< ref "docs/overview/distribution.md" >}}),
* Use its [batch archetype](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22org.seedstack%22%20a%3A%22domain-archetype%22) to generate the project.

The process is interactive and will ask you a few questions about the project to be created.
{{< /callout >}}

# Result

After execution, a single module project is created which contains only the domain layer. This module is intended to be
included in other modules such as a [Web application]({{< ref "getting-started/project-templates/web.md" >}}),  
a [CLI tool]({{< ref "getting-started/project-templates/cli.md" >}}), or any runnable project.

You should see a structure similar to the following:

```plain
- mydomain
    |- src
        |- main
        |   |- java
        |   |   |- org.generated.project
        |   |       |- domain                
        |   |           |- model              <-- domain model
        |   |           |- services           <-- domain services
        |   |           |- shared             <-- shared value objects
        |   |- resources
        |       |- application.yaml           <-- main configuration
        |- test
            |- java
            |- resources
        |       |- application.override.yaml  <-- test configuration
```

# More resources

* [Business framework documentation]({{< ref "docs/business" >}}).
* [Simple business example](https://github.com/seedstack/samples/tree/master/business).
