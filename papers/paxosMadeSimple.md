# Paxos Made Simple 
## by Lamport

## The consensus algorithm

### Choosing a value
- at least one value needs to be chosen => acceptors have to accept the first value it receives
- value is chosen when it is accepted by majority => acceptors must be allowed to accept multiple values
- multiple **proposals** can be chosen, but they must have the same value =>
	if any proposal with value v is chosen, then every higher-numbered proposal that is chosen has value v.
- 
