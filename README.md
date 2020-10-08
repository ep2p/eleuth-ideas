# eleuth-ideas

This repository contains ideas that comes to my mind about Eleuth project.

## Network Structure
Eleuth network is combination of structured and unstructured decentralized (p2p) networks.

It's unstructured in terms that many nodes are not stable, perhaps because they are only home desktops with dynamic IP addresses.
It's structured in terms that stable nodes have opportunity to create a decentralized ring network.

Therefore if we look at the network in wide angle, we observe multiple rings and multiple chains. Each node in a ring or chain can be connected to multiple nodes of other rings and chains.

## Rings
Each ring contains group of nodes that are connected with same ring id. Each node of ring creates connection to `(log n)` nodes.

<img src="https://github.com/idioglossia/eleuth-ideas/raw/main/images/Ring%20Broadcast.svg" width="350" alt="Ring Broadcast"/>

 
