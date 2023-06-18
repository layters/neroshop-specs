# neroshop DHT specification

Authors: [@larteyoh](https://github.com/larteyoh)

## Overview
The neroshop DHT is based on the Kademlia Distributed Hash Table (DHT) with some additional protocol message types.



## Definitions

### Replication factor or maximum closest nodes (`k`)
The amount of replication is governed by the replication parameter `k`. The
default value for `k` is currently 10, but may increase to 20 or higher depending on node churn.

### Distance

In all cases, the distance between two keys/node ids is `XOR(sha3_256(key1),
sha3_256(key2))`.

### Routing table
...



## Protocol messages

* `ping`: This query type is often used to check the liveliness of a node.

* `find_node`: This query type is used to request a list of nodes with an id closest to a specific id or key.

The response is a `nodes` message containing information about a node such as its IP address and port.


* `put`: This query type is used to store key-value data in a receiving node's hash table. 

The querying node sends a `put` message to the `NEROSHOP_DHT_REPLICATION_FACTOR` nodes with an id closest to the key.
When the querying node makes a put request and it is the originator of the data, it also stores its own data in its hash table right after sending the `put` to the other nodes in its routing table.
On receiving a `put` request, a node will `store` the key-value pair data in its own hash table.

The kind of data stored in the hash table is typically related to user account, listings, orders, and ratings/reviews. 
Each data contains metadata which specifies the type of data that is being stored.


* `get`: This query type is used to find a value to a specific key. 

The querying node typically sends a `get` message to the `k` nodes with an id closest to the key and when any of those closests nodes receive the `get`, they respond with a message containing the value to the requested key. In the case that any of the closest nodes do not have the value in their hash table, they attempt to send `get` messages to the closest nodes in their routing table until the value is found or not.


* `map`: Like `put`, except it uses the value to create search terms and map them to the corresponding key.
Unlike `put`, `map` stores data in the local database file in order to make use of SQLite's FTS5 (Full-Text Search 5) feature and perform advanced and efficient searches.

The processing of mapping search terms to DHT keys is known as inverted indexing or simply, indexing.


* `set`: Same as put but it updates/modifies the value to a key without changing the key.

This query also checks and validates the signature beforehand, using the user's public keys to ensure that the data is only altered by its originator.



## References

[BitTorrent](http://bittorrent.org/beps/bep_0005.html)
