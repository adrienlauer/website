---
title: "Factories"
type: "home"
zones:
    - "Business"
sections:
    - "BusinessManual"
tags:
    - domain-driven design
menu:
    BusinessManual:
        weight: 30
---

{{% callout def %}}
**A factory is responsible for creating a whole, internally consistent aggregate when it is too complicated to do
it in a constructor.**
{{% /callout %}}
<!--more-->

# Characteristics

## Objects produced

A factory is part of the domain and responsible for creating some domain objects. In the business framework a factory
can only create domain objects implementing {{< java "org.seedstack.business.Producible" >}}:

* Aggregates through their aggregate root,
* Value objects,
* Domain events.

Note that non-root entities are not produced by factories but should be created by their aggregate root.

## Identity creation

Being responsible for creating valid aggregates, factories may need to create their identity. This can be done from 
input parameters given to the factory or by using a generation mechanism. This mechanism can be automated by the business
framework, see [below]({{< ref "docs/business/manual/factories.md#identity-generation" >}}). 

# Explicit factory

## Declaration

To declare a factory with the business framework, create an interface extending {{< java "org.seedstack.business.domain.Factory" >}}
with at least one method for creating the produced object: 
  
```java
public interface SomeFactory extends Factory<SomeAggregate> {
    
    SomeAggregate createFromName(String name);
}
```

{{% callout info %}}
This interface must be co-located with the produced object. In the case of a factory producing an aggregate, the 
interface should be placed in the corresponding aggregate package.
{{% /callout %}}
  
Then implement this interface in a class extending {{< java "org.seedstack.business.domain.BaseFactory" >}}:
  
```java
public class SomeFactoryImpl extends BaseFactory<SomeAggregate> implements SomeFactory {
    
    SomeAggregate createFromName(String name) {
        SomeAggregate someAggregate = new SomeAggregate(new Name(name));
        someAggregate.initialize(new Date());
        return someAggregate;
    }
}
```  

{{% callout info %}}
Normally, a factory implementation should not depend upon technical aspects like a library or a particular technology. 
As such you can put the implementation along the interface, in the aggregate package. This allows to declare the aggregate 
constructors with default (package) visibility and force client code to use the factory. 
{{% /callout %}}

## Usage

To use your factory, simply [inject it]({{< ref "docs/seed/dependency-injection.md" >}}) where required: 

```java
public class SomeClass {
    @Inject
    private SomeFactory someFactory;
    
    public void someMethod() {
        SomeAggregate someAggregate = someFactory.createFromName("someName");
        // do something with aggregate
    }
}
```

{{% callout info %}}
By default, factories are instantiated each time they are injected, avoiding the risk to wrongly keep an internal state 
between uses. In some cases, after having well considered the issue, you can choose to make your factory a singleton by
annotating the factory implementation with {{< java "javax.inject.Singleton" "@" >}}.
{{% /callout %}}  
  
# Default factory

The business framework provides a default factory for each class implementing {{< java "org.seedstack.business.Producible" >}}
that does not already has an explicit factory.
    
A default factory only provides a `create(...)` method. It will try to find a constructor of the produced class matching 
the given arguments and invoke it. It can be used like this:
    
```java
public class SomeClass {
    @Inject
    private Factory<SomeAggregate> someAggregateFactory;
    
    public void someMethod() {
        SomeAggregate someAggregate = factory.create(new Name("John Doe"));
    }
}
```

{{% callout info %}}
The benefit of using the default factory instead of just invoking the constructor manually is that it will trigger the
business framework identity generation mechanism on the produced object if necessary. 
See [below]({{< ref "docs/business/manual/factories.md#identity-generation" >}}) for details about identity generation.
{{% /callout %}}

# Identity generation

## Declaration 

The business framework provides an identity generation mechanism that can be automatically triggered after the creation
of an object by a factory. It can also be manually triggered if necessary. To use this mechanism, annotate the field of
the aggregate root holding the identity with {{< java "org.seedstack.business.domain.Identity" "@" >}}:


```java
public class SomeAggregate extends BaseAggregateRoot<UUID> {
    @Identity(generator = UuidGenerator.class)
    private UUID id;
    
    // other fields and methods
}
```

The {{< java "org.seedstack.business.domain.Identity" "@" >}} annotation takes the class of the generator as parameter.
While you can directly specify a concrete implementation of the generator, it is recommended to only specify a generation 
strategy interface like {{< java "org.seedstack.business.domain.UuidGenerator" >}} or {{< java "org.seedstack.business.domain.SequenceGenerator" >}}
and specify its implementation using [class configuration]({{< ref "docs/seed/configuration.md#class-configuration" >}}):

```yaml
classes:
  org:
    mycompany:
      myapp:
        domain:
          model:
            myaggregate:
              SomeAggregate:
                identityGenerator: simpleUUID
```

In this case, the implementation of {{< java "org.seedstack.business.domain.UuidGenerator" >}} annotated with the
qualifier `@Named("simpleUUID")` will be used for this aggregate.

## Usage

To automatically trigger the identity generation mechanism at the end of a factory creation method, annotate it with the 
{{< java "org.seedstack.business.domain.Create" "@" >}} annotation:

```java
public class SomeFactoryImpl extends BaseFactory<SomeAggregate> implements SomeFactory {
    
    @Create
    SomeAggregate createFromName(String name) {
        SomeAggregate someAggregate = new SomeAggregate(new Name(name));
        someAggregate.initialize(new Date());
        return someAggregate;
    }
}
```  

{{% callout info %}}
After the method has returned, an interceptor will apply the chosen identity strategy on the returned object. 
{{% /callout %}}

As an alternative you can apply the identity generation strategy programmatically by injecting the {{< java "org.seedstack.business.domain.IdentityService" >}}:

```java
public class SomeClass {
    @Inject
    private IdentityService identityService;

    public void someMethod() {
        MyAggregate myAggregate = new MyAggregate();
        identityService.identify(myAggregate);
        return myAggregate;
    }
}
```

{{% callout warning %}}
Note that identity generation does not walk the object graph to generate identities for eventual sub-entities. You must
trigger identity generation (automatically or manually) separately on each entity.
{{% /callout %}}

## Custom identity generator

Below is an example of a basic Timestamp id generation strategy:

```java
package org.mycompany.myapp.infrastructure.identity;

import org.seedstack.business.domain.BaseEntity;
import org.seedstack.business.domain.IdentityGenerator;

@Named("timestamp-id")
public class TimestampIdentityGenerator implements IdentityGenerator<BaseEntity<Long>, Long> {
    
    @Override
    public Long generate(BaseEntity<Long> entity, Map<String, String> entityConfig) {
        return new Date().getTime();
    }
}
```

## Built-in identity generators

### Sequence

The sequence generator provides a unique ever-incrementing number. Numbers are not required to be contiguous. Implementations 
of this strategy must implement the {{< java "org.seedstack.business.domain.SequenceGenerator" >}} interface. 

A test-only in-memory implementation is provided by {{< java "org.seedstack.business.util.inmemory.InMemorySequenceGenerator" >}}. 
No state is preserved across application restarts. It is configured as below:

```yaml
classes:
  org:
    mycompany:
      myapp:
        domain:
          model:
            myaggregate:
              SomeAggregate:
                identityGenerator: inMemorySequence
```

{{% callout tips %}}
Sequence generators for relational databases are provided in the [JPA add-on]({{< ref "addons/jpa/index.md" >}}). 
{{% /callout %}}

### UUID

The UUID generator uses a [Universally Unique Identifier](https://en.wikipedia.org/wiki/Universally_unique_identifier) as
identity. Implementations of this generator must implement the {{< java "org.seedstack.business.domain.UuidGenerator" >}}
interface.
 
An implementation that uses the {{< java "java.util.UUID" >}} Java class is provided by {{< java "org.seedstack.business.domain.SimpleUuidGenerator" >}}.
It is configured as below:  

```yaml
classes:
  org:
    mycompany:
      myapp:
        domain:
          model:
            myaggregate:
              SomeAggregate:
                identityGenerator: simpleUUID
```

# Example

## The interface

```java
public interface OrderFactory extends GenericFactory<Order> {

    Order createOrder(Customer customer, List<Pair<ProductId, Integer>> orderedProducts);
    
    Order createRepeatOrder(Order previousOrder);
    
    Order createCreditOrder(Order orderToCredit);    
}
```

## The implementation

```java
@Create
public class OrderFactoryImpl extends BaseFactory<Order> implements OrderFactory {
    
    @Override
    public Order createOrder(CustomerId customerId) {
        return new Order(customerId);
    }
    
    @Override
    public Order createRepeatOrder(Order previousOrder) {
        Order order = new Order(previousOrder.getCustomerId());
        previousOrder.items().forEach(order::addItem);
        return order;
    }
    
    @Override
    public Order createCreditOrder(Order orderToCredit) {
        return new CreditOrder(orderToCredit.getCustomerId(), orderToCredit.getId());
    }
}
```  
