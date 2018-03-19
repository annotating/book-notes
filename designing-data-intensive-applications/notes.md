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