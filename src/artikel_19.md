# Modules en zichtbaarheid

Wanneer een programma groter wordt, zetten we code in meerdere modules. Zo
blijft ieder bestand overzichtelijk.

## Een module in hetzelfde bestand

```rust
mod speler {
    pub fn begroet() {
        println!("Welkom, speler!");
    }
}

fn main() {
    speler::begroet();
}
```

`pub` maakt een functie zichtbaar buiten de module. Zonder `pub` kan alleen de
module zelf haar gebruiken.

## Een struct openbaar maken

```rust
mod game {
    pub struct Speler {
        pub naam: String,
    }
}

fn main() {
    let speler = game::Speler {
        naam: String::from("Arin"),
    };
    println!("{}", speler.naam);
}
```

De struct en het veld zijn hier allebei `pub`, omdat `main` ze buiten de module
gebruikt.

## Meerdere bestanden

Een project kan bijvoorbeeld zo groeien:

```text
src/
├── main.rs
├── speler.rs
└── vijand.rs
```

In `main.rs` schrijf je dan `mod speler;`. Rust zoekt vervolgens de module in
`src/speler.rs`.

## Oefeningen

1. [Een module aanroepen](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_19/1_module.rs)
2. [Een publieke functie](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_19/2_pub.rs)
3. [Een publieke struct](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_19/3_struct.rs)
4. [Code ordenen](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_19/4_game.rs)