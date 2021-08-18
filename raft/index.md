# Research Paper - Raft

<!more>

## Why Raft
* It provides a consensus algorithm allowing a collection of machines
to work as a group that can survive the failures of some of its members.

## Features
* **Strong leader**
	* logs only flow from leader to others.
* **Leader election**
	* Raft uses randomized timers to elect leaders.
* **Membership changes**
	* Raft uses *joint consensus* approach where two different configurations
	overlap largely during transitions.


## Replicated state machines
![Alt text](https://github.com/ArberSephirotheca/czy.github.io/raw/master/raft/state_machine.png "State Machine")
* The state machines process identical sequences of commands from the logs.
* Each servers stores a replicated log.

## Raft Consensus algorithm
* It ensures *safety* under all [non-Byzantine](https://web.archive.org/web/20170205142845/http://lamport.azurewebsites.net/pubs/byz.pdf) conditions, including:
	1. network delays
	2. partitions
	3. packet loss
	4. duplication
	5. reordering
* It decomposes consensus problem into three subproblems:
![avatar](https://github.com/ArberSephirotheca/czy.github.io/raw/master/raft/state.png)
![avatar](https://github.com/ArberSephirotheca/czy.github.io/raw/master/raft/append.png)
![avatar](https://github.com/ArberSephirotheca/czy.github.io/raw/master/raft/request.png)
![avatar](https://github.com/ArberSephirotheca/czy.github.io/raw/master/raft/rule.png)
	1. **Leader election** - A leader must br chosen when an exisiting leader fails (Section **5.2**).
	2. **Log replication** - the leader must accept log entries from clients and replicates them across the cluster (Section **5.3**).
	3. **Safety** - see figure below.
![Alt text](https://github.com/ArberSephirotheca/czy.github.io/raw/master/raft/safety.png "Safety")


## Basic Structure
* Each server has one of state:
	1. Candidate - used to elect a new leader.
	2. Follwer - simply respond to rquests from leaders and candidates.
	3. Leader - handles all client requests(follower will redirect request to leader).
![Alt text](https://github.com/ArberSephirotheca/czy.github.io/raw/master/raft/state_change.png "State Change")

## RPCs
* Two type of RPC:
	1. RequestVote RPCs - initiated by **candidates**.
	2. AppendEntries RPCs - initiated by **leader** to replicate log and send heartbeat.

## Leader election
* When server start up, they begin as **followers**.
* Leader send periodic heartbeats(append with **no log entries**) to all followers.
* When a follower receive no message over a period (*election timeout*), it starts a election:
	* Follower increments its **term** and becomes **candidate**.
	* Candidate **vote** itself and issues requestVote RPCs.
	* Candidate mantains its state until:
		1. It wins election.
		2. Another server becoms a leader.
		3. No winner over a period of time.
	* When a candidate receives RPCs from other candidates with **<** term, it rejects.
	* When a candidate receives RPCs from other candidates with **>=** term, it becomes follower.

