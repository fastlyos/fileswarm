fileswarm
=========

> Share and download files securely over a p2p network.

## Installation

```sh
$ npm install fileswarm
```

## Usage

```js
const fileswarm = require('fileswarm')
const crypto = require('crypto')
const ram = require('random-access-memory')

const secret = crypto.randomBytes(32)

const src = './path/to/source/file'
const dst = './path/to/destination/file'

// seeds `src` and broadcasts as an enciphered feed of bytes
// where each "block" in the feed stores a 24 byte nonce in
// a `seed.nonces` storage used for the cipher and that should
// be accessible to the downloader along with the
// corresponding shared secret
const seed = fileswarm.seed(src, ram, { secret }, (err) => {
  const { key, nonces } = seed
  // without `secret` and `nonces` the download cannot decipher the
  // seed's feed storage after verification
  fileswarm.download(dst, { key, nonces, secret }, (err) => {
    // `dst` points to decrypted file downloaded from seed
  })
})
```

## API

### `fileswarm.seed(pathspec, storage[, opts[, callback])`

Seed file at `pathspec` where `pathspec` can be a local file path or a
HTTP URL. The underlying [hypercore][hypercore] `storage` must be
specified as the enciphered file at `pathspec` is stored there and
replicated in the [network swarm][hyperswarm]. If a shared secret is
given then the data stored in the underlying feed is encrypted. Readers
must be given access to the shared secret as well as a
[random-access-storage][ras] instance containing the nonces created by
the seeder. When indexing and seeding is complete, `callback()` will be
called. `callback(err)` is also called when an error occurs.

```js
const fileswarm = require('fileswarm')
const crypto = require('crypto')
const path = require('path')
const raf = require('random-access-file')
const fs = require('fs')

const pathspec = 'https://example.com/video.mp4'
const filename = path.basename(pathspec)
const nonces = raf(filename + '.nonces')
const secret = crypto.randomBytes(32) // share this
const seed = fileswarm.seed(pathspec, { secret }, onseed)

function onseed(err) {
  if (err) throw err

  const basename = path.basename(seed.pathspec)
  const keyFile = basename + '.key'

  fs.writeFile(keyFile, secret, () => {
  })
}
```

### `fileswarm.download(storage[, opts[, callback]])`

> TODO

## License

MIT

[hyperswarm]: https://github.com/hyperswarm/hyperswarm
[hypercore]: https://github.com/mafintosh/hypercore
[ras]: https://github.com/random-access-storage/random-access-storage
