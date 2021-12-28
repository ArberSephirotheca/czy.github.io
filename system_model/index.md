# Asynchronous Model vs Synchronous Model


### Synchronous Distributed System
#### Lots of assumptions
- **Upper Bound on Message Delivery**
  - There exists a **upper bound** on message delay
- **Ordered Message Delivery**
  - communication between machines are expected to be FIFIO.
- **Globally Synchronous Clocks**
  - Each node has a local clock, and the clocks of all nodes are always in sync with each other. 

### Asynchronous Distributed System
- **No Upper Bound Time Limit**
  - Message can be delayed for arbitrary amount of times.
- **Message Delivery can be unordered**
  - Not necessarily FIFO
