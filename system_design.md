
# General Stragey

## Ask refining questions

Functional requirements
* The clients need directly—for example, the ability to send
  messages in near real-time to friends.
Non-Functional requirements
* messaging service performance shouldn’t degrade with increasing user load.

## Handle the data

* What’s the size of the data right now?
* At what rate is the data expected to grow over time?
* How will the data be consumed by other subsystems or end users?
* Is the data read-heavy or write-heavy?
* Do we need strict consistency of data, or will eventual consistency work?
* What’s the durability target of the data?
* What privacy and regulatory requirements do we require for storing or
  transmitting user data?


## Discuss the components

Discuss the standard components

An example could be the type of database - will a conventional database work,
or should we use a NoSQL database?

* Front-end components,
* load balancers,
* caches,
* databases,
* firewalls, and
* CDNs are just some examples of system components.

## Discuss trade-offs

We should point out weaknesses in our design to our interviewer and explain why
we haven’t tackled them yet. An example could be that our current design can’t
handle ten times more load, but we don’t expect our system to reach that level
anytime soon. We have a monitoring system to keep a very close eye on load
growth over time so that a new design can be implemented in time. This is an
example where we intentionally had a weakness to reduce system cost.

Something is always failing in a big system. We need to integrate fault
tolerance and security into our design.


# What to not do

Here are a few things that we should avoid doing in a system design interview:

* Don’t write code in a system design interview.
* Don’t start building without a plan.
* Don’t work in silence.
* Don’t describe numbers without reason. We have to frame it.
* If we don’t know something, we don’t paper over it, and we don’t pretend to
  know it.

# Abstractions

## Network Abstraction

* Rpc calls
    * client stub and server stub respectively abstract the remote side to
      the local side.

# Consistency


## Cap Theorm

Only 2 out of the 3, can be guaranteed

* Consistency
  * Every read receives the most recent write or an error.
* Availability
  * Every request receives a (non-error) response, without the guarantee that
    it contains the most recent write.
* Partition tolerance
  * The system continues to operate despite an arbitrary number of messages
    being dropped (or delayed) by the network between nodes.

## ACID for database transations

* Atomicity
  * Atomicity guarantees that each transaction is treated as a single "unit",
    which either succeeds completely or fails completely
* Consistency
  * Consistency ensures that a transaction can only bring the database from one
    consistent state to another, preserving database invariants:
* Isolation
  * Isolation ensures that concurrent execution of transactions leaves the
    database in the same state that would have been obtained if the
    transactions were executed sequentially.
* Durability
  * Durability guarantees that once a transaction has been committed, it will
    remain committed even in the case of a system failure (e.g., power outage
    or crash). This usually means that completed transactions (or their
    effects) are recorded in non-volatile memory

## Different Consistency Paradigms

Eventual        Causal           Sequential        Strict
Consistency     Consistency      Consistency       Consistency/Linearizability

* Eventual Consistency
  * 2 writes done independantly will be eventually updated everwhere.
  * Makes for a highly available system
  * Eg: DNS
* Causal consistency
    * works by categorizing operations into dependent and independent
      operations.
    * Dependent operations are also called causally-related operations.
    * Causal consistency preserves the order of the causally-related operations.
    * Eg, a comment and reply must appear one after the other.
* Sequential Consistency
    * Sequential consistency is stronger than the causal consistency model.
    * It preserves the ordering specified by each client’s program.
    * However, sequential consistency doesn’t ensure that the writes are
      visible instantaneously or in the same order as they occurred according
      to some global clock.
    * Eg, post from different friends can be re-ordered, but the posts by the
      same friend should appear in order
* Strict consistency aka linearizability
    * Usually, synchronous replication is one of the ingredients for achieving
      strong consistency, though it in itself is not sufficient.
    * We might need consensus algorithms such as Paxos and Raft to achieve
      strong consistency.

# Failure Models

## Fail-stop

* In this type of failure, a node in the distributed system halts permanently.
* However, the other nodes can still detect that node by communicating with it.
* From the perspective of someone who builds distributed systems, fail-stop
  failures are the simplest and the most convenient.

## Crash

* In this type of failure, a node in the distributed system halts silently, and
  the other nodes can’t detect that the node has stopped working.

## Omision Failures

* In omission failures, the node fails to send or receive messages.
* There are two types of omission failures.
  * If the node fails to respond to the incoming request, it’s said to be a
    send omission failure.
  * If the node fails to receive the request and thus can’t acknowledge it,
    it’s said to be a receive omission failure.

## Temporal failures

* In temporal failures, the node generates correct results, but is too late to
  be useful. This failure could be due to bad algorithms, a bad design
  strategy, or a loss of synchronization between the processor clocks.

## Byzantine failures

* In Byzantine failures, the node exhibits random behavior like transmitting
  arbitrary messages at arbitrary times, producing wrong results, or stopping
  midway.
* This mostly happens due to an attack by a malicious entity or a
  software bug. A byzantine failure is the most challenging type of failure to
  deal with.

# Availability

Availability is the percentage of time that some service or infrastructure is
accessible to clients and is operated upon under normal conditions.


100% available => said service/functions resopnsds as intended all the time.

```
Availability    (Total Time - Amount Of Time Service Was Down)
(in percent) =  ---------------------------------------------- * 100
                             Total Time
```
### what exactly is avaialble

* Available for everybody
    * If its down for a subset of users, is it counted as available?
* Planned downtimes
* Downtimes due to cyberattacks

## in nines

Availability Percentages versus Service Downtime

Availability %      | Downtime per Year | Downtime per Month | Downtime per Week
-------             | -------           | -------            | -------
90% (1 nine)        | 36.5 days         | 72 hours           | 16.8 hours
99% (2 nines)       | 3.65 days         | 7.20 hours         | 1.68 hours
99.5% (2 nines)     | 1.83 days         | 3.60 hours         | 50.4 minutes
99.9% (3 nines)     | 8.76 hours        | 43.8 minutes       | 10.1 minutes
99.99% (4 nines)    | 52.56 minutes     | 4.32 minutes       | 1.01 minutes
99.999% (5 nines)   | 5.26 minutes      | 25.9 seconds       | 6.05 seconds
99.9999% (6 nines)  | 31.5 seconds      | 2.59 seconds       | 0.605 seconds
99.99999% (7 nines) | 3.15 seconds      | 0.259 seconds      | 0.0605 seconds

# Reliability

MTBF =  Total Elapsed Time - Sum of Downtime
        ------------------------------------
              Total Number of Failures

MTTF =  Total Number of Repairs
        -----------------------
        Total Maintenance Time

* The measurement of availability is driven by time loss
* whereas the frequency and impact of failures drive the measure of reliability

# Scalability

Scalability is the ability of a system to handle an increasing amount of
workload without compromising performance

Eg of workload:

The workload can be of different types, including the following:

* Request workload: This is the number of requests served by the system.
* Data/storage workload: This is the amount of data stored by the system.

## Dimensions

Here are the different dimensions of scalability:

* Size scalability:
    * A system is scalable in size if we can simply add additional users and
      resources to it.
* Administrative scalability:
    * This is the capacity for a growing number of organizations or users to
      share a single distributed system with ease.
* Geographical scalability:
    * This relates to how easily the program can cater to other regions while
      maintaining acceptable performance constraints. In other words, the
      system can readily service a broad geographical region, as well as a
      smaller one.

## Approaches

* Vertical:
    * Add capabilities to existing device. Eg: RAM, CPU
    * Usually costly as the cost of exotic components increase than commodity
    * synonym: scaling up
* Horizontal
    * synonym: scaling out
    * increase num of machines in the network.

# Maintainability

## what is it

* Operability:
    * This is the ease with which we can ensure the system's smooth operational
      running under normal circumstances and achieve normal conditions under a
      fault.
* Lucidity:
    * This refers to the simplicity of the code. The simpler the code base, the
      easier it is to understand and maintain it, and vice versa.
* Modifiability:
    * This is the capability of the system to integrate modified, new, and
      unforeseen features without any hassle.

## measuring

Maintainability, M, is the probability that the service will restore its
functions within a specified time of fault occurrence. M measures how
conveniently and swiftly the service regains its normal operating conditions.

For example, suppose a component has a defined maintainability value of 95% for
half an hour. In that case, the probability of restoring the component to its
fully active form in half an hour is 0.95.


# Fault Tolerance

## what is it

Fault tolerance refers to a system’s ability to execute persistently even if
one or more of its components fail. Here, components can be software or
hardware. Conceiving a system that is hundred percent fault-tolerant is
practically very difficult.

## techniques

### Replication

* replicate both service and data
* Updating data in replicas is a challenging job

### Checkpointing

* save computational state at a given point

# Back of envelope calculations

* 
