
# Обзор бандлеров JS

Материалы доклада Первого Московского Митапа по WebAssembly (WebAssembly Moscow Meetup #1).

репозиторий с примерами - https://github.com/krasnobaev/webasm-jsbundlers

## Run

```bash
reveal-md wasmjsbundlers.md -w --css style.css --template template.html
```

## Build

```bash
reveal-md wasmjsbundlers.md -w --css style.css --template template.html --static dist
```

## Print

* open `localhost:1948/wasmjsbundlers.md?print-pdf` or press `e`-key
* adapt paper size (216mm x 279mm)

### Desktape

won't work properly. e.g.

```bash
decktape reveal --load-pause=1000 --slides=11 http://localhost:1948/wasmjsbundlers.md\#/ ./out.pdf
```

## Информация о митапе

https://www.meetup.com/ru-RU/wasm-moscow/events/261041755/

https://webassembly-moscow.timepad.ru/event/969924/
