# neroshop DHT specification

Authors: [@larteyoh](https://github.com/larteyoh)

## Overview
The neroshop DHT is based on the Kademlia Distributed Hash Table (DHT) with some additional protocol message types.



## Definitions

### Replication factor (`r`)
Represents the number of nodes closest to a key, in which `put` messages are sent to.
The default value for `r` is currently 10 and it is usually the same value as the maximum closest nodes (`k`) or lower.

###  Maximum closest nodes (`k`)
Represents the number of nodes closest to a key or id, in which `get` and `find_node` messages are sent to. The default value for `k` is currently 20.

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
Unlike `put`, `map` stores data in the local database file rather than the in-memory hash table in order to make use of SQLite's FTS5 (Full-Text Search 5) feature and perform advanced and efficient searches.

The process of mapping search terms to DHT keys is known as inverted indexing or simply, indexing.


* `set`: Same as put, except it updates the value to a key without changing the key.

This query also checks and validates the signature beforehand, using the user's RSA or Monero keys to ensure that the data can only be modified by its originator.

Set messages can only be processed while in IPC mode so that outside nodes cannot modify other peers' data anyhow.


## Features
- [x] Boostraping mechanism(s)
    - Hardcoded bootstrap nodes
- [x] Keys/Node IDs
    - represented as SHA-3-256 hashes
- [x] KBuckets
    - represented as an unordered_map of <int, std::vector<std::unique_ptr<Node>>>. A max of 256 kbuckets is set, each of them containing up to 25 elements.
- [x] XOR Distance between Keys
- [x] Basic protocol message types: `ping`, `find_node`, `put`, and `get` as well as two additional message types specific to this implementation: `map` and `set`.
- [x] Periodic health checking
    - checks node liveliness every `NEROSHOP_DHT_PERIODIC_CHECK_INTERVAL` seconds
- [x] Periodic republishing (refresh)
    - republishes data to the network every `NEROSHOP_DHT_REPUBLISH_INTERVAL` hours
    - in addition to periodic republishing, node data is automatically republished upon termination of the GUI client
- [x] Distributed indexing
    - the moment a node joins the network, it receives indexing data from the initial `k` closest nodes it contacts via the `map` protocol message type.
- [x] Data validation
    - validates data correctness
- [x] Data integrity verification
    - verifies data integrity using either **monero** or RSA digital signatures
- [ ] node IP address blacklisting
- [ ] Keyword/search term blacklisting
- [x] Expiration dates on certain data such as `orders`

## Data serialization (examples)
**User:**
```json
{
    "avatar": {
        "name": "avatars-xK2cS4dDrz8LNke4-X4YlUw-t500x500.jpg",
        "size": 72163
    },
    "display_name": "dude",
    "metadata": "user",
    "monero_address": "5ARm5QSDW3uWT8EEeoQ5mHghcw82SqYrUHzS99CS1TaxY3XDMGCyeDUN59veZwYfLCNH6mvvzKwJYiLv7Bag1Swn7sSEF9e",
    "public_key": "-----BEGIN PUBLIC KEY-----\nMIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEAoGKa3cN7JNj6xKgvG49f\nkg4Izod2oqNqoraYHMLAhI29MWXUIc8FizhhJUlO4Ts3i2qFaq1ng5jM15K2SQUo\na1ve4rATAbL0j+AnFMPQ/i883m6UJRdO5rUdtjLiKELuXckaQ3pbOmJ06lXI3qQ+\nzrPtgHp7hK2zfJSI+lgqHfU5AoQThOo5+QhuVgu1eJrvTMphc+XSPQ/8/7aPAdsB\nWI3FFy/QRtKr7RsjrsC2rTzYkqUo+MBXocN6d/1qCEPOgLuKJZvsKHt0VtWEHcQ2\ndTIqn4MhCh7Vz7psS1rBlfFW/j+PHXsQS7IRtU8RIy9yAoCKRO80aBhUe1ktvLaP\n2zXnW87vrRABYCc9fy9e3ZzCbPbAE/avozF/qR+XusBEKuvNUmMKW7CBet4CsGXc\nCAbJh5HIwrp+lgMZlgGLTjQ5cfegqtYtXWlJz09BbaY1+VNKhhhd0X/CN7apmqpu\njq94C9Db3dQGqTowHYflnwB3G3iXEcllmwECgFAIy4AG1cx3p/xkyXKe7r8QKcZF\nywE/2IQg8xmqtNDyQvcrIA9y6nyARskYZuqio62eQTaLKiYPTELmg5XmXyyVarJw\nIVXeWjqFz7WtS3RnM7RxoN8cwnd4JPIFTusGkucLAd/TBYKytPrsV68dvUmyrm1F\no63Sv7T7kBpeGIn5VOFY0wUCAwEAAQ==\n-----END PUBLIC KEY-----\n",
    "signature": "SigV2PagGh1SMm7KVrahMXeb7177Mzox7RHr1ciS1juk5WLfWatZiEods4ougHZKzgujp1oYhtG2vDguYcH3ac5zMYtjY"
}
```
```json
{
    "avatar": {
        "name": "pixelArt-1689350451803.png",
        "size": 3348
    },
    "display_name": "lupita",
    "metadata": "user",
    "monero_address": "59PFsC9wHepdrz5oseoDW3auk8a1HtUZbDHmPWwbBFjsa3kcue9N4GbK2hMRoFGgyLGNv14Z2QjDGjT1TrsC1UerPK3igm4",
    "public_key": "-----BEGIN PUBLIC KEY-----\nMIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEAwtXNtHoyaiMGbtB2R1m+\nVMI15xjzERCnUsqL2qrxCo/QCcCqtKjEhdknPQcMO5taAmaUJnovDfJcmIA5M0kQ\n4ZKjKtzREN/YSanvgAemkX75I5rQPXIMQAz4Cs5Ie+cheOrl1WYo8DoAzlNRi8z0\nT2MDNBisMaWhqGfwEUgNtXQjxZpcYs9GmiM4qgIdCSniA6ixuPAsLNCI/LXhEBxX\ng1tyYJW4nOm9Sb0vlDr/ZiQRPDz23TdvlF6C1OXotsdd2dhNsbSEtXE7wEUYPX6O\n2UXbBF5PtzrmTuB6zoC9U4qfXrpkFJ9uPxpjER2fa0RSdcRhmywh/WTgVOUjR8iT\nRqSEs+8ByyZ2qV6TUSzKo2dWwCQIO6NHcX6ITbSrUeGFddIWuUmWVRGtG9d62yZD\n5o2895Qd+0J51L7l7xUNcFHmjxSm35AiGsvHJ4u583pz4DUodB8vASODuz/85X3x\nFdmLHV0IpeKZvpaSmYIRUsizg6SzXHNTYHEmOIlYCCJ16Zz2WO9P3OkzNms0ScKq\nfnL+OSBC47zquWpI905bB625SVJJaUZJvlIg6/wU2fdfVteeuboEiMiPyted9aId\nS8Vq8Ks8yFTZJzvhyOqpzSLx6/v5OuA+ABQH+AHOax36QRB+i6Fe3Vnc/YJLjmTp\n/8+KD7atMZb0UOpGCy4INeUCAwEAAQ==\n-----END PUBLIC KEY-----\n",
    "signature": "SigV2gASDvXcQkeW5LLzt7mMxZu5NvBb3s9rawYzf9wDVoWqeUtxtKTtWhvLVM9pn3rJ6aBYvKNX47tuhv57BPRP6jTyL"
}
```

**Listing**:
```json
{
    "condition": "New",
    "currency": "USD",
    "date": "2023-07-20T06:52:01Z",
    "id": "44d7f6f9-9a94-4fd9-84ad-7dc22a182ada",
    "location": "Germany",
    "metadata": "listing",
    "price": 10.0,
    "product": {
        "category": "Sports & Outdoors",
        "description": "",
        "id": "0442c628-beab-4564-b6de-8a6baa7c6490",
        "images": [
            {
                "id": 0,
                "name": "monero-symbol-on-white-480.png",
                "size": 6768
            }
        ],
        "name": "Monero ball"
    },
    "quantity": 1,
    "seller_id": "5ARm5QSDW3uWT8EEeoQ5mHghcw82SqYrUHzS99CS1TaxY3XDMGCyeDUN59veZwYfLCNH6mvvzKwJYiLv7Bag1Swn7sSEF9e",
    "signature": "SigV2bo1rMpaK9xeWU6CEuSnSCJ7buT8qANS8gUsjRx4wVpqpdxwj7LbLitkjYfD1NJeYmvJndYM9bUe2qWGrn7W35ebz"
}
```
```json
{
    "condition": "New",
    "currency": "USD",
    "date": "2023-07-20T07:14:26Z",
    "id": "2cd40b7b-a142-425f-b17a-afe9e6064a78",
    "location": "Thailand",
    "metadata": "listing",
    "price": 5.0,
    "product": {
        "category": "Food & Beverages",
        "description": "",
        "id": "cfe34584-d57c-4ec3-badf-f9b406a7b4d2",
        "images": [
            {
                "id": 0,
                "name": "gitea.png",
                "size": 26874
            }
        ],
        "name": "Green Tea"
    },
    "quantity": 100,
    "seller_id": "59PFsC9wHepdrz5oseoDW3auk8a1HtUZbDHmPWwbBFjsa3kcue9N4GbK2hMRoFGgyLGNv14Z2QjDGjT1TrsC1UerPK3igm4",
    "signature": "SigV2ZqBnw1sass4BrpWeq5HPacdf1zgdfkikYEW356kJw4XrazHuRPrirYydSeGVcm2wcRTWWLu5g1oX2PfxogjnnYNQ"
}

```

**Product rating**:
```json
{
    "comments": "This product is aiight",
    "metadata": "product_rating",
    "product_id": "2b715653-da61-4ea0-8b5a-ad2754d78ba1",
    "rater_id": "59HiXx4PNJDd4sWVWhWtqb8m9pB7Huyc9HgybAwjFGkj27HozxkaLXNDhFH4wtzfYCe8GLzhA5YUHPUGWnjfRhvdL5HHE1C",
    "signature": "SigV2BExHYCAKkhoTbC3L4tx2EeHTaz2ZAtLfu7YgWcu1Ex3JXVk4X8XssfPE3TLBSQGRNLFENo4iAMa3gFV3u3bKoEvY",
    "stars": 3
}
```
**Seller rating**:
```json
{
    "comments": "I love you!!!",
    "metadata": "seller_rating",
    "rater_id": "59JKGvtkpPQ7UocLgK5UV79GTEYJqDsJMgF9CiKmmms858VKvVvx1uX8DfXKYLWtoK9LVL38WQgf1JVn8yGWXZuJJzVwWGt",
    "score": 1,
    "seller_id": "5AncSFWauoN8bfA68uYpWJRM8fFxEqztuhXSGkeQn5Xd9yU6XqJPW7cZmtYETUAjTK1fCfYQX1CP3Dnmy5a8eUSM5n3C6aL",
    "signature": "SigV2idt52jHJfAic6ccT5yQwkJ3iP7cmNZ8Y1DYFBoQme9C9foRBzM4b2XeZwCQyCwvoDPM4xbWQ3FBP9NvVCBMe6gkr"
}   
```

**Order**:
```json
{
	"coin": "Monero",
	"created_at": "2023-07-27T05:42:29Z",
	"customer_id": "52P6EwT4zhpEfFhDLc9WSWGkSaEudfF7t597EnFRnQ8sYv4SqdS7BwDPddiJtxeQUHSP7SYAfgycaPhwMvWLc8p8M26eDkB",
	"delivery_option": "Delivery",
	"discount": 0.0,
	"id": "43f5ab3f-7288-4ad5-8295-adaada079101",
	"items": [{
		"listing_key": "2e6b41de8e53b7cd8c143d4632d13fe3aafcc7ab0ce6c94a7fac232ea0e4634c",
		"quantity": 1,
		"seller_id": "53TTZUaFGVB9koKDF1tpxc3APGUVEBMqURpr1aMFZwhJPP8oJksaspq6XgHVaoZijwDnDHRYuMDFvbzNZUYcH2tt8NzMqtd"
	}, {
		"listing_key": "2e8c94bb4b3448478ab1cc7db22ee95f8c607a595d90f3284efb39c033c15c55",
		"quantity": 1,
		"seller_id": "53TTZUaFGVB9koKDF1tpxc3APGUVEBMqURpr1aMFZwhJPP8oJksaspq6XgHVaoZijwDnDHRYuMDFvbzNZUYcH2tt8NzMqtd"
	}],
	"metadata": "order",
	"notes": "12 Robot Dr. Boston MA 02115",
	"payment_option": "Escrow",
	"shipping_cost": 0.0,
	"status": "New",
	"subtotal": 0.036584724958608955,
	"total": 0.036584724958608955
}
```

**Message**:
```json
{
    "content": "EPwhlCsN/r+P9PQr2+WyvNrxgV+32eBtWjS9UMmyrjBsXvJ0j4JJ++8WSpDUBClC4Us7PTmz8zospCK+FgH1P1vZm6NS+GFzXmxE7Zo5WXq6xBGe2KU7BCtOzXX7axm93+Jlb/1cWl7QRv4SXf/7DXEhRS6wjByCgk0Jl3ltcDpSWJeppZeJM9K4EXRNR5aoxyzeSi6TWfPOadf3/VCNryjqroOYojAhthaQ+qGJxEJ78Ef1lxn4jExvthqIsHFIyxpbdJN+/NiW9jHpHJoJqR8/xk5okCZmBFPWs6gDSPUenQK7Mk62yzLE6+ccVWTPdEmT39nPjYjHJLR+p8bWpyCRWCe6uOs4Mlhft//CWvS/eX06yLUhhMVoPMwKUSAcVAswKw4hC1XcJwOP+YQQkY8HFjphFo/eRgXtUcVWiKK3tiwW3aJontucH2mNnVyXEeoqQ20UtSfaCsHn4Hs2Jzw0BtxC7+G381k+ferRvG1jpNRm7UmN86LpEqwPK6BMdm90bOEP5FWJLOMGkwvR3inkzLy8dO2W4FILgnKWKFrKKjpNh/goHr/zeiBjtqk6UMa5vSOrGtAHQuPzWJttvb9PJ/wRxSTlZVTZO2lNxcw6dofikxV7n0ZapywN1BM34awUE54EYc3oYkDVFlmYdjCyQxUOD57W8h5z8WViBto=",
    "metadata": "message",
    "recipient_id": "57k6T5W8H7gQWwCgHuN3mdg7q6Jk1g6UUYc6E8LvgrKfMdJj5sFViBeZxgF1T79z5TZpzf9mLAWUrH8KMUpuiEzNTiFjGgP",
    "sender_id": "ecUlBKvTUFDN39iIwP7OMm51S7BgUFQtli7TCaaugyDCl8LQckocLvS4SiD5fBdvf059994yk0wOGSJy7i2aQGC7kn5sv2gqoDGbYlKsfU5RGfApgkFeE7tgXfEM2oiygNnGThRT0K4M1Q3347VRzAZqAt0M/1VDXgH3eONbFqkbdKhrxWtOUTKMcB0Y3WRcnqOsrOEhCblJyvPn7+/lZN2xo10eVLtzRUcK8e8btcWIgW8vsMR5Ma8I5Q/ZVC6RZXSOiy1ixRKsNGC0z414Wp7rC0onvDVF+Hd0imtdT/dgIN++MCj4zyuDIgJJFZR7qCOJ2AfbSEpONQJI4GbOO1uSf7IgUPnuOWJwO+9FeKO1pWCpfmefxdzTXolL3KGJnAlvW84EurBt7y4RZ27qFqYuzZ7M0282nOOC91kMo6rREWn9zRMFrdecv5MhB3rGVxAdV1jeaf9eW3nsJBJn55y9tccMahSxvQNfGyTU6f2x2KKJpzDayRs7PL4NzRoXJ1UuDAG58lPHasre5H8HVHFXW3wzJHNP4QVPrFPsELwrV6rmSykD7U+nlilA5s0mps+bEdj6ou6ZtgP2qalrZTV/jzpwOEJGPKT8m+k2DxIEVlnIpZHYDvJe8sLtikMdZIFJrBycm4ddqrHQA9A4NQm7FyXlapJPMmBr8Tl+Gos=",
    "timestamp": "2023-07-28T04:10:38Z"
}
```

## RPC messages
[`msgpack`](https://msgpack.org/) is the binary serialization format used to transmit data throughout the neroshop network. JSON data is first converted to msgpack before it is transmitted. This is because msgpack reduces the JSON data size by 32%, making it smaller, faster and more efficient to transfer data across the network.

There are three types of messages that are sent in the neroshop network: `query`, `response`, and `error`. A query is basically a request made by a peer. The primary query types used are `ping`, `find_node`, `put`, `get`, `map`, and `set`.
When a query is sent, normally a response should be expected. Responses are typically a sign of the successful execution of a query. Sometimes the result of a query may be an error if a function could not be executed or no value is returned from a `get` request, or perhaps something else.

Each query consists a `query` field containing the query type, an `args` field containing the arguments for the specific query type. A response consists of any field/value representing the result of the response. All messages must have an `id` field containing the node's id and a `tid` to identify which message matches with which response. With the exception of IPC mode where the local GUI client sends queries to the local daemon server. A `version` field is also included in each message to specify which version of the neroshop DHT is currently being used by a peer.


## References

[BitTorrent](http://bittorrent.org/beps/bep_0005.html)

[anarkiocrypto](https://twitter.com/AnarkioC) - for helping me come up with the design for the data structure for users, listings, orders etc.
