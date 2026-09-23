# Traits: gedrag delen

Een struct bevat gegevens. Een **trait** beschrijft gedrag dat verschillende
types kunnen hebben. Je kunt een trait lezen als een afspraak:

> Een type dat deze trait gebruikt, heeft deze method.

## 1. Een trait beschrijven

```rust
trait Levende {
    fn is_levend(&self) -> bool;
}
```

De trait zegt nog niet hoe `is_levend` werkt. Dat bepaalt ieder type zelf.

## 2. Een trait implementeren

```rust
trait Levende {
    fn is_levend(&self) -> bool;
}

struct Speler {
    gezondheid: i32,
}

impl Levende for Speler {
    fn is_levend(&self) -> bool {
        self.gezondheid > 0
    }
}

fn main() {
    let speler = Speler { gezondheid: 10 };
    println!("{}", speler.is_levend());
}
```

`impl Levende for Speler` betekent: `Speler` volgt de afspraken van `Levende`.

## 3. Dezelfde trait voor meerdere types

```rust
trait Levende {
    fn is_levend(&self) -> bool;
}

struct Vijand {
    gezondheid: i32,
}

impl Levende for Vijand {
    fn is_levend(&self) -> bool {
        self.gezondheid > 0
    }
}
```

Een speler en een vijand kunnen dus allebei `is_levend()` gebruiken, ook al
zijn het verschillende structs.

## Oefeningen

1. [Een trait maken](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_13/1_trait.rs)
2. [Een trait implementeren](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_13/2_speler.rs)
3. [Een vijand implementeren](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_13/3_vijand.rs)
4. [Levende types gebruiken](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_13/4_trait_levend.rs)