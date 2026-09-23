# `HashMap`: werken met sleutel en waarde

Een `HashMap` bewaart paren van een sleutel en een waarde. Dat is handig voor
een inventory, scores of instellingen.

## Een HashMap maken

```rust
use std::collections::HashMap;

fn main() {
    let mut inventory = HashMap::new();
    inventory.insert("drankje", 3);
    inventory.insert("pijl", 12);

    println!("Drankjes: {}", inventory["drankje"]);
}
```

## Een waarde veilig opzoeken

Een sleutel bestaat misschien niet. `get` geeft daarom een `Option` terug:

```rust
use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();
    scores.insert("Arin", 100);

    if let Some(score) = scores.get("Arin") {
        println!("Score: {score}");
    }
}
```

## Waarden aanpassen

Met `entry` kun je een beginwaarde gebruiken wanneer een sleutel nog ontbreekt:

```rust
use std::collections::HashMap;

fn main() {
    let mut verslagen = HashMap::new();
    *verslagen.entry("Goblin").or_insert(0) += 1;
    *verslagen.entry("Goblin").or_insert(0) += 1;
    println!("{} Goblins verslagen", verslagen["Goblin"]);
}
```

## Oefeningen

1. [Een inventory maken](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_24/1_inventory.rs)
2. [Een score opzoeken](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_24/2_score.rs)
3. [Een item toevoegen](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_24/3_entry.rs)
4. [Scores optellen](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_24/4_tellen.rs)