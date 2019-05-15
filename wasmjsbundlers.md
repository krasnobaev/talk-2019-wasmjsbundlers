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
  Рассмотрим способы подключения WebAssembly модулей в веб-приложения.

---

### План на сегодня

*
* рассмотрим варианты импорта .wasm файлов
* увидим примеры сборки

---

### простой пример `add.wasm`

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
  полноценная программа

  .wat - официальный стандарт для описания WebAssembly модуля

  .wat может быть транслиован в бинарный формат, удобный для передачи по сети.

  .wat и .wasm транслируются взаимно-однозначно

----

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
  Также можно встретить формат .wast, более удобный для чтения.

  но этот формат не стандартизирован, хотя и описан в официальном стандарте

---

### Варианты импорта 1️⃣

* объявление внутри js – бинарный формат <mark>TODO:</mark>
* объявление внутри js – `.wat`-исходники <mark>TODO:</mark>
* ручная загрузка файла .wasm <mark>TODO:</mark>

----

* объявление внутри js – бинарный формат <mark>TODO:</mark>
```js
let instance = WebAssembly.instantiate(Buffer.from([0,61,…]));
alert(`a=1, b=2, a+b=${instance.exports.add(1, 2)}`);
```

----

* объявление внутри js – `.wat`-исходники <mark>TODO:</mark>
```js
const src = '(module (type $t0 (func (param i32 i32) (result i32)))…';
let module = webassemblyjs.instantiateFromSource(src);
alert(`a=1, b=2, a+b=${module.exports.add(1, 2)}`);
```

----

* ручная загрузка файла .wasm <mark>TODO:</mark>

```js
request = new XMLHttpRequest();
request.open('GET', '/add.wasm');
request.responseType = 'arraybuffer';
request.send();

request.onload = function() {
  webassemblyjs
    .instantiate(request.response)
    .then((module) => {
      alert(`a=1, b=2, a+b=${module.instance.exports.add(1, 2)}`);
    });
};
```
<!-- .element: class="big code" -->

Note:
  импорт файла .wasm

  пример на основе https://webassembly.js.org/docs/example-add-wasm.html

  в заголовках html грузить нельзя, импорт только через javascript

  проблема в том, что

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

  Варианты сборщиков для Rust: cargo, wasm-bindgen, wasm-pack

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
* <!-- .element: class="grayed" --> ручная загрузка файла .wasm
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

Возможности бандлера
* <!-- .element: class="fragment" data-fragment-index="1" --> сериализация .wasm внутрь .js
* <!-- .element: class="fragment" data-fragment-index="1" --> асинхронный импорт
* <!-- .element: class="fragment" data-fragment-index="1" --> <mark>TODO:</mark> wasm_bindgen extern?
* <!-- .element: class="fragment" data-fragment-index="2" --> прямой импорт (.rs/.toml)
* <!-- .element: class="fragment" data-fragment-index="2" --> livereload при изменении исходников (.c/.rs/…)

также интересно: <!-- .element: class="fragment" data-fragment-index="3" -->
* размер бандла <!-- .element: class="fragment" data-fragment-index="3" -->
* скорость сборки <!-- .element: class="fragment" data-fragment-index="3" -->

Note:
  основные фичи: сериализация

  второстепенные (DX): прямой импорт, livereload

  НФТ: размер бандла / скорость сборки

---

### Структура проекта (Rust)

design-time <!-- .element: class="fragment" data-fragment-index="1" -->
* <!-- .element: class="fragment" data-fragment-index="1" --> `./{package.json, index.js}`
* <!-- .element: class="fragment" data-fragment-index="1" --> `./src/`
* <!-- .element: class="fragment" data-fragment-index="2" --> `./{Cargo.toml, lib.rs}`

build-time (Rust) <!-- .element: class="fragment" data-fragment-index="3" -->
<!-- * `./bin/` – бинарники wasm-bindgen -->
* <!-- .element: class="fragment" data-fragment-index="4" --> `./target/` – результат работы сборщика
* <!-- .element: class="fragment gray" data-fragment-index="5" --> `./pkg/` – предварительный wasm, вместе с ts-определениями (только для wasm-pack?)
* <!-- .element: class="fragment" data-fragment-index="6" --> `./dist/` – готовый релиз (wasm+js)

Note:
  предположим, что всё располагаем

---

### JS-бандлеры

* <!-- .element: class="fragment highlight-current-blue" data-fragment-index="1" --> Parcel
* <!-- .element: class="fragment highlight-current-blue" data-fragment-index="1" --> Rollup
* <!-- .element: class="fragment highlight-current-blue" data-fragment-index="1" --> WebPack
* Grunt
* Gulp
<!-- <li class="li-gulp"> <i class="fab fa-gulp"></i> Gulp </li> -->
<!-- <li class="li-grunt"> <i class="fab fa-grunt"></i> Grunt </li> -->

Note:
  рассматриваем WebPack, Parcel, Rollup

  Gulp/Grunt - не так интересно, наличие awesome

---

<!-- .slide: class="plugstable" -->
### Доступные плагины для основных бандлеров (Rust)

|plugin                                                          |GitHub|builder         |                          |
|----------------------------------------------------------------|------|----------------|--------------------------|
|parcel-plugin-cargo-web  <!-- .element: class="green-text" -->  |37 ⭐ |`cargo build`   |                          |
|parcel-plugin-rustwasm   <!-- .element: class="red-text"   -->  |2 ⭐  |`wasm-bindgen`  |                          |
|parcel-plugin-wasm.rs    <!-- .element: class="green-text" -->  |18 ⭐ |`wasm-pack`     |                          |
|rollup-plugin-rust       <!-- .element: class="red-text"   -->  |17 ⭐ |`cargo build`   |                          |
|rollup-plugin-wasm                                              |64 ⭐ |–               |                          |
|wasm-pack via webpack    <!-- .element: class="green-text" -->  |34 ⭐ |`wasm-pack`     |                          |

<span>usable <!-- .element: class="green-text" --> </span>
<span>unusable <!-- .element: class="red-text" --> </span>

Note:
  usable – билд отрабатывает как и подразумевается

  unusable – не удаётся получить билд без дополнительных телодвижений

  parcel-plugin-wasm.rs - есть пример использования у команды rustwasm https://github.com/rustwasm/rust-parcel-template

  parcel-plugin-cargo-web     |37 GH stars |Latest commit 5498476  on Jan 28        |                          |
  parcel-plugin-rustwasm      |2  GH stars |Latest commit 09ff5fe  on Sep 25, 2018  |last commit 8 months ago  |
  parcel-plugin-wasm.rs       |18 GH stars |Latest commit dbc50f1  on Apr 22        |                          |
  rollup-plugin-rust          |17 GH stars |Latest commit ef411b8  on Feb 2         |допилить напильником      |
  @wasm-tool/wasm-pack-plugin |34 GH stars |Latest commit 8db33a1  on Mar 14        |                          |
  |rollup-plugin-wasm         |64          |                                        |необходим готовый .wasm   |

---

<!-- .slide: class="plugstable" -->
### Возможности плагинов (Rust)

|plugin                                                          |livereload|.wasm async load|import .rs|import .toml|wasm_bindgen extern(<mark>TODO:</mark>)|serialize .wasm in .js|
|----------------------------------------------------------------|----------|----------|----------|------------|------------|-----------|
|parcel-plugin-cargo-web  <!-- .element: class="green-text" -->  |.js       |YES       |sync/async|            |            |           |
|parcel-plugin-rustwasm   <!-- .element: class="red-text" -->    |.js       |YES       |sync/async|            |YES         |           |
|parcel-plugin-wasm.rs    <!-- .element: class="green-text" -->  |.js       |          |sync      |sync        |YES         |           |
|rollup-plugin-rust       <!-- .element: class="red-text" -->    |<i>NO</i> |YES       |async     |            |            |YES        |
|<span class="green-text">wasm-pack via webpack</span>           |<span>.js/.rs</span><!-- .element: class="fragment highlight-green" data-fragment-index="2" -->|YES| | |NO? |  |
<!-- |rollup-plugin-wasm                                              |          |?         |          |            |            |YES        | -->

Note:
  serialize .wasm file in .js - специально в конце - т.к. сомнительная функция

  parcel-plugin-cargo-web -

  parcel-plugin-rustwasm  -

  parcel-plugin-wasm.rs   -

  rollup-plugin-rust      -

  wasm-pack via webpack   - webpack require loading of compiled wasm binary

---

<!-- .slide: class="plugstable" -->
### Минимальный пример (Rust)

|plugin                                                          |project sources |distro zip      |build time *  |
|----------------------------------------------------------------|---------------:|---------------:|--------------|
|parcel-plugin-cargo-web  <!-- .element: class="green-text" -->  |     87&nbsp;MiB|    416&nbsp;KiB|≈13s          |
|parcel-plugin-rustwasm   <!-- .element: class="red-text"   -->  |    108&nbsp;MiB|     17&nbsp;KiB|<i>≈124s</i>  |
|parcel-plugin-wasm.rs    <!-- .element: class="green-text" -->  |    110&nbsp;MiB|     17&nbsp;KiB|≈150s         |
|rollup-plugin-rust       <!-- .element: class="red-text"   -->  |     10&nbsp;MiB|     35&nbsp;KiB|≈12s          |
|wasm-pack via webpack    <!-- .element: class="green-text" -->  |  392.4&nbsp;MiB|    975&nbsp;KiB|≈40s          |

\* <small>примерное время без усреднений (без сборки wasm-bindgen)</small>

Note:
  данные на тот момент когда это всё собиралось (12/2018)

  во всех примерах нет готового .wasm (сборка из исходников)

  код для замера скорости
  ```bash
  npm run clean
  rm -rf ~/.cargo/registry/cache
  npm run build
  ```

  |plugin                 |distro size/zip             |
  |-----------------------|---------------------------:|
  |parcel-plugin-cargo-web|  1.4&nbsp;MiB/416&nbsp;KiB |
  |parcel-plugin-rustwasm |   43&nbsp;KiB/17&nbsp;KiB  |
  |parcel-plugin-wasm.rs  |   42&nbsp;KiB/17&nbsp;KiB  |
  |rollup-plugin-rust     |  220&nbsp;KiB/35&nbsp;KiB  |
  |wasm-pack via webpack  |  3.4&nbsp;MiB/975&nbsp;KiB |

----

### 1️⃣ parcel-plugin-cargo-web <!-- .element: class="green-text" -->

```js
import module from './lib.rs';
// module.greet('John Doe');
alert(`a=1, b=2, a+b=${module.add(1, 2)}`);
```

Note:

----

### 2️⃣ parcel-plugin-rustwasm <!-- .element: class="red-text"   -->

```js
import('./lib.rs').then(module => {
  module.greet('John Doe');
  alert(`a=1, b=2, a+b=${module.add(1, 2)}`);
});
```

Note:

----

### 3️⃣ parcel-plugin-wasm.rs <!-- .element: class="green-text" -->

```js
import module from './Cargo.toml';
module.greet('John Doe');
alert(`a=1, b=2, a+b=${module.add(1, 2)}`);
```

Note:

----

### 4️⃣ rollup-plugin-rust <!-- .element: class="red-text"   -->

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

----

### 5️⃣ wasm-pack via webpack <!-- .element: class="green-text" -->

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
