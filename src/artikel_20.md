# `Option<T>`: een waarde die misschien ontbreekt

Soms bestaat een waarde wel en soms niet. Een speler kan bijvoorbeeld wel of
geen wapen hebben. Rust gebruikt daarvoor `Option<T>`.

## `Some` en `None`

```rust
fn main() {
    let wapen: Option<String> = Some(String::from("Zwaard"));
    let geen_wapen: Option<String> = None;

    println!("Heeft wapen: {}", wapen.is_some());
    println!("Heeft wapen: {}", geen_wapen.is_some());
}
```

`Some(waarde)` betekent dat er een waarde is. `None` betekent dat er geen
waarde is.

## Een `Option` gebruiken met `match`

```rust
fn toon_wapen(wapen: Option<&str>) {
    match wapen {
        Some(naam) => println!("Wapen: {naam}"),
        None => println!("Geen wapen"),
    }
}

fn main() {
    toon_wapen(Some("Boog"));
    toon_wapen(None);
}
```

Met `if let` schrijf je een korte controle wanneer je vooral één variant nodig
hebt:

```rust
if let Some(naam) = Some("Staf") {
    println!("Wapen: {naam}");
}
```

## Oefeningen

1. [Some maken](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_20/1_some.rs)
2. [None controleren](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_20/2_none.rs)
3. [Een wapen tonen](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_20/3_wapen.rs)
4. [Een getal zoeken](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_20/4_zoeken.rs)