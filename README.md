# neroshop DHT specification

Authors: [@larteyoh](https://github.com/larteyoh)

## Overview
The neroshop DHT is based on the Kademlia Distributed Hash Table (DHT) with some additional protocol message types.



## Definitions

### Replication factor (`r`)
Represents the number of nodes closest to a key, in which `put` messages are sent to.
The default value for `r` is currently 10 and it is usually the same value as the maximum closest nodes (`k`).

###  Maximum closest nodes (`k`)
Represents the number of nodes closest to a key or id, in which `get` messages are sent to. The default value for `k` is currently 10.

### Distance

In all cases, the distance between two keys/node ids is `XOR(sha3_256(key1),
sha3_256(key2))`.

### Routing table
...

### Hash table
...



## Protocol messages

* `ping`: This query type is often used to check the liveliness of a node.

* `find_node`: This query type is used to request a list of nodes with an id closest to a specific id or key.

The response is a `nodes` message containing information about nodes such as their IP addresses and ports.


* `put`: This query type is used to store key-value data in a receiving node's hash table. 

The querying node sends a `put` message to the `r` nodes with an id closest to the key.
If any of the closest nodes fail to respond to the put request, they are replaced by other closest nodes within the routing table.

When the querying node makes a put request and it is the originator of the data, it also stores its own data in its hash table right after sending the `put` to the other nodes in its routing table.
On receiving a `put` request, a node will `store` the key-value pair data in its own hash table.

The kind of data stored in the hash table is typically related to user account, listings, orders, and ratings/reviews. 
Each data contains metadata which specifies the type of data that is being stored.


* `get`: This query type is used to find a value to a specific key. 

The querying node typically sends a `get` message to the `k` nodes with an id closest to the key and when any of those closests nodes receive the `get`, they respond with a message containing the value to the requested key. In the case that any of the closest nodes do not have the value in their hash table, they attempt to send `get` messages to the closest nodes in their routing table until the value is found or not.


* `map`: Like `put`, except it uses the value to create search terms and map them to the corresponding key.
Unlike `put`, `map` stores data in the local database file rather than the hash table in order to make use of SQLite's FTS5 (Full-Text Search 5) feature and perform advanced and efficient searches.

The processing of mapping search terms to DHT keys is known as inverted indexing or simply, indexing.


* `set`: Same as put, except it updates the value to a key without changing the key.

This query also checks and validates the signature beforehand, using the user's public keys to ensure that the data can only be modified by its originator.



## References

[BitTorrent](http://bittorrent.org/beps/bep_0005.html)
