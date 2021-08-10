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
![Alt text](https://github.com/ArberSephirotheca/czy.github.io/raw/master/raft/state.png)
![Alt text](https://github.com/ArberSephirotheca/czy.github.io/raw/master/raft/append.png)
![Alt text](https://github.com/ArberSephirotheca/czy.github.io/raw/master/raft/request.png)
![Alt text](https://github.com/ArberSephirotheca/czy.github.io/raw/master/raft/rule.png)
	1. **Leader election** - A leader must br chosen when an exisiting leader fails (Section **5.2**).
	2. **Log replication** - the leader must accept log entries from clients and replicates them across the cluster (Section **5.3**).
	3. **Safety** - see figure below.
![Alt text](https://github.com/ArberSephirotheca/czy.github.io/raw/master/raft/safety.png "Safety")


## Basic Structure

