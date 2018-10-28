# Chapter20-ZooKeeper
Hadoop’s distributed coordination service, called ZooKeeper

ZooKeeper characteristics:
1. simple
2. expressive
3. highly available
4. facilitates loosely coupled interactions
5. is a library

## Installing and Running ZooKeeper
1. `% tar xzf zookeeper-x.y.z.tar.gz`
2. `% export ZOOKEEPER_HOME=~/sw/zookeeper-x.y.z`  
`% export PATH=$PATH:$ZOOKEEPER_HOME/bin`
3. `zoo.cfg`  
`tickTime=2000  
dataDir=/Users/tom/zookeeper
clientPort=2181`  
`% zkServer.sh start`
4. `% echo ruok | nc localhost 2181` # check ZooKeeper is running

## An Example

### Group Membership in ZooKeeper
`znode` act as a container of data(like file) and a container of other znodes(like directory)
### Creating the Group

### Joining a Group

### Listing Members in a Group

### Deleting a Group

## The ZooKeeper Service

### Data Model


### Operations

### Implementation

### Consistency

### Sessions

### States

## Building Application


### A Configuration Service


### The Resilient ZooKeeper Application

### A Lock Service

### More Distributed Data Strucures and Protocols

## ZooKeeper in Production

### Resilience and Performance

### Configuration

## Further Reading


































---
