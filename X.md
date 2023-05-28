NIP-X
======

Secure Communication Channels (SCC)
-------------

`draft` `optional` `author:egge` `author:starbuilder`

The purpose of this NIP is to allow the creation and management of secure bidirectional channels of communication. The approach is inspired by the SimpleX protocol, but modified to fit into the existing nostr ecosystem and without relying in out-of-band messaging. This NIP introduces a single new kind Y, that will be used to initialize, negotiate or modify secure communication channels between to peers. It is based on NIP-4 and can gain additional privacy by enforcing NIP-42.

## Secure Communication Channels

A secure communication channel is a kind-4 channel between two peers that has no public ties to their main identity. Once initialized it can be valid forever, while allowing the peers to negotiate modifications like key-rotations or change of relays for enhanced privacy.

## Flow

This exemplary flow describes the creation of a SCC between Alice and Bob.

1. Alices derives two random key pairs (`IK` and `SK`) that she never used before. She then creates a Kind-Y event of type `create`, including the required metadata (relays: <list of relays>, sender: <pubkey(SK)>, secret: <secret>) and signs it using her own private key. She then creates a kind-4 event using `IK`, adds a encrypted version of the kind Y, prefixed by `Y:` as content, and publishes it to relays that she expects Bob to read from. She can now safely delete `IK`.

2. Bob receives the kind-4 event published by pubkey(IK) and decrypts it. Because its content is prefixed with `Y:` he knows to handle it like a SCC event. Because its a `SCC create` event Bob can now link Alice's identity to the sender key found in the metadata, after verifying its signature.

3. Bob derives a new key pair (`RK`) and creates a kind y event of type `created`, including the required metadata (secret: <secret>) and signs it using his own private key. He then creates a kind-4 event using `RK` addressed to `pub(SK)`, adds a encrypted version of the Kind y event, prefixed by `Y:` ,as content and publishes it to the relays that he found in the `SCC create` metadata.

4. Alice receives Bob's kind-4 event and decrypts it. After parsing the contained kind-Y event, its signature and secret, she can extract pub(RK) and link it to Bob's identity. A SCC between SK and RK is now established. Alice and Bob can now communicate using SK and RK and regular kind-4 events without correlation to their main identities.

## Modifying an existing SCC

### Changing relays

### Extending SCC

### Key-Rotation

## NIP-42

While NIP-42 is not mandatory for this approach to work, it still reduces the surface for analysis and correlation. It is absolutely advised to use relays that enfore NIP-42 for kind-4 events.


## Event format

This NIP specifies the use of the `Y` event type, having in `content` a type, and a list of varying metadata tags:

### create
* `relays` a list of relays that shall be used for this SCC
* `sender` the pubkey the initializing peer will use for this SCC
* `secret` a random secret
* `expiry?` (optional) time in seconds after which this SCC will be discarded by both peers.

### created
* `secret` the secret the initializing peer used in their `create` event

### acknowledged
* `id` id of the acknowledged event.

### reject
* `id` id of the rejected event.
* `reason` (optional) text explaining why a peer decided to reject a request.

A exemplary `SCC create` event

```json
{
  "id": <32-bytes lowercase hex-encoded sha256 of the the serialized event data>,
  "pubkey": <32-bytes lowercase hex-encoded public key of the event creator>,
  "created_at": <unix timestamp in seconds>,
  "kind": Y,
  "tags": [
    ["relays",<comma-seperated list of relays>],
    ["sender",<pub(SK)>],
    ["secret", <a random secret>],
    ["expiry",<time in seconds>]
  ],
  "content": "create",
  "sig": <64-bytes hex of the signature of the sha256 hash of the serialized event data, which is the same as the "id" field>
}
```
