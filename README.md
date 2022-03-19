# Ephemeral Futaba Channel Blockchain

v0.0.1

## Concepts

- Ephemeral Thread Lifetime Algorithm `Work in Progress`
- Wallet must hold a minimum of 1 unit to post.
- Each non-saged reply bumps a thread within the ephemerality algorithm.
- Spam is curtailed by a asymmetic low compression rate/high decompression rate compression algorithm, stored and signed as the signature of each post (contenders: brieflz -9, zstd -22).

## Spec

### On-chain Interface

```json5
{
        // network generated id
        "_txid": "0xb90d5acb8733c0d36ff3a0fd2f71eef37022d335e854d77d8ccf650cc292d145", 

        // network generated ISO 8601 timestamp
        "_date": "2022-03-19T20:34:37+0100", 

        // poster public key
        "_pk": "5GnVP7jkGUjsqhHdtBp1xSgcnNjbUDAnpfYTLWarRHG9QTWm",

        // payload signature, exluding (^\_|^\+)
        // using brieflz -9 to create signature limits spammers by computational overhead (this algorithm may change)
        "+sig": "brieflz -9: <sig>",

        // Post Subject (Used when creating a thread)
        "subject": "My First Post",

        // Post Asset - Client Brings Off-Chain Storage
        "asset": "protocol://path-to-asset",

        // Post Message - Client brings own msg format text, markdown, 2chan, bbode, json, etc
        "message": "My First Message",

        // Thread Reply TXID (Used only when replying to a thread) 
        "reply-to": "0xb90d5acb8733c0d36ff3a0fd2f71eef37022d335e854d77d8ccf650cc292d145",

        // Post Tag (Used when creating a thread)
        "tag": "general",

        // Sage Flag (Default False): Will not bump thread
        "sage": false // saged as to not influcing bumping a thread
    }

```

## API

### Posting

`newMessage(keys={public,private}, message)`

```json5
    {
        "+sig": "brieflz: <sig>", // payload signature, exludes (^\_|^\+)
        "subject": "My First Post", // Subject
        "asset": "protocol://path-to-asset", // client brings own asset storage
        "message": "My First Message", // client brings own format text, markdown, 2chan, bbode, json, etc
        "tag": "general" // tags
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

Although decentralized there may be a need for a client to manage control over what is displayed and filtered within their application.
These administrative messages can be any format of a clients choosing. It is left up to the client as to which administrative public keys to honor.

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

`Work in Progress`
