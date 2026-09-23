# Strings in de praktijk

Rust heeft twee teksttypes die je vaak ziet. Een `String` kan groeien en
veranderen. Een `&str` is een geleende tekst die je alleen leest.

## Een `String` veranderen

```rust
fn main() {
    let mut bericht = String::from("Hallo");
    bericht.push('!');
    bericht.push_str(" Welkom.");
    println!("{bericht}");
}
```

Gebruik `push` voor één `char` en `push_str` voor een stuk tekst.

## Tekst lenen

Functies die tekst alleen lezen, kunnen `&str` gebruiken:

```rust
fn eerste_woord(tekst: &str) -> &str {
    tekst.split_whitespace().next().unwrap_or("")
}

fn main() {
    let bericht = String::from("De deur opent");
    println!("{}", eerste_woord(&bericht));
}
```

`&str` werkt met zowel een `String` als een tekstliteral.

## Veelgebruikte methods

```rust
fn main() {
    let tekst = "De draak verschijnt";
    println!("Lengte: {}", tekst.len());
    println!("Bevat draak: {}", tekst.contains("draak"));
}
```

Let op: `len()` telt bytes. Voor eenvoudige Nederlandse tekst is dat meestal
geen probleem, maar bij alle Unicode-tekens is tekst iets ingewikkelder.

## Oefeningen

1. [Tekst opbouwen](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_18/1_opbouwen.rs)
2. [Tekst lenen](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_18/2_lenen.rs)
3. [Een woord vinden](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_18/3_eerste_woord.rs)
4. [Tekst controleren](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_18/4_contains.rs)