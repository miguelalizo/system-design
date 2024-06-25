# Reliability Scalability and Maintainability


## Reliability

Making systems that work correctly even when faults occur (fault tolerance)
- faults can be:
    - bugs, human error, hardware fault

## Scalability

Keeping performance good even when load increases

### Load parameters
 
Used to quantitatively measure load and performance

Choice of load parameters depends on the architecture of the system: may be requests per second to a web server, ratio of reads to writes in a database, number of simultaneously active users in a chatroom, etc

- throughput: number of records that can be processed per second or total time it takes to run a job on a dataset of a certain size
- response time: tuime between client sending a request and receiving a response

Fanout: in transaction processing systems, this term is used to describe the number of requests to other services that we need to make in order to serve one upcoming request

## Maintainability

Making life better for the engineering and operations teams who need to work with the system.
- good visibility into the systems health (loggiong)
- good abstractions that reduce complexity

- operability: make it easy for operations teams to keep the system running smoothly
- simplicity: make it easy for new engineers to understand the system
- evolvability: make it easy for engineers to make changes