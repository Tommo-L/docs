# Consensus

## List of Terms

* **Proof of Stake** `PoS` - A type of algorithm which uses network consensus to handle fault tolerance.

* **Proof of Work** `PoW` - A type of algorithm which uses computing power to handle fault tolerance.

* **Byzantine Fault** `BF` - A failure in which a node remains functional, but operates in a dishonest manner.

* **Delegated Byzantine Fault Tolerance** `DBFT` - A consensus algorithm implemented within the NEO blockchain to guarantee fault tolerance.

* **View** `v` - The dataset used during a consensus activity in NEO `DBFT`

## Roles
In the NEO consensus algorithm, Consensus Nodes are elected by NEO holders and vote on validity of transactions.  These nodes have also been referred to as 'Bookkeepers'.  Moving forward, they will be referred to as Consensus Nodes.

  - <img style="vertical-align: middle" src="../../assets/nNode.png" width="25"> **Consensus Node** - This node participates in the consensus activity.  During a consensus activity, consensus nodes take turns assuming the following two roles:
  - <img style="vertical-align: middle" src="../../assets/speakerNode.png" width="25"> **Speaker** `(One)` - The **Speaker** is responsible for transmitting a block proposal to the system.
  - <img style="vertical-align: middle" src="../../assets/cNode.png" width="25"> **Delegate** `(Multiple)` - **Delegates** are responsible for reaching a consensus on the transaction.


## Introduction

One of the fundamental differences between blockchains is how they can guarantee fault tolerance given defective, non-honest activity on the network.

Traditional methods implemented using PoW can provide this guarantee as long as a majority of the network's computational power is honest.  However, because of this schema's dependency on compute, the mechanism can be very inefficient (computational power costs energy and requires hardware).  These dependencies expose a PoW network to a number of limitations, the primary one being the cost of scaling.

NEO implements a Delegated Byzantine Fault Tolerance consensus algorithm which takes advantage of some PoS-like features(NEO holders vote on **Consensus Nodes**) which protects the network from Byzantine faults using minimal resources, while rejecting some of its issues.  This solution addresses performance and scalability issues associated with current blockchain implementations without a significant impact to the fault tolerance.

## Theory

The Byzantine Generals Problem is a classical problem in distributed computing.  The problem defines a number of **Delegates** that must all reach a consensus on the results of a **Speaker's** order.  In this system, we need to be careful because the **Speaker** or any number of **Delegates** could be traitorous.  A dishonest node may not send a consistent message to each recipient.  This is considered the most disastrous situation.  The solution to the problem requires that the **Delegates** identify if the **Speaker** is honest and what the actual command was as a group.

For the purpose of describing how DBFT works, we will primarily be focusing this section on the justification of the 66.66% consensus rate used in Section 5.  Keep in mind that a dishonest node does not need to be actively malicious, it could simply not be functioning as intended. 

For the sake of discussion, we will describe a couple of scenarios.  In these simple examples, we will assume that each node sends along the message it received from the **Speaker**.   This mechanism is used in DBFT as well and is critical to the system. We will only be describing the difference between a functional system and disfunctional system.  For a more detailed explanation, see the references.


### **Honest Speaker**

  <p align="center"><img src="../../assets/n3.png" width="300"><br> <b>Figure 1:</b> An n = 3 example with a dishonest <b>Delegate</b>.</p>

  In **Figure 1**, we have a single loyal **Delegate** (50%).  Both **Delegates** received the same message from the honest **Speaker**.  However, because a **Delegate** is dishonest, the honest Delegate can only determine that there is a dishonest node, but is unable to identify if it is the block nucleator (The **Speaker**) or the **Delegate**.  Because of this, the **Delegate** must abstain from voting, changing the view.

  <p align="center"><img src="../../assets/n4.png" width="400"><br> <b>Figure 2:</b> An n = 4 example with a dishonest <b>Delegate</b>.</p>

  In **Figure 2**, we have a two loyal **Delegates** (66%).  All **Delegates** received the same message from the honest **Speaker** and send their validation result, along with the message received from the speaker to each other **Delegate**.  Based on the consensus of the two honest **Delegates**, we are able to determine that either the **Speaker** or right **Delegate** is dishonest in the system.  


### **Dishonest Speaker** 

  <p align="center"><img src="../../assets/g3.png" width="300"><br> <b>Figure 3:</b> An n = 3 example with a dishonest <b>Speaker</b>. </p>

  In the case of **Figure 3**, the dishonest **Speaker**, we have an identical conclusion to those depicted in **Figure 1**.  Neither **Delegate** is able to determine which node is dishonest.

  <p align="center"><img src="../../assets/g4.png" width="400"><br> <b>Figure 4:</b> An n = 4 example with a dishonest <b>Speaker</b>. </p>

In the example posed by **Figure 4**, the blocks received by both the middle and right node are not validatable.  This causes them to defer for a new view which elects a new **Speaker** because they carry a 66% majority.  In this example, if the dishonest **Speaker** had sent honest data to two of the three **Delegates**, it would have been validated without the need for a view change.


## Practical Implementation

The practical implementation of DBFT in NEO uses an iterative consensus method to guarantee that consensus is reached.  The performance of the algorithm is dependent on the fraction of honest nodes in the system. **Figure 5** depicts the
expected iterations as a function of the fraction of dishonest nodes.  

Note that the **Figure 5** does not extend below 66.66% **Consensus nodes** honesty.  Between this critical point and 33% **Consensus nodes** honesty, there is a 'No-Man's Land' where a consensus is unattainable.  Below 33.33% **Consensus nodes** honesty, dishonest nodes (assuming they are aligned in consensus) are able to reach a consensus themselves and become the new point of truth in the system.

<img src="../../assets/consensus.iterations.png" width="800">

**Figure 5:** Monte-Carlo Simulation of the DBFT algorithm depicting the iterations required to reach consensus. {100 Nodes; 100,000 Simulated Blocks with random honest node selection}


### Definitions

**Within the algorithm, we define the following:**

  - `t`: The amount of time allocated for block generation, measured in seconds.
    - Currently: `t = 15 seconds`
    - This value can be used to roughly approximate the duration of a single view iteration as the consensus activity and communication events are fast relative to this time constant.

  - `n`: The number of active **Consensus Nodes**.

  - `f`: The minimum threshold of faulty **Consensus Nodes** within the system. 
     - `f = (n - 1) / 3`

  - `h` : The current block height during consensus activity.

  - `i` : **Consensus Node** index.


  - `v` : The view of a **Consensus Node**.  The view contains the aggregated information the node has received during a round of consensus.  This includes the vote (`prepareResponse` or `ChangeView`) issued by all Delegates.


  - `k` : The index of the view `v`.  A consensus activity can require multiple rounds.  On consensus failure, `k` is incremented and a new round of consensus begins.


  - `p` : Index of the **Consensus Node** elected as the **Speaker**.  This calculation mechanism for this index rotates through **Consensus Nodes** to prevent a single node from acting as a dicator within the system. 
     - `p = (h - k) mod (n)`


  - `s`: The safe consensus threshold.  Below this threshold, the network is exposed to fault.  
     - `s = ((n - 1) - f)`


### Requirements

**Within NEO, there are three primary requirements for consensus fault tolerance:**

1. `s` **Delegates** must reach a consensus about a transaction before a block can be committed.

2. Dishonest **Consensus Nodes** must not be able to persuade the honest consensus nodes of faulty transactions. 

3. At least `s` **Delegates** are in same state (`h`,`k`) to begin a consensus activity

### Algorithm
**The algorithm works as follows:**

1. A **Consensus Node** broadcasts a transaction to the entire network with the sender's signatures.

   <p align="center"><img src="../../assets/consensus1.png" width="450"><br> <b>Figure 6:</b> A <b>Consensus Node</b> receives a transaction and broadcasts it to the system. </p>

2. **Consensus Nodes** log transaction data into local memory.

3. The first view `v` of the consensus activity is initialized.

4. The **Speaker** is identified. **Wait** `t` seconds.
     
<p align="center"><img src="../../assets/consensus2.png" width="450"><br/> <b>Figure 7:</b> A <b>Speaker</b> has been identified and the view has been set.</p>
     
5. The **Speaker** broadcasts the proposal : `<prepareRequest, h, k, p, bloc, [block]sigp>`

     <p align="center"><img src="../../assets/consensus3.png" width="450"><br> <b>Figure 8:</b> The <b>Speaker</b> mints a block proposal for review by the <b>Delegates</b>. </p>

6. The **Delegates** receive the proposal and validate:

    - Is the data format consistent with the system rules?
    - Is the transaction already on the blockchain?
    - Are the contract scripts correctly executed?
    - Does the transaction only contain a single spend?(i.e. does the transaction avoid a double spend scenario?)
    - **If Validated Proposal Broadcast:**  `<prepareResponse, h, k, i, [block]sigi>`
    - **If Invalidated Proposal Broadcast:**  `<ChangeView, h,k,i,k+1>`

   <p align="center"><img src="../../assets/consensus4.png" width="500"><br> <b>Figure 9:</b> The <b>Delegates</b> review the block proposal and respond. </p>

7. After receiving `s` number of 'prepareResponse' broadcasts, a **Delegate** reaches a consensus and publishes a block.

8. The **Delegates** sign the block.

   <p align="center"><img src="../../assets/consensus5.png" width="500"><br> <b>Figure 10:</b> A consensus is reached and the approving <b>Delegates</b> sign the block, binding it to the chain. </p>

9. When a **Consensus Node** receives a full block, current view data is purged, and a new round of consensus begins. 
  
  - `k = 0`

---

> [!Note]
>
>  If after   (![timeout](../../assets/consensus.timeout.png) )  seconds on the same view without consensus:
>  
>   - **Consensus Node** broadcasts: <ChangeView, h,k,i,k+1>
>
>   - Once a **Consensus Node** receives at least `s` number of broadcasts denoting the same change of view, it increments the view `v`, triggering a new round of consensus.	
>

## References
- [A Byzantine Fault Tolerance Algorithm for Blockchain](dbft.md)
- [Practical Byzantine Fault Tolerance](http://pmg.csail.mit.edu/papers/osdi99.pdf)
- [The Byzantine Generals Problem](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/12/The-Byzantine-Generals-Problem.pdf)

