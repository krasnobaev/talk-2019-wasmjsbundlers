
## Run

```bash
reveal-md wasmjsbundlers.md -w --css style.css --template template.html
```

## Print

* open `localhost:1948/wasmjsbundlers.md?print-pdf` or press `e`-key
* adapt paper size (216mm x 279mm)

### Desktape

won't work properly. e.g.

```bash
decktape reveal --load-pause=1000 --slides=11 http://localhost:1948/wasmjsbundlers.md\#/ ./out.pdf
```
