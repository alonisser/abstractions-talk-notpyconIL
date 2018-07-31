class: center, middle

# Abstractions.
## And 3 sad tales of the slow app

[live presentation](https://alonisser.github.io/abstractions-talk/) <br/>
[twitter](alonisser@twitter.com), [medium](https://medium.com/@alonisser/)

Works at [zencity.io](https://zencity.io/). We help cities leverage AI to understand their citizen's needs.

---
# Castles of the mind

>*The programmer, like the poet, works only slightly removed from pure thought-stuff. He builds his castles in the air, from air, creating by exertion of the imagination.”*
 
>Frederick P. Brooks Jr., The Mythical Man-Month: Essays on Software Engineering

---
# Abstractions

Software is about abstractions. It's the way we communicate. It's an important part of the way we reason about software.

We are building our castles with abstractions.

--

And yes. It's **abstractions all the way down**.
.img-half-container.center[![turtles all the way down](./turtle.png)]

---

# The old tale about the DB connection
A simple, but ugly start
```python
//Application code

db = MongoClient('mongodb://127.0.0.1:27017/mydb')

return db.find({})
```

---

# The old tale about the DB connection

A little wrapping abstraction

```python
# Technical repo module
db = MongoClient('mongodb://127.0.0.1:27017/mydb') # <= a URI here

class DataStore(object):
    @classmethod
    get(query):
        return db.find(query)
        
# And in application module:

from datastores import DataStore
DataStore.get(somequery)         
```

---

# Intermezzo about URI

URI - Uniform Resource Identifier

```python

mongo_url="mongodb://127.0.0.1:27017/mydb"
         
```

> *Uniformity provides several benefits.  It allows different types
  of resource identifiers to be used in the same context, even when
  the mechanisms used to access those resources may differ.  It
  allows uniform semantic interpretation of common syntactic
  conventions across different types of resource identifiers.  It
  allows introduction of new types of resource identifiers without
  interfering with the way that existing identifiers are used.  It
  allows the identifiers to be reused in many different contexts,
  thus permitting new applications or protocols to leverage a pre-
  existing, large, and widely used set of resource identifiers.* 
  
> From the URI current definition By the Internet Engineering task force (IETF) in rfc-3986

---

# Intermezzo about URI

URI - Uniform Resource Identifier

```python

mongo_url="mongodb://127.0.0.1:27017/mydb"
         
```

> *Uniformity provides several benefits.  It allows **different types**
  of resource identifiers to be used in the same context, even when
  **the mechanisms used to access those resources may differ**.  It
  allows uniform semantic interpretation of common syntactic
  conventions across **different types of resource** identifiers.  It
  allows introduction of new types of resource identifiers without
  interfering with the way that existing identifiers are used.  It
  **allows the identifiers to be reused in many different contexts**,
  thus permitting new applications or protocols to leverage a pre-
  existing, large, and widely used set of resource identifiers.* 
  
> From the URI current definition By the Internet Engineering task force (IETF) in rfc-3986

--

The URI is used to abstract differences between resources

---
# Intermezzo about URI

URI - Uniform Resource Identifier

```python

mongo_url="mongodb://127.0.0.1:27017/mydb"
         
```
It's a powerful tool. It's a concise abstraction which delivers lots of data:
 a protocol, identification, resource location and even configuration

---

# The old tale about the DB connection

We change from using a concrete ip address to a localhost name.

```python

mongo_url="mongodb://localhost:27017/mydb"
# Technical code here .. 
# And in application module:

from datastores import DataStore
DataStore.get(somequery)  
         
```
--

A new abstraction: DNS. instead of providing hard coded addresses we provide a name

---

# The old tale about the DB connection
  
Names! Is there a more used (and litigated) software abstraction? 

.img-half-container.center[![Atreo riding falcor](./falcor.jpg)]
 
---

# The old tale about the DB connection

As we are moving to deploy we opt to use a mongo as a service from a provider.

```python

mongo_url="mongodb://myuser:mypass@ds666.mlabs.com:61055/mydb"

# Technical code here .. 
# And in application module:

from datastores import DataStore
DataStore.get(somequery)  

         
```
--

Note that from the application code, nothing changes. Our abstraction is fulfilling its destiny!

---
# The old tale about the DB connection

Finally we move to production, with a mongo cluster and a ssl connection


```python

mongo_url="mongodb://myuser:mypass@ds666-a0.mlabs.com,ds666-a1.mlabs.com:61055/mydb?replicaSet=ds666&ssl=true"

# Technical code here .. 
# And in application module:

from datastores import DataStore
DataStore.get(somequery)  

         
```
Note that from the application code, still nothing changes. 

---

# The old tale about the DB connection

Lets reflect on the way we went.

--

From the crude local mongo code in the application to an abstracted db api which actually uses a highly available cloud service.

This was done by small iterative changes, in good **refactoring** practice.

--

And from application code perspective nothing changes

--

OR DOES IT?

---

class: center, middle

# 
```python

mongo_url="mongodb://myuser:mypass@ds666-a0.mlabs.com,ds666-a1.mlabs.com:61055/mydb?replicaSet=ds666&ssl=true"
```
.img-container[![we are not in kansas any more](We-are-not-in-Kansas-Anymore1.jpg)]
---

# Leaky abstractions

>*"All non-trivial abstractions, to some degree, are leaky"* Joel Spolsky from: [The Law of Leaky Abstractions
](https://www.joelonsoftware.com/2002/11/11/the-law-of-leaky-abstractions/)
---

# Leaky DB abstraction
So while from the application code perspective "nothing has changed" . LOTS HAS ACTUALLY CHANGED:

--

* local socket communication with a local process has been replaced with communication over the unreliable internet network via tcp/ip 
* So we now need to reason about ssl, latency, consistency (It's a cluster!).
* Failure handling and connection pooling which are usually a small concern on a local install, are an order of magnitude more complex under those conditions

---

# Numbers

.img-container[![Undersea internet cables](./submarine_cables.png)]

---

# Numbers

.img-container[![Undersea internet cables](./submarine_laying_cable.jpg)]

---

# Numbers

.img-container[![Numbers that every programmer should know](https://camo.githubusercontent.com/77f72259e1eb58596b564d1ad823af1853bc60a3/687474703a2f2f692e696d6775722e636f6d2f6b307431652e706e67)]

from https://gist.github.com/hellerbarde/2843375

---

# Open question

Is it good that that all this new complexity was added completely abstracted from our application code? 

--

Is there a way we can "mark" this kind of code as "Dangerously slow over unreliable network?" Should we?

.img-half-container.center[![slippery alery](./slippery.png)]
---

class: center, middle

# First tale: staging failure

---

# First tale: staging failure

One of the first things I was told when starting to work at the company was: "there is no staging" It's broken

Staging was just too slow to be useful.

--

Naturally new code was "tested" and tinkered with at production.
 
--

Yes I do know that **the only real staging environment is production** but having no non-local, long living playground is unacceptable.

---
# First tale: staging failure

Lets find our killer.

--

Previous fixing efforts went into throwing more cpu power/memory GBs on the staging environment instance.
 Or on the staging MongoDB installation.

--

It didn't help. Every moving part had more than enough cpu and more than enough memory. Throwing more at it wouldn't improve it. 

--

We had a blind spot, the network connecting the Db and our app.

---
# Networks

The world is a big place

.img-half-container.center[![world](./world2.gif)]

And networks have real constraints: from routers to the physics limiting the bandwidth and speed of submarine communications cables  


---
# Networks

.img-container.center[![world_in_hand](./world_in_hand.jpg)]

But the internet makes it seem small

--

I can ssh into a server in South Asia exactly the same as I ssh into a router in our office. For a human, it **feels** the same.

--

But not for a computer

---

# Latency: A slow killer

Turns out that the staging MongoDB installation by our provider was in North America AWS us-east-1a.

--

Sadly, our servers were in central europe

--

The inter aws region average latency for those zones is 88 ms.

On my dev machine Localhost latency is approx 0.05 ms. The db installation was 3 (+) **orders of magnitude** slower. about 1760 times slower.

---
# Blind spot

why was that a blind spot while we were searching for our staging environment killer?

--

I suspect this has a lot to do with Abstracted geography, physics under elegant software constructs, hiding the ugly face of network latency.
And with the way multiple software layers and abstractions, make db configuration removed from our reflection.

--

It might also have to do with software development "silos". Whose concern is it? 
it's not a db vendor's concern nor a saas db provider's concern but it's also out of the immediate responsibilty of app devs.

---

# Blind spot

Software products can do a better job in shedding light on those dark corners. for example, A MongoDB driver could at least log an error or warning about high latency.

---

class: center, middle

# Second tale: Who ate my cpu?

.img-container[![Goldilocks eats](3_bears.jpg)]

---

# Second tale: Who ate my cpu?

The CPU is in the heart of computing, a tremendously complex hardware and software.

For at least half a century we have been building better and higher abstractions about communicating with the cpu.
 We've moved from punch cards to assembly,
from Assembly to c and from c to lots of higher level and very high level languages. 

.img-half-container.center[![cpu scheme](./cpu4960-I7.jpg)]
---
# Second tale: Who ate my cpu?

Again, we were slow.. the app was just sluggish to the point of unusable.

At this point, all of our code was running in the same instance, both "cron" workers and webapp. 

Looking at the instance we could see rocket high cpu and monitoring showed long blocking periods in the node.js event loop.

Naturally we went hunting for a CPU hog. resource intensive, long, sync, compute in our app.

--

And we just couldn't find one..

--

Oh, after stopping worker processes for a while, we did get quicker for a short period of time.  

---

# Abstracted compute

Can you spot the important difference?

.img-container[![remote top](./remote_top.png)]

.img-container[![local top](./local_top.png)]

--
No you can't, It's abstracted away

---

# Abstracted compute

One represents the actual machine underneath, the other some logical vm

.img-container[![remote top](./remote_top.png)]

.img-container[![local top](./local_top.png)]

---

# Abstracted compute

When you purchase cloud compute from AWS your instance has a number of vCPUs: 

VCPU? 
>*hyperthread of an Intel Xeon core for M4, M3, C4, C3, R3, HS1, G2, I2, and D2.*

--

There is also a mention of ECU:

>*Transitioning to a utility computing model fundamentally changes how developers have been trained to think about CPU resources. Instead of purchasing or leasing a particular processor to use for several months or years, you are renting capacity by the hour. Because Amazon EC2 is built on commodity hardware, over time there may be several different types of physical hardware underlying EC2 instances. Our goal is to provide a consistent amount of CPU capacity no matter what the actual underlying hardware.*

In my words: the way AWS controls your access to processing power

---

# Abstracted compute


So while abstracted CPU processing is **quite different** from a "real" cpu. yet top shows them as exactly* the same. 

*(You can get a some limited clues)

---

# Abstracted compute

Took us some time to notice high levels of cpu "steal time" in the monitoring graphs

.img-container[![cpu steal time](./steal_time_cpu.png)]

>[graph source](http://mamchenkov.net/wordpress/2015/12/11/cpu-steal-time/)

---

# Abstracted compute

Turned out that the real reason our CPU maxed out, was we were being throttled out of CPU cycles by AWS.

It's the curse of the T2 EC2 instances. You don't get a "full" vCPU (what ever that means) you get a **percentage** of a "full" vCPU.
Depending on the number of ECUs you bought.

--

So while abstracted processing power is a necessary abstraction in the world of Infrastructure As A Service(IAAS); it's completely missing from our tools.
 From the point of view of the server it's very hard to tell the difference between a maxed out cpu to a 'stolen' one.
And impossible to tell the difference between a 'steal' happening from noisy neighbour, over sold server or being throttled by hypervisor.  

--

I'm sure this isn't a good thing

---
# Second tale: the cpu strikes back

Two months later we started getting a really degraded performance from our (hosted) MongoDB installation. 

Turned out that it was the T2 instance hell all over again, this time with the db provider hosting instances hitting the limit, due to our queries churning the cpu.


--

This time with instances we don't even control.

--

Yes, it was in the Plan specifics, everyone missed it. including, for a while, the support engineers from the provider.
 
---

# Mitigating abstraction disaster
I can't offer a lot of help here.

But good metrics timeseries of **different** aspects for your system would prove invaluable to debugging these kinds of scenarios.

--

You need to collect: system metrics, network metrics, application inner metrics, APM data, cloud provider metrics, errors and commits and yes, business KPI metrics.

Metric exploration and overlaying might shed some light into the places your own abstractions hide from you; 

**just behind those beautiful castles of the mind** 


---
# Discussion

* Abstractions are evolving while our tools and languages less so. this is becoming painful. What are our missing tools?

* What are the ways we can mitigate this abstraction pain?

* When and where is the point of "too much" abstraction? 

* Is it possible to automate "linting" of too much abstraction?

---
# Ideas

* Adding abstraction review into general code review process

* Dev team deep diving once in a while in one particular underlying abstraction, understanding what's beneath.

* "Abstraction debt management" 

---

class: center, middle

#Open source rocks!

---

class: center, middle

#Thanks for listening!

---
