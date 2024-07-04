# System Design Review

## Contents

1.  [System Design Framework](#system-design-framework)
1.  [Vert vs Horz scaling](#vertical-vs-horizontal-scaling)
1.  [Latency vs Throughput](#latency-vs-throughput)
1.  [Load balancer](#load-balancer)
1.  [* DBs](#databases)
    - [SQL](#sql)
        - [ACID](#acid)
    - [NoSQL](#nosql)
1.  [DB-Partitioning-Replication Consistency-Availability](#db-partitioning-replication-consistency-availability)
    - [DB Vertical Scaling](#db-vertical-scaling)
    - [DB Horizontal Scaling](#db-horizontal-scaling)
    - [Partitioning](#partitioning)
    - [Replication](#replication)
    - [Consistency](#consistency)
    - [Availability](#availability)
1.  [CAP Theroem](#cap-theorem)
1.  [Cache](#cache)
1.  [CDN](#cdn)
1.  [DNS](#dns)
1.  [Reverse Proxy](#reverse-proxy)
1.  [Application Layer](#application-layer)
1.  [Data centers](#data-centers)
1.  [* Asynchronism Message Queue](#message-queue)
1.  [Web Tier; stateless vs stateful](#web-tier)
1.  [* Networking / HTTP / Web Sockets](#networking-http-web-sockets)
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

## Latency-vs-throughput

[Back to main](#contents)

Latency is the time to perform some action or to produce some result.

Throughput is the number of such actions or results per unit of time.

Generally, you should aim for maximal throughput with acceptable latency.

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

### SQL

Relational Database Management System (RDBMS)

A relational database like SQL is a collection of data items organized in tables.

#### ACID

ACID is a set of properties of relational database transactions.
- Atomicity - Each transaction is all or nothing
    - Atomicity ensures that each transaction is treated as a single, indivisible unit of work. This means that either all operations within the transaction are completed successfully and committed, or if any operation fails, the entire transaction is rolled back (undone) to maintain data consistency.
    - For example, in a banking transaction where money is transferred from one account to another, atomicity ensures that if the withdrawal from one account succeeds, the deposit to the other account will also succeed. If either operation fails, the entire transaction (both operations) is aborted.
- Consistency - Any transaction will bring the database from one valid state to another
    - Consistency guarantees that the database remains in a consistent state before and after the execution of any transaction. This means that transactions cannot violate any integrity constraints or rules defined for the database schema.
        - **Note: This refers to a single DB not replication consistency**
     - Database consistency ensures that all data written to the database is valid according to all defined rules, including constraints, cascades, and triggers. It prevents transactions from leaving the database in an invalid state.
- Isolation - Executing transactions concurrently has the same results as if the transactions were executed serially
    - Isolation ensures that multiple transactions can execute concurrently without affecting each other. Each transaction should operate independently as if it were the only transaction being executed on the database.
    - Isolation prevents interference between transactions, such as ensuring that one transaction's intermediate state is not visible to another transaction until it is committed. This is achieved through techniques such as locking, concurrency control, and isolation levels.
- Durability - Once a transaction has been committed, it will remain so
    - Durability guarantees that once a transaction has been committed, it will persist even in the event of a system failure (such as power outage, crash, or error). Committed transactions should not be lost and should survive any form of system failure.
    - This is typically achieved by writing transaction changes (committed data) to non-volatile storage (usually disk) in a manner that ensures data persistence. Even if the database system crashes immediately after a transaction is committed, the changes made by the transaction should still be available when the system is recovered.

- Stores data in tables and rows
- Supports joins across tables and supprots many-to-many relationships
- Examples:
    - MySQL
    - PostgreSQL
    - Oracle

#### Techniques for scaling RDBMS
- There are many techniques to scale a relational database:
    - [master-slave replication](#master-slave-replication)
    - [master-master replication](#master-master-replication)
    - [federation](#federation)
    - [sharding](#sharding)
    - [denormalization](#denormalizing)
    - [SQL tuning](#sql-tuning)

### NoSQL

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

### DB Vertical Scaling

- adding more CPU, RAM, DISK, etc to an existing machine
- drawbacks:
    - there are hardware limits. If you have a large user base a single db server is not enough
    - greater risk of SPOF
    - overall cost of vertical scaling is high

### DB Horizontal Scaling

- [Replication](#replication)
    - Master-slave
    - Master-master
- [Partitioning](#partitioning)
    - Federation
    - Sharding
- [Denormalizing](#denormalizing)
- [SQL Tuning](#sql-tuning)

### Partitioning

(Horizontally scaling)

For large applications, it is infeasable top fit the entire dataset in a single server. The simplest solution is to split the data into smaller partitions and store them in multiple servers.

There are two challenges when partitioning data:
- how to distribute data evenly across the servers
- minimize the data movement when nodes are added or removed

#### Federation

Federation (or functional partitioning) splits up databases by function.

For example, instead of a single, monolithic database, you could have three databases: forums, users, and products, resulting in less read and write traffic to each database and therefore less replication lag. Smaller databases result in more data that can fit in memory, which in turn results in more cache hits due to improved cache locality. With no single central master serializing writes you can write in parallel, increasing throughput.

Disadvantage(s): federation
- Federation is not effective if your schema requires huge functions or tables.
- You'll need to update your application logic to determine which database to read and write.
- Joining data from two databases is more complex with a server link.
- Federation adds more hardware and additional complexity.


#### Sharding

Horizontal Scaling (sharding)
- adding more servers!
- separates large databases into smaller, more easily managed parts called shards. Each shard shares the same schema although the data is unique to each shard
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

##### Consistent Hashing
**Consistent Hashing** solves these problems:
- servers are placed on a hashring
- key is hashed onto the ring and the data is stored in the first server encountered while moving in clockwise direction

Advantges of consistent hashing:
- automatic scaling: servers can be added or removed depending on additional load; easy to horizontally scale
- heterogeneuty: using virsual nodes on the ring, a server can have a proportional amount of virtual nodes on the ring with its capacioty
- only k/n nodes need to be remapped on average. Where k is the number of keys and n is the number of slots
- SHA1 is a common hashing function

#### Denormalizing

Denormalization attempts to improve read performance at the expense of some write performance. Redundant copies of the data are written in multiple tables to avoid expensive joins. Some RDBMS such as PostgreSQL and Oracle support materialized views which handle the work of storing redundant information and keeping redundant copies consistent.

Once data becomes distributed with techniques such as federation and sharding, managing joins across data centers further increases complexity. Denormalization might circumvent the need for such complex joins.

In most systems, reads can heavily outnumber writes 100:1 or even 1000:1. A read resulting in a complex database join can be very expensive, spending a significant amount of time on disk operations.

Disadvantage(s): denormalization
- Data is duplicated.
- Constraints can help redundant copies of information stay in sync, which increases complexity of the database design.
- A denormalized database under heavy write load might perform worse than its normalized counterpart.


#### SQL Tuning

SQL tuning refers to the process of optimizing SQL queries and database performance to improve efficiency, reduce execution time, and enhance overall system responsiveness. Here’s a quick summary of SQL tuning:

 1. Identifying Bottlenecks: Analyzing the performance of SQL queries to identify bottlenecks, such as slow-running queries or resource-intensive operations.

2. Query Optimization: Rewriting SQL queries to improve efficiency, using techniques like:
- Restructuring queries to minimize unnecessary joins or subqueries.
- Optimizing WHERE clauses and using appropriate indexes to speed up data retrieval.
- Avoiding unnecessary data sorting or aggregations.

3. Indexing: Creating and maintaining appropriate indexes on tables to facilitate quick data access and query optimization. This includes understanding different types of indexes (e.g., B-tree, hash indexes) and their impact on query performance.

4. Statistics Management: Ensuring that database statistics are up-to-date to help the query optimizer generate efficient execution plans. This involves regularly updating statistics on tables and indexes.

5. Database Schema Design: Designing an efficient database schema that minimizes data redundancy and optimizes query performance. This includes normalization and denormalization strategies based on specific use cases.

6. Query Execution Plan: Understanding and analyzing the query execution plan generated by the database optimizer. This helps in identifying potential performance issues and optimizing query execution paths.

7. Monitoring and Profiling: Monitoring database performance metrics and profiling SQL queries to capture and analyze execution times, resource consumption, and inefficiencies.

8. Application Design Considerations: Optimizing SQL queries should also consider application design patterns and access patterns to ensure that database interactions align with application requirements and performance goals.

### Replication

To achieve high availability and reliability, data must be replicated asynchronously over N servers, where N is a configurable parameter.

#### Master Slave Replication

Master/Slave relatiopnship can be used for data replication in database management systems.

A master database generally is used for write operations and slave databases are used to distribute read operations. (Generally the access pattern is more reads than writes so you may have more slaves).

#### Master-Master Replication

Both masters serve reads and writes and coordinate with each other on writes. If either master goes down, the system can continue to operate with both reads and writes.

Disadvantage(s): master-master replication
- You'll need a load balancer or you'll need to make changes to your application logic to determine where to write.
- Most master-master systems are either loosely consistent (violating ACID) or have increased write latency due to synchronization.

Advantages of database replication:
- Better performance
    - all writes and updates happen in master nodes. All reads are distributed across slave nodes. Improves performance because operations can be done in parallel
- Reliability:
    - In case a database server fails, you dont need to worry about data loss because the data is replicated across multiple locations.
    - High availability: Website remains operational even if one database is offline.

Disadvantages of replication:
- There is a potential for loss of data if the master fails before any newly written data can be replicated to other nodes.
- Writes are replayed to the read replicas. If there are a lot of writes, the read replicas can get bogged down with replaying writes and can't do as many reads.
- The more read slaves, the more you have to replicate, which leads to greater replication lag.
- More hardware and additional complexity

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

### Availability

Two complimentary patterns to support high availability:
1. Fail-over
2. Replication

1. Fail-over (2 types)
- Active-passive:
    - With active-passive fail-over, heartbeats are sent between the active and the passive server on standby. If the heartbeat is interrupted, **the passive server takes over the active's IP** address and resumes service.
    - The length of downtime is determined by whether the passive server is already running in 'hot' standby or whether it needs to start up from 'cold' standby. Only the active server handles traffic.
    - **Active-passive failover can also be referred to as master-slave failover.**
- Active-active:
    - In active-active, both servers are managing traffic, spreading the load between them.
    - If the servers are public-facing, the DNS would need to know about the public IPs of both servers. If the servers are internal-facing, application logic would need to know about both servers.
    - Active-active failover can also be referred to as master-master failover.

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

A CND is a network of geographically dispersed proxy servers used to deliver static content (CSS, HTML, JS, Images, Videos, etc).

**Amazon has a DNS service: CloudFront**

How it works:
- When a user visits a website, a CDN server closet to the user will deliver static content. Intuitively users that are farther away from the CDN will experience slower website loads.

Push CDNs
    - Push CDNs receive new content whenever changes occur on your server. You take full responsibility for providing content, uploading directly to the CDN and rewriting URLs to point to the CDN. You can configure when content expires and when it is updated. Content is uploaded only when it is new or changed, minimizing traffic, but maximizing storage.
    - Sites with a small amount of traffic or sites with content that isn't often updated work well with push CDNs. Content is placed on the CDNs once, instead of being re-pulled at regular intervals.
- Pull CDNs
    - Pull CDNs grab new content from your server when the first user requests the content. You leave the content on your server and rewrite URLs to point to the CDN. This results in a slower request until the content is cached on the CDN.
    - A time-to-live (TTL) determines how long content is cached. Pull CDNs minimize storage space on the CDN, but can create redundant traffic if files expire and are pulled before they have actually changed.
    - Sites with heavy traffic work well with pull CDNs, as traffic is spread out more evenly with only recently-requested content remaining on the CDN.

Consideratins of using a CDN:
- Cost: usually CDN is run by a third-party provider and you are charged for data transfers in and out of the CDN. Caching indfrequently used assets has no benefit of a CDN and should be moved out of the CDN
- Setting an appropriate cache expiry. For time-sensitive content, setting an approproate expriry time is important.
- CDN fallback: you should consider how your application deals with CDN failure. If there is a temporary CDN failiure, requests should be sent to the origin.
- Invalidating files: Use object versioning to serve a different version of the object. Maybe by adding a parameter to the url


## DNS

[Back to main](#contents)

The Domain Name System (DNS) is the phonebook of the Internet. When users type domain names such as ‘google.com’ or ‘nytimes.com’ into web browsers, DNS is responsible for finding the correct IP address for those sites.

Services such as CloudFlare and **Amazon's Route 53** provide managed DNS services. Some DNS services can route traffic through various methods:

    Weighted round robin
        Prevent traffic from going to servers under maintenance
        Balance between varying cluster sizes
        A/B testing
    Latency-based
    Geolocation-based

Disadvantage(s): DNS
- Accessing a DNS server introduces a slight delay, although mitigated by caching described above.
- DNS server management could be complex and is generally managed by governments, ISPs, and large companies.


## Reverse Proxy

[Back to main](#contents)

A reverse proxy is a web server that centralizes internal services and provides unified interfaces to the public. Requests from clients are forwarded to a server that can fulfill it before the reverse proxy returns the server's response to the client.

Additional benefits include:

- Increased security - Hide information about backend servers, blacklist IPs, limit number of connections per client
- Increased scalability and flexibility - Clients only see the reverse proxy's IP, allowing you to scale servers or change their configuration
- SSL termination (the process of decrypting encrypted traffic before passing it along to a web server) - Decrypt incoming requests and encrypt server responses so backend servers do not have to perform these potentially expensive operations
    - Removes the need to install X.509 certificates on each server
- Compression - Compress server responses
- Caching - Return the response for cached requests
- Static content - Serve static content directly
    - HTML/CSS/JS
    - Photos
    - Videos
    - Etc

Load balancer vs reverse proxy
- Deploying a load balancer is useful when you have multiple servers. Often, load balancers route traffic to a set of servers serving the same function.
- Reverse proxies can be useful even with just one web server or application server, opening up the benefits described in the previous section.
- Solutions such as NGINX and HAProxy can support both layer 7 reverse proxying and load balancing.

Disadvantage(s): reverse proxy
- Introducing a reverse proxy results in increased complexity.
- A single reverse proxy is a single point of failure, configuring multiple reverse proxies (ie a failover) further increases complexity.


## Application Layer

[Back to main](#contents)

Separating out the web layer from the application layer (also known as platform layer) allows you to scale and configure both layers independently. Adding a new API results in adding application servers without necessarily adding additional web servers. The single responsibility principle advocates for small and autonomous services that work together. Small teams with small services can plan more aggressively for rapid growth.

Microservices
- Related to this discussion are microservices, which can be described as a suite of independently deployable, small, modular services. Each service runs a unique process and communicates through a well-defined, lightweight mechanism to serve a business goal. 1
- Pinterest, for example, could have the following microservices: user profile, follower, feed, search, photo upload, etc.

Disadvantage(s): application layer
- Adding an application layer with loosely coupled services requires a different approach from an architectural, operations, and process viewpoint (vs a monolithic system).
- Microservices can add complexity in terms of deployments and operations.


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