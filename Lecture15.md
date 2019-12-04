# Clusters

## CAP Theorem

- theorem: no distributed system is safe from partition tolerance, so we must choose between consistency and availability
  - consistency: every read receives the most recent write (or an error)
  - availability: every request receives a non-error response
  - partition tolerance: system functions despite an arbitrary number of messages being dropped or delayed by network

## ACID vs. BASE

- utilized by RDBMS's that favor consistency over availability
  - atomicity, consistency, isolation, durability
- utilized by NoSQL databases that favor availability over consistency
  - basically available, soft state, eventual consistency

## Joining

- initialize the cluster with a seed node (e.g. via application.conf)
- new node sends message to all seed nodes
  - conveys intent to join to first responder

## PubSub

- allows sending message to an actor even though sender doesn't know which node actor is on
- actor will "subscribe" to a certain topic
- sender will "publish" to that topic 
- pubsub mediator will deliver the published message to all actors that subscribed to that topic

## Singleton

- cluster singleton is a single actor of a certain type running in the system
- singleton is a single entry point to the system (single point of failure)

## Sharding

- distribute actors across several nodes in the cluster and address them with logical identifier rather than location