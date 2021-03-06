---
title: Обзор бандлеров в JS
# theme: solarized
revealOptions:
  transition: 'fade'
---
# Обзор бандлеров в JS
## Как приготовить .wasm

<small>Алексей Краснобаев, BDO Unicon</small>

Note:
  Рассказать предысторию, как я выбирал билдер полгода назад.

  На выхлопе получилась пара демок.

---

### План на сегодня

* как я начал заниматься WebAssembly (демки) <!-- .element: class="fragment highlight-current-green" data-fragment-index="1" -->
* импортируем готовые .wasm файлы в JS
* собираем .wasm через бандлер
* сравним сборщики (на примере Rust)

Note:

---

### Демки

* Простые примеры использования бандлеров https://github.com/krasnobaev/webasm-jsbundlers
* FM-синтезатор (WIP) https://github.com/krasnobaev/webasm-fmosc
* WebGL-tutorial на основе MDN-примеров (WIP) https://github.com/krasnobaev/webasm-webgl-tutorial

Note:

----

FM-синтезатор (WIP)

<small>https://github.com/krasnobaev/webasm-fmosc</small>

<iframe data-src="https://krasnobaev.github.io/webasm-fmosc/index.html" style="background-color:#fff;height:60rem;width:100%;"></iframe>

----

#### WebGL-tutorial на основе MDN-примеров (WIP)

<small>https://github.com/krasnobaev/webasm-webgl-tutorial</small>

<iframe data-src="https://krasnobaev.github.io/webasm-webgl-tutorial/dist/#rust-5" style="background-color:#fff;height:30rem;width:100%;"></iframe>

Note:
  исходные примеры на JS https://mdn.github.io/webgl-examples/

----

#### Примеры использования бандлеров

<small>https://github.com/krasnobaev/webasm-jsbundlers</small>

* Просто Hello World для каждого плагина
* Примеры будут в конце презентации

---

### План на сегодня

* как я начал заниматься WebAssembly (демки)
* импортируем готовые .wasm файлы в JS <!-- .element: class="green-text" -->
* собираем .wasm через бандлер
* сравним сборщики (на примере Rust)

---

### простой пример add.wasm

.wasm – бинарный формат
```bash
0000000 00 61 73 6d 01 00 00 00 01 07 01 60 02 7f 7f 01
0000010 7f 03 02 01 00 07 07 01 03 61 64 64 00 00 0a 09
0000020 01 07 00 20 00 20 01 6a 0b
0000029
```

.wat – [WebAssembly text format](https://webassembly.github.io/spec/core/text/index.html)
```bash
(module
  (type $t0 (func (param i32 i32) (result i32)))
  (func $add (type $t0) (param $p0 i32) (param $p1 i32) (result i32)
    get_local $p0
    get_local $p1
    i32.add)
  (export "add" (func $add)))
```

Note:
  полноценная программа

  .wat - официальный стандарт для описания WebAssembly модуля

  .wat может быть транслиован в бинарный формат, удобный для передачи по сети.

  .wat и .wasm транслируются взаимно-однозначно, можно использовать официальный WebAssembly/binaryen

  Предпросмотр доступен в хроме или в VS Сode

----

* .wast – .wat с ассертами, диалект для тестирования
* online WebAssembly IDE https://webassembly.studio
* небольшие примеры .wasm https://github.com/Hanks10100/wasm-examples

Note:

---

### Способы импорта .wasm в JS 1️⃣

#### готовый .wasm

* ручная загрузка файла .wasm по HTTP
* сериализованный .wasm в JS
* компиляция .wat в runtime

Note:
  импорт wasm в js потому что другого способа использования в браузерном окружении не существует.

  сериализация в BASE64, либо побайтово, как любят делать некоторые плагины.

  в index.html нельзя заинклюдить.

  https://github.com/xtuc/webassemblyjs/tree/master/packages/webassemblyjs

----

#### ручная загрузка файла .wasm по HTTP

```js
fetch('https://krasnobaev.github.io/webasm-jsbundlers/add.wasm')
.then(response =>
  response.arrayBuffer()
).then(bytes =>
  WebAssembly.instantiate(bytes)
).then(results => {
  alert(`a=1, b=2, a+b=${results.instance.exports.add(1,2)}`);
});
```

```js
WebAssembly.instantiateStreaming(
  fetch('https://krasnobaev.github.io/webasm-jsbundlers/add.wasm')
).then(results => {
  alert(`a=1, b=2, a+b=${results.instance.exports.add(1,2)}`);
});
```

Note:
  пожалуй самый логичный способ, но wasm-файл уже должен быть собран

  примеры на основе
  https://developer.mozilla.org/en-US/docs/WebAssembly/Loading_and_running
  https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WebAssembly/instantiate

  в заголовках html грузить нельзя, импорт только через javascript

----

#### сериализованный .wasm в JS

```js
let Buffer = new Uint8Array([
    '00', '61', '73', '6d', '01', '00', '00', '00', '01', '07', '01', '60', '02', '7f', '7f', '01',
    '7f', '03', '02', '01', '00', '07', '07', '01', '03', '61', '64', '64', '00', '00', '0a', '09',
    '01', '07', '00', '20', '00', '20', '01', '6a', '0b',
].map(num => parseInt(num, 16)));
let wasmModule = new WebAssembly.Module(Buffer);
let wasmInstance = new WebAssembly.Instance(wasmModule, []);
alert(`a=1, b=2, a+b=${wasmInstance.exports.add(1, 2)}`);
```

----

#### компиляция .wat в runtime

<pre class="javascript"><code data-trim data-noescape>
const src = `(module
  (func $add (param i32) (param i32) (result i32)
    (get_local 0)
    (get_local 1)
    (i32.add))
  (export "add" (func $add)))`;
let module = <mark>webassemblyjs</mark>.instantiateFromSource(src);
alert(`a=1, b=2, a+b=${module.exports.add(1, 2)}`);
</code></pre>

---

### План на сегодня

* как я начал заниматься WebAssembly (демки)
* импортируем готовые .wasm файлы в JS
* собираем .wasm через бандлер <!-- .element: class="green-text" -->
* сравним сборщики (на примере Rust)

---

### На чём готовить .wasm

Привычные языки <!-- .element: class="fragment" data-fragment-index="1" -->
* <!-- .element: class="fragment nobullets" data-fragment-index="1" --> 🐥 C/C++ via emscripten/LLVM
* <!-- .element: class="fragment nobullets" data-fragment-index="1" --> <div><!-- .element: class="fragment highlight-current-blue" data-fragment-index="3" --> 🐥 Rust via wasm-pack</div>
* <!-- .element: class="fragment nobullets" data-fragment-index="1" --> 🐥 Go via GOARCH=wasm
* <!-- .element: class="fragment nobullets" data-fragment-index="1" --> 🐣 JavaScript via Duktape

Специализированные языки <!-- .element: class="fragment" data-fragment-index="2" -->
* <!-- .element: class="fragment nobullets" data-fragment-index="2" --> 🐥 AssemblyScript (TS-based, Binaryen)
* <!-- .element: class="fragment nobullets" data-fragment-index="2" --> 🐣 Walt (alternative JS-based syntax for .wat)
* <!-- .element: class="fragment nobullets" data-fragment-index="2" --> 🐣 Wam (.wast superset)

Note:
  https://github.com/appcypher/awesome-wasm-langs

  binaryen – https://github.com/WebAssembly/binaryen

  walt - https://ballercat.github.io/walt/

  TODO: `.walt` транспилируется в .wat?

----

### На чём готовить .wasm

#### 🐥 Rust

* cargo build --target=wasm32-unknown-unknown
* wasm-bindgen
* <!-- .element: class="green-text" --> wasm-pack

Note:
  https://rustwasm.github.io/wasm-bindgen/examples/hello-world.html

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

  `#[no_mangle]` vs `#[wasm_bindgen]`

---

### Способы импорта wasm в JS 2️⃣

#### Автоматизируем сборку .wasm

* импорт конкретных модулей
```js
import module from './lib.rs';
```

* импорт всего бандла
```js
import module from './Cargo.toml';
```
* динамический import()
```js
let module = await import('./Cargo.toml'); ////
```

Note:
  удобство модулей - небольшие конкретные функции, обработка сайд-эффектов

  удобство бандлов - мб движок, большой код

  динамический `import()` как модулей так и бандлов

---

Возможности потенциального сборщика .wasm
* <!-- .element: class="fragment" data-fragment-index="1" --> доступность JS-окружения в .wasm (e.g. <code class="nowrap" data-trim data-noescape>#[wasm_bindgen]</code>)
* <!-- .element: class="fragment" data-fragment-index="1" --> сериализация .wasm внутрь .js
* <!-- .element: class="fragment" data-fragment-index="1" --> асинхронная загрузка .wasm
* <!-- .element: class="fragment" data-fragment-index="2" --> прямой импорт (.rs/.toml)
* <!-- .element: class="fragment" data-fragment-index="2" --> livereload при изменении исходников (.rs/.c/…)

также интересно: <!-- .element: class="fragment" data-fragment-index="3" -->
* размер бандла / скорость сборки <!-- .element: class="fragment" data-fragment-index="3" -->
* отсутствие лишних абстракций <!-- .element: class="fragment" data-fragment-index="3" -->

Note:
  доступность JS-окружения в .wasm - по умолчанию можно только читать числа из .wasm, чтобы общаться более сложными объектами или управлять из .wasm например canvas'ом, нужно использовать дополнительные обёртки типа wasm_bindgen (на примере rust).

  основные фичи: сериализация

  второстепенные (DX): прямой импорт, livereload

  НФТ: размер бандла / скорость сборки

  так например rollup-plugin-rust вынуждает делать ручной импорт buffer-es6 в исходниках

---

### Структура проекта (Rust)

design-time <!-- .element: class="fragment" data-fragment-index="1" -->
* <!-- .element: class="fragment" data-fragment-index="1" --> ./package.json, ./index.js
* <!-- .element: class="fragment" data-fragment-index="1" --> ./src/
* <!-- .element: class="fragment" data-fragment-index="2" --> ./Cargo.toml, ./lib.rs

build-time (Rust) <!-- .element: class="fragment" data-fragment-index="3" -->
* <!-- .element: class="fragment" data-fragment-index="4" --> ./target/, ./pkg/ – результат работы сборщика
* <!-- .element: class="fragment" data-fragment-index="6" --> ./dist/ – готовый релиз (.wasm+.js)

Note:
  всё располагаем в src

  ещё бывает wasm-bindgen мусорит в ./bin/

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

### План на сегодня

* как я начал заниматься WebAssembly (демки)
* импортируем готовые .wasm файлы в JS
* собираем .wasm через бандлер
* сравним сборщики (на примере Rust) <!-- .element: class="green-text" -->

Note:
  Зачем с нуля настраивать свой pipeline если есть готовый.

---

<!-- .slide: class="plugstable" -->
### Доступные плагины для основных бандлеров (Rust)

|plugin                                                          |GitHub|builder (cli) |                          |
|----------------------------------------------------------------|------|--------------|--------------------------|
|parcel-plugin-cargo-web  <!-- .element: class="green-text" -->  |37 ⭐ |cargo build   |                          |
|parcel-plugin-rustwasm   <!-- .element: class="red-text"   -->  |2 ⭐  |wasm-bindgen  |                          |
|parcel-plugin-wasm.rs    <!-- .element: class="green-text" -->  |18 ⭐ |wasm-pack     |                          |
|rollup-plugin-rust       <!-- .element: class="red-text"   -->  |17 ⭐ |cargo build   |                          |
|rollup-plugin-wasm                                              |64 ⭐ |–             |                          |
|wasm-pack via webpack    <!-- .element: class="green-text" -->  |34 ⭐ |wasm-pack     |                          |

<span>usable <!-- .element: class="green-text" --> </span>
<span>unusable <!-- .element: class="red-text" --> </span>
untested

Note:
  usable – билд отрабатывает как и подразумевается

  unusable – не удаётся получить билд без дополнительных телодвижений

  parcel-plugin-wasm.rs - есть пример использования у команды rustwasm https://github.com/rustwasm/rust-parcel-template

  Rust official docs: https://rustwasm.github.io/docs/wasm-bindgen/examples/without-a-bundler.html

  Rust official docs2: https://rustwasm.github.io/docs/wasm-bindgen/reference/deployment.html

  `cargo build` means `cargo build --target=wasm32-unknown-unknown`

  parcel-plugin-cargo-web     |37 ⭐ |Latest commit 5498476  on Jan 28        |                          |
  parcel-plugin-rustwasm      |2 ⭐  |Latest commit 09ff5fe  on Sep 25, 2018  |last commit 8 months ago  |
  parcel-plugin-wasm.rs       |18 ⭐ |Latest commit dbc50f1  on Apr 22        |                          |
  rollup-plugin-rust          |17 ⭐ |Latest commit ef411b8  on Feb 2         |допилить напильником      |
  rollup-plugin-wasm          |64 ⭐ |                                        |необходим готовый .wasm   |
  @wasm-tool/wasm-pack-plugin |34 ⭐ |Latest commit 8db33a1  on Mar 14        |                          |

---

<!-- .slide: class="plugstable" -->
### Возможности плагинов (Rust)

|plugin                                        |#[wasm_bindgen] in .rs|livereload|.wasm async load|import .rs|import .toml|serialize .wasm in .js|
|----------------------------------------------------------------|----|----------|----------------|----------|------------|----------------------|
|parcel-plugin-cargo-web  <!-- .element: class="green-text" -->  |NO  |.js       |YES             |sync/async|CATCH FIRE  |                      |
|parcel-plugin-rustwasm   <!-- .element: class="red-text" -->    |YES |.js       |YES             |sync/async|            |                      |
|parcel-plugin-wasm.rs    <!-- .element: class="green-text" -->  |YES |.js       |                |sync      |sync        |                      |
|rollup-plugin-rust       <!-- .element: class="red-text" -->    |NO  |<i>NO</i> |YES             |async     |            |YES                   |
|<span class="green-text">wasm-pack via webpack</span>           |YES |<span>.js/.rs</span><!-- .element: class="fragment highlight-green" data-fragment-index="2" -->|YES| | | |
<!-- |rollup-plugin-wasm                                              |    |?         |                |          |            |YES                   | -->

Note:
  CATCH FIRE - не зависнет, но призадумается

  livereload - какой-то из плагинов использует порт по умолчанию (35675?)

  конечно стоит делать Issue

  serialize .wasm file in .js - специально в конце - т.к. сомнительная функция

  parcel-plugin-cargo-web - нет поддержки директивы #[wasm_bindgen]

  parcel-plugin-rustwasm  - не собирается, раньше всё работало, нужно перепроверять

  parcel-plugin-wasm.rs   - всё бы ничего но нет livereload по .rs

  rollup-plugin-rust      - не работает так как заявлено в документации к плагину

  wasm-pack via webpack   - некрасивый импорт ./pkg и большой билд, в остальном всё в порядке

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

\* <small>примерное время без усреднений и сборки с #[wasm_bindgen]</small>

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

### lib.rs

```rust
extern crate wasm_bindgen;

use wasm_bindgen::prelude::*;

#[wasm_bindgen]
extern {
  fn alert(s: &str);
}

#[wasm_bindgen]
pub fn greet(name: &str) {
  alert(&format!("Hello, {}!", name));
}

// #[no_mangle]
#[wasm_bindgen]
pub fn add(a: i32, b: i32) -> i32 {
  return a + b
}
```
<!-- .element: class="bigcode" -->

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

----

#### Troubleshooting

```bash
npm run clean # убрать все build-time артефакты
rm -r ~/.cargo/registry/cache
rustup default nightly
rustup update
# обновить зависимости package.json/Cargo.toml
```

Note:
  ~/.cargo/registry/cache – только кэш собранных пакетов, исходники в другом месте

  обновить зависимости package.json/Cargo.toml – удобнее использовать плагины с подсветкой текущей последней версии, доступны как для Cargo.toml так и для package.json (VSCode OFK)

---

### Время выводов

* Я остановился на @wasm-tool/wasm-pack-plugin
* Если строить свой pipeline – лучше брать за основу wasm-pack или wasm-bindgen
* Если делаете свои плагины, делайте папочку example

Note:

---

Спасибо за внимание

<img src="/asset/qr-gist-github-com.svg" />

Алексей Краснобаев <alekseykrasnobaev@gmail.com>
