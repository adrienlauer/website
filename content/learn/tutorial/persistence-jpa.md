---
title: "14. JPA persistence"
subsection: "tutorial"    
tags:
    - domain-driven design
    - persistence
    - tutorial
menu:
    learn-tutorial:
        parent: "business"
        weight: 2
---

To demonstrate true independence from persistence technology, let's switch to a JPA!<!--more--> 

{{< callout warning >}}
As we will add new dependencies and some configuration, **the application must be stopped**.
{{< /callout >}}

### Dependencies

Add the SeedStack [JPA add-on]({{< ref "addons/jpa" >}}%):

{{< dependency g="org.seedstack.addons.jpa" a="jpa" >}}

Then add an implementation of the JPA standard like [Hibernate](http://hibernate.org/):

{{< dependency g="org.hibernate" a="hibernate-entitymanager" v="5.3.7.Final" >}}

Hibernate requires the following dependency if you're using Java 9 or more recent:

{{< dependency g="javax.xml.bind" a="jaxb-api" v="2.3.0" >}}

Then add an in-memory capable database like [HyperSQL](http://hsqldb.org/):

{{< dependency g="org.hsqldb" a="hsqldb" v="2.4.1" >}}

{{< callout tips >}}
To connect to an external database, you would add its JDBC driver dependency instead.  
{{< /callout >}}

### Configure a database

JPA is structured around the concept of **"persistence units"**. We have to configure at least one unit to use JPA and such unit needs
a JDBC datasource to work with. Add the following configuration in the `application.yaml` file:

```yaml
jdbc:
  datasources:
    myDatasource:
      url: jdbc:hsqldb:mem:mydb
```

The configuration above will declare a JDBC datasource named `myDatasource` pointing to an auto-created, in-memory, 
H2 database. Now declare a JPA unit using this datasource:

```yaml
jpa:
  units:
    myUnit:
      datasource: myDatasource
      properties:
        hibernate.dialect: org.hibernate.dialect.HSQLDialect
        hibernate.hbm2ddl.auto: update
```

The configuration above will declare a JPA unit named `myUnit`, referencing our datasource. 

{{< callout info >}}
The properties are here to configure Hibernate, our JPA provider and are specific to it. Their role here is to tell 
Hibernate what SQL dialect should be used and that we need the tables to be created or updated on startup.
{{< /callout >}}

### Add the JPA annotations

JPA entities must be mapped to a relational model. While this can be done through XML mapping files, a simpler way is
to annotate the classes.

The PersonId class becomes:

```java
package org.generated.project.domain.model.person;

import javax.persistence.Embeddable;
import org.seedstack.business.domain.BaseValueObject;

@Embeddable
public class PersonId extends BaseValueObject {
    private String email;

    private PersonId() {
        // needed for Hibernate
    }

    public PersonId(String email) {
        this.email = email;
    }

    // ...
}
```

The Person class becomes:

```java
package org.generated.project.domain.model.person;
 
import javax.persistence.EmbeddedId;
import javax.persistence.Entity;
import org.seedstack.business.domain.BaseAggregateRoot;

@Entity
public class Person extends BaseAggregateRoot<PersonId> {
    @EmbeddedId
    private PersonId id;
    private String firstName;
    private String lastName;

    private Person() {
        // needed for Hibernate
    }
 
    public Person(PersonId id) {
        this.id = id;
    }
 
    // ...
}
```

{{< callout info >}}
Note that we also added a private no-arg constructor to allow Hibernate to instantiate the classes.
{{< /callout >}}

### Change the qualifiers

Now just replace the {{< java "org.seedstack.business.util.inmemory.InMemory" "@" >}} injection qualifier with
the {{< java "org.seedstack.jpa.Jpa" "@" >}} qualifier common from the JPA add-on. This will tell SeedStack to choose
a JPA-based implementation (also provided by the add-on) for injection instead of the in-memory one.

Let's start with the `SampleDataGenerator` qualifier:

```java
public class SampleDataGenerator implements LifecycleListener {
    @Inject
    @InMemory
    private Repository<Person, PersonId> personRepository;
    
    // ...
}
```

Becomes:

```java
public class SampleDataGenerator implements LifecycleListener {
    @Inject
    @Jpa
    private Repository<Person, PersonId> personRepository;
    
    // ...
}
```

Then the HelloResource class:

```java
@Path("hello")
public class HelloResource {
    @Inject
    @InMemory
    private PersonRepository personRepository;
    
    // ...
}
```

Becomes:

```java
@Path("hello")
public class HelloResource {
    @Inject
    @Jpa
    private PersonRepository personRepository;
    
    // ...
}
```

### Declare a transaction

**JPA database work should be done in a transaction.** A good place to start and end a transaction would be on an application
service that orchestrates all the needed operations for a particular use-case. We don't have such a service in our little
tutorial, so we will put the transaction on the `hello()` method of our REST resource.

Two annotations are needed here:

* The {{< java "org.seedstack.seed.transaction.Transactional" "@" >}} annotation to declare the transaction boundaries,
* And the {{< java "org.seedstack.jpa.JpaUnit" "@" >}} annotation to declare on which resource the transaction should be
done. 

In the `HelloResource` class:

```java
@Path("hello")
public class HelloResource {
    // ...

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    @Transactional
    @JpaUnit("myUnit")
    public String hello() {
        // ...
    }
}
```

And in the `SampleDataGenerator` class: 

```java
public class SampleDataGenerator implements LifecycleListener {
    /// ...
 
    @Override
    @Transactional
    @JpaUnit("myUnit")
    public void started() {
        // ...
    }
    
    // ...
}
```

### Run the application again

Startup the application again by issuing the following command:

```bash
mvn seedstack:watch
```

You can now see that the `/hello` REST endpoint shows the same behavior as before but backed by JPA persistence. **You
can see the JPA subsystem being initialized in the application startup logs.**

### What happened ?

Business-framework compatible persistence add-ons (like [JPA]({{< ref "addons/jpa" >}}) or [MongoDB]({{< ref "addons/mongodb/basics" >}}))
each provide:
 
* An implementation for all standard operations for repositories. 
* The ability to translate a domain specification into the corresponding technology-specific query. 

Both are used here, behind the scenes. The specification used in the `findByName()` method gets translated into a JPA 
criteria query, turned then by Hibernate into the proper SQL query. **The results are available as a {{< java "java.util.stream.Stream" >}}
of domain objects, loaded dynamically from the database as they are consumed.**

## Now what ?

### What we learned

In this page you have learned:

* How to configure a JDBC datasource.
* How to configure a JPA unit.
* How to switch to JPA persistence layer with minimal code impact.

{{< callout ref >}}
If you want to go further on the topic of persistence, see what you can read about [repositories]({{< ref "docs/business/behavior/repositories.md" >}})
and [specifications]({{< ref "docs/business/model/specifications.md" >}}).

You can also explore SeedStack [persistence add-ons]({{< baseUrl >}}addons) documentation: [JDBC]({{< ref "addons/jdbc" >}}), 
[JPA]({{< ref "addons/jpa" >}}), [MongoDB]({{< ref "addons/mongodb/basics" >}}), etc...   
{{< /callout >}}

{{< callout tips >}}
If you can't get this to work, check the [troubleshooting page]({{< ref "support/troubleshooting" >}}).
{{< /callout >}}
