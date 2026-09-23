# Enums: één van meerdere mogelijkheden

Een `enum` beschrijft een waarde die één van meerdere mogelijkheden kan zijn.
Dat is handig voor een wapen, een actie of de toestand van een game.

## 1. Een enum maken

```rust
enum Wapen {
    Zwaard,
    Boog,
    Staf,
}

fn main() {
    let wapen = Wapen::Zwaard;
    let _ = wapen;
}
```

De mogelijkheden heten **varianten**. We schrijven de naam van de enum en de
variant samen met `::`.

## 2. Een enum controleren met `match`

`match` kijkt welke variant aanwezig is:

```rust
enum Actie {
    Aanvallen,
    Verdedigen,
    Genezen,
}

fn beschrijf(actie: Actie) {
    match actie {
        Actie::Aanvallen => println!("De speler valt aan."),
        Actie::Verdedigen => println!("De speler verdedigt."),
        Actie::Genezen => println!("De speler geneest."),
    }
}

fn main() {
    beschrijf(Actie::Aanvallen);
}
```

Een `match` moet iedere mogelijkheid behandelen.

## 3. Gegevens in een variant

Een variant kan extra gegevens bevatten:

```rust
enum Schade {
    Normaal(i32),
    Kritiek { hoeveelheid: i32, bonus: i32 },
}

fn main() {
    let aanval = Schade::Kritiek {
        hoeveelheid: 20,
        bonus: 10,
    };

    match aanval {
        Schade::Normaal(hoeveelheid) => println!("Schade: {hoeveelheid}"),
        Schade::Kritiek { hoeveelheid, bonus } => {
            println!("Kritieke schade: {}", hoeveelheid + bonus)
        }
    }
}
```

## Oefeningen

1. [Een wapen-enum maken](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_12/1_wapen.rs)
2. [Een actie tonen](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_12/2_actie.rs)
3. [Match gebruiken](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_12/3_match.rs)
4. [Een enum met getal](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_12/4_enum_schade.rs)
5. [Game-status](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_12/5_status.rs)