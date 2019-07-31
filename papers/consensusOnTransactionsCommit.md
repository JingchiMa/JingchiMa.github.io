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


