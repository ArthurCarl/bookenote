# Chapter 20 HBase
## HBasics
HBase is a Distributed column-oriented database built on top of HDFS.HBase is the Hadoop application to use when you require real-time read/write random access to very large datasets.

HBase is not RDBMS and does not support SQL,it is able to do what an RDBMS can't:host very large,sparsely populated tables on cluster made from commodity hardware.

### Backdrop

## Concepts

### Whirlwind Tour of the Data Model
Tables are made of rows and columns. Table cells are versioned.

A table's colum families must be specified up front as part of the table schema definition,but new column family members can be added on demand.All column family members are stored togetherd on the filesystem.Because tunning and storage specifications are done at the column family level,it is advised that all column family members have the same general access pattern and size characteristics.

Tables are automatically partitioned horizontally by HBase into **regions**.

Row updates are atomic,no matter how many row columns constitute the row-level transaction.

### Implementation
HBase is made up of an HBase `master` node orchestrating a cluster of one or more `regionserver` workers.

The HBase master is responsible for bootstrapping a virgin install,for assigning regions to registered regionservices,and for recovering regionserver failures.

The regionservers carry zero or more regions and field client read/write requests.They also manage regions splits,informing the HBase master about the new daughter regions so it can manage the offlining of parent regions and assignment of the replacement daughters.

HBase persists data via the Hadoop filesystem API.

HBase keeps a special catalog table named `hbase:meta`,within which it maintains the cuurent list,state,and locations of all user-space regions afloat on the cluster.Entries in `hbase:meta` are keyed by region name,where a region name is made up of the name of the table the region belongs to,the region's start row,its time of creation ,and finally,an MD5 hash of all of these.

To save on having to make three round-trips per row operation,clients cache all they learn while doing lookups for `hbase:meta`.They cach locations as well as uer-space region start and stop rows,so they can figure out hosting regions themselves without having to go back to the `hbase:meta` table.Clients constinue to use the cached entries as they work,until there is a fault.When this happens,the client consults the `hbase:meta` table again to learn the new location.If the consulted `hbase:meta` region has moved,then ZooKeeper is consulted.
## Installation
1. `% tar xzf hbase-x.y.z.tar.gz`
2. `% export HBASE_HOME=~/sw/hbase-x.y.z`  
`% export PATH=$PATH:$HBASE_HOME/bin`
3. `% hbase`

### Test Drive
1. `% start-hbase.sh` # start a standalone instance of HBase.
2. `% hbase shell` # To administer HBase
3. `hbase(main):001:0> create 'test','data'` # Tabel `test`,column family `data`
4. `hbase(main):002:0>list` # Prove the new table was created.
5. `hbase(main):003:0>put 'test','row1','data:1','value:1'` #insert  
`hbase(main):004:0>put 'test','row2','data:2','value:2'`  
`hbase(main):005:0>put 'test','row3','data:3','value:3'`  
`hbase(main):006:0>get 'test','row1'` # query  
`hbase(main):007:0>scan 'test'`
6. `hbase(main):009:0>disable 'test'`#disable before drop table.  
`hbase(main):010:0>drop 'test'`  
`hbase(main):011:0>list`
7. `% stop-hbase.sh` #shut down HBase  

## Clients


### Java
`Scanner` like tranditional database cursor or Java iterators; it return rows in order.
### MapReduce

### REST and Thrift

## Building an Online Query Application
1. Datasets is massive ,update frequently.
2. Application display the most up-to-date observation a second or so of receipt

### Schema Design
Two tables :
1. `stations` `observations`  
`stations` row key: `stationid` , column family:`info`-key-value dictionary,dictionary keys:`info:name` `info:location` `info:description`  
`observations` row key:`stationid`+timestamp, column family:`data`-`airtemp`

1. `hbase(main):001:0>create 'stations',{NAME => 'info'}`  
`hbase(main):002:0> create 'observations' ,{NAME=>'data'}`

### Loading Data
`habase HBaseStationImporter input/ncdc/metadata/stations-fixed-width.txt`

Bulk load
two-step process:
1. `HFileOutputFormat2` write HFiles to an HDFS directory using a MapReduce Job
2. Move HFiles from HDFS into an HBase table
### Online Queries


## HBase Versus RDBMS

### Successful Service

### HBase

## Praxis

### HDFS

### UI

### Metrics

### Counters
































---
