[![NPM version](https://img.shields.io/npm/v/yauzl-promise.svg)](https://www.npmjs.com/package/yauzl-promise)
[![Build Status](https://img.shields.io/github/actions/workflow/status/overlookmotel/yauzl-promise/test.yml?branch=master)](https://github.com/overlookmotel/yauzl-promise/actions)
[![Coverage Status](https://img.shields.io/coveralls/overlookmotel/yauzl-promise/master.svg)](https://coveralls.io/r/overlookmotel/yauzl-promise)

# `yauzl` unzipping with Promises

## Usage

Promisified version of [yauzl](https://www.npmjs.com/package/yauzl) for unzipping ZIP files.

### Installation

```
npm install yauzl-promise
```

### Methods

#### `open()` / `fromFd()` / `fromBuffer()` / `fromRandomAccessReader()`

These methods all work as before, but return a Promise rather than taking a callback.

```js
const yauzl = require('yauzl-promise');
const zipFile = await yauzl.open('/path/to/file.zip');
```

`lazyEntries` option is automatically enabled. Get file entries using methods listed below.

`autoClose` option is automatically disabled. `ZipFile`s must be closed manually with `.close()`.

#### `zipFile.close()`

Closes file and returns Promise which resolves when all streams are closed.

Files **must** be closed when finished with to avoid resource leakages.

```js
const zipFile = await yauzl.open('/path/to/file.zip');
await zipFile.close();
```

#### `zipFile.readEntry()`

Same as original yauzl method, but returning a promise. Promise resolves to an instance of `yauzl.Entry`, or rejects if there is an error.

```js
const entry = await zipFile.readEntry();
console.log(entry);
```

Calling `.readEntry()` again returns the next entry. When there are no entries left, it returns `null`.

#### `zipFile.readEntries( [numEntries] )`

Read several entries and return as an array.

```js
const entries = await zipFile.readEntries(3);
entries.forEach(console.log);
```

If `numEntries` is `0`, `null` or `undefined`, reading will continue until all entries are read.

WARNING: This is dangerous. If ZIP contains a large number of files, could lead to crash due to out of memory. Use `.walkEntries()` instead.

#### `zipFile.walkEntries( callback [, numEntries] )`

Read several entries and call `callback` for each.

If `callback` returns a promise, the promise is awaited before reading the next entry. If `callback` throws an error or returns a rejected promise, walking stops and the promise returned by `.walkEntries()` is rejected.

Returns a promise which resolves when all have been read.

```js
await zipFile.walkEntries(entry => {
  console.log(entry);
});
console.log('Done');
```

If `numEntries` is `0`, `null` or `undefined`, reading will continue until all entries are read.

#### `zipFile.openReadStream( entry [, options] )`

Same as original method but returns promise of a stream.

```js
const readStream = await zipFile.openReadStream(entry);
readStream.pipe(writeStream);
```

#### `entry.openReadStream( [options] )`

As above, but called on an `Entry` object.

```js
const entry = await zipFile.readEntry();
const readStream = await entry.openReadStream();
readStream.pipe(writeStream);
```

### Events

`ZipFile` objects are from a subclass of yauzl's original `ZipFile` class. They are event emitters but do not emit any of the events original yauzl module emits (`entry`, `end`, `close` or `error`).

These events are replaced by the resolution/rejection of promises returned by the methods listed above.

If an `error` event is emitted unexpectedly within yauzl at a time when no operation (`readEntry()` etc) is in progress, that event is consumed to prevent the process from crashing. The next time `readEntry()`, `close()` or `openReadStream()` is called, the promise returned from that method will reject with the previously emitted error.

### Customization

#### Alternative Promise implementation

Promises returned by default are native JS Promises.

`.usePromise()` returns a new `yauzl` object where the methods return promises from the specified Promise constructor.

```js
const Bluebird = require('bluebird');
const yauzl = require('yauzl-promise').usePromise(Bluebird);

const p = yauzl.open('/path/to/file.zip');
assert(p instanceof Bluebird);
```

NB This does not alter the original `yauzl` object, only the one returned from `.usePromise()`.

```js
const Bluebird = require('bluebird');
const yauzl = require('yauzl-promise');
const yauzlBluebird = yauzl.usePromise(Bluebird);

const p = yauzl.open('/path/to/file.zip');
assert(p instanceof Promise);
assert(!(p instanceof Bluebird));

const p = yauzlBluebird.open('/path/to/file.zip');
assert(p instanceof Bluebird);
assert(!(p instanceof Promise));
```

#### Using another version of `yauzl`

`.useYauzl()` method promisifies a specific `yauzl` object.

Only useful if you have a modified version of yauzl which you want to promisify.

```js
const yauzlCrc = require('yauzl-crc');
const yauzl = require('yauzl-promise').useYauzl(yauzlCrc);
```

The yauzl object passed is cloned before it is modified, unless you set `clone` option to `false`:

```js
const yauzlFork = require('my-yauzl-fork');
const yauzl = require('yauzl-promise').useYauzl(yauzlFork, { clone: false });
assert(yauzl === yauzlFork);
```

#### Using another version of `yauzl` and alternative Promise implementation

`.use()` method does both of the above.

```js
const yauzlCrc = require('yauzl-crc');
const Bluebird = require('bluebird');
const yauzl = require('yauzl-promise').use(Bluebird, yauzlCrc);
```

The yauzl object passed is cloned before it is modified, unless you set `clone` option to `false`:

```js
const yauzlFork = require('my-yauzl-fork');
const yauzl = require('yauzl-promise').use(Bluebird, yauzlFork, { clone: false });
assert(yauzl === yauzlFork);
```

## Versioning

This module follows [semver](https://semver.org/). Breaking changes will only be made in major version updates.

All active NodeJS release lines are supported (v16+ at time of writing). After a release line of NodeJS reaches end of life according to [Node's LTS schedule](https://nodejs.org/en/about/releases/), support for that version of Node may be dropped at any time, and this will not be considered a breaking change. Dropping support for a Node version will be made in a minor version update (e.g. 1.2.0 to 1.3.0). If you are using a Node version which is approaching end of life, pin your dependency of this module to patch updates only using tilde (`~`) e.g. `~1.2.3` to avoid breakages.

## Tests

Use `npm test` to run the tests. Use `npm run cover` to check coverage.

## Changelog

See [changelog.md](https://github.com/overlookmotel/yauzl-promise/blob/master/changelog.md)

## Issues

If you discover a bug, please raise an issue on Github. https://github.com/overlookmotel/yauzl-promise/issues

## Contribution

Pull requests are very welcome. Please:

* ensure all tests pass before submitting PR
* add tests for new features
* document new functionality/API additions in README
* do not add an entry to Changelog (Changelog is created when cutting releases)
