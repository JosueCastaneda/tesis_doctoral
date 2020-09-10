# Strong eventual consistency (SEC)

## Definitions

### Eventual consistency [source](https://pages.lip6.fr/Marc.Shapiro/papers/RR-7687.pdf)

We need three properties for EC

1. Eventual delivery: An update executed at a correct replica eventually executes at all correct replicas.
2. Termination: All update executions terminate.
3. Convergence: Correct replicas that have executed the same updates eventually reach equivalent state (and stay)

Pure liveness guarantees, no safety. This does not preclude diverging, rolling back, reconciling.
You never know **when** you reach consistency

### Strong eventual consistency [source](https://pages.lip6.fr/Marc.Shapiro/papers/RR-7687.pdf)

We need three properties for EC

1. Eventual delivery: An update executed at a correct replica eventually executes at all correct replicas.
2. Termination: All update executions terminate.
3. **Strong** Convergence: Correct replicas that have executed the same updates **have** equivalent state.

No rollbacks, every update is immediatly persistent (durable). Another way of looking at it whenever two nodes see the same events they are in the same state.

## Properties [source](https://youtu.be/oyUHd894w18?t=1025)

### Local update
* No synchronisation
* Fast, responsive

### Solves the CAP problem
* (Strong eventual) consistency
* Available
* Partition-tolerant (up to n-1 failures)

### Does not require consensus
* Cheap
* Not universal (Not all problems can be solved using SEC)


## Syncronization approaches. [source](https://pages.lip6.fr/Marc.Shapiro/papers/RR-7687.pdf)

### State-based
Send complete state asynchronously over the network and merge with states and works if it converges.

Sufficient condition to converge: The data has to be a monotonic semi-lattice (a set with a partial order with an operation that can take two values that gives the upper bound of the two values).

1. Payload of messages for a semi-lattice.
2. Updates are increasing.
3. merge computes Least Upper Bound.

An example is the increment and your merge is your max of the two values then it works.

Think of git auto-merging (without conflict). Nice for **reasoning**.

### Operation-based

Whenever there is an operation, send the update to everyone (similar to the brodact for causal delivery using vector clocks). Don't send the whole state (less overhead).

Difference between state and operation based, is the amount of data sent and the recency. For operation you need to send the information after the update, with the state is less restringent. Nice for **production**.

Sufficient conditions for replicas to converge:
1. (Liveness) all replicas execut all operations in delivery order
2. (Safety) all concurrent operations commute.

### Equivalence between state and operation based approaches. [source](https://youtu.be/oyUHd894w18?t=1762)

1. A state-based object can emulate an operation-based object, and vice-versa.
2. State-based emulation on an op-based CRDT is a CRDT
3. Operation-based emulation on an state-based CRDT is a CRDT

Monothonic semi-lattice is equivalent to commute and vice-versa.

### Combining two CRDTs: Sharded CRDT.

If you take two CRDTs and combine them, the result will also be a CRDT if partitions are static.

### SEC and blockchain [source](https://youtu.be/B5NULPSiOGw?t=867)  

Blockchains: Total order with byzantine consensus (to prevent double-spending) which means picking **one** of several proposed values. 

Collaboration (like in CRDT) you want to pick **all** proposed values and merge them.


### Examples of CRDT:

Counter: [source](https://youtu.be/9xFfOhasiOE?t=298)


### Drawbacks of CRDTs:

* Can be implemented badly.
* Metadata overhead [source](https://github.com/automerge/automerge-perf/blob/master/columnar/README.md)
* Specific to application. Difficult to generalize, since the CRDT limits what an application can employ [source] (https://queue.acm.org/detail.cfm?id=2462076)

# Consistency As Logical Monotonicity (CALM) [source](https://cacm.acm.org/magazines/2020/9/246941-keeping-calm/fulltext)

## Questions:
* Does my program produce deterministic outcomes despite non-determinism in the runtime system?
* Which programs can be consistenly computed while remaining available under partition?

## Monotonic definition
A program P is monotonic if for any input sets *S*,*T* where *S* ⊆ *T*, *P(S) ⊆ P(T)* 

## Confluence
In the context of nondeterministic message delivery, an operation on a single machine is confluent if it produces the same set of outputs for any nondeterministic ordering and batching of a set of inputs. Following our discussion of sets of information S and T above, a confluent single-machine operation can be viewed as a deterministic function from sets to sets, abstracting away the nondeterministic order in which its inputs happen to appear in a particular run of a distributed system. 

Confluence makes no requirements or promises regarding notions of recency (e.g., a read is not guaranteed to return the result of the latest write request issued) or ordering of operations (e.g., writes are not guaranteed to be applied in the same order at all replicas).


## Theorem. [source](http://alpha.uhasselt.be/frank.neven/papers/jacm2013.pdf)

A program has a consistent, coordination-free distributed implementation if and only if it is monotonic.

### Observation 1.

Coordination-freeness is equivalent to availability under partition.

  