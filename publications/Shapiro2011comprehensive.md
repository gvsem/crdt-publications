## A comprehensive study of Convergent and Commutative Replicated Data Types
Marc Shapiro, Nuno Preguiça, Carlos Baquero, and Marek Zawirski. Research Report 7506, INRIA, January 2011.

![](https://img.shields.io/badge/-introduction-brightgreen)

### Summary

- CRDT (Convergent or Commutative Replicated Data Types) is a novel consensus-free approach to design
replicated data structures which are proved to be eventually consistent based on simple
mathematical properties.
- A formalism for asynchronous object replication is proposed - state-based (CvRDT) and
operation-based (CmRDT). CRDT replicas provably converge to a common state which is equivalent to some correct sequential execution.
- To be continued...

### Content

#### Section 1. Introduction

- Global total order of operations [1] even in presence of faults [2] in distributed systems is researched,
  however serialisation affects perfomance and scalability.
- It is worth to imply CAP theorem [3] and rely on optimistic replication [4] (eventual consistency [5]) instead
  of consistency. No synchronization *a priori* is required, operations are sent asynchronously, every replica
  eventually applies all updates, there is no strict order. Conflicts are resolved by a background consensus algorithm
  [6], [7]. Weaker consistency is more perfomant and acceptable in some cases.
- Correctness of optimistic system when shown ad-hoc is error-prone (e.g. Amazon Shopping Cart [8])
- In this paper CRDTs as a simple, theoretically sound approach are proposed and studied. They are extremely scalable,
  fault-tolerant and consensus-free. However, these requirements have strong limitations. Trivial example of CRDT
  is G-Counter, another example is Treedoc [9]. In this paper variations on registers, counters, sets, graphs and sequences
  are proposed.
- Some CRDT designs suffer from unbounded growth, that is why garbage collection may be required. Collecting garbage
  requires a weak form of synchronization [10].
- It is wise to design CRDTs with commutative time-critical operations and delayable rare operations which require synchronization.
  It concurs with Brewer's suggestion for side-stepping the CAP impossibility [11]; and this approach is similar the shopping cart
  design by Alvaro et al. [12]: updates (as time-critical operations) commute, but reads (check-outs) require coordination.
- We migrate from **linearisability** [13] to **quiescent consistency** [14, Section 3.3]; CRDTs are weaker than non-blocking constructs
  (including lock-free and wait-free algorithms) which are generally based on a hardware consensus primitive.
- Some contributions of this paper:
  - A specification language suited to asynchronous replication (Section 2)
  - A formalisation of state-based and operation-based replication (Section 2)
  - Two sufficient conditions for eventual consistency (Section 2)
  - A comprehensive collection of useful data type designs: counters, registers,
    container types (sets and maps with add/remove clean semantics), complex types (graphs,
    monotonic DAGs, sequence) (Section 3)
  - A study of the problem of garbage-collecting meta-data. (Section 4)
  - Example of constructing a shopping cart using CRDTs (Section 5)
  - A comparison with previous work (Section 6)
  


### References

1. **Lamport Clock** [24] Leslie Lamport. Time, clocks, and the ordering of events in a distributed system. Communications of the ACM, 21(7):558–565, July 1978.
2. [8] Tushar Deepak Chandra, Vassos Hadzilacos, and Sam Toueg. The weakest failure detector for solving consensus. Journal of the ACM, 43(4):685–722, 1996.
3. **CAP theorem** [13] SethGilbertandNancyLynch.Brewer’sconjectureandthefeasibilityofconsistent,available,partition- tolerant web services. SIGACT News, 33(2):51–59, 2002.
4. **Optimistic replication** [37] Yasushi Saito and Marc Shapiro. Optimistic replication. ACM Computing Surveys, 37(1):42–81, March 2005.
5. **Eventual consistency** [41]. Werner Vogels. Eventually consistent. ACM Queue, 6(6):14–19, October 2008.
6. [4] Lamia Benmouffok, Jean-Michel Busca, Joan Manuel Marquès, Marc Shapiro, Pierre Sutra, and Geor- gios Tsoukalas. Telex: A semantic platform for cooperative application development. In Conf. Française sur les Systèmes d’Exploitation (CFSE), Toulouse, France, September 2009.
7. [40] Douglas B. Terry, Marvin M. Theimer, Karin Petersen, Alan J. Demers, Mike J. Spreitzer, and Carl H. Hauser. Managing update conflicts in Bayou, a weakly connected replicated storage system. In 15th Symp. on Op. Sys. Principles (SOSP), pages 172–182, Copper Mountain, CO, USA, December 1995. ACM SIGOPS, ACM Press.
8. **Amazon Shopping Cart** [10]. Giuseppe DeCandia, Deniz Hastorun, Madan Jampani, Gunavardhan Kakulapati, Avinash Lakshman, Alex Pilchin, Swaminathan Sivasubramanian, Peter Vosshall, and Werner Vogels. Dynamo: Amazon’s highly available key-value store. In Symp. on Op. Sys. Principles (SOSP), volume 41 of Operating Systems Review, pages 205–220, Stevenson, Washington, USA, October 2007. Assoc. for Computing Machinery.
9. **Treedoc** [32]. Nuno Preguiça, Joan Manuel Marquès, Marc Shapiro, and Mihai Leţia. A commutative replicated data type for cooperative editing. In Int. Conf. on Distributed Comp. Sys. (ICDCS), pages 395–403, Montréal, Canada, June 2009.
10. [25] Mihai Leţia, Nuno Preguiça, and Marc Shapiro. CRDTs: Consistency without concurrency control. In SOSP W. on Large Scale Distributed Systems and Middleware (LADIS), volume 44 of Operating Systems Review, pages 29–34, Big Sky, MT, USA, October 2009. ACM SIG on Operating Systems (SIGOPS), Assoc. for Comp. Machinery.
11. [6] Eric Brewer. On a certain freedom: exploring the CAP space. Invited talk at PODC 2010, Zurich, Switzerland, July 2010.
12. **Alvaro et al. Shopping Cart** [1] Peter Alvaro, Neil Conway, Joe Hellerstein, and William Marczak. Consistency analysis in Bloom: a CALM and collected approach. In Biennial Conf. on Innovative DataSystems Research (CIDR), Asilomar, CA, USA, January 2011.
13. **Linearizability** [18] Maurice Herlihy and Jeannette Wing. Linearizability: a correcteness condition for concurrent objects. ACM Transactions on Programming Languages and Systems, 12(3):463–492, July 1990.
14. **The Art of Multiprocessor Programming** [17] Maurice Herlihy and Nir Shavit. The Art of Multiprocessor Programming. Morgan Kaufmann, March 2008.

### BibTeX

```tex
@techreport{Shapiro2011comprehensive,
  author = {Shapiro, Marc and Pregui{\c c}a, Nuno and Baquero, Carlos and Zawirski, Marek},
  title = {A comprehensive study of Convergent and Commutative Replicated Data Types},
  year = {2011},
  number = {7506},
  month = jan,
  institution = {INRIA},
  type = {Research Report},
  url = {http://hal.inria.fr/inria-00555588/},
  keywords = {introduction}
}
```
