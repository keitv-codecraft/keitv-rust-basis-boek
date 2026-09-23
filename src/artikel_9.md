# Borrowing: tijdelijk iets gebruiken

In het vorige artikel gaf een functie de ownership van een `String` over. Vaak
hoeft een functie de waarde alleen te lezen. Dan kunnen we de waarde tijdelijk
lenen. Dat heet **borrowing**.

## 1. Een reference met `&`

```rust
fn toon_naam(naam: &String) {
    println!("Welkom, {naam}!");
}

fn main() {
    let naam = String::from("Arin");
    toon_naam(&naam);
    println!("{naam} kan hier nog steeds gebruikt worden.");
}
```

`&naam` betekent: leen `naam` tijdelijk. De functie wordt geen eigenaar.

Meestal schrijven we voor tekst `&str` in plaats van `&String`:

```rust
fn toon_naam(naam: &str) {
    println!("{naam}");
}

fn main() {
    let naam = String::from("Arin");
    toon_naam(&naam);
}
```

Een `&String` kan automatisch als `&str` worden gebruikt.

## 2. Een mutable reference

Met `&mut` leen je een waarde en mag je haar tijdelijk veranderen:

```rust
fn neem_schade(gezondheid: &mut i32, schade: i32) {
    *gezondheid -= schade;
}

fn main() {
    let mut gezondheid = 100;
    neem_schade(&mut gezondheid, 25);
    println!("Gezondheid: {gezondheid}");
}
```

Het sterretje `*` betekent hier: gebruik de waarde waarnaar de reference wijst.

## 3. De belangrijkste regels

- Een gewone reference met `&` mag lezen.
- Een mutable reference met `&mut` mag veranderen.
- Je hebt niet tegelijk een mutable reference en andere references naar
  dezelfde waarde.
- Een reference wordt nooit de eigenaar van de waarde.

Deze regels helpen Rust om onveilige situaties tijdens het compileren te
vinden.

## Oefeningen

1. [Een String lenen](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_9/1_lenen.rs)
2. [Een tekst tonen](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_9/2_tekst.rs)
3. [Gezondheid lenen](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_9/3_schade.rs)
4. [Een String aanpassen](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_9/4_aanpassen.rs)
5. [Twee keer lenen](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_9/5_twee_keer.rs)

De laatste oefening laat zien dat lezen zonder ownership over te dragen veilig
kan worden herhaald.