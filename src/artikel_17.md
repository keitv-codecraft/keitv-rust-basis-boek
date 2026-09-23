# Automatische tests

Tot nu toe controleerden we programma's vooral door ze uit te voeren. Rust kan
ook automatisch controleren of een functie het juiste resultaat geeft.

## 1. Een test schrijven

```rust
fn verdubbel(getal: i32) -> i32 {
    getal * 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn verdubbel_werkt() {
        assert_eq!(verdubbel(5), 10);
    }
}

fn main() {}
```

`#[test]` markeert een testfunctie. `assert_eq!` controleert of twee waarden
gelijk zijn.

## 2. Tests uitvoeren

In een Cargo-project voer je tests uit met:

```text
cargo test
```

Cargo zoekt functies met `#[test]` en voert ze automatisch uit. Een test die
faalt geeft informatie over de verwachte en werkelijke waarde.

Andere handige controles zijn:

```rust
fn main() {
    let gezondheid = 10;
    let eerste = 1;
    let tweede = 2;

    assert!(gezondheid > 0);
    assert_ne!(eerste, tweede);
}
```

Gebruik meerdere kleine tests. Dan zie je sneller welk onderdeel niet werkt.

## Oefeningen

1. [Een functie testen](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_17/1_test.rs)
2. [Schade testen](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_17/2_schade_test.rs)
3. [Een bool testen](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_17/3_assert.rs)
4. [Meerdere tests schrijven](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_17/4_meerdere_tests.rs)