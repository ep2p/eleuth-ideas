# eleuth-ideas

This repository contains ideas that comes to my mind about Eleuth project.

## Network Structure
Eleuth network is combination of structured and unstructured decentralized (p2p) networks.

It's unstructured in terms that many nodes are not stable, perhaps because they are only home desktops with dynamic IP addresses.
It's structured in terms that stable nodes have opportunity to create a decentralized ring network.

Therefore if we look at the network in wide angle, we observe multiple rings and multiple chains. Each node in a ring or chain can be connected to multiple nodes of other rings and chains.

## Rings
Each ring contains group of nodes that are connected with same ring id. Each node of ring creates connection to `(log n)` nodes when it boots.
There is no decentralized solutions in finding seed nodes. To boot up a ring, the new node should know address for at least one other node of the ring.

[This video](https://www.youtube.com/watch?v=kXyVqk3EbwE) explains enough information about rings and algorithms for finding members of a ring.

Also there are cases when you are looking for members outside of current ring. For example, one node could be hoping [at least] one of the other ring nodes has information about some other node in eleuth network.
In order to find which nodes of current ring have information about a node out of current ring, a broadcast should be sent. It's like a find query for something outside the current ring so every node of the ring should receive this find query.

<img src="https://github.com/idioglossia/eleuth-ideas/raw/main/images/Ring%20Broadcast.svg" width="350" alt="Ring Broadcast"/>


Image above shows how this broadcast can be sent to all members. Since each node is connected to 3 other nodes (only in above example), there will be lots of duplicate messages.
For example, 0 sends message to `[1,2,4]` and 1 has to send same message to `[2,3,5]` so this means 2 will receive the same message twice.
Also, note that when 0 is connected to 2, it means that 2 is connected to 0. So 2 should avoid sending this message back to 0 again.

In order to avoid these duplicate messages, each node adds information about which other nodes should be excluded from broadcast. As seen in image above, when 0 is sending message to 1, it tells node 1 to avoid sending message to nodes: `[0,2,4]`.
Now when 1 sends this message to 3, it tells 3 to avoid sending this message to `[1,5]` (1 is sender itself, 5 is another connected node, and 2 is skipped cause its lower number than the receiver which is 3).
Also note that `[1,5]` is added to previous exclude list: `[0,2,4]` which results in `[0,2,4,1,5,]`.
 
## Availability
When a node becomes available, it publishes a signed message to all its seeders. Seeder nodes will also publish this availability message to other nodes, and this process could go on till *N edges are passed* or *message reaches to one ring with at least x members*.
For now we focus on scenario when availability message moves forward in a chain till it reaches first ring. 

<img src="https://github.com/idioglossia/eleuth-ideas/blob/main/images/Multi Ring Lookup.svg?raw=true" width="600" alt="Multi ring Lookup"/>


When Availability message reaches to a ring, a broadcast will be sent too all members of ring, therefor all members will know a node is available.
In image above, all `light blue` colored nodes, know that a node with id of A is available. Also all members of *ring-2* know that node A is available through *ring-node-6*.

Now if B looks up for A, the find message will move through chain till it reaches *Ring-1* and then it will be broadcast to all members of *ring-1* and finally it is passed to *ring-2* and that's when *ring-2-node-4* knows where node A is and responses.

### Node Address Caching
Something that is not mentioned directly in **Availability** but is definitely playing important part in routing B->A is route cache.

<img src="https://github.com/idioglossia/eleuth-ideas/blob/main/images/Single%20Ring%20Cache.svg?raw=true" width="500" alt="Multi ring Lookup"/>

The image above can explain how routing and caching is done in Eleuth network. When node `A` goes online, it tells all the other nodes about its availability.
So first-level nodes know that they have connection to A.
Then they tell next level nodes that they have access to A.
So second-level nodes know the way to get to A is from previous level nodes. In above example, D knows to send a message to A, it should send it to B and B will pass that message to A.

This is a reliable mechanism in a way, cause *ring-2-node-7* knows A is available in network but it only knows to communicate with D if it wants pass something to A. In other word, *ring-2-node-7* has no information where A is exactly.

This path should be cached in each node for a TTL and only as long as previous level node is online. Which means if B goes down, D can no longer rely on this path. Also, to update ttl, it would be fair to ask node A to send periodic heartbeats (availability message).

<img src="https://github.com/idioglossia/eleuth-ideas/blob/main/images/Multi%20Ring%20Cache.svg?raw=true" width="500" alt="Multi ring Lookup"/>

Now, back to the first scenario where B sent message which passed another ring (ring-1). Since every member of ring-2 know how to reach to A, its safe that ring-2 creates a cache that says:
- A is available in ring-2
- ring-2 is available in through nodes 5 and 7

This also means that this cache is `one-to-many`. In other words, each node can have `list of other nodes` to use for accessing a single node.