# multiformats

This library is for building an interface for working with various
inter-related multiformat technologies (multicodec, multihash, multibase,
and CID).

The interface contains all you need for encoding and decoding the basic
structures with no codec information, codec encoder/decoders, base encodings
or hashing functions. You can then add codec info, codec encoders/decoders,
base encodings, and hashing functions to the interface.

This allows you to pass around an interface containing only the code you need
which can greatly reduce dependencies and bundle size.

```js
import * as CID from 'multiformats/cid'
import { sha256 } from 'multiformats/hashes/sha2'
import dagcbor from '@ipld/dag-cbor'
import { base32 } from 'multiformats/bases/base32'
import { base58btc } from 'multiformats/bases/base58'

const bytes = dagcbor.encode({ hello: 'world' })

const hash = await sha256.digest(bytes)
// raw codec is the only codec that is there by default
const cid = CID.create(1, dagcbor.code, hash)
```

However, if you're doing this much you should probably use multiformats
with the `dag` API.

```js
// Import basics package with dep-free codecs, hashes, and base encodings
import { dag } from 'multiformats/basics'
import { sha256 } from 'multiformats/hashes/sha2'
import cbor from '@ipld/dag-cbor'
import json from '@ipld/dag-json'

const encoder = dag.encoder({ multicodec: dag.or(json), hasher: sha256 })

const hello = encoder.encodeBlock({ hello: 'world' })
const cid = await hello.cid()

const greeting = encoder.encode({ code: json.code, value: { greeting: hello } })
await greeting.cid()

const decoder = dag.decoder({ multicodec: dag.or(json), hasher: sha256 })
decoder.decode({ code: json.code, bytes: greeting.bytes })
```

# Plugins

By default, no base encodings, hash functions, or codec implementations are included with `multiformats`.
However, you can import the following bundles to get a `multiformats` interface with them already configured.

| bundle | bases | hashes | codecs |
|---|---|---|---|
| `multiformats/basics` | `base32`, `base64` | `sha2-256`, `sha2-512` | `json`, `raw` |

## Base Encodings (multibase)

| bases | import | repo |
 --- | --- | --- |
`base16` | `multiformats/bases/base16` | [multiformats/js-multiformats](https://github.com/multiformats/js-multiformats/tree/master/bases) |
`base32`, `base32pad`, `base32hex`, `base32hexpad`, `base32z` | `multiformats/bases/base32` | [multiformats/js-multiformats](https://github.com/multiformats/js-multiformats/tree/master/bases) |
`base64`, `base64pad`, `base64url`, `base64urlpad` | `multiformats/bases/base64` | [multiformats/js-multiformats](https://github.com/multiformats/js-multiformats/tree/master/bases) |
`base58btc`, `base58flick4` | `multiformats/bases/base58` | [multiformats/js-multiformats](https://github.com/multiformats/js-multiformats/tree/master/bases) |

## Hash Functions (multihash)

| hashes | import | repo |
| --- | --- | --- |
| `sha2-256`, `sha2-512` | `multiformats/hashes/sha2` | [multiformats/js-multiformats](https://github.com/multiformats/js-multiformats/tree/master/hashes) |
| `sha3-224`, `sha3-256`, `sha3-384`,`sha3-512`, `shake-128`, `shake-256`, `keccak-224`, `keccak-256`, `keccak-384`, `keccak-512` | `@multiformats/sha3` | [multiformats/js-sha3](https://github.com/multiformats/js-sha3) |
| `murmur3-128`, `murmur3-32` | `@multiformats/murmur3` | [multiformats/js-murmur3](https://github.com/multiformats/js-murmur3) |

## Codec Implementations (multicodec)

| codec | import | repo |
| --- | --- | --- |
| `raw` | `multiformats/codecs/raw` | [multiformats/js-multiformats](https://github.com/multiformats/js-multiformats/tree/master/codecs) |
| `json` | `multiformats/codecs/json` | [multiformats/js-multiformats](https://github.com/multiformats/js-multiformats/tree/master/codecs) |
| `dag-cbor` | `@ipld/dag-cbor` | [ipld/js-dag-cbor](https://github.com/ipld/js-dag-cbor) |
| `dag-json` | `@ipld/dag-json` | [ipld/js-dag-json](https://github.com/ipld/js-dag-json) |

# API

# multiformats([table])

Returns a new multiformats interface.

Can optionally pass in a table of multiformat entries.

## multiformats.configure

## multiformats.varint

## multiformats.bytes

## multiformats.digest

## multiformats.hasher

# multiformats.CID

Changes from `cids`:

* All base encodings are cached indefinitely.
* CID's can be created without any multiformat data.
  * The new API is entirely based on parsing the varints
    so it doesn't need the table metadata in order to associate
    string names.

There are also numerous deprecations. These deprecations all stem from the
fact that we no longer know the full set of available multicodec information.
It's actually quite possible to provide a CID interface without this, you can
still do everything you used to do, you just need to use ints instead of strings
and do some of the fancier V0 coercions outside this library.

Deprecation List:
  * the multibase encoding is no longer cached during instantiation.
    * this being indeterministic was causing some nasty problems downstream
      since `toString()` needs to be used as a cache key and it's not possible
      to encode V0 into anything but base58btc. this means that you can't have
      deterministic hash keys without also requiring base58btc support, so we
      removed this feature.
  * no more .toBaseEncodedString(), just toString()
  * no more .multibaseName
  * no more .prefix()
  * no more .codec
    * new property ".code" is the multiformat integer.
    * this is going to be a painful transition but we have to get off of using
      the string if we ever want to drop the full table. while the DX for this is
      nice it forces into bloating the bundle and makes using new codecs very
      painful.
