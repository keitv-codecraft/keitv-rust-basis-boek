# Inventaris en uitrusting

Een inventory kan een `Vec<String>` of een `HashMap<String, i32>` zijn. Een
wapen past goed in een eigen struct:

```rust
struct Wapen {
    naam: String,
    schade: i32,
}

struct Inventaris {
    wapens: Vec<Wapen>,
}
```

Laat methods zoals `voeg_toe` en `eerste_wapen` de regels van de inventory
bewaken. De rest van het spel hoeft dan niet te weten hoe de lijst werkt.