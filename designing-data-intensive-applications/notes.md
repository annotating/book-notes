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
    