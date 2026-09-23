# `impl`-blokken en methods

In het vorige artikel maakten we structs. Met een `impl`-blok zetten we gedrag
dat bij een struct hoort op een overzichtelijke plek.

## 1. Een method met `&self`

```rust
struct Speler {
    naam: String,
    gezondheid: i32,
}

impl Speler {
    fn toon_naam(&self) {
        println!("Speler: {}", self.naam);
    }
}

fn main() {
    let speler = Speler {
        naam: String::from("Arin"),
        gezondheid: 100,
    };
    speler.toon_naam();
}
```

`&self` betekent dat de method de struct alleen leest. We roepen een method
aan met een punt: `speler.toon_naam()`.

## 2. Een method met `&mut self`

Een method die gegevens verandert, gebruikt `&mut self`:

```rust
struct Speler {
    gezondheid: i32,
}

impl Speler {
    fn neem_schade(&mut self, schade: i32) {
        self.gezondheid -= schade;
    }
}

fn main() {
    let mut speler = Speler { gezondheid: 100 };
    speler.neem_schade(25);
    println!("{}", speler.gezondheid);
}
```

De waarde moet `mut` zijn als een method haar verandert.

## 3. Een method met een resultaat

Een method kan ook informatie teruggeven:

```rust
struct Speler {
    gezondheid: i32,
}

impl Speler {
    fn is_levend(&self) -> bool {
        self.gezondheid > 0
    }
}

fn main() {
    let speler = Speler { gezondheid: 10 };
    println!("Levend: {}", speler.is_levend());
}
```

## Oefeningen

1. [Een method maken](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_11/1_method.rs)
2. [Een naam tonen](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_11/2_naam.rs)
3. [Schade als method](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_11/3_method_schade.rs)
4. [Levend controleren](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_11/4_levend.rs)
5. [Genezen](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_11/5_genezen.rs)