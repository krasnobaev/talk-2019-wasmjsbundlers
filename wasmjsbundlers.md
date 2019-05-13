---
title: Обзор бандлеров в JS
# theme: solarized
revealOptions:
  transition: 'fade'
---
# Обзор бандлеров в JS
## Как приготовить .wasm

<!-- ![cat](sub/cat.jpg) -->

Note:

---

`.wat` – [WebAssembly text format](https://webassembly.github.io/spec/core/text/index.html)
```bash
(module
  (type $t0 (func (param i32 i32) (result i32)))
  (func $add (type $t0) (param $p0 i32) (param $p1 i32) (result i32)
    get_local $p0
    get_local $p1
    i32.add)
  (export "add" (func $add)))
```

`.wasm` – бинарный формат
```bash
0000000 00 61 73 6d 01 00 00 00 01 07 01 60 02 7f 7f 01
0000010 7f 03 02 01 00 07 07 01 03 61 64 64 00 00 0a 09
0000020 01 07 00 20 00 20 01 6a 0b
0000029
```

Note:
  .wat - официальный стандарт
  .wat и .wasm транслируются взаимно-однозначно

---

`.wat` – [WebAssembly text format](https://webassembly.github.io/spec/core/text/index.html)
```bash
(module
  (type $t0 (func (param i32 i32) (result i32)))
  (func $add (type $t0) (param $p0 i32) (param $p1 i32) (result i32)
    get_local $p0
    get_local $p1
    i32.add)
  (export "add" (func $add)))
```

`.wast` – неофициальный формат, в основном для тестирования

```bash
(module
  (func $add (param i32) (param i32) (result i32)
    (get_local 0)
    (get_local 1)
    (i32.add)
  )
  (export "add" (func $add))
)
```

Note:
  .wast - неофициальный, но описан там же

---

### Варианты импорта 1️⃣

* объявление внутри js – бинарный формат <mark>TODO:</mark>
```js
let instance = WebAssembly.instantiate(Buffer.from([0,61,…]));
alert(`a=1, b=2, a+b=${instance.exports.add(1, 2)}`);
```

* объявление внутри js – `.wat`-исходники <mark>TODO:</mark>
```js
const src = '(module (type $t0 (func (param i32 i32) (result i32)))…';
let module = webassemblyjs.instantiateFromSource(src);
alert(`a=1, b=2, a+b=${module.exports.add(1, 2)}`);
```

* импорт файла .wasm <mark>TODO:</mark>

```js
request = new XMLHttpRequest();
request.open('GET', '/add.wasm');
request.responseType = 'arraybuffer';
request.send();

request.onload = function() {
  webassemblyjs
    .instantiate(request.response)
    .then((module) => {
      var res = module.instance.exports.add(1, 1);
      document.getElementById("res").innerHTML = res;
    });
};
```
<!-- .element: class="bigcode" -->

Note:
  импорт файла .wasm - https://webassembly.js.org/docs/example-add-wasm.html
  в html грузить нельзя, импорт только через

---

### На чём готовить `.wasm`

Привычные языки <!-- .element: class="fragment" data-fragment-index="1" -->
* <!-- .element: class="fragment" data-fragment-index="1" --> 🐥 C, LLVM via `emscripten`
* <!-- .element: class="fragment" data-fragment-index="1" --> <div><!-- .element: class="fragment highlight-current-blue" data-fragment-index="3" --> 🐥 Rust via `wasm-pack`</div>
* <!-- .element: class="fragment" data-fragment-index="1" --> 🐥 Go via `GOARCH=wasm`
* <!-- .element: class="fragment" data-fragment-index="1" --> 🐣 JavaScript via `Duktape`

Специализированные языки <!-- .element: class="fragment" data-fragment-index="2" -->
* <!-- .element: class="fragment" data-fragment-index="2" --> 🐥 AssemblyScript (TS-based)
* <!-- .element: class="fragment" data-fragment-index="2" --> 🐣 Walt (alternative JS-based syntax for `.wat`)
* <!-- .element: class="fragment" data-fragment-index="2" --> 🐣 Wam (`.wast` superset)

Note:
  https://github.com/appcypher/awesome-wasm-langs
  @publicquestion Кто здесь frontend?
  @publicquestion А кто stand-alone/mobile?

  Go - нативная поддержка
  walt - https://ballercat.github.io/walt/
  TODO: `.walt` транспилируется в `.wat`?
  TODO: Можно ли называть AssemblyScript/Walt/Wam прямыми альтернативами?

---

### Пример на Rust

```rust
#[no_mangle]
pub fn add(a: i32, b: i32) -> i32 {
  a + b
}
```

<pre class="javascript"><code data-trim data-noescape>
import('<mark>./Cargo.toml</mark><!-- .element: class="fragment" data-fragment-index="2" -->')
.then(module => {
  alert(\`a=1, b=2, a+b=${module.add(1, 2)}\`);
})
.catch(console.error);
</code></pre>
<!-- .element: class="fragment fade-in" data-fragment-index="1" -->

Note:
  1. Преположим, что мы реализовали логику на Rust

---

### Варианты импорта 2️⃣

* <!-- .element: class="grayed" --> объявление внутри js – бинарный формат
* <!-- .element: class="grayed" --> объявление внутри js – `.wat`-исходники
* <!-- .element: class="grayed" --> импорт файла .wasm

* модуль
```js
import module from './lib.rs';
```

* бандл
```js
import module from './Cargo.toml';
```

* динамический `import()`
```js
let module = await import('./Cargo.toml')
```

Note:

---

### Что нам интересно от сборщика

* прямой импорт (.rs/.toml)
* асинхронный импорт
* livereload при изменении исходников

также интересно: <!-- .element: class="fragment" data-fragment-index="1" -->
* размер бандла <!-- .element: class="fragment" data-fragment-index="1" -->
* скорость сборки <!-- .element: class="fragment" data-fragment-index="1" -->

Note:

---

### Структура проекта (Rust)

design-time <!-- .element: class="fragment" data-fragment-index="1" -->
* <!-- .element: class="fragment" data-fragment-index="1" --> `./{package.json,index.js}`
* <!-- .element: class="fragment" data-fragment-index="1" --> `./src/`
* <!-- .element: class="fragment" data-fragment-index="2" --> `./{Cargo.toml,lib.rs}`

build-time <!-- .element: class="fragment" data-fragment-index="3" -->
<!-- * `./bin/` – бинарники wasm-bindgen -->
* <!-- .element: class="fragment" data-fragment-index="4" --> `./target/` – результат работы `cargo build`
* <!-- .element: class="fragment gray" data-fragment-index="5" --> `./pkg/` – предварительный wasm, вместе с ts-определениями (только для wasm-pack?)
* <!-- .element: class="fragment" data-fragment-index="6" --> `./dist/` – готовый бандл (wasm+js)

Note:
  предположим, что всё располагаем

---

### JS-бандлеры

* <!-- .element: class="fragment highlight-current-blue" data-fragment-index="1" --> WebPack
* <!-- .element: class="fragment highlight-current-blue" data-fragment-index="1" --> Parcel
* <!-- .element: class="fragment highlight-current-blue" data-fragment-index="1" --> Rollup
* Gulp/Grunt

Note:
  рассматриваем WebPack, Parcel, Rollup
  Gulp/Grunt - не так интересно

---

<!-- .slide: class="plugstable" -->
### Доступные плагины для основных бандлеров (Rust)

|plugin                 |status        |comment                 |
|-----------------------|--------------|------------------------|
|parcel-plugin-cargo-web|OK            |                        |
|parcel-plugin-rustwasm |won't build   |last commit 8 months ago|
|parcel-plugin-wasm.rs  |OK            |                        |
|rollup-plugin-rust     |OK            |Require additional steps*|
|wasm-pack via webpack  |<mark>TODO:</mark>         |                        |

\* <mark>TODO:</mark> <!-- .element: class="fragment" data-fragment-index="1" -->

Note:
  parcel-plugin-cargo-web     |37 GH stars |Latest commit 5498476  on Jan 28
  parcel-plugin-rustwasm      |2  GH stars |Latest commit 09ff5fe  on Sep 25, 2018
  parcel-plugin-wasm.rs       |18 GH stars |Latest commit dbc50f1  on Apr 22
  rollup-plugin-rust          |17 GH stars |Latest commit ef411b8  on Feb 2
  @wasm-tool/wasm-pack-plugin |34 GH stars |Latest commit 8db33a1  on Mar 14

---

<!-- .slide: class="plugstable" -->
### Возможности плагинов (Rust)

|plugin                 |livereload|async load|import .rs|import .toml|wasm_bindgen extern|
|-----------------------|----------|----------|----------|------------|------------|
|parcel-plugin-cargo-web|JS        |YES       |sync/async|NO          |NO          |
|parcel-plugin-rustwasm |JS        |YES       |sync/async|NO          |YES         |
|parcel-plugin-wasm.rs  |JS        |NO        |sync      |sync        |YES         |
|rollup-plugin-rust     |Won't Work|YES       |async     |NO          |NO          |
|<span>wasm-pack via webpack</span><!-- .element: class="fragment highlight-red" data-fragment-index="3" -->  |<span>JS/Rust</span><!-- .element: class="fragment highlight-red" data-fragment-index="2" -->   |YES<sup>*</sup>|NO   |NO          |NO?         |

\* webpack require loading of compiled wasm binary  <!-- .element: class="fragment" data-fragment-index="1" -->

Note:
  TODO: move wasm_bindgen extern into first feature column

---

<!-- .slide: class="plugstable" -->
### Минимальный пример (Rust)

|plugin                 |project sources |distro zip         |deps in Cargo.toml + build time                     |status|
|-----------------------|---------------:|------------------:|----------------------------------------------------|------|
|parcel-plugin-cargo-web|     87&nbsp;MiB|       416&nbsp;KiB|no deps in Cargo.toml - ✨  Built in 3.87s.         |OK    |
|parcel-plugin-rustwasm |    108&nbsp;MiB|        17&nbsp;KiB|wasm-bindgen          - ✨  Built in 124.13s.       |NO? <mark>TODO:</mark>   |
|parcel-plugin-wasm.rs  |    110&nbsp;MiB|        17&nbsp;KiB|wasm-bindgen          - ✨  Built in 95.29s.        |OK    |
|rollup-plugin-rust     |     10&nbsp;MiB|        35&nbsp;KiB|no deps in Cargo.toml - created dist/index.js in 2s|Require additional steps and revise|
|<span>wasm-pack via webpack</span><!-- .element: class="fragment highlight-current-red" -->  |  392.4&nbsp;MiB|       975&nbsp;KiB|wasm-bindgen, web-sys - 68s                        |OK    |

Note:
  |plugin                 |distro size/zip             |
  |-----------------------|---------------------------:|
  |parcel-plugin-cargo-web|  1.4&nbsp;MiB/416&nbsp;KiB |
  |parcel-plugin-rustwasm |   43&nbsp;KiB/17&nbsp;KiB  |
  |parcel-plugin-wasm.rs  |   42&nbsp;KiB/17&nbsp;KiB  |
  |rollup-plugin-rust     |  220&nbsp;KiB/35&nbsp;KiB  |
  |wasm-pack via webpack  |  3.4&nbsp;MiB/975&nbsp;KiB |

---

### 1️⃣ parcel-plugin-cargo-web

```js
import module from './lib.rs';
// module.greet('John Doe');
alert(`a=1, b=2, a+b=${module.add(1, 2)}`);
```

Note:

---

### 2️⃣ parcel-plugin-rustwasm

```js
import('./lib.rs').then(module => {
  module.greet('John Doe');
  alert(`a=1, b=2, a+b=${module.add(1, 2)}`);
});
```

Note:

---

### 3️⃣ parcel-plugin-wasm.rs

```js
import module from './Cargo.toml';
module.greet('John Doe');
alert(`a=1, b=2, a+b=${module.add(1, 2)}`);
```

Note:

---

### 4️⃣ rollup-plugin-rust

```js
import './node_modules/buffer-es6/index.js';
import instantiateWasm from './lib.rs';

async function init() {
  const { instance } = await instantiateWasm();
  return instance.exports;
}

init().then(({ add, greet }) => {
  // greet('John Doe');
  alert(`a=1, b=2, a+b=${add(a, b)}`);
});
```

Note:

---

### 5️⃣ wasm-pack via webpack

```js
import('./pkg')
.then(module => {
  // module.greet('John Doe');
  alert(`a=1, b=2, a+b=${module.add(1, 2)}`);
})
.catch(console.error);
```

Note:

---

Время вопросов

Note:
  помещайте пример в папочку `example`
