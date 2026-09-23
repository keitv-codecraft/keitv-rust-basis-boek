# Interactie en NPC's

Een NPC kan een naam, tekst en actie hebben. Een enum maakt acties duidelijk:

```rust
enum NpcActie {
    Praat,
    GeefItem(String),
    GeenActie,
}
```

Houd gesprekken klein en voorspelbaar. Een nieuwe NPC hoort vooral nieuwe data
te zijn, niet een kopie van de hele spel-lus.