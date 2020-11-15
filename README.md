# eleuth-ideas

This repository contains ideas that comes to my mind about Eleuth project.

## Network Structure
Eleuth network is combination of structured and unstructured decentralized (p2p) networks.

It's unstructured in terms that many nodes are not stable, perhaps because they are only home desktops with dynamic IP addresses.
It's structured in terms that stable nodes have opportunity to create a decentralized ring network.

Therefore if we look at the network in wide angle, we observe multiple rings and multiple chains. Each node in a ring or chain can be connected to multiple nodes of other rings and chains.

---

## Node types

### Rings

Rings are type of node which are connected to eachother using Kademlia Algorithm ([check kademlia-api](https://github.com/ep2p/kademlia-api)). Each ring contains group of nodes that are connected with same ring id.

There are 2 type of ring networks. Private rings and public rings.

You can setup your private ring and other nodes will need to authorizate themselves in order to join your ring. Or you can setup public ring and everyone else can join the ring. Or you can connect to multiple other rings as well. And ofcource you will need authorizate yourself to the private ones.

Rings store data of [routes](#Routing) to other nodes (using kademlia storage algorithm). This data is coppied into multiple nodes of ring so failures wont effect it. Therefore, if you connect to any member of a ring, you can query about any other node in the network, and if a path to that node is available in current ring, your node will find it.


Rings can also broadcast messages between members of ring.

<img src="https://github.com/idioglossia/eleuth-ideas/raw/main/images/Ring%20Broadcast.svg" width="350" alt="Ring Broadcast"/>


Image above shows how this broadcast can be sent to all members. Each node passes the message to other nodes in ring that it has in its routing table. These nodes then pass the same message to other nodes they know of, excluding the ones already used by previous nodes. This process goes on till there is no longer a node left to send message to.

### Proxies

Proxy nodes basically have to forward incoming requests to other nodes. The other nodes can be devices, or ring nodes.

---

## Availability
When a node becomes available, it publishes a signed message to all its seeders. Seeder nodes will also publish this availability message to other nodes, and this process could go on till *N edges are passed* or *message reaches to one ring with at least x nodes*.

When Availability message reaches to a ring, a broadcast will be sent too all members of ring, therefor all members will know a node is available.
Each node of ring then passes this availiblity message to all the nodes it knows out of the current ring.

In that case, N (as in "this process could go on till N edges are passed") should be a constant number that wont pressure the whole network. If two nodes are far than N nodes, they might never have the chance to communicate.

---

### Routing and cache
Something that is not mentioned directly in **Availability** but is definitely playing important part in routing is the route cache.

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
- ring-2 is available through nodes 5 and 7

This also means that this cache is `one-to-many`. In other words, each node can have list of other nodes as routes.


## IDEA ISSUES - Availability

Availability explanation has some issues. Its almost impossible to make sure a second ring will have any information about nodes from first ring (as in picture). Therefore, multiple ring idea has some issues. 

Possible fix the idea of "Community based" network:

```
Eleuth should initialize a community for initial and main ring.
Other rings can create a connection to this ring, so the availablity message moves around the network.
Or they should let the device and proxy nodes know that their ring has no connection to main ring. 
Also, there should be a way to prove that each secondary ring has connection to main ring.
```
```
 Each community needs to store couple of nodeIds with public key in a reachable web address or hard coded in their custom forked application builds to let clients make sure they can access the community ring by pinging those nodes.
```
