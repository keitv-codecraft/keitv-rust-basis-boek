# Iterators

Een iterator geeft waarden uit een verzameling één voor één. Dat is een
andere manier om door een `Vec` te lopen.

## `iter`

```rust
fn main() {
    let namen = vec!["Arin", "Borin", "Cedric"];

    for naam in namen.iter() {
        println!("{naam}");
    }
}
```

`iter()` leent de waarden. De `Vec` blijft daarna beschikbaar.

## `map`

Met `map` maakt je voor ieder element een nieuwe waarde:

```rust
fn main() {
    let scores = vec![10, 20, 30];
    let hogere_scores: Vec<i32> = scores.iter().map(|score| score + 5).collect();
    println!("{hogere_scores:?}");
}
```

`collect()` verzamelt de resultaten in een nieuwe verzameling.

## `filter`

Met `filter` houd je alleen elementen over die aan een voorwaarde voldoen:

```rust
fn main() {
    let scores = vec![10, 60, 30, 90];
    let hoge_scores: Vec<&i32> = scores.iter().filter(|score| **score >= 50).collect();
    println!("{hoge_scores:?}");
}
```

Begin bij iedere iterator-keten met één stap. Voeg pas daarna een volgende
stap toe.

## Oefeningen

1. [Een iterator gebruiken](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_22/1_iter.rs)
2. [Scores aanpassen](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_22/2_map.rs)
3. [Scores filteren](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_22/3_filter.rs)
4. [Een nieuwe Vec verzamelen](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_22/4_collect.rs)