# De eerste speelbare RPG

Een eerste versie kan uit drie onderdelen bestaan:

```rust
struct Speler { gezondheid: i32 }
struct Vijand { gezondheid: i32 }

fn aanval(speler: &Speler, vijand: &mut Vijand) {
    let _ = speler;
    vijand.gezondheid -= 10;
}
```

Voeg daarna een eenvoudige spelstatus toe: menu, spelen of game over. Bouw
steeds één werkende stap en test die voordat je de volgende toevoegt.