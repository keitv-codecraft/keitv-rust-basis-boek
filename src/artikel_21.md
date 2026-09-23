# `Result<T, E>` en foutafhandeling

`Option` vertelt of iets bestaat. `Result` vertelt ook waarom iets niet is
gelukt. Een `Result` is `Ok(waarde)` of `Err(fout)`.

## Een Result teruggeven

```rust
fn deel(getal: i32, deler: i32) -> Result<i32, String> {
    if deler == 0 {
        Err(String::from("Delen door nul kan niet."))
    } else {
        Ok(getal / deler)
    }
}

fn main() {
    match deel(10, 2) {
        Ok(uitkomst) => println!("Uitkomst: {uitkomst}"),
        Err(fout) => println!("Fout: {fout}"),
    }
}
```

Een functie die kan mislukken geeft de fout terug aan de code die haar heeft
aangeroepen. Zo kan de gebruiker een begrijpelijke melding krijgen.

## `?` gebruiken

Met `?` geef je een fout automatisch door aan de aanroeper:

```rust
fn deel(getal: i32, deler: i32) -> Result<i32, String> {
    if deler == 0 {
        Err(String::from("Delen door nul kan niet."))
    } else {
        Ok(getal / deler)
    }
}

fn deel_en_tel_op(getal: i32, deler: i32) -> Result<i32, String> {
    let uitkomst = deel(getal, deler)?;
    Ok(uitkomst + 1)
}
```

`?` werkt alleen in een functie die zelf een passend `Result` teruggeeft.

## Oefeningen

1. [Een Ok-resultaat](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_21/1_ok.rs)
2. [Een fout teruggeven](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_21/2_err.rs)
3. [Een Result matchen](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_21/3_result_match.rs)
4. [Delen controleren](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_21/4_delen.rs)