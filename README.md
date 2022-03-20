# Ephemeral Futaba Channel Blockchain

v0.0.1

## Concepts

- Ephemerality Algorithm
- Minimum existential deposit to post
- Spam is curtailed by an asymmetric low compression rate/high decompression rate compression algorithm, stored and signed
  as the signature of each post (contenders: brieflz -9, zstd -22).

## Spec

### On-chain Interface

```json5
{
  // Network Generated ID
  "_txid": "0xb90d5acb8733c0d36ff3a0fd2f71eef37022d335e854d77d8ccf650cc292d145",
  
  // Network Generated ISO 8601 timestamp
  "_date": "2022-03-19T20:34:37+0100",
  
  // Signer Public Key
  "_pk": "5GnVP7jkGUjsqhHdtBp1xSgcnNjbUDAnpfYTLWarRHG9QTWm",
  
  // Compressed and Private Key Encrypted Payload, excluding (^\_|^\+)
  // When decryption and/or decompression fails to match included
  // payload, message is discarded and prevented from entering the block.
  // 
  // Format: <PROTOCOL> <OPTIONS>: <COMPRESSED_ENCRYPTED_PAYLOAD>
  "+sig": "brieflz -9: <sig>",
  
  // Post Subject - On Thread Create
  "subject": "My First Post",
  
  // Post Asset - Client Brings Off-Chain Storage
  "asset": "protocol://path-to-asset",
  
  // Post Message - Client brings own msg format text, markdown, 2chan, bbcode, json, etc
  "message": "My First Message",
  
  // Thread Reply TXID - On Reply
  "reply-to": "0xb90d5acb8733c0d36ff3a0fd2f71eef37022d335e854d77d8ccf650cc292d145",
  
  // Channel - On Thread Create
  // Utilized for segmenting boards.
  "channel": "general",
  
  // Sage Flag (Default: False) - On true, post is ignored by ephemerality algorithm. 
  "sage": false
}
```

## API

### Posting

`newMessage(keys={public,private}, message)`

```json5
{
  "+sig": "brieflz: <sig>",
  "subject": "My First Post",
  "asset": "protocol://path-to-asset",
  "message": "My First Message",
  "channel": "general"
}
```

### Replying

`newMessage(keys={public,private}, message)`

```json5
{
  "+sig": "brieflz: <sig>",
  "asset": "protocol://path-to-asset",
  "message": "My First Reply",
  "reply-to": "0xb90d5acb8733c0d36ff3a0fd2f71eef37022d335e854d77d8ccf650cc292d145",
  "sage": false
}
```

### Administrative

Although decentralized there may be a need for a client to manage control over what is displayed and filtered within
their application. These administrative messages can be any format of a clients choosing. It is left up to the client as
to which administrative public keys to honor.

`newMessage(keys={public,private}, message)`

```json5
{
  "+sig": "brieflz: <sig>",
  "reply-to": "0xb90d5acb8733c0d36ff3a0fd2f71eef37022d335e854d77d8ccf650cc292d145",
  // This can be any format json, yaml, msgpack, etc.
  "message": "{\"pin\": \"2022-03-19T20:34:37+0100\", \"by\":\"5GnVP7jkGUjsqhHdtBp1xSgcnNjbUDAnpfYTLWarRHG9QTWm\"}",
  "sage": true
}
```

#### Example Functions

Locking a thread:

```json5
"message": "{\"lock\": \"2022-03-19T20:34:37+0100\", \"by\":\"5GnVP7jkGUjsqhHdtBp1xSgcnNjbUDAnpfYTLWarRHG9QTWm\"}"
```

Pinning a thread (an off-chain storage engine will be requried to pin long term messages):

```json5
"message": "{\"pin\": \"2022-03-19T20:34:37+0100\", \"by\":\"5GnVP7jkGUjsqhHdtBp1xSgcnNjbUDAnpfYTLWarRHG9QTWm\"}"
```

Deleting a thread:

```json5
"message": "{\"delete\": \"2022-03-19T20:34:37+0100\", \"by\":\"5GnVP7jkGUjsqhHdtBp1xSgcnNjbUDAnpfYTLWarRHG9QTWm\"}"
```

## Ephemerality Algorithm

In classic futaba style threads must be ephemeral and discarded from the chain under certain conditions.

To determine this three set of thresholds can trigger thread removal.

### Deletion by Age

Threads older than 48 hours will be removed from the chain.

### Deletion by Thread Count

Multiplier: 1.5

Minimum Count: 50

1. Sort threads by last non-saged reply timestamp (descending order).
2. Calculate number of threads on-chain per hour.
3. Calculate the average, multiplied by a constant multiplier.
4. Return the largest value between the result and the minimum constant.
5. Remove threads whose sort order fall below the threshold.

### Deletion by Reply Count

Multiplier 0.5
Minimum Count: 10

1. Retrieve threads with last replies older than 1 hour.
2. Sort replies by last non-saged reply timestamp (descending order).
3. Calculate number of replies per thread on-chain per hour.
4. Calculate the average, multiplied by a constant multiplier.
5. Return the largest value between the result and the minimum constant.
6. Remove threads with reply counts whose sort order fall below the threshold.

### Calculating Ephemerality Threshold

```javascript
// arr: list of thread or reply counts
const ephemeralityThreshold = (arr, multiplier = 1.5, min = 50) => {
  // avg w/ min value
  return Math.max((arr.reduce((a, b) => a + b, 0) / arr.length) * multiplier, min);
}

ephemeralityThreshold([123, 60, 200, 23, 45, 60])
// result (max posts): 127.75

```
