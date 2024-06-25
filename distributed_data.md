# Distributed Data

There are various reasons why to distribute database across multiple machines:
- scalability:
    - if your data volume, read load or write load grows bigger than a single machine can handle, you can potentially spread the load across different machines
- fault tolerance/high availability:
    - application needs to continue working even if one machine goes down. Provides redundancy
- latency:
    - If you have users around the world you might want to havbe servers at various locations worldwide so that each user can be served from a datacenter that is geographically close to them (avoids having to wait for a data packet to travel halfway around the world)

## Scaling up
Also called vertical scaling
- Throws more compute power at a problem (more cpu, more ram, etc) on a single machine
- the cost of scaing up grows faster than linearly: more ram, more cpu, etc typically can cost more than just higher count of avg machines
- offers limited fault tolerance
- limited to a single geographic location (high latency)

## Shared nothing architecture

Also called horizontal scaling or scaling out. 

- each machine or virtual machine is called a node
- each node uses its own CPU, disks, RAM, etc.
- coordination between nodes is done at the SW level, using a ocnventional network

- no special hardware is required in this architecture -> use whatever machines give the optimal price to performance ratio
- gives the ability to distribute data across multiple geographic locations (reduce latency)

## Replication vs partitioning (of data)

Two common ways data is distributed across multiple nodes:

1. replication
    - keeping a copy of the same data on diferent nodes and potentially diffewrent locations. 
        - Provides redundancy (data can be served by available nodes)
        - helps improve performance. How?
2. partitioning
    - splits a database into smaller subsets called partitions so that different partitions can be assigned to diffeerent nodes (sharding). how?
    