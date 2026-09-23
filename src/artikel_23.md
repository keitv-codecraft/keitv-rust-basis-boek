# Generics: code hergebruiken voor verschillende types

Soms doet dezelfde functie hetzelfde met verschillende types. Met een
**generic** schrijven we die functie één keer.

## Een generieke functie

```rust
fn eerste<T>(waarden: &[T]) -> Option<&T> {
    waarden.first()
}

fn main() {
    let getallen = [10, 20];
    let namen = ["Arin", "Borin"];

    println!("{}", eerste(&getallen).unwrap());
    println!("{}", eerste(&namen).unwrap());
}
```

`T` is een plaatsaanduiding voor een type. De functie werkt daardoor met
getallen, tekst en andere types.

## Een generieke struct

```rust
struct Schatkist<T> {
    inhoud: T,
}

fn main() {
    let goud = Schatkist { inhoud: 100 };
    let wapen = Schatkist {
        inhoud: String::from("Zwaard"),
    };
    println!("{} en {}", goud.inhoud, wapen.inhoud);
}
```

## Generics met een trait

Een functie kan vragen om een type dat bepaald gedrag heeft:

```rust
use std::fmt::Display;

fn toon<T: Display>(waarde: T) {
    println!("Waarde: {waarde}");
}
```

`T: Display` betekent dat `T` met `{}` geprint kan worden.

## Oefeningen

1. [Een generieke functie](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_23/1_generiek.rs)
2. [Een generieke kist](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_23/2_kist.rs)
3. [Een trait-bound](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_23/3_display.rs)
4. [De eerste waarde](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_23/4_eerste.rs)