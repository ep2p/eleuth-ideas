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

<img src="https://github.com/idioglossia/eleuth-ideas/blob/main/images/Multi Ring Lookup.svg?raw=true" width="500" alt="Multi ring Lookup"/>


When Availability message reaches to a ring, a broadcast will be sent too all members of ring, therefor all members will know a node is available.
In image above, all `light blue` colored nodes, know that a node with id of A is available. Also all members of *ring-2* know that node A is available through *ring-node-6*.

Now if B looks up for A, the find message will move through chain till it reaches *Ring-1* and then it will be broadcast to all members of *ring-1* and finally it is passed to *ring-2* and thats when 

