# Beslissingen met `if` en `else`

Een programma hoeft niet altijd dezelfde regels uit te voeren. Met `if` laat
je code alleen uitvoeren wanneer een voorwaarde waar is.

```rust
fn main() {
    let gezondheid = 0;

    if gezondheid <= 0 {
        println!("Game over");
    } else {
        println!("Speel verder");
    }
}
```

Een voorwaarde is een `bool`: `true` of `false`. Veelgebruikte vergelijkingen
zijn `==`, `!=`, `>`, `<`, `>=` en `<=`.

## Meerdere mogelijkheden

Met `else if` test je meer dan twee mogelijkheden:

```rust
fn rang(score: i32) -> &'static str {
    if score >= 1000 {
        "Legende"
    } else if score >= 500 {
        "Held"
    } else {
        "Beginner"
    }
}

fn main() {
    println!("Rang: {}", rang(650));
}
```

Let op: de blokken van een `if` moeten hetzelfde type opleveren als je `if`
als waarde gebruikt.

## Oefeningen

Maak de oefeningen [1](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_6/1_levens.rs),
[2](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_6/2_even.rs),
[3](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_6/3_rang.rs) en
[4](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_6/4_game_over.rs).