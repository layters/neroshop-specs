# neroshop DHT specification

Authors: [@larteyoh](https://github.com/larteyoh)

## Overview
The neroshop DHT is based on the Kademlia Distributed Hash Table (DHT) with some additional protocol message types.



## Definitions

### Replication factor (`r`)
Represents the number of nodes closest to a key, in which `put` messages are sent to.
The default value for `r` is currently 10 and it is usually the same value as the maximum closest nodes (`k`).

###  Maximum closest nodes (`k`)
Represents the number of nodes closest to a key or id, in which `get` and `find_node` messages are sent to. The default value for `k` is currently 10.

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


* `map`: This query type takes the value then uses it to create search terms and map them to the corresponding key.
Unlike `put`, `map` stores data in the local database file rather than the hash table in order to make use of SQLite's FTS5 (Full-Text Search 5) feature and perform advanced and efficient searches.

The process of mapping search terms to DHT keys is known as inverted indexing or simply, indexing.


* `set`: Same as put, except it updates the value to a key without changing the key.

This query also checks and validates the signature beforehand, using the user's public keys to ensure that the data can only be modified by its originator.



## Features
- [x] Boostraping mechanism(s)
    - Hardcoded bootstrap nodes
- [x] Keys/Node IDs
    - represented as SHA-3-256 hashes
- [x] KBuckets
    - represented as an unordered_map of <int, std::vector<std::unique_ptr<Node>>>. A max of 256 kbuckets is set, each of them containing up to 25 elements.
- [x] XOR Distance between Keys
- [x] Basic protocol message types: `ping`, `find_node`, `put`, and `get`
- [x] Periodic health checking
    - checks node liveliness every `NEROSHOP_DHT_PERIODIC_CHECK_INTERVAL` seconds
- [x] Periodic republishing (refresh)
    - republishes data to the network every `NEROSHOP_DHT_REPUBLISH_INTERVAL` hours
- [x] Distributed indexing
    - the moment a node joins the network, it receives indexing data from the initial `k` nodes it contacts via the `map` protocol message type.
- [x] Data validation
    - validates data correctness
- [ ] Data integrity verification
    - verifies data integrity using digital signatures
- [ ] node IP address Blacklisting
- [ ] Keyword/search term blacklisting
- [ ] Expiration dates on certain data such as `orders`


## References

[BitTorrent](http://bittorrent.org/beps/bep_0005.html)
