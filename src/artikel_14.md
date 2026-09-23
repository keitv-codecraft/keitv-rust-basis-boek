# Derive-attributen: automatisch gedrag toevoegen

Met `#[derive(...)]` vraagt je aan Rust om eenvoudige standaardfunctionaliteit
voor een type te maken. Dat voorkomt herhaling.

## `Debug`

Met `Debug` kunnen we een hele waarde tijdens het programmeren bekijken:

```rust
#[derive(Debug)]
struct Speler {
    naam: String,
    gezondheid: i32,
}

fn main() {
    let speler = Speler {
        naam: String::from("Arin"),
        gezondheid: 100,
    };
    println!("{speler:?}");
}
```

## `Clone`

Met `Clone` kan een waarde bewust worden gekopieerd:

```rust
#[derive(Clone)]
struct Wapen {
    naam: String,
}

fn main() {
    let eerste = Wapen {
        naam: String::from("Zwaard"),
    };
    let tweede = eerste.clone();
    println!("{} en {}", eerste.naam, tweede.naam);
}
```

## `PartialEq`

Met `PartialEq` kunnen we waarden vergelijken met `==` en `!=`.

```rust
#[derive(Debug, PartialEq)]
enum Status {
    Menu,
    Spelen,
}

fn main() {
    let status = Status::Spelen;
    assert_eq!(status, Status::Spelen);
}
```

## `Default`

`Default` geeft een handige beginwaarde. Voor getallen is dat `0` en voor een
`String` een lege tekst. Je kunt ook eigen beginwaarden maken met een functie.

## Oefeningen

1. [Debug gebruiken](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_14/1_debug.rs)
2. [Een wapen klonen](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_14/2_clone.rs)
3. [Status vergelijken](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_14/3_gelijk.rs)
4. [Een standaardspeler](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_14/4_default.rs)