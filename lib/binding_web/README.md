# Web Tree-sitterin your `package.json`:

```js
"
> [!NOTE]
> The `web-tree-sitter.js` file on GH releases is an ES6 module. If you are interested in using a pure CommonJS library, such
> as for Electron, you should use the `web-tree-sitter.cjs` file instead.

### Basic Usage

First, create a parser:

```js
const parser = new Parser();
```

Then assign a language to the parser. Tree-sitter languages are packaged as individual `.wasm` files (more on this below):

```js
const { Language } = require('web-tree-sitter');
const JavaScript = await Language.load('/path/to/tree-sitter-javascript.wasm');
parser.setLanguage(JavaScript);
```

Now you can parse source code:

```js
const sourceCode = 'let x = 1; console.log(x);';
const tree = parser.parse(sourceCode);
```

and inspect the syntax tree.

```, to parse JavaScript, you can install the `tree-sitter-javascript`
package:
][gh release js].

#### Generating `.wasm` files

You can also generate the `.wasm` file for your desired grammar. Shown below is an example of how to generate the `.wasm`
file for the JavaScript grammar.

> [!NOTE]
> Since v0.26.1, `tree-sitter build --wasm` uses [wasi-sdk][] and will automatically download it on first use.
> No additional tools need to be installed.

First install `tree-sitter-cli`, and the tree-sitter language for which to generate `.wasm`
(`tree-sitter-javascript` in this example):

```sh
npm install --save-dev tree-sitter-cli tree-sitter-javascript
```

Then just use tree-sitter cli tool to generate the `.wasm`.

```sh
npx tree-sitter build --wasm node_modules/tree-sitter-javascript
```

If everything is fine, file `tree-sitter-javascript.wasm` should be generated in current directory.

### Running .wasm in Node.js

Notice that executing `.wasm` files in Node.js is considerably slower than running [Node.js bindings][node bindings].
However, this could be useful for testing purposes:

```javascript
const Parser = require('web-tree-sitter');

(async () => {
  await Parser.init();
  const parser = new Parser();
  const Lang = await Parser.Language.load('tree-sitter-javascript.wasm');
  parser.setLanguage(Lang);
  const tree = parser.parse('let x = 1;');
  console.log(tree.rootNode.toString());
})();
```

### Loading a pre-compiled WebAssembly module

Some environments, such as Cloudflare Workers and Vercel Edge Functions, import
`.wasm` files as `WebAssembly.Module` objects. You can pass those modules to `Language.loadSync`:

```javascript
import treeSitterJavaScript from 'tree-sitter-javascript.wasm';
// treeSitterJavaScript is of type `WebAssembly.Module`
const JavaScript = Language.loadSync(treeSitterJavaScript);
parser.setLanguage(JavaScript);
```

### Running .wasm in browser

`web-tree-sitter` can run in the browser, but there are some common pitfalls.

#### Loading the .wasm file

`web-tree-sitter` needs to load the `tree-sitter.wasm` file. By default, it assumes that this file is available in the
same path as the JavaScript code. Therefore, if the code is being served from `http://localhost:3000/bundle.js`, then
the Wasm file should be at `http://localhost:3000/tree-sitter.wasm`.

For server side frameworks like NextJS, this can be tricky as pages are often served from a path such as
`http://localhost:3000/_next/static/chunks/pages/index.js`. The loader will therefore look for the Wasm file at
`http://localhost:3000/_next/static/chunks/pages/tree-sitter.wasm`. The solution is to pass a `locateFile` function in
the `moduleOptions` argument to `Parser.init()`:

```javascript
await Parser.init({
  locateFile(scriptName: string, scriptDirectory: string) {
    return scriptName;
  },
});
```

`locateFile` takes in two parameters, `scriptName`, i.e. the Wasm file name, and `scriptDirectory`, i.e. the directory
where the loader expects the script to be. It returns the path where the loader will look for the Wasm file. In the NextJS
case, we want to return just the `scriptName` so that the loader will look at `http://localhost:3000/tree-sitter.wasm`
and not `http://localhost:3000/_next/static/chunks/pages/tree-sitter.wasm`.

For system library.
Since this doesn't exist in the browser, the bundlers will get confused. For Webpack, you can fix this by adding the
following to your webpack config:

```javascript
{
  resolve: {
    fallback: {
      fs: false
    }
  }
}
```

[
