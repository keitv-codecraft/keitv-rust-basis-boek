# Ownership: wie is de eigenaar?

Rust gebruikt **ownership** om veilig met geheugen om te gaan. Het belangrijkste
idee is eenvoudig:

> Iedere waarde heeft één eigenaar.

Wanneer de eigenaar niet meer nodig is, kan Rust de waarde opruimen.

## 1. Een waarde heeft een eigenaar

```rust
fn main() {
    let naam = String::from("Arin");
    println!("{naam}");
}
```

`naam` is eigenaar van de `String`. De tekst staat in het geheugen en hoort bij
de variabele `naam`.

## 2. Een move

Bij sommige types maakt Rust automatisch een eenvoudige kopie. Bij een
`String` gebeurt dat niet zomaar. De eigenaar wordt dan verplaatst:

```rust
fn main() {
    let eerste_naam = String::from("Arin");
    let tweede_naam = eerste_naam;

    println!("{tweede_naam}");
}
```

Na `let tweede_naam = eerste_naam;` is `tweede_naam` de nieuwe eigenaar. De
oude naam mogen we niet meer gebruiken. Dit noemen we een **move**.

> [!WARNING]
> Een move is geen fout. Rust voorkomt hiermee dat twee variabelen tegelijk
> denken dat zij dezelfde `String` mogen opruimen.

## 3. Een bewuste kopie maken

Soms willen we echt twee aparte strings. Gebruik dan `clone()`:

```rust
fn main() {
    let eerste_naam = String::from("Arin");
    let tweede_naam = eerste_naam.clone();

    println!("{eerste_naam} en {tweede_naam}");
}
```

`clone()` maakt een nieuwe waarde. Dat kost extra geheugen, dus gebruik het
wanneer je echt een kopie nodig hebt.

## 4. Scope

Een waarde bestaat binnen een bepaald bereik, een **scope**:

```rust
fn main() {
    {
        let schat = String::from("Goud");
        println!("{schat}");
    }
}
```

Wanneer de scope eindigt, kan Rust de waarde opruimen.

## 5. Ownership en functies

Een functie kan eigenaar worden van een waarde:

```rust
fn toon_naam(naam: String) {
    println!("Welkom, {naam}!");
}

fn main() {
    let naam = String::from("Arin");
    toon_naam(naam);
}
```

Na de aanroep is de functie de eigenaar geweest. Wil `main` de naam daarna
ook nog gebruiken, maak dan een kopie of wacht op het volgende artikel over
borrowing.

## Oefeningen

Maak de oefeningen in de Rustlings-repository:

1. [Een String maken](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_8/1_string.rs)
2. [Een move begrijpen](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_8/2_move.rs)
3. [Een kopie maken](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_8/3_clone.rs)
4. [Een functie krijgt ownership](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_8/4_functie.rs)
5. [Een naam doorgeven](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_8/5_naam.rs)

De laatste oefening vormt de overgang naar borrowing.