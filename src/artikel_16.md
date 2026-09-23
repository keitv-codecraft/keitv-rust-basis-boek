# Closures: gedrag als waarde

Een closure is een klein stukje gedrag dat we in een variabele kunnen bewaren.
De parameters staan tussen verticale strepen.

```rust
fn main() {
    let verdubbel = |getal| getal * 2;
    println!("{}", verdubbel(5));
}
```

Je kunt types expliciet opschrijven:

```rust
let tel_op = |eerste: i32, tweede: i32| -> i32 { eerste + tweede };
```

## Een closure op een groep waarden gebruiken

Een closure kan samen met een `for`-lus een actie uitvoeren op ieder element:

```rust
struct Vijand {
    gezondheid: i32,
}

fn main() {
    let mut vijanden = vec![Vijand { gezondheid: 30 }, Vijand { gezondheid: 50 }];
    let vuurstorm = |vijand: &mut Vijand| {
        vijand.gezondheid -= 20;
    };

    for vijand in &mut vijanden {
        vuurstorm(vijand);
    }
}
```

De closure is hier een aanval die we op alle vijanden toepassen. Je kunt een
andere closure maken voor een ijsaanval zonder de lus opnieuw te schrijven.

## Een closure uit de omgeving gebruiken

Een closure kan waarden lezen uit de plek waar hij gemaakt is:

```rust
fn main() {
    let bonus = 10;
    let geef_bonus = |score: i32| score + bonus;
    println!("{}", geef_bonus(100));
}
```

## Oefeningen

1. [Een closure maken](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_16/1_closure.rs)
2. [Twee parameters](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_16/2_parameters.rs)
3. [Een bonus gebruiken](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_16/3_bonus.rs)
4. [Een AoE-closure](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_16/4_aanval.rs)
