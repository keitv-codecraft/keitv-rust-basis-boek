# Bestanden lezen en schrijven

Een savegame moet buiten het programma kunnen worden bewaard. Rust kan met
`std::fs` bestanden lezen en schrijven.

```rust
use std::fs;

fn main() -> Result<(), std::io::Error> {
    fs::write("savegame.txt", "Arin;100;25")?;
    let inhoud = fs::read_to_string("savegame.txt")?;
    println!("{inhoud}");
    Ok(())
}
```

De `?` geeft een lees- of schrijffout door. In een echte game kies je een
geschikte locatie en behandel je ontbrekende bestanden met een duidelijke
melding.