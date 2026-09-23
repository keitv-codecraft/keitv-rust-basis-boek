# De spelwereld

Een wereld bestaat uit locaties. Een locatie kan een beschrijving, buren en
een mogelijke vijand hebben.

```rust
struct Locatie {
    naam: String,
    beschrijving: String,
    vijand: Option<String>,
}
```

Begin met een klein aantal locaties. Gebruik `Option` wanneer een locatie niet
altijd een vijand bevat.