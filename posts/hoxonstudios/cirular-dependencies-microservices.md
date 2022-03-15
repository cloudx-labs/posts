---
title: 'Circular dependencies between Microservices'
published: false
description: There is a bad practice in Microservices architecture that seems to be totally forgotten
tags: 'architecture, programming, microservies, distributedsystems'
cover_image: ./assets/leaves.jpg
---

Circular dependencies are a bad practice in Microservices architecture that usually goes unnoticed. Yet avoiding them is crucial to reduce complexity in this architectural pattern.

When working in monolithic systems, there's widespread agreement that these dependencies are evil. On microservices, though, the story is completely different. In this pattern, we treat services as independent modules that communicate almost without restriction. This approach is very prone to creating circular dependencies.

Having circular dependencies between microservices will result in services that are hardly maintainable. You'll not be able to think on a single service at a time as they'll be highly coupled.

Let's explore some practices that we want to avoid to get rid of circular dependencies in our Microservices architecture.

## Leaking domain knowledge

A very imperceptible way of creating circular dependencies between services is sending requests with Ids, value objects, or any other kind of knowledge that belongs to the client service domain.

### Example

Suppose we have these services:

- ProductService
- InvoiceService

Inside our **ProductService** we will surely have a **Product** with each entity having a **ProductId**. In this example **ProductService** will be sending a request to **InvoiceService** to generate an invoice that could be something like this:

```text
POST /invoice
{ productId: 3, quantity: 1, ... }
```

In this situation **ProductService** has to know that some invoice service exists to send the request. However, our **InvoiceService** will also be aware of the existence of some **Product** entity that is in the domain of the caller. A good way to verify if we are creating a circular dependency is thinking about the **InvoiceService** as a separated entity. If we completely remove the **ProductService**, will this request still make sense as it is? The obvious answer will be 'No'.

To solve this, first we have to decide which service will depend on the other, say Product on Invoice. After we decide that **InvoiceService** will not depend on **ProductService**, we need to think about the former in isolation. Do we really need to receive a **ProductId** to validate the existence of the product, get its unit price and description, and then generate the invoice? Or do we just need a **ProductCode** to display in the invoice? Most of the time the answer will be that receiving the **ProductCode**, **UnitPrice** and **Description** in the request is good enough. Trusting the information received from services behind our control is crucial to avoid unnecessary validations.

```text
POST /invoice
{
  productCode: 'A123',
  description: 'Nice product',
  unitPrice: 99.9,
  quantity: 1,
  ...
}
```

This is just a simple example but keep in mind that there are a lot of ways to leak domain knowledge to other services without us noticing.

## Choreography

This communication pattern seems to be very common in Event-Driven systems. In it, services emit events into a common Event Bus so others can react to them to complete a complex distributed process. 

### Example

- **UserService** emits a *UserRegisteredEvent*
- **CreditService** receives that *UserRegisteredEvent* and validates the credit of that person
- **CreditService** emits a *CreditValidatedEvent*
- **UserService** receives that *CreditValidatedEvent* and finishes the user creation process

Some people may think that, as we are using events to communicate  between services, we are completelly decoupling them. That is only partially true, because the only kind of coupling we are removing is  the temporal coupling. On the other hand, we are now creating a circular dependency between them because events, as requests, are part of the API of our services.

One possible solution to this issue, without the need of removing the asynchronous nature of our system, is using commands and callbacks for responses. In our example, our *UserService* is clearly wanting to validate the credit of the person in order to finish the bigger process of registering a user. Modeling this as events is a clear mistake because what we really have here is a _command_ in disguise, something to which we expect some response to continue with our work.

To overcome this, we will need to change a few things. First, our **CreditService** will need to have an input topic/queue to receive commands. Apart from that, instead of emiting a **UserReigsteredEvent** *(we can still do it for other purposes)* now we are going to explicitly send a **ValidateCreditCommand** into that topic and in the header of that command we need to specify the callback topic where we want to receive the response, in this case the input topic of our **UserService**.

Now, if we look at our **CreditService** in isolation, we will notice that it doesn't know anything about our **UserService** anymore, we had removed the circular dependency we this simple tweak.

## Conclusion

There are plenty of examples that I may be missing, but the main point is that cyclic dependencies are something that almost every developer want to avoid. As an advice, we also need to pay atention when implementing Microservices to avoid them at all costs. This will allow us to think in one service at a time and reduce the cognitive load in our brain. Having to think or change a lot of services in every business requirement is tiresome and a good symptom of having circular dependencies in our Microservices architecture.
