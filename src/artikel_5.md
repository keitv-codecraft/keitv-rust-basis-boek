# Commentaar

Commentaar is tekst voor mensen die de code lezen. Rust voert commentaar niet
uit. Gebruik commentaar om een reden of een lastige keuze uit te leggen, niet
om iedere regel te vertalen.

## 1. Een regel commentaar

Een regel die begint met `//` is commentaar:

```rust
fn main() {
    // De speler begint met drie levens.
    let levens = 3;
    println!("Levens: {levens}");
}
```

Rust negeert alles na `//` tot het einde van de regel.

## 2. Meerdere regels

Met `/*` en `*/` maak je commentaar over meerdere regels:

```rust
/*
Deze tekst wordt niet uitgevoerd.
Je kunt hier tijdelijk uitleg zetten.
*/
fn main() {
    println!("De game start!");
}
```

> [!TIP]
> Commentaar kan tijdelijk helpen tijdens het leren. Verwijder commentaar dat
> later niets meer toevoegt.

## Oefeningen

Maak daarna de oefeningen [1](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_5/1_regel.rs),
[2](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_5/2_bericht.rs) en
[3](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_5/3_meer_regels.rs).

De laatste oefening verwijst terug naar dit artikel.