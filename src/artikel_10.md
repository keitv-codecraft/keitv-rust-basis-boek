# Structs: gegevens bij elkaar bewaren

Een speler heeft meer dan één stukje informatie. Met een `struct` bundelen we
gegevens die bij elkaar horen.

## 1. Een struct maken

```rust
struct Speler {
    naam: String,
    gezondheid: i32,
    goud: i32,
}
```

`Speler` is nu een eigen datatype. `naam`, `gezondheid` en `goud` heten de
velden van de struct.

## 2. Een struct gebruiken

```rust
struct Speler {
    naam: String,
    gezondheid: i32,
    goud: i32,
}

fn main() {
    let speler = Speler {
        naam: String::from("Arin"),
        gezondheid: 100,
        goud: 25,
    };

    println!("{} heeft {} goud.", speler.naam, speler.goud);
}
```

Met een punt (`.`) lezen we een veld uit.

## 3. Velden veranderen

De hele struct moet `mut` zijn als we een veld willen veranderen:

```rust
struct Speler {
    naam: String,
    gezondheid: i32,
}

fn main() {
    let mut speler = Speler {
        naam: String::from("Arin"),
        gezondheid: 100,
    };

    speler.gezondheid -= 25;
    println!("{} heeft nog {} gezondheid.", speler.naam, speler.gezondheid);
}
```

## 4. Structs en functies

Een functie kan een struct lenen. Zo blijft de functie eigenaarloos:

```rust
struct Speler {
    naam: String,
    gezondheid: i32,
}

fn toon_speler(speler: &Speler) {
    println!("{}: {} gezondheid", speler.naam, speler.gezondheid);
}

fn main() {
    let speler = Speler {
        naam: String::from("Arin"),
        gezondheid: 100,
    };
    toon_speler(&speler);
}
```

## Oefeningen

1. [Een speler-struct maken](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_10/1_struct.rs)
2. [Velden uitlezen](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_10/2_velden.rs)
3. [Gezondheid veranderen](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_10/3_veranderen.rs)
4. [Een struct lenen](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_10/4_lenen.rs)
5. [Een vijand maken](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_10/5_vijand.rs)

In het volgende artikel voegen we gedrag toe met `impl`-blokken.