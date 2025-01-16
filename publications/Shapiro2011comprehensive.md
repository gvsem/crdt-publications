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
  
#### Section 2. Background and system model

Properties of considered distributed system:
- Asynchronous network which may partition and recover
- Processes are non-byzantine, but they may crash and recover
- Process memory is durable, i.e. it survives crashes

##### 2.1 Atoms and objects

Process may store:
- **Atom** – immutable data type identified by its literal content.
  - integers, strings, sets, tuples etc. (atom types are written in lower case)
- **Object** – mutable replicated data type consisting of:
  - Identity. Objects with the same identity, but located in different processes are called *replicas*.
  - Payload. It may be composed of number of atoms or objects.
  - Initial state. Aka default constructor for object.
  - Interface. It consists of *operations*.
 
*Example (Fig 1)*. Let us consider an object with payload consisting of three atoms. It may have three replicas, every replica has its own current state of each atom.

Objects are assumed independent; transactions are not considered. Without loss of generality, we always focus on a single object at a time, thus process = replica.

##### 2.2 Operations

**Operation** is what object's interface consists of. The environment has an unspecified number of *clients* that query and update objects on an arbitrary *source* replica:
- **Query**. Executes locally, i.e. entirely at one local replica.
- **Update**. Executes in two phases:
  - **Modify part.** Operation is called at the source (possibly, with some initial processing).
  - **Downstream part.** The update is transmitted asynchronously to all replicas [4].

Some good requirements to satisfy:
- *Liveness* – any update eventually reaches the causal history of every replica, i.e. $\exists C: \forall x_i: \lim_{t \rightarrow \infty}{x_i(t)} \rightarrow C$.
- *Happens-before relation on operations* – partial order is based on causal history, operation $f$ happens-before operation $g$ if causal history of $f$ preceeds $g$, i.e. $f \rightarrow g \leftrightarrow C(f) \subset C(g)$.

##### 2.2.1 State-based replication (passive replication)

An update occurs entirely at the source, then propagates by transmitting the modified payload between replicas.

**State-based object type specification T**:
- static Payload initial() – get initial payload value
- Payload query(...) – check precondition, evaluate synchronously, no side effects
- void update(...) - check precondition, evaluate at source synchronously, side-effects at source to execute synchronously
- boolean compare(T other) – is this <= other in semilattice?
- void mergeWith(T other) - least upper bound of this and other, at any replica

Operations are executed atomically. Operations are enabled only if a given pre-condition of operation holds in the source's current state. E.g. an element can be removed from a Set only if it is in the Set at the source.

State between arbitrary pairs of replicas is transmitted in order to propagate changes between different nodes. The replica payload is updated using current local payload state, the received state and the merge operation.

**Definition 2.1.** Causal History [15] of state-based Object $C(x_i)$ of replica $x_i$:
- Initially, $C(x_i) = \emptyset$
- After executing update operation $f$: $C(f(x_i)) = C(x_i) \cup {f}$
- After executing merge against states $x_i$ and $x_j$: $C(merge(x_i, x_j)) = C(x_i) \cup C(x_j)$

NB! $C$ is a logical function, not a part of the object.

Thus, $f$ happens-before $g$ if causal history of $f$ preceeds $g$, i.e. $f \rightarrow g \leftrightarrow C(f) \subset C(g)$.

To meet the requirement of liveness, underlying system is assumed to be transmitting states between pairs of replicas at unspecified times, infinitely often, in a connected graph of replicas.

##### 2.2.2 Operation-based replication (active replication)

**Operation-based object type specification T**:
- static Payload initial() – get initial payload value
- Payload query(...) – check precondition, evaluate locally and synchronously, no side effects
- void update(...) 
  - atSource(...): check precondition, evaluate at source synchronously, no side-effects
  - downstream(...): check precondition, evaluate asynchronously, side-effects

**Definition 2.2.** Causal History of state-based Object $C(x_i)$ of replica $x_i$:
- Initially, $C(x_i) = \emptyset$
- After executing downstream phase of operation $f$ at replica $x_i$: $C(f(x_i)) = C(x_i) \cup {f}$

**Delivery order** $\le_{d}$ – order of operations' delivery where the downstream precondition is true.

**Causal delivery order** $\le_{d}$ – if $f \rightarrow g$ ($f$ happened-before $g$), then $f$ is delivered before $g$.

We note that all downstream preconditions in this paper are satisfied by causal delivery, i.e. $f \le_{d} g \Rightarrow f \le_{\rightarrow} g$

To meet the requirement of liveness, underlying system needs to have reliable broadcast that delivers every update to every replica in an order $\le_{d}$ (*delivery order*).

##### 2.3 Convergence

**Definition 2.3.** (Eventual Convergence) Two replicas $x_i$ and $x_j$ of an object $x$ converge eventually if:
- $C(x_i) = C(x_j)$ implies that states $i$ and $j$ are equivalent, i.e. all query operations return the same values (Safety)
- $f \in C(x_i) \rightarrow f \in C(x_j)$

Pairwise eventual convergence implies that any non-empty subset of replicas of the object converge, if updates are delivered to every replica.

##### 2.3.1 State-based CRDT (CvRDT)

**Definition 2.4** (Least Upper Bound on partial order $\leq_{v}$ - LUB $\sqcup_{v}$):
- commutative
- associative
- idempotent

**Definition 2.5** (Join Semilattice). An ordered set $(S, \leq_{v})$ is a Join Semilattice iff $\forall x,y \in S: \exists x \sqcup_{v} y$

**Convergent Replicated Data Type (CvRDT, monotonic semilattice)** - state-based object with payload whose values form a semilattice and defined operation $merge(x,y) \eq x \sqcup_{v} y$ which receives updates monotonically advancing upwards according to $\leq_v$:
- $compare(x, y) = x \leq_v y$
- $equals(x, y) = x \leq_v y \wedge y \leq_v x$
- $merge(x, y) = x \sqcup_{v} y$

**Proposition 2.1** (Eventual Convergence of CvRDT replicas within liveness) + Proof

##### 2.3.2 Operation-based CRDT (CmRDT)

**Definition 2.6** (Commutativity). Operations $f$ and $g$ commute, iff for any reachable replica state $S$ where their source pre-condition is enabled, the source precondition of $f$ (resp. $g$) remains enabled in state $S \dot g$ (resp. $S \dot f$), and $S \dot f \dot g = S \dot g \dot f$.

If all concurrent operations commute, then all execution orders consistent with relivery order are equivalent, and all replicas converge to the same state.

**Commutative Replicated Data Type (CmRDT)** - operation-based with commuting concurrent operations.

**Proposition 2.2** (Eventual Convergence of CmRDT replicas within liveness) + Proof

##### 2.4 Relation between the two approaches

TBC

#### 3 Portfolio of basic CRDTs

TBC

#### 4 Garbage collection

TBC

#### 5 Putting CRDTs to work

TBC

#### 6 Commutativity with previous work

TBC

#### 7 Conclusion

TBC

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
15. **Causality Relationship** R. Schwarz and F. Mattern. Detecting causal relationships in distributed computations: In search of the holy grail. Distributed Computing, 3(7):149–174, 1994.
16. **Lattices** [9] B. A. Davey and H. A. Priestley. Introduction to Lattices and Order. Cambridge University Press, 1990.

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
