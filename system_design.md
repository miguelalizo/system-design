# System Design Review

## Contents

1.  [System Design Framework](#system-design-framework)
1.  [Vert vs Horz scaling](#vertical-vs-horizontal-scaling)
1.  [Load balancer](#load-balancer)
1.  [* DBs](#databases)
1.  [DB-Partitioning-Replication Consistency-Availability](#db-partitioning-replication-consistency-availability)
1.  [DB Scaling](#db-scaling)
1.  [CAP Theroem](#cap-theorem)
1.  [* ACID](#acid-cap)
1.  [Cache](#cache)
1.  [CDN](#cdn)
1.  [DNS](#dns)
1.  [Data centers](#data-centers)
1.  [Message Queue](#message-queue)
1.  [Web Tier; stateless vs stateful](#web-tier)
1.  [Networking / HTTP / Web Sockets](#networking-http-web-sockets)
1.  [Logging / Metrics / Automation](#logging-metrics-and-automation)
1.  [* Patterns](#patterns)
    -  [* Consistent hashing and why](#consistent-hashing)

## System design Framework

[Back to main](#contents)

Step 1: Understand the problem and establish the design scope
- Think deeply and ask clarifying questions
- try to ask questions that narrow the design scope and write these down as you get the answers
- typical questions:
    - what specific features are we going to build?
    - How many users does a product have?
    - How fast does the company anticipate to scale up? What are the anticipated scales in 3 months, 6 months, a year?
    - What is the companies tech stack?
    - What existing services might you leverage in the design?
    - Mobile app or web app? or both?
    - Most improtant features of the product?
    - is the newsfeed sorted in reverse chonological order?
    - DAU? Daily active users?
    - Make a verbal assumption about memory and ask if that is fair. Does 20Bytes per average query sound reasonable (for search auto-complete system)?

Step 2: Propose high-level design and get buy in
- Collaborate with the interviewer through this process
    - treat interviewer as a teammate
- Come up with initial blueprint for the design and ASK FOR FEEDBACK.
- draw box diagram with key components on whiteboard
    - this might include clients (mobile/web), APIs, web services, data stores, cache, CDN, message queue, etc.
- Do back of the envolope calcs to evaluate if the blueprint fits the requirements
- if possible go through a few concrete use cases. This will help you frame the high level design. Also might help discover edge cases.

Step 3: Design deep-dive
- At this point you should have a already achieved the following objectives
    - agreed on the overall goals and feature scope
    - sketched out high-level bleprint for the overall design
    - obtained feedback from your interviewwer on the high-level design
    - maybe had some initial ideas of areas to deep dive into

Step 4: Wrap up
- Give the interviewer a recap of the design to refresh their memory
- Never say your design is perfect. Always consider how the design could be better
- Talk about potential error case scenarios and how they are handled by the design
- How to handle the next scale curve

Dos:
- Ask for clarification. Do not assume your assumptions are correct.
- Understand the requirements of the problem
- Communicate with your interviewer your thought process
- Suggest multiple approaches if possioble
- Bounce ideas off the interviewer
- Never give up
- Ask for hints if stuck!
- Try not to think in silence
- Ask for feedback early and often



## Vertical vs Horizontal Scaling

[Back to main](#contents)

### Vertical Scaling

"Scaling up" or "Vertical Scaling" is the process of adding more power (CPU, RAM, etc) to your servers.

Pros
- Great option when traffic is low
- Offers simplicity

Cons
- Has a hard limit; impossible to add unlimited CPU and memory to a single server
- Does not have failover and reduncancy. Single point of failure

### Horizontal Scaling

"Scaling out" is the process of adding more servers to your pool of resources.

More desirable for large scale applications

Pros
- Generally it costs less to have higher amount of basic servers than one really great server
- Offers failover and reduncancy
- In the case of Datacenters, offers better latency
- In the case of databases, offers better read and write performance



## Load Balancer

[Back to main](#contents)

A load balancer evenly distributes incoming traffic amonng web servers that are defined in the load balanced set.

Users connect to the public IP of the load balancer directly. From there the load balancer forwards the request to a server in the load-balanced pool using private IP.

Pros
- Provides failover
    - If one worker server is down, the load balancer could stop forwarding requests to that one and instead to a different one.
- Improves availaibility of the web tier
- Provides horizontal scaling capablity
    - If the website traffic grows rapidly, you can add more servers to the load balanced pool (scale out)


## Databases

[Back to main](#contents)

### SQL vs NoSQL

Relational (SQL)
- Stores data in tables and rows
- Supports joins across tables and supprots many-to-many relationships
- Examples:
    - MySQL
    - PostgreSQL
    - Oracle

NoSQL:
- 1. Key Value Stores
- 2. Graph stores
- 3. Document stores
- 4. Column stores


1. Key-Value Stores

High level description

Pros / Cons

When to use

Examples (3rd pty, Amazon, Azure)


1. Graph Stores

High level description

Pros / Cons

When to use

Examples (3rd pty, Amazon, Azure)


1. Document Stores

High level description

Pros / Cons

When to use

Examples (3rd pty, Amazon, Azure)


1. Column Stores

High level description

Pros / Cons

When to use

Examples (3rd pty, Amazon, Azure)


## DB-Partitioning-Replication Consistency-Availability

[Back to main](#contents)

### Partitioning

For large applications, it is infeasable top fit the entire dataset in a single server. The simplest solution is to split the data into smaller partitions and store them in multiple servers.

There are two challenges when partitioning data:
- how to distribute data evenly across the servers
- minimize the data movement when nodes are added or removed

#### Consistent Hashing
**Consistent Hashing** solves these problems:
- servers are placed on a hashring
- key is hashed onto the ring and the data is stored in the first server encountered while moving in clockwise direction

Advantges of consistent hashing:
- automatic scaling: servers can be added or removed depending on additional load; easy to horizontally scale
- heterogeneuty: using virsual nodes on the ring, a server can have a proportional amount of virtual nodes on the ring with its capacioty
- only k/n nodes need to be remapped on average. Where k is the number of keys and n is the number of slots
- SHA1 is a common hashing function

### Replication

To achieve high availability and reliability, data must be replicated asynchronously over N servers, where N is a configurable parameter.

Master/Slave relatiopnship can be used for data replication in database management systems.

A master database generally is used for write operations and slave databases are used to distribute read operations. (Generally the access pattern is more reads than writes so you may have more slaves).

Advantages of database replication:
- Better performance
    - all writes and updates happen in master nodes. All reads are distributed across slave nodes. Improves performance because operations can be done in parallel
- Reliability:
    - In case a database server fails, you dont need to worry about data loss because the data is replicated across multiple locations.
    - High availability: Website remains operational even if one database is offline.

What if a database server goes offline?
- Slave failiure:
    - If there are multiple slaves: the read requests can be distributed to other slaves.
    - If only one slave: they can be performed by the master until a new slave can go online.
- Master failiure (failover):
    - A slave gets promoted to master.
    - Circular replication?
    - Multi-master replication?

### Consistency

Quorum consistency: Since data is replicated in multiple nodes, quarum consistency can help configure acceptable availability and consistency for the application

Definitions:
- N: Number of replicas
- W: A write quorum of size W means W nodes must acknowledge a write operation for it to be considered succesful
- R: A read quorum of size R means R nodes must acknowledge a read operation for it to be considered succesful
- Coordinator node: acts as a proxy between the client and the nodes to ensure W and R quorums are met

Configuration of W, N and R are a tradeoff between latency and consistency:
- R = 1 and W = N: the system is optimized for fast read
- W = 1 and R = N, the sustem is optimized for fast write
- W + R > N: Strong consistency is guaranteed
- W + R <= N: Strong consistency is not guaranteed ( there must be at least one overlapping node that has the latest data to ensure consistency )


#### Consistency models:
- strong consistency: Any read operation returns a value corresponding to the result of thwe most recent write data item. Clients never send out of date data
    - banks, meduical aplications
- weak conistency: subsequent read operations may not see the most up to date data
- eventual consistency: Specific form of weak consistency. Given enough time all nodes will return the most up to date data
    - most systems use this approach
    - tunable

Failiure detection:
- gossip protocol and heartbeat
    - nodes periodically increments a heartbeat counter to each other
    - if the heartbeat has not increased for more than a predefined amount of time the nodes will gossip with each other to confirm the counter and if it is confirmed then the node casn be marked down

Handling failiures:
- Temporary failiures:
    - in strict quorum approach, the system could temporarily disable all reads and writes
    - otherwise, sloppy quorum can be used where the first W healthy nodes are used for writes and the first R nodes are used for reads
        -     - other nodes will temporarily process requests
    - once downed sevrer is back up or replaced, all the changes and data will be pushed back to the new or old node through hinted handoff
- Data center outage:
    - this is why we must have multiple data centers in different locations to accoutn for power outages and the likes
    - users should still be able to process requests even if a data center is down

## DB Scaling

[Back to main](#contents)

Vertical Scaling
- adding more CPU, RAM, DISK, etc to an existing machine
- drawbacks:
    - there are hardware limits. If you have a large user base a single db server is not enough
    - greater risk of SPOF
    - overall cost of vertical scaling is high

Horizontal Scaling (sharding)
- adding more servers!
- separates large databases into smaller, more easily managed parts called shards. Each sshard shares the same schema although the data is unique to each shard
- Sharding key (also known as a partition key) is the most important factor to consider when implementing a sharding strategy. Determines how data is distributed
    - A sharding key allows you to retreive and modify data efficiently by routing the database queries to the correct database.
    - **One of the most important criteria is chosing a sharding key that can evenly distrubute the distributed data.**
- sharding introduces additional complexity and challenges
    - resharding data: resharding is needed when
        1. a single shard could no longer hold the data due to growth
        2. Uneven distribution of data might cause certain shards to experience shard exhaustion faster
    - when shard exhaustion happens, updating the sharding function and redistributing thew data must occur.
        - [Consistent Hashing](#consistent-hashing) is a commonly used technique to solve this problem
    - Celebrity problem: also called a hotspot key problem.
        - Exessive access to a specific shard can cause server overload. To mitigate this problem, a single or multiple shards may need to be allocated to a celebrity or hot key
    - Join and de-normalization: It is hard to perform join across a db that has been sharded. A common work around is to denormalize the shard so queries can be performed in a single table

## CAP Theorem

[Back to main](#contents)

CAP theroem states that it is impossible for a distributed system to simultaneously provide more than two of the following guarantees:
- consistency: means all clients see the same data at the same time, no matter which node they connected to
- availability: means any client which requests data gets a response even if some of the nodes are down
- partition tolerance: a partition indicates a communication break between two nodes. Partition tolerance means the system continues to operate despite network partitions

CP: Consistency and Partittion tolerance System
- Supports consistency and parition tolerance while sacrificing availability

AP: Availability and partition tolerance
- Supports availability and partition tolerance while sacrificing consistency

CA:  Network failiures are unavoidable. For a system to support consistency OR availability, it must be partition tolerant.

## Cache

[Back to main](#contents)

A cache is a temporary storage area that stores the result of expensive responses or frequemntly accessed data in memory so subsequent requests can be served more quickly.

Mitigates the problem of application performance being affected by repeated calls to the database.

Cache tier is a temporary storage layer, much faster than the database.

Pros
- Better performance
- Ability to reduce database workloads
- Ability to scale cache tier independently

Strategy:
- Read through cache: After receiving a request, the cache tier checks to see if it has the available data, otherwise it will perform the database query, stores it in cache, and sends it back to client.

Considerations for using a cache:
- When data is read frequently but modified infrequently.
- Tuning a good expoiration policy. How long the data stays in the cache before it is expired. Helps data not become stale. On the flip side, too small an expiration time results in a heavy burden on the databases.
- Consistency: This involves keeping the data store and cache in sync. Maintaining consistency is challenging.
- Mitigating failiures: A single cache server represents a single point of failiure (SPOF)
- Eviction policy: Once the cache is full, an eviction policy (such as LRU or FIFO) is used to satisfy different use cases

## CDN

Content Delivery Network

[Back to main](#contents)

A CND is a network of geographically dispersed servers used to deliver static content (CSS, HTML, JS, Images, Videos, etc).

How it works:
- When a user visits a website, a CDN server closet to the user will deliver static content. Intuitively users that are farther away from the CDN will experience slower website loads.

Consideratins of using a CDN:
- Cost: usually CDN is run by a third-party provider and you are charged for data transfers in and out of the CDN. Caching indfrequently used assets has no benefit of a CDN and should be moved out of the CDN
- Setting an appropriate cache expiry. For time-sensitive content, setting an approproate expriry time is important.
- CDN fallback: you should consider how your application deals with CDN failure. If there is a temporary CDN failiure, requests should be sent to the origin.
- Invalidating files: Use object versioning to serve a different version of the object. Maybe by adding a parameter to the url


## DNS

[Back to main](#contents)

The Domain Name System (DNS) is the phonebook of the Internet. When users type domain names such as ‘google.com’ or ‘nytimes.com’ into web browsers, DNS is responsible for finding the correct IP address for those sites.

## Data Centers

[Back to main](#contents)

Improve availability and proivide better user experience, supporting multiple data centers is crucial.

Requests can be GeoDNS routed (geo-routed) to the closest data center. GeoDNS is a service that allows domain names to be resolved to IP addresses based on the locaiton of a user.

Pros
- In the event of a data center outage, all traffic can be directed to another data center
- Provides higher availability and reduncancy

Considerations of Data Centers
- Traffic routing: using GeoDNS requests can be routed to the nearest data center to help with latency
- Data syncronization: Users from different regions could use different local databases or caches. In failover cases, the traffic could be routed to a different data center. Therefore a common strategy is to replciate the data among data centers.
- Test and deployment: With multi-data center setups, it is important to test the website/app in different locations. Automated deployment/test/build tools are crucial.

## Message Queue

[Back to main](#contents)

A message queue is a dirable component, stored in memory, that supports async communication. It serves as a buffer and distributes async requests.

The basic architecture is simple:
- Input services (called producers/publishers), create messages and publish them to a message queue.
- Other services or servers, called consumers/subscribers, connect to the queue and perform actions defined by the messages.

Pros
- Decoupling! Using message queues decouples the producer from the consumer.
    - if the consumer is out, the producer can still add messages to the queue and vise-versa
- Allows for independent scaling of producers and consumer servers
    - If the amount of jobs in the queue grows too large, the consumers can be horizontally scaled. If the queue is empty most of the time, the consumers can be scaled down

## Web-Tier

[Back to main](#contents)

Stateless Web Tier
- Keeps no information about client data
- HTTP requests can be sent to any web servers, which fetch data from a shared data store
- Moves sessiondata out of the server and into a shared data store
    - NoSQL is chosen here usually as it is easy to scale
- Pros:
    - Simpler, more robust, and scalable (compared to stetful)
    - Auto-scaling is much more simple by having the user session data in a shared data store
- Uses:
    - HTTP requests and REST APIs


Stateful Web Tier
- Remembers client data from one request to the next
- Issues:
    - Every request from the same client must be routed to the same server. Can b edone with sticky sessions in load balancerS BUT this adds overhead.
    - Adding and removing web servers is more difficult with this approach
- Uses:
    - Chat clients through Web Sockets


## Networking-HTTP-Web Sockets

[Back to main](#contents)

HTTP:
- Time-tested and most widely used web protocol
- keep alive header allows for connection to stay open and reduce number of TCP handshakes
- Great for client initiated connection

Simulating server initiated connections:
- polling
    - technique where the client periodically asks the server if there is any new data. (Busy waiting)
        - drawbacks - inefficient and resource intensive
- long polling
    - similar to polling but the client keeps the connection open until there are actually messages available or timeout is reached.
        - drawbacks - also inefficeint
- Web Sockets
    - most common solution for sending async updates from the server to the client
    - connection is initiated by the client. It is bi-directional and persistent.
    - It starts its life as an HTTP connection and could be upgraded via some well defined handshake to Web Socket connection.
    - Server could send uypdates to a client
    - works even with firewallas in palce because uses port 80 or 443 which are used by HTTP and HTTPS connections

TCP vs UDP

Transmission Control Protocol (TCP) is a communications standard that enables application programs and computing devices to exchange messages over a network. It is designed to send packets across the internet and ensure the successful delivery of data and messages over networks.

TCP is one of the basic standards that define the rules of the internet and is included within the standards defined by the Internet Engineering Task Force (IETF). It is one of the most commonly used protocols within digital network communications and ensures end-to-end data delivery.

TCP organizes data so that it can be transmitted between a server and a client. It guarantees the integrity of the data being communicated over a network. Before it transmits data, TCP establishes a connection between a source and its destination, which it ensures remains live until communication begins. It then breaks large amounts of data into smaller packets, while ensuring data integrity is in place throughout the process.

As a result, high-level protocols that need to transmit data all use TCP Protocol.  Examples include peer-to-peer sharing methods like File Transfer Protocol (FTP), Secure Shell (SSH), and Telnet. It is also used to send and receive email through Internet Message Access Protocol (IMAP), Post Office Protocol (POP), and Simple Mail Transfer Protocol (SMTP), and for web access through the Hypertext Transfer Protocol (HTTP).

An alternative to TCP in networking is the User Datagram Protocol (UDP), which is used to establish low-latency connections between applications and decrease transmissions time. TCP can be an expensive network tool as it includes absent or corrupted packets and protects data delivery with controls like acknowledgments, connection startup, and flow control.

UDP does not provide error connection or packet sequencing nor does it signal a destination before it delivers data, which makes it less reliable but less expensive. As such, it is a good option for time-sensitive situations, such as Domain Name System (DNS) lookup, Voice over Internet Protocol (VoIP), and streaming media.




## Logging Metrics and Automation

[Back to main](#contents)

Logging: Monitoring error logs is impotant because it helps identify errors and problems in your system.

Metrics: Collecting different types of metrics helps us to gain business insights and understand the health and status of our system.
- Helpful metrics:
    - Host level metrics: CPU, Memory, disk I/O, etc.
    - Aggregated level metrics: For example the performance of the dayabase tier, cache tier, etc
    - Key business metrics: Daily active users, retention, revenue, etc.

Automation: When a system gets big and complex, we need to leverage automation tools to improve productivity.
- CICD is good practice, with good testing teams can detect problems quickly.
- CICD also helps improve developer productivity (CI build/test/deploy)


## Consistent Hashing

[Back to main](#contents)

[Back to DB Scaling](#db-scaling)