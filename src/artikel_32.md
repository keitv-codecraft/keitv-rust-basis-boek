# Gevechten

Een gevecht combineert structs, methods, borrowing, enums en lussen. Begin met
één aanval en voeg daarna verdedigen, genezen en kritieke schade toe.

```rust
fn aanval(gezondheid: &mut i32, schade: i32) {
    *gezondheid = (*gezondheid - schade).max(0);
}
```

Test grenzen: nul gezondheid blijft nul en een grote aanval maakt een vijand
niet negatief.