# Hadoop

## Architecture

- master-slave topology with one master node and multiple slave nodes
  - master node assigns tasks to slave nodes and manages resources
  - slave nodes do the actual computing and store real data

## Principles

- data is distributed around the network and replicated to support fault tolerance
- computation is sent to the data (i.e. code to execute is sent to the slave nodes)

## Fault Tolerance

- worker failure is assumed when worker stops responding to master's periodic pings
- if a master fails a new master can be restarted from the last checkpoint
  - HDFS data replication ensures that data isn't lost in either case
