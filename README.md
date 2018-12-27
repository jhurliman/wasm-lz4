# wasm-lz4

https://github.com/lz4/lz4 compiled to WebAssembly. For now only decompression is supported. PRs welcome!

## API

`wasm-lz4` exports a single function:

```js
export default (buffer: Buffer, destinationByteSize: number) => Buffer
```

Here is an example of using the module:

```js
import fs from 'fs'
import decompress from 'wasm-lz4';

const compressedBytes = fs.readFileSync('compressed.lz4');

// currently you need to know the exact expected size of the output buffer
// so the wasm runtime can allocate enough bytes to decompress into
// TODO: this should not be necessary...it's exposed in the lz4 C code
const outputBytes = compressedBytes * 10;
const decompressedBytes = decompress(compressedBytes, outputBytes);
```

## Using the module in a browser

Emscripten compiled WebAssembly modules are built in 2 parts: a `.js` side and a `.wasm` side.  In the browser the `.js` side needs to download the `.wasm` side from the server so it can compile it.  There is [more information available in the emscripten documentation](https://kripken.github.io/emscripten-site/docs/compiling/Deploying-Pages.html).  When loading the module in the browser via webpack you need to make sure your sever will send the `.wasm` part of the module to the browser when the `.js` side of the module requests it.

## Asynchronous loading & compiling

After the `.wasm` binary is loaded (via `fetch` in the browser or `require` in node) it must be compiled by the WebAssembly runtime.  If you call `decompress` before it is finished compiling it will throw an error indicating it isn't ready yet.  To wait for the module to be loaded you can do something like the following:

```
import decompress from 'wasm-lz4'

async function doWork() {
  await decompress.isLoaded;
  decompress(buffer, outputByteLength);
}
```

## Developing locally

### Building

#### Step 1

First, and most importantly, you need to install emscripten and activate it into your terminal environment.  [Follow these instructions](https://kripken.github.io/emscripten-site/docs/getting_started/downloads.html).

#### Step 2

Next, clone the module recursively as it includes [lz4](https://github.com/lz4/lz4) as a git submodule:

```sh
$ git clone git@github.com:cruise-automation/wasm-lz4.git --recursive
```

#### Step 3

Run `npm run build` to invoke emcc and compile the code in `wasm-lz4.c` as well as the required lz4 source files from the lz4 git submodule.

#### Step 4

To run the tests, run `npm install` followed by `npm test`.

To run the tests in Docker, first make sure Docker is installed, and then run `npm run docker:test`.

## LICENSE

This software is licensed under the Apache License, version 2 ("ALv2"), quoted below.

Copyright (c) 2018-present, GM Cruise LLC

Licensed under the Apache License, Version 2.0 (the "License"); you may not
use this file except in compliance with the License. You may obtain a copy of
the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
License for the specific language governing permissions and limitations under
the License.
