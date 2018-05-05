# Chapter 1
## Reliable, Scalable, and Maintainable Applications

With data-Intensive applications, we seek to achieve the following:  
1. Reliability  
   * The system continues to work correctly even when things go wrong
   * We cannot prevent faults entirely, only reduce them by designing fault-tolerant systems
   * The rate of faults is typically higher in fault-tolerant systems, because they can handle and are able to allow more faults
2. Scalability
   * The system should be able to handle growing data volume, traffic, or complexity 
   * Twitter example
     * Hybrid approach
       * Most tweets fanned out to home timelines at time of posting (write to mailbox example)
       * For celebrities with many followers, their tweet is fetched and merged when follower accesses their home timeline
3. Maintainability
   * The system should be easy to operate, understand, and make future changes on

# Chapter 2
## Data Models and Query Languages
* Declarative vs imperative programming
* Historically, the hiearchical model was used (data represented as one big tree), but representing many-to-many relationships was challenging, hence the invention of relational model solve that problem. But nowadays, the relational model isn't quite sufficient either.
  * NoSQL: non relational databases
    * Document databases
      * Data comes as self-contained documents and relationships between one document and another is rare
    * Graph databases
      * Good for use cases where anything is potentially related to everything

# Chapter 3
## Storage and Retrieval
* Storage engines (index structure)
    * Log-structured storage engines
      * log: an append-only sequence of records
      * index: an additional structure derived from the primary data; it serves as a signpost and speeds up read, but also write is slower because it needs to update the index as well
        * Hash index example
      * SSTable: "Sorted String Table" (hash index improvement)
          * Key-value pairs is sorted by key
          * Each key only appears once within each merged segment file
          * To keep keys sorted, can maintain a tree data structure in memory such as red-black trees or AVL trees
          * This in-memory balanced tree data structure is called a memtable
      * LSM: "Log-Structured Merge-Tree"
          * Storage engines that are based on the principle of merging and compacting sorted files (such as SSTable)
    * Page-orinted storage engines 
      * B-Trees 
        * Are the mostly widely used indexing structure
        * Standard index implementation in almost all relational databases, and many nonrelational databases as well
        * Keep key-value pairs sorted like SSTables, but have different design philosophy
          * Log-structured indexes break the database down into variable-size segments, that are some MB+ in size, and always write a segment sequentially
          * B-trees break down the database into fixed-size pages (or blocks), traditionally 4KB(+), and read and write one page at a time (the underlying hardware is also arranged in fixed-sized blocks)
        * (works sorta of like a big interval tree)
        * Branching factor: num of children pages per node (in practice, typically several hundred)
        * Index is stored on disk instead of memory
        * To handle crashes, each write operation also writes to a WAL (write-ahead log) before it is executed
          * Copy-on-write (optimized) is sometimes used instead
        * With multiple threads, latches are used to protect the tree's data structures
    * Other indexes
      * In key-value pair, the value can be the actual row (clustered index), or a reference to the row stored elsewhere (heap file)
      * Concatenated index: type of multi-column index that combines several fields into one key by appending one column to another
* Data warehouses
  * OLTP (optimized for transaction processing) vs OLAP (optimized for analytics)
  * A separate, read-only copy, of data in various OLTP systems of the company used by analysts 
    * Typically optimized for analytic access patterns
    * Star schema
      * Fact table: each row represents an event that occured at a particular time 
        * Columns are information, or foreign key references to other tables (called dimension tables)
  * Column oriented storage

# Chapter 4
## Encoding and Evolution
* Formats for encoding data
  * In memory, data is kept in data structure objects like arrays, hash tables, trees, etc...
  * When want to write data to a file or send it over the network, the data needs to be encoded into some self-contained sequence of bytes (eg. JSON, XML CSV).
  * REST vs SOAP
    * REST is a design philosophy (ie. using URL for identifying resources)
    * SOAP is an XML-based protocol for making network API requests
      * WSDL: an XML-based language that describes the API of a SOAP web service
  * RPC: remove procedure calls
    * Tries to make a request to a remote network serivce look like local procedure calls
    * Lots of flaws, since a network request behaves very differently from a local function call
    * Frequently used for the case of requests between services within same datacenter

# PART 2
## Distributed data
* Vertical vs horizontal scaling
* Horizontal scaling
  * Data is distributed across multiple nodes in two common ways:
    1. Replication: keep a copy of the same data on several different nodes
    2. Partitioning: splitting the database into smaller subsets called partitions, which is then assigned to different nodes (sharding) 
  
# Chapter 5
## Replication
* Reasons to replicate data:
  * Keep data geograpically close to users
  * Fault tolerance
  * Increase read throughput by having more machines that can serve reads
* (In chapter 5, we assume each machine can hold a copy of the entire dataset)
* Popular algorithms for replicating changes between nodes:
  1. Single-leader: a replica is designated as leader
    * All write requests are done by the leader, which also sends the data change to all its followers, as part of a replication log or change stream. Each follower processes the update. Reads can be done on any leader or follower, but writes are only accepted on the leader.
      * Often asynchronous, ie. leader sends message to followers, but doesn't wait for response before reporting success to user
      * Setting up new followers
        * Take consistent snapshot of leader's database and copy to new follower node. Follower connects to leader and requests all data changes that have happened since snapshot was taken. When follower has processed backlog of the data changes, it has caught up, and can process new changes from leader as they happen.
    * Follow failure: catch up recover
      * Follower detects changes since it failed, connects to leader, updates all changes
    * Leader failure: failover
      * Election of new leader
      * Can go wrong in several ways, due to mismatched data, conflicts between old vs new leader, discarding old leader writes, etc.
    * Replication logs
      1. Statement-based replication
        * Because many edge cases, other replication methods are generally preferred
      2. Write-ahead (WAL) log shipping
        * Used by PostgreSQL
      3. Logical (row-based) replication
        * Logical log: a sequence of records describing writes to database tables at the granularity of a row
       * Used by MySQL binlog
    * Replication lag
      * Delay in replication until eventual consistency
      * Read-after-write consistency: guarantees that if a user writes and reloads the page, they should see the update they just submitted
        * Impl 1: always read own profile from leader, and other users' profiles from a follower
        * Impl 2: client can remember timestamp of most recent write, and system serving reads for user ensures updates at least until that timestamp (otherwise let another replica handle read, or delay query until caught up)
      * Cross-device read-after-write consistency: if user enters info on one device, they should be able to view the info on another device immediately
      * Monotonic reads: if user makes several reads in sequence, they should not see time go backwards due to asynchronous updates
        * Impl 1: each user reads their reads from same replica
      * Consistent prefix reads: anyone reading a sequence of writes will see them in the same order
  2. Multi-leader
    * Extensiion of single leader model, and allows multiple nodes to accept writes. Each leader is simultaneously a follower to another leader.
    * Use cases
      * Multi-datacenter operation: each datacenter has its own leader. Replication between datacenters is lead by each database leader. Has advantages in performance and reliability, but one big downside. The same data may be concurrently modified in two different datacenters, and write conflicts must be resolved. In addition, it may cause issues with features like autoincrementing keys, triggers, and integrity constraints. Multi-leader replication is often dangerous and should be avoided.
      * Clients with offline operation: changes that are made offline are synced with a server when online again. With multiple devices, each device has a local database that acts as leader. The sync process between all devices for application is done via asynchronous multi-leader replication.
      * Collaborative editing
    * Handling write conflicts
      * The biggest problem with multi-leader replication is that write conflicts can occur, which means that conflict resolution is required
      * Conflict avoidance: preferred, since many implementations of multi-leader replication handle conflicts poorly. One strategy is to make sure requests from a particular user is always routed to and use the same datacenter leader for reading and writing. 
      * Converging towards consistent state: 
    * Multi-Leader Replication Topologies
      * How writes are proprogated from one leader to another
        * Circular: MySQL, bad if one node fails
        * Star: bad if one node fails
        * All-to-all: most general: bad if some network links faster than others (may cause misordering in write  replication). Use version vectors to order events correctly.
  3. Leaderless
    * Dynamo, Riak, Cassandra, Voldemort
    * Failover does not exist. Ie. if a portion of (w)rite/(n)nodes is successful, call that success. Then when client read from one replica, the request is also sent to several nodes in parallel, and newest value is returned.
      * (w+r) > n, can expect to get up-to-date value when reading
        * Reads/writes that follow this rule are called "quorum" reads and writes

# Chapter 6
## Partioning
* Key-value partitioning
  * Range partioning
    * Hot spots
  * Hash partitioning
    * eg. Fowler– Noll–Vo function(Cassandra and MongoDB), MD5 (Voldemort)
* Key-value partioning (with secondary index)
  * Document based partitioning
    * Each partition maintains its secondary indexes independently of others (local index). 
    * Reading is scatter-gather since still need to send query to all partitions, then combine results. As such, read queries on secondary indexes can be expensive. 
  * Term-based partitioning
    * A global index is partitioned and becomes the secondary indexes. 
    * Writes are now more complicated, since a write could affect multiple partitions (eg. each updating term is on a different node). Updates to the secondary indexes are often asychronous. 
* Rebalancing partitions
  * Strategies
    * Example of bad strategy: change hash key to (hash mod N). This moves the keys around a lot and makes rebalancing expensive.
    * Fixed number of nodes: create more partitions than there are nodes, with each node having approximately equal number of partitions (can be difficult to pick out "perfect" number if data size is highly variable). When adding new node, it can plucks existing partitions away from current nodes. When removing partition, it distributes its partitions amongst the current nodes. Size of partitions proportional to size of dataset.
    * Dynamic partitioning (for key-range partitions): when partition exceeds configured size, split into two partitions. Number of partitions is proportional to size of dataset.
    * Partition proportional to nodes: have a fixed number of partitions per node. Size of each partition grows as datasize increases, but number of nodes remain unchanged. When increase number of nodes, the partitions become smaller again.
  * Request Routing
    * Client can get forwarded to correct node to handle request, go through routing tier, or connect with node directly 

# Chapter 7
## Transactions
* Transaction: one logical unit of several reads/writes. Atomic, either the entire thing succeds or fails
* ACID: atomicity, consistency, isolation, durability
* Transaction isolation
  * Read commited: no dirty reads, no dirty writes (can only see/overwrite commited data)
  * Preventing dirty writes: often implemented by using row-level locks
  * Preventing dirty reads: for every object written, remember new and committed value. While transaction is ongoing, read old value. When transaction complete, read new value.
* Nonrepeatable read or read skew
  * Read at end of transaction is different than saw in previous query
  * Eg. Alice has $1000 in two bank accounts. If she transfers $100 from account1 to account2, and she views the transaction as it is being processed, she may only see $900 ($100 has disappeared).
  * Example of places where read skew is intolerable
    * Backups: may end up with parts of backup containing older/newer versions of data
    * Analytic queries and integrity checks: may return nonsensical queries if observing database at different times
  * Solution is to use snapshot isolation
    * Each transaction reads from consistent snapshot of database, ie. database data ending at start of transaction
    * Implementation
      * Write locks to prevent dirty writes
      * Readers never block writers, writers never block readers
      * Multiple version concurrency control (MVCC)
    * Snapshot isoluation is also called serializable (Oracle), repeatable read (PostgreSQL/MySQL)
* SELECT * FOR UPDATE : explicit row locking
* Write skew
* Serializable isolation : regarded as the strongest isolation level. Guarantees that even if transactions execute in parallel, end result is the same as if they had executed serially concurrency
  * Implementataions
    * Actual serial execution
      * Doesn't scale well
    * Two-phase locking 
      * Doesn't perform well
    * Optimistic concurrency control techniques
      * Serializable snapshot isolation (SSI)