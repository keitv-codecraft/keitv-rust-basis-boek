# Arrays en Vecs: groepen waarden

In een game hebben we vaak meerdere vijanden, scores of items. Een array of
`Vec` bewaart meerdere waarden van hetzelfde type.

## Arrays

Een array heeft een vaste lengte:

```rust
fn main() {
    let scores = [100, 250, 500];
    println!("Eerste score: {}", scores[0]);
}
```

De eerste index is `0`. Een array met drie waarden heeft dus indexen `0`, `1`
en `2`.

## Een `Vec`

Een `Vec` kan tijdens het programma groter of kleiner worden:

```rust
fn main() {
    let mut vijanden = Vec::new();
    vijanden.push("Goblin");
    vijanden.push("Ork");

    println!("Aantal vijanden: {}", vijanden.len());
}
```

Je kunt ook meteen waarden toevoegen met `vec!`:

```rust
let vijanden = vec!["Goblin", "Ork", "Trol"];
```

## Door een `Vec` lopen

Met `for` lees je ieder element:

```rust
fn main() {
    let vijanden = vec!["Goblin", "Ork", "Trol"];

    for vijand in &vijanden {
        println!("{vijand}");
    }
}
```

Gebruik `&mut` wanneer je elementen wilt veranderen:

```rust
struct Vijand {
    gezondheid: i32,
}

fn main() {
    let mut vijanden = vec![Vijand { gezondheid: 30 }, Vijand { gezondheid: 50 }];

    for vijand in &mut vijanden {
        vijand.gezondheid -= 10;
    }
}
```

## Oefeningen

1. [Een array lezen](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_15/1_array.rs)
2. [Een Vec maken](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_15/2_vec.rs)
3. [Waarden toevoegen](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_15/3_push.rs)
4. [Een Vec doorlopen](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_15/4_for.rs)
5. [Groepsschade](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_15/5_aoe.rs)