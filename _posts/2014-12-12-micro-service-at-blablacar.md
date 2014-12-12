---
layout:     post
title:      "Micro-services at BlaBlaCar"
tags:       tech
authors:    [benjamin-fraud]
---

Going international comes with a bunch of technical issues that need to be addressed early. How do we easily **scale** our platform ? How do we
 provide our service to users who are thousands of kilometers away ? How do we ensure that everything goes smoothly with our **constant
 delivery processes** ?

To answer all these questions, we must adopt a global technical strategy which gives us enough **flexibility** each and every day. Like many other
  platforms, our service is mainly structured around a **main functional brick**, with a few smaller ones gravitating around. That is why we are
currently working on slicing our **monolithic application** into smaller, independent functional bricks. But what does independent mean ?

In a user-centric application where everything is connected, how do you trace **relevant boundaries** and decide what can be isolated and what cannot ?

## Event sourcing

Although there is no simple answer, we must first ask ourselves how we would like to make our hypothetical different services **talk** to each others.
At some point, user actions will have to trigger **background operations**. If you take the time to list these operations, and the way they are connected
 to each other, you will most likely end up with a **spaghetti bowl**, where nothing seems to be independent enough to be isolated.

At this point you'll have basically two choices : let things globally untouched, or review the way you allow things to **communicate between them**.
Since the first solution is not an option, we specified inter-service communication around two axes :

* A service **should never** ask directly another service to perform business operations, it may only ask for data which is meaningful to perform its
 own operations ;
* A service **may** trigger an **event** whenever an operation is processed to notify every other service that might be interested that some
 operation was processed. These services will then be able to trigger operations of their own, with their own language and logic, completely
 independently from one another.

This can be done by using a **messaging system** (or message broker), where every service may publish into or consume from. This allows
to drastically reduce the coupling between services. At Blablacar, we use **RabbitMQ**.

![event-sourcing](../../images/2014-12-12-event-sourcing.png)

## Specialized layers

But while event sourcing allows us to design independent services, we still have no precise idea of what these services should look like.
Indeed, we may have identified a bunch of **functional bricks** which may be isolated from one another, but what about the data ? Going international
will raise issues regarding data access from **multiple countries and services**, and we need to find a way to make data available.

For these reasons, we imagined a global architecture made of **three specialized layers** :

### Application

Basically any client which requires functionality or data. The application is responsible for **routing** the client request,
asking **layers below** for a functionality, and formatting an understandable response. At Blablacar, we work with multiple applications :
 our main Symfony2 application (the website), our **public API** (the entry-point for mobile apps), a bunch of **workers** (to perform asynchronous tasks)...
Each of these applications has its own needs and should therefore ask for operations to the appropriate services.

The application layer, being responsible for **aggregating calls** to different services, may also provide the user with **partial functionality**, in case of
unavailability from one service.

### Business

Contains the whole domain logic. It acts as the **middleware** between application and data. It enforces the **business rules**, and is
described using a language understandable by everyone. With that in mind, **domain-driven design** (DDD) offers a good starting point to
describe and partially implement the boundaries we designed. Using DDD, we are able to imagine effectively the concepts that need to be dealt
with by refocusing on their true meaning.

Business layers are kept **simple** and focus only on pure business processes.

### Data

Contains access methods to our data back-ends. Each data type has its own entry point, allowing us to easily switch from one back-end
 to another that may use a **different technology**. Communication between data layers is **not allowed** : if a business functionality requires
 data from multiple storages, it must be responsible for **aggregating** it according to its needs. This obviously raises questions such as pagination
 capabilities, and at some point, technical choices we make will have an **impact on the product**. For this reason, it is mandatory to raise
 awareness of **technical constraints** we face and to make them understandable for everyone.

![layers](../../images/2014-12-12-layers.png)

## Think API

By thinking micro-services, the idea behind is that each part should provide a **generic interface** to allow easy inter-communication.
Although business layers may only communicate with each other through event sourcing, there is still the need to actually call the
different business and data methods, directly from some application code if the layer is a dependency (a vendor of some worker for instance),
or through an **HTTP client** if it is on a remote location (which is what we are aiming at).

For this reason, every service (business or data) should expose its actions **through an API**. The client code should have no awareness of the implementations
behind the scene, and only requests for business actions or data through this generic interface. This interface is responsible for routing the request
towards the appropriate layer.

At BlaBlaCar, we designed this communication with the **Request/Handler/Response** pattern for the business layer :

* The **request** encapsulates the client needs and provides access to each attribute. It is easily **serializable**, in order to be passed through the
network ;
* The **handler** performs the actual operation based on the request. Its responsibility is to process the needed operation and to return a response ;
* The **response** encapsulates the result of the business operation and is easily serializable as well. As part of an **API-oriented service**, it should
provide a status code, informing on the operation success or failure, that may be **translated** into HTTP code for instance.

![pattern](../../images/2014-12-12-pattern.png)

## Conclusion

As of now, we support our international growth by isolating our services one step at at time. By thinking micro-services, we force ourselves to
imagine things globally and to abstract the layers involved.

Among all the technical challenges which are faced during this research phase, the major difficulty is probably to make it a global concern,
because everyone involved will hit the wall at some point.