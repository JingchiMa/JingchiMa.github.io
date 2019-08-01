# Consensus on Transaction Commit

## Problem Setup
A transaction is performed by a collection of processes called resource managers (RMs). A transaction will end if one of the RM issues a request to either commit or abort the transaction.

### Goal and Requirements
- Goal
- Safety
  - **stability** if one RM has ever entered *commited* or *aborted* state, it will remains in that state forever.
  - **consistency** it's impossible for one RM to be in the *commited* state while another in the *aborted* state.
- Liveness (not precisely defined, see paper for more details)
  - Non-triviality
    if all RMs reach *prepared* state, 
      eventurally all RMs reach the *commited* state.
    if one of the RMs reach *aborted* state,
      eventually all RMs reach the *aborted* state.
  - Non-blocking

### RM: resource manager
- Faulty
- Communications
  - via messages
  - unreliable, but messages will not be corrupted (undetectably)
- States
  - working: initial state
  - prepared
  - commited
  - aborted
- State transitions (behaviors)
  - predicates 
    - canCommit: true iff **all** RMs are in prepared or committed state. 
    - notCommited: true iff no RM is in commited state.
  - behavoirs
    - prepare
      change from working state to prepared state.
    - decide
      if in *prepared* state and *canCommit* is true, 
        change  to *commited*.
      else if in *working* or *prepared* state, and *notCommited* is true,
        change to *aborted*.


## Two Phase Commit
### TM: transaction manager
- States
  - init: initial state
  - preparing
  - commited
  - aborted
      
### The protocol
- The protocol starts when an RM entered the *prepared* state from *working* state. The RM will also send a *Prepared* message to the TM.
- Upon receipt of the *Prepared* message, the TM will send *Prepare* message to every other RM. 
- Upon receipt of the *Prepare* message, RMs will enter *Prepared* state, and send *Prepared* message to TM.
- When TM receives *Prepared* from all RMs, it enters the *commited* state, and send *Commit* message to all RMs.
- RM enters *commited* state after receiving *Commit* message from TM.

### Problem with Two Phase Commit
- Failure of TM will cause the protocol to block until the TM is repaired. 

## Paxos Commit
### Requirement 
- Safety
  - Only a single value can be chosen.
  - The chosen value must be proposed by one client.
- Liveness

### Paxos Protocol
In this paper, the paxos protocol has an explicit leader election phase, and is explained with different terminologies. 

- **Phase 1a**: leader sends proposal with ballot number *bal* to every accetor.
- **Phase 1b**: 
  When acceptor receives **phase 1a** message from leader,
  if acceptor has performed actions on ballots numbered bal or higher,
    ignore the **phase 1a** message.
  else 
    reply with a **phase 1b** message consisting of 
    - the largest ballot number for which it receives a **phase 1a** message.
    - the **phase 2b** message with the highest ballot number it has sent, if any
- **Phase 2a**:
  if the leader has received **phase 1b** message from a majority of the acceptors, there are two possiblities:
  - **Free**
    None of the majority of the acceptors report having sent a **phase 2b** message, so the algorithm has not yet chosen a value.
  - **Forced**
    Let *u* be the highest ballot number of all reported **phase 1b** messages. Let *M<sub>u</sub>* be the set of all **phase 1b** messages with ballot number *u*. All the messages in *M<sub>u</sub>* have the same value *v*, which might already be chosen.
  In free case, leader can choose whatever value. In the forced case, the leader will try to get the value *v* chosen, by sending **phase 2a** message with value *v* and ballot number *bal* to every acceptor.
- **Phase 2b**: 
  When an acceptor receives **phase 2a** message with ballot number *bal* and value *v*,
  if has received a **phase 1a** or **phase 2a** message with higher ballot number than *bal*,
    ignore the *phase 2a* message.
  else 
    reply a **phase 2b** message with ballot number *bal* and value *v* to the leader.
- **Phase 3**:
  if the leader receives **phase 2b** message from a majority of acceptors, broadcase the value *v* in that **phase 2b** message to all acceptors.

### Paxos Commit
Let *A<sub>1</sub>*, *A<sub>2</sub>*, ... *A<sub>2F + 1</sub>* be the acceptors, where F is some nonnegative integer. Let *leader* be the current leader of the paxos instance<sup>[1]</sup>. 

Let *RM<sub>1</sub>*, *RM<sub>1</sub>*, ..., *RM<sub>N</sub>* be the resource managers (the participants of the transaction).

Leader will maintain a vector of size *N*, each entry of which will be the decision of each resource manager. If all the entries are *prepared*, then the transaction is commited. Otherwise aborted.

#### The protocol
- Some RM (let's say RM<sub>1</sub>) will start the transaction by sending a *BeginCommit* message to *leader*. 
- After receiving *BeginCommit*, *leader* sends out *Prepare* message to all RMs.
- For *RM<sub>i</sub>*, *i* = 1, ..., N, its *decision* is either *prepared*, or *aborted*. And *RM<sub>i</sub>* sends a ballot 0 *phase 2a* message with *decision* as the value to the acceptors to run a separate paxos instance<sup>[2]</sup>. When the acceptors receive the *phase 2a* messages, they will send *phase 2b* messages to *leader* (*not* RM).
- If *leader* receives *F + 1* or more *phase 2b* messages for a paxos instance, the decision for the correpsonding RM is made. *leader* can broadcast this result to all RMs.
- *leader* can send *phase 1a* message with larger ballot number to acceptors to learn about the decisions or try to get *aborted* accepted. 

#### Cost of Paxos Commit
- Messages
  - 5 message delays for RMs to learn that transaction is commited. 
    - *Begin Commit*
    - *leader* to RMs (*Prepare*)
    - RM to acceptors
    - acceptors to *leader*
    - *leader* to RMs (*Commit*)
  - Total messages required: see papers for more detail.
- Stable storage write
  - 2 stable storage write delay (assuming concurrent RM prepare)
    - RM: before entering *prepared* state (sending out *phase 2a* message). 
    - Acceptors: before sending *phase 2b* message.
  - N + F + 1 writes in total


[1] A paxos instance roughly means one setup for paxos algorithm. Each paxos instance will have its own "log" so different instances are not relevant to each other.

[2] This means *RM<sub>i</sub>* will now act as a leader and start another paxos instance to let the acceptors accept its value. 


## Mumbling
This paper *Consensu on Transaction Commit* by *Lamport* mainly talks about the nonblocking transaction algorithm: *Paxos Commit*. *Lamport* explained how he came up with this algorithm:
> I then looked for applications of consensus in which there is a single special proposer whose proposed value needs to be chosen quickly.  I realized there is a "killer app"--namely, distributed transaction commit.  Instead of regarding transaction commit as one consensus problem that chooses the single value commit or abort, it could be presented as a set of separate consensus problems, each choosing the commit/abort desire of a single participant.  Each participant then becomes the special proposer for one of the consensus problems.  This led to what I call the Paxos Commit algorithm. 

I feel the key design point of this algorithm is "using multiple stable storages to replace the single TM in two phase commit". At first, it may be appealing to directly view the transaction commit problem as a special case for consensus problem -- RMs need to agree on either *aborted* or *commited*. The problem with this approach is that, before entering *commited*, people want the RMs to be in *prepared* state first. Therefore, this may take two paxos rounds to implement. As in the paper, 
> However, in the normal case, the leader must learn that each RM has prepared before it can try to get the value committed chosen. Having the RMs tell the leader that they have prepared requires at least one message delay.

Another intuitive way is to make TM fault-tolerant by implementing it using paxos. As a result, TM is not a single node but a paxos cluster. The papers commented this approach:
> Had we used Paxos simply to obtain consensus on a single decision value, this would have been equivalent to replacing the TM’s stable storage by the acceptors’ stable storage, and replacing the single TM by a set of possible leaders. Our Paxos Commit algorithm goes further in essentially eliminating the TM’s role in making the decision. In Two-Phase Commit, the TM can unilaterally decide to abort. In Paxos Commit, a leader can make an abort decision only for an RM that does not decide for itself.

To be honest, I don't fully understand why *Paxos Commit* outperforms this naive replicating approach. According to the paper, it seems to be because "Paxos Commit algorithm goes further in essentially eliminating the TM’s role in making the decision". But is there any performance benefit? 

So finally it comes the *Paxos Commit* algorithm. It uses multiple nodes to store the decision for **each** RM, and make sure the nodes are in sync by using paxos. This might give some insights on other problems.

## Questions
I would say this paper is not that easy to understand. First of all, the description of paxos algorithm seems to be quite different from the original one. And I feel the *Paxos Commit* algorithm is not fully discussed. Currently, my questions are:
1. After sending *prepared* or *aborted* to acceptors, does the RM need to send this *phase 2a* message ever again? Paper mentions that it's possible that consensus cannot be reached on ballot 0. Why is that? Is it because, for example, the RM cannot talk to a majority of acceptors? If so, what the RM should do? 
2. The paper mentions that "a leader can make an abort decision only for an RM that does not decide for itself". I feel this has two potential problems. (1) The timeout value has to be set carefully. If the timeout for *leader* to start sending *phase 1a* message is too small, the *leader* can easily make the decision to *abort* for the RM. (2) It's still possible that even if an RM chooses *prepared* (meaning the RM stores *prepared* into its stable storage, and then sends out *phase 2a* messages to acceptors), *leader* can still make the decision *aborted* because *leader* is proposing at higher ballot, and not learn the value proposed by the RM.

