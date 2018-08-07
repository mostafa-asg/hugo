---
title: "Reactive Systems vs Reactive Programming"
date: 2018-08-07T15:34:56+04:30
draft: false
tags: [digest, reactive programming, reactive systems]
---
This is the digest of [Reactive Programming versus Reactive Systems](https://www.lightbend.com/reactive-programming-versus-reactive-systems)
by *Jonas Bonér* and *Viktor Klang*. This is for anyone who does not have enough time to
study the original white paper. It is the shortest version without sacrificing important stuff.

# Reactive - A Set Of Design Principles
* “Reactive” is a <span style="color:red">set of design principles</span> for creating cohesive systems.
* In a Reactive System, it’s the interaction between the individual parts that makes all the difference,
which is the ability to operate individually yet act in concert to achieve their intended result.
* We see that it’s possible to write a single application in a Reactive style
(i.e. using Reactive Programming); however, that’s merely one piece of the puzzle.
Though each of the above aspects may seem to qualify as “Reactive” in and of themselves <span style="color:red">
they do not make a system Reactive</span>.
* When people talk about Reactive in the context of software development and design,
they generally mean one of three things:
  * Reactive Systems (architecture and design)
  * Reactive Programming (declarative event-based)
  * Functional Reactive Programming (FRP)
* The main driver behind modern systems is the notion of <span style="color:red">Responsiveness</span>:
the acknowledgement that if the client/customer does not get value in a timely fashion then they will go
somewhere else. Fundamentally there is no difference between not getting value and not getting value when
it is needed.
* In order to facilitate Responsiveness, two challenges need to be faced:
  * being Responsive **under failure**, defined as <span style="color:red">Resilience</span>
  * and being Responsive **under load**, defined as <span style="color:red">Elasticity</span>
* The [Reactive Manifesto](https://www.reactivemanifesto.org/) prescribes that in order to achieve this,
the system needs to be <span style="color:red">Message-driven</span>

{{< fluid_imgs
        "center|/static/reactive-system-vs-programming/1.png|Reactive Manifesto"
>}}
# Functional Reactive Programming (FRP)
* Functional Reactive Programming, commonly called “FRP,” <span style="color:red">is frequently misunderstood</span>. FRP was very
precisely defined 20 years ago by [Conal Elliott](http://conal.net/papers/icfp97/). The term has most recently been used incorrectly to describe
technologies like [Elm](http://elm-lang.org/), [Bacon.js](https://baconjs.github.io/), and Reactive Extensions (RxJava, Rx.NET, RxJS) amongst others.
Most libraries claiming to support FRP are almost exclusively talking about Reactive Programming and it will
therefore not be discussed further.

# Reactive Programming
* Reactive Programming, not to be confused with Functional Reactive Programming,
is a subset of Asynchronous Programming and a paradigm where <span style="color:red">the availability of new information</span>
drives the logic forward rather than having control flow driven by a thread-of-execution.
* It supports decomposing the problem into multiple discrete steps where each can be executed in an asynchronous
and nonblocking fashion, and then be composed to produce a workflow—possibly unbounded in its
inputs or outputs.
* [Asynchronously](https://www.reactivemanifesto.org/glossary#Asynchronous) is a very important technique in Reactive Programming since it allows for non-blocking
execution—where threads of execution competing for a shared resource don’t need to wait by blocking
(preventing the thread of execution from performing other work until current work is done),
and can as such perform other useful work while the resource is occupied.
Amdahl’s Law tells us that <span style="color:red">contention is the biggest enemy of scalability</span>,
and therefore a Reactive program should rarely, if ever, have to block.
{{< fluid_imgs
        "center|/static/reactive-system-vs-programming/2.png|Non Blocking"
>}}
* Reactive Programming is generally <span style="color:red">Event-driven</span>, in contrast to
Reactive Systems, which are <span style="color:red">Message-driven</span>
* The Application Program Interface (API) for Reactive Programming libraries are generally either:
  * <span style="color:red">Callback-based</span>—where anonymous, side-effecting callbacks are attached to event sources,
    and are being invoked when events pass through the dataflow chain.
  * <span style="color:red">Declarative</span>—through functional composition, usually using well established combinators like map,
    filter, fold etc.
* Most libraries provide a mix of these two styles, often with the addition of stream-based operators like
  windowing, counts, triggers, etc.
* <span style="color:red">It would be reasonable to claim that Reactive Programming is related to Dataflow Programming</span>,
  since the emphasis is on the flow of data rather than the flow of control.
* Popular libraries supporting the Reactive Programming techniques on the JVM include, but are not limited to,
  [Akka Streams](https://akka.io/), [Ratpack](https://ratpack.io/), [Reactor](https://projectreactor.io/),
  [RxJava](https://github.com/ReactiveX/RxJava) and [Vert.x](https://vertx.io/).

# The Benefits Of Reactive Programming
* The primary benefits of Reactive Programming are: <span style="color:red">increased utilization</span> of
  computing resources on multicore and multi-CPU hardware; and <span style="color:red">increased performance</span> by
  reducing serialization points.
* A secondary benefit is one of <span style="color:red">developer productivity</span> as traditional programming paradigms have all
  struggled to provide a straightforward and maintainable approach to dealing with asynchronous and
  nonblocking computation and IO. Reactive Programming solves most of the challenges here since it
  typically removes the need for explicit coordination between active components.
* In order to take full advantage of asynchronous execution, the inclusion of
  <span style="color:red">back-pressure</span> is crucial to avoid over-utilization,
  or rather unbounded consumption of resources.
* But even though Reactive Programming is a very useful piece when constructing modern software,
  in order to reason about a system at a higher level one has to use another tool:
  <span style="color:red">Reactive Architecture</span>—the process of designing Reactive Systems.

# Event-Driven VS Message-Driven
* As mentioned previously, Reactive Programming—focusing on computation through ephemeral <span style="color:red">dataflow</span>
  chains—tend to be <span style="color:red">Event-driven</span>, while Reactive Systems—focusing on
  <span style="color:red">resilience</span> and <span style="color:red">elasticity</span> through the communication,
  and coordination, of distributed systems—is <span style="color:red">Message-driven</span> (also referred to as Messaging).
* The <span style="color:red">main difference</span> between a Message-driven system with long-lived addressable components,
  and an Event-driven dataflow-driven model, is that Messages are inherently directed, Events are not.
  <span style="color:red">Messages have a clear, single, destination; while Events are facts for others to observe</span>.
  Furthermore, messaging is preferably asynchronous, with the sending and the reception decoupled
  from the sender and receiver respectively.
* Messages are needed to communicate across the network and forms the basis for communication in distributed systems,
  while Events, on the other hand, are emitted locally. It is common to use Messaging under the hood to
  bridge an Event-driven system across the network by sending Events inside Messages.
* Messaging forces us to <span style="color:red">embrace the reality and constraints of distributed systems</span>—things
  like partial failures, failure detection, dropped/duplicated/reordered messages, eventual consistency,
  managing multiple concurrent realities, etc.—and tackle them head on instead of hiding them behind a
  leaky abstraction—pretending that the network is not there—as has been done too many times in the past (e.g. EJB, RPC, CORBA, and XA).
* These differences in semantics and applicability have profound implications in the application design,
  including things like resilience, elasticity, mobility, location transparency and management of the
  complexity of distributed systems.
{{< fluid_imgs
        "center|/static/reactive-system-vs-programming/3.png|Event-Driven VS Message-Driven"
>}}

# Reactive Systems And Architecture
* The foundation for a Reactive System is <span style="color:red">Message-Passing</span>, which creates a temporal boundary between
  components which allows them to be decoupled in time—this allows for concurrency—and space—which allows
  for distribution and mobility. This decoupling is a requirement for full isolation between components,
  and forms the basis for both Resilience and Elasticity.
* The world is becoming increasingly interconnected, which mean software is increasingly dependent on other
  software to function properly.
* So the key to building <span style="color:red">Resilient</span>, self-healing systems is to allow failures to be: contained, reified as
  messages, sent to other components (that act as supervisors), and managed from a safe context outside the failed
  component. Here, being Message-driven is the enabler: moving away from strongly coupled, brittle,
  deeply nested synchronous call chains that everyone learned to suffer through…or ignore.
  The idea is to decouple the management of failures from the call chain,
  freeing the client from the responsibility of handling the failures of the server.
* <span style="color:red">Elasticity</span> is about Responsiveness under load—meaning that the throughput of a system scales up or down
  (i.e. adding or removing cores on a single machine) as well as in or out
  (i.e. adding or removing nodes/machines in a data center) automatically to meet varying demand as
  resources are proportionally added or removed. It is the essential element needed to take advantage of
  the promises of Cloud Computing: allowing systems to be resource efficient, cost-efficient,
  environmentally-friendly and pay-per-use.
* Systems need to be adaptive—allowing for intervention-less auto-scaling, replication of state and behavior,
  load-balancing of communication, failover and upgrades, all without rewriting or even reconfiguring
  the system. The enabler for this is <span style="color:red">Location Transparency</span>: the ability to scale the system in the same way,
  using the same programming abstractions, with the same semantics, across all dimensions of scale—from CPU
  cores to data centers.
* Reactive Systems represent the most productive system architecture that we know of (in the context of multicore,
  Cloud and Mobile architectures):
  * Isolation of failures offer [bulkheads](https://skife.org/architecture/fault-tolerance/2009/12/31/bulkheads.html) between components, preventing failures from cascading and limiting the scope and severity of failures.
  * Supervisor hierarchies offer multiple levels of defences paired with self-healing capabilities, which removes a lot of transient failures from ever incurring any operational cost to investigate.
  * Message-passing and location transparency allow for components to be taken offline and replaced or rerouted without affecting the end-user experience. This reduces the cost of disruptions, their relative urgency, and also the resources required to diagnose and rectify.
  * Replication reduces the risk of data loss, and lessens the impact of failure on the availability of retrieval and storage of information.
  * Elasticity allows for conservation of resources as usage fluctuates, allowing for minimizing operational costs when load is low, and minimizing the risk of outages or urgent investment into scalability as load increases.
{{< fluid_imgs
        "center|/static/reactive-system-vs-programming/4.png|Bulkheads"
>}}
### How Does Reactive Programming Relate To Reactive Systems?
* Reactive Programming is a great technique for managing internal logic and dataflow transformation, locally within the components, as a way of optimizing code clarity, performance and resource efficiency. Reactive Systems, being a set of architectural principles, puts the emphasis on distributed communication and gives us tools to tackle resilience and elasticity in distributed systems.
* One common problem with only leveraging Reactive Programming is that its tight coupling between computation stages in an Event-driven callback-based or declarative program makes Resilience harder to achieve because its transformation chains are often ephemeral and its stages—the callbacks or combinators—are anonymous, i.e. not addressable.
* If one of the stages in the dataflow chain fails, then the whole chain needs to be restarted, and the client notified. This is in contrast to a Message-driven Reactive System, which has the ability to self-heal without necessitating notifying the client.
* Another contrast to the Reactive Systems approach is that pure Reactive Programming allows decoupling in time, but not space (unless leveraging Message-passing to distribute the dataflow graph under the hood, across the network, as discussed previously).

{{< fluid_imgs
        "center|/static/reactive-system-vs-programming/5.png|"
>}}
{{< fluid_imgs
        "center|/static/reactive-system-vs-programming/6.png|"
>}}

### How Does Reactive Programming & Systems Relate To Fast Data Streaming?
* Underneath the end-user API, it typically uses Message-passing and the principles of Reactive
  Systems in-between nodes supporting a distributed system of stream processing stages,
  durable event logs, replication protocols—although these parts are typically not exposed to the developer.
  **It is a good example of using Reactive Programming at the user level and Reactive Systems at the system level**.

### How Does Reactive Programming & Systems relate to Mobile Applications And The Internet Of Things (IoT)?
* There’s a need for strategies for handling device failures, for when information is lost, and when services fail—because they will. The back-end systems managing all this needs to be able to scale on demand and be fully resilient, in other words, there’s a need for **Reactive Systems**.
* Having lots of sensors generating data, and being unable to deal with the rate with which this data arrives—a common problem set seen for the back-end of IoT—indicates a need to implement back-pressure for devices and sensors. Looking at the end-to-end data flow of an IoT system—with tons of devices: the need to store data, cleanse it, process it, run analytics, without any service interruption—the necessity of asynchronous, non-blocking, fully back-pressured streams becomes critical, this is where **Reactive Programming** really shines.

### How Does Reactive Programming & Systems Relate To Traditional Web Applications?
* Web Applications can greatly benefit from a **Reactive Programming** style of development
* Most recently, server push á la Server-Sent Events and WebSockets have become increasingly used, **Reactive Programming** has tools for this, more specifically Streams and Futures, which makes it straightforward to do non-blocking and asynchronous transformations and push those to the clients.
* **Reactive Programming** can also be valuable in the data access layer—updating and querying data in resource efficient manner—preferably using SQL or NoSQL databases with asynchronous drivers.
* Web applications also benefit from **Reactive System** design for things like: distributed caching, data consistency, and cross-node notifications. Traditional web applications normally use stateless nodes. But as soon as you start using Server-Sent-Events (SSE) and WebSockets, your nodes become stateful, since at a minimum, they are holding the state of a client connection, and push notifications need to be routed to them accordingly. Doing this effectively requires a Reactive System design, since it is an area where directly addressing the recipients through messaging is important.