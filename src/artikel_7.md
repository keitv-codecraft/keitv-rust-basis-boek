# Lussen

Een lus voert code meerdere keren uit. Dat is handig voor een countdown, een
aanval of meerdere beurten.

## `loop`

`loop` blijft doorgaan tot `break` wordt uitgevoerd:

```rust
fn main() {
    let mut beurt = 1;

    loop {
        println!("Beurt {beurt}");
        beurt += 1;
        if beurt > 3 {
            break;
        }
    }
}
```

## `while`

Een `while`-lus gaat door zolang de voorwaarde waar is:

```rust
fn main() {
    let mut levens = 3;
    while levens > 0 {
        println!("Nog {levens} levens");
        levens -= 1;
    }
}
```

## `for`

Met `for` loop je over een bereik. `1..=3` bevat 1, 2 en 3.

```rust
fn main() {
    for beurt in 1..=3 {
        println!("Beurt {beurt}");
    }
}
```

> [!NOTE]
> Een bereik met twee puntjes, zoals `0..3`, stopt vóór 3. Met `..=` neem je
> de laatste waarde wel mee.

## Oefeningen

Maak de oefeningen [1](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_7/1_lus.rs),
[2](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_7/2_while_lus.rs),
[3](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_7/3_for_lus.rs) en
[4](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_7/4_aoe_schade.rs).