# Rust 27 — Een groter Rust-project structureren

Tot nu toe hebben we veel afzonderlijke onderdelen van Rust geleerd.

We kunnen inmiddels onder andere:

- variabelen gebruiken
- functies schrijven
- structs maken
- methodes schrijven
- enums en traits gebruiken
- `Vec` en `HashMap` gebruiken
- iterators gebruiken
- `Option` en `Result` gebruiken
- bestanden lezen en schrijven
- automatische tests schrijven.

Dat is genoeg om een veel groter programma te bouwen.

Maar er ontstaat nu een nieuw probleem.

Als we alles in één bestand zetten, krijgen we al snel iets als:

```text
main.rs
  ├── Speler
  ├── Vijand
  ├── Wapen
  ├── Inventaris
  ├── Gevecht
  ├── Winkel
  ├── Savegame
  ├── Tests
  └── heel veel code
```

Dat wordt moeilijk te lezen en moeilijk te onderhouden.

Daarom gaan we onze RPG opdelen in **modules** en bestanden.

---

## 1. Waarom meerdere bestanden?

Stel dat we een RPG hebben met:

```text
Speler
Vijand
Wapen
Inventaris
Gevecht
Savegame
```

Het is logisch om onderdelen die bij elkaar horen bij elkaar te zetten.

Bijvoorbeeld:

```text
src/
├── main.rs
├── speler.rs
├── vijand.rs
├── wapen.rs
├── inventaris.rs
├── gevecht.rs
└── savegame.rs
```

Nu heeft ieder bestand een duidelijk doel.

```text
speler.rs
    → alles wat direct met spelers te maken heeft

vijand.rs
    → vijanden

wapen.rs
    → wapens

inventaris.rs
    → inventarissen

gevecht.rs
    → gevechtslogica

savegame.rs
    → opslaan en laden
```

`main.rs` wordt vervolgens het startpunt van het programma.

---

## 2. We hebben modules al gezien

In het eerdere hoofdstuk over modules hebben we bijvoorbeeld gezien:

```rust,ignore
mod speler;
```

en een bestand:

```text id="wz1v7j"
src/speler.rs
```

Rust behandelt dat bestand als de module `speler`.

We kunnen daarna iets uit die module gebruiken:

```rust,ignore
use speler::Speler;
```

Dit principe gaan we nu gebruiken voor onze RPG.

---

## 3. Een nieuw project

Maak voor dit hoofdstuk een nieuw project:

```text id="8w9fws"
cargo new rpg
```

Ga daarna naar het project:

```text id="u3pj8x"
cd rpg
```

De eerste structuur is:

```text id="x4q5fb"
rpg/
├── Cargo.toml
├── src/
│   └── main.rs
└── ...
```

---

## 4. De eerste module

Maak:

```text id="2w5q44"
src/speler.rs
```

Zet daarin:

```rust,ignore
pub struct Speler {
    pub naam: String,
    pub gezondheid: i32,
}
```

In `main.rs`:

```rust,ignore
mod speler;

use speler::Speler;

fn main() {
    let speler = Speler {
        naam: String::from("Arin"),
        gezondheid: 100,
    };

    println!("{} heeft {} HP.", speler.naam, speler.gezondheid);
}
```

We hebben nu twee bestanden:

```text id="tqk7t0"
src/
├── main.rs
└── speler.rs
```

Maar het programma is nog steeds één programma.

---

## 5. `mod` en `use` hebben verschillende functies

Dit onderscheid is belangrijk.

Met:

```rust,ignore
mod speler;
```

zeggen we:

> Er is een module `speler` die bij dit programma hoort.

Met:

```rust,ignore
use speler::Speler;
```

zeggen we:

> Ik wil `Speler` rechtstreeks kunnen gebruiken.

Zonder `use` kunnen we ook schrijven:

```rust,ignore
let speler = speler::Speler {
    naam: String::from("Arin"),
    gezondheid: 100,
};
```

`use` is dus vooral bedoeld om namen gemakkelijker te gebruiken.

---

## 6. `pub` is belangrijk

In `speler.rs` hadden we:

```rust,ignore
pub struct Speler
```

Waarom staat daar `pub`?

Omdat `Speler` vanuit `main.rs` gebruikt moet kunnen worden.

Zonder `pub`:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
}
```

is de struct privé voor de module.

Dan kan `main.rs` hem niet rechtstreeks gebruiken.

Hetzelfde geldt voor functies.

Bijvoorbeeld:

```rust,ignore
pub fn maak_speler() -> Speler {
    Speler {
        naam: String::from("Arin"),
        gezondheid: 100,
    }
}
```

---

## 7. Niet alles hoeft `pub` te zijn

Dit is een belangrijk onderdeel van een goede projectstructuur.

Stel dat `speler.rs` deze functie heeft:

```rust,ignore
fn controleer_gezondheid(gezondheid: i32) -> i32 {
    if gezondheid < 0 {
        0
    } else {
        gezondheid
    }
}
```

Deze functie hoeft niet door andere modules gebruikt te worden.

Dan hoeft er geen `pub` te staan.

We kunnen dus bijvoorbeeld hebben:

```rust,ignore
pub struct Speler {
    pub naam: String,
    pub gezondheid: i32,
}

fn controleer_gezondheid(gezondheid: i32) -> i32 {
    // ...
}
```

Hiermee beperken we wat andere modules rechtstreeks kunnen gebruiken.

---

## 8. Encapsulatie

We hebben eerder al geleerd dat velden van een struct privé kunnen zijn.

Dat wordt nu veel belangrijker.

We kunnen bijvoorbeeld:

```rust,ignore
pub struct Speler {
    naam: String,
    gezondheid: i32,
}
```

De struct is publiek, maar de velden zijn privé.

Andere modules kunnen dan niet zomaar:

```rust,ignore
speler.gezondheid = -500;
```

doen.

In plaats daarvan bieden we een methode aan:

```rust,ignore
impl Speler {
    pub fn gezondheid(&self) -> i32 {
        self.gezondheid
    }
}
```

en bijvoorbeeld:

```rust,ignore
pub fn neem_schade(&mut self, schade: i32) {
    self.gezondheid -= schade;

    if self.gezondheid < 0 {
        self.gezondheid = 0;
    }
}
```

De module houdt daarmee controle over de interne toestand van de speler.

---

## 9. Een constructor

We kunnen ook een constructorachtige associated function maken:

```rust,ignore
impl Speler {
    pub fn nieuw(naam: String) -> Self {
        Self {
            naam,
            gezondheid: 100,
        }
    }
}
```

Dan kan `main.rs` schrijven:

```rust,ignore
let speler = Speler::nieuw(String::from("Arin"));
```

`Self` betekent hier:

> het type waar deze `impl` bij hoort.

In dit geval is dat `Speler`.

---

## 10. Een nette `speler.rs`

Ons bestand kan nu bijvoorbeeld worden:

```rust,ignore
pub struct Speler {
    naam: String,
    gezondheid: i32,
}

impl Speler {
    pub fn nieuw(naam: String) -> Self {
        Self {
            naam,
            gezondheid: 100,
        }
    }

    pub fn naam(&self) -> &str {
        &self.naam
    }

    pub fn gezondheid(&self) -> i32 {
        self.gezondheid
    }

    pub fn neem_schade(&mut self, schade: i32) {
        self.gezondheid -= schade;

        if self.gezondheid < 0 {
            self.gezondheid = 0;
        }
    }

    pub fn is_levend(&self) -> bool {
        self.gezondheid > 0
    }
}
```

Dit bestand bevat alles wat rechtstreeks met `Speler` te maken heeft.

---

## 11. `main.rs` wordt kleiner

Onze `main.rs` kan nu heel eenvoudig zijn:

```rust,ignore
mod speler;

use speler::Speler;

fn main() {
    let mut speler = Speler::nieuw(String::from("Arin"));

    println!(
        "{} heeft {} HP.",
        speler.naam(),
        speler.gezondheid()
    );

    speler.neem_schade(30);

    println!(
        "{} heeft nog {} HP.",
        speler.naam(),
        speler.gezondheid()
    );
}
```

De implementatie van de speler zit ergens anders.

`main.rs` beschrijft vooral wat het programma doet.

---

## 12. Een tweede module: `vijand`

Maak:

```text id="9j7y1c"
src/vijand.rs
```

Bijvoorbeeld:

```rust,ignore
pub struct Vijand {
    naam: String,
    gezondheid: i32,
    schade: i32,
}

impl Vijand {
    pub fn nieuw(naam: String, gezondheid: i32, schade: i32) -> Self {
        Self {
            naam,
            gezondheid,
            schade,
        }
    }

    pub fn naam(&self) -> &str {
        &self.naam
    }

    pub fn gezondheid(&self) -> i32 {
        self.gezondheid
    }

    pub fn schade(&self) -> i32 {
        self.schade
    }

    pub fn neem_schade(&mut self, schade: i32) {
        self.gezondheid -= schade;

        if self.gezondheid < 0 {
            self.gezondheid = 0;
        }
    }

    pub fn is_levend(&self) -> bool {
        self.gezondheid > 0
    }
}
```

Voeg daarna in `main.rs` toe:

```rust,ignore
mod speler;
mod vijand;
```

We kunnen beide types gebruiken:

```rust,ignore
use speler::Speler;
use vijand::Vijand;
```

---

## 13. Meerdere modules gebruiken

Onze structuur is nu:

```text id="3w5bq6"
src/
├── main.rs
├── speler.rs
└── vijand.rs
```

`main.rs`:

```rust,ignore
mod speler;
mod vijand;

use speler::Speler;
use vijand::Vijand;

fn main() {
    let speler = Speler::nieuw(String::from("Arin"));

    let vijand = Vijand::nieuw(
        String::from("Goblin"),
        30,
        8,
    );

    println!(
        "{} vecht tegen {}.",
        speler.naam(),
        vijand.naam()
    );
}
```

We hebben nu al een klein programma met een duidelijke structuur.

---

## 14. Gevechtslogica hoort niet in `main.rs`

Stel dat we dit in `main.rs` schrijven:

```rust,ignore
vijand.neem_schade(10);
```

Dat is prima.

Maar wat als een gevecht ingewikkelder wordt?

Bijvoorbeeld:

```text
speler valt aan
    ↓
bereken schade
    ↓
vijand neemt schade
    ↓
controleer of vijand dood is
    ↓
vijand valt terug aan
    ↓
controleer of speler dood is
```

Dan wordt het handig om deze logica in:

```text id="9f9ahh"
gevecht.rs
```

te zetten.

---

## 15. De module `gevecht`

Maak:

```text id="d7azn7"
src/gevecht.rs
```

Bijvoorbeeld:

```rust,ignore
use crate::speler::Speler;
use crate::vijand::Vijand;

pub fn speler_valt_aan(speler: &Speler, vijand: &mut Vijand) {
    let schade = 10;

    vijand.neem_schade(schade);

    println!(
        "{} doet {} schade aan {}.",
        speler.naam(),
        schade,
        vijand.naam()
    );
}
```

Hier zien we iets nieuws:

```rust,ignore
crate::
```

---

## 16. Wat betekent `crate::`?

`crate` betekent:

> de huidige Rust-crate, oftewel ons programma.

Als we schrijven:

```rust,ignore
crate::speler::Speler
```

bedoelen we:

```text
ons programma
    ↓
module speler
    ↓
Speler
```

Daarom kunnen we in `gevecht.rs` schrijven:

```rust,ignore
use crate::speler::Speler;
```

en:

```rust,ignore
use crate::vijand::Vijand;
```

---

## 17. `gevecht` toevoegen

In `main.rs`:

```rust,ignore
mod speler;
mod vijand;
mod gevecht;
```

Daarna:

```rust,ignore
use gevecht::speler_valt_aan;
```

We kunnen dan:

```rust,ignore
let mut vijand = Vijand::nieuw(
    String::from("Goblin"),
    30,
    8,
);

speler_valt_aan(&speler, &mut vijand);
```

De gevechtslogica staat nu niet meer in `main.rs`.

---

## 18. Een belangrijk verschil: data versus gedrag

Een handige manier om de projectstructuur te bedenken:

### `speler.rs`

Bevat:

```text
Speler
    eigenschappen
    methodes
```

### `vijand.rs`

Bevat:

```text
Vijand
    eigenschappen
    methodes
```

### `gevecht.rs`

Bevat:

```text
logica die Speler en Vijand samen gebruikt
```

Dit onderscheid helpt bij het bepalen waar nieuwe code thuishoort.

---

## 19. Een inventarismodule

Onze speler heeft inmiddels een inventaris nodig.

Maak:

```text id="v1f3o8"
src/inventaris.rs
```

We kunnen bijvoorbeeld een struct maken:

```rust,ignore
use std::collections::HashMap;

pub struct Inventaris {
    items: HashMap<String, i32>,
}
```

De `HashMap` zelf hoeft niet publiek te zijn.

Dat is bewust.

Andere modules hoeven niet rechtstreeks:

```rust,ignore
inventaris.items
```

te kunnen wijzigen.

---

## 20. Methoden voor de inventaris

We kunnen bijvoorbeeld schrijven:

```rust,ignore
impl Inventaris {
    pub fn nieuw() -> Self {
        Self {
            items: HashMap::new(),
        }
    }

    pub fn voeg_item_toe(&mut self, item: &str) {
        let aantal = self
            .items
            .entry(String::from(item))
            .or_insert(0);

        *aantal += 1;
    }

    pub fn aantal(&self, item: &str) -> i32 {
        self.items.get(item).copied().unwrap_or(0)
    }
}
```

Hier zien we een nieuw klein stukje Rust:

```rust,ignore
copied()
```

---

## 21. `copied()`

`get()` gaf eerder:

```text id="y7y5v9"
Option<&i32>
```

Maar onze functie:

```rust,ignore
aantal(&self, item: &str) -> i32
```

moet een gewone `i32` teruggeven.

Omdat `i32` `Copy` is, kunnen we:

```rust,ignore
.copied()
```

gebruiken.

Bijvoorbeeld:

```rust,ignore
self.items.get(item).copied()
```

verandert:

```text
Option<&i32>
```

in:

```text
Option<i32>
```

Daarna kunnen we:

```rust,ignore
.unwrap_or(0)
```

gebruiken.

Het volledige stuk:

```rust,ignore
self.items
    .get(item)
    .copied()
    .unwrap_or(0)
```

betekent:

> Zoek het item op. Als het bestaat, geef het aantal als gewone `i32`. Als het niet bestaat, geef `0`.

---

## 22. Inventaris testen

Omdat `Inventaris` een zelfstandige module is, kunnen we hem ook zelfstandig testen.

Onderaan `inventaris.rs`:

```rust,ignore
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn nieuw_is_leeg() {
        let inventaris = Inventaris::nieuw();

        assert_eq!(inventaris.aantal("Potion"), 0);
    }

    #[test]
    fn item_toevoegen() {
        let mut inventaris = Inventaris::nieuw();

        inventaris.voeg_item_toe("Potion");

        assert_eq!(inventaris.aantal("Potion"), 1);
    }

    #[test]
    fn item_meerdere_keer_toevoegen() {
        let mut inventaris = Inventaris::nieuw();

        inventaris.voeg_item_toe("Potion");
        inventaris.voeg_item_toe("Potion");

        assert_eq!(inventaris.aantal("Potion"), 2);
    }
}
```

Dit is precies waarom modules nuttig zijn.

We kunnen onderdelen afzonderlijk testen.

---

## 23. `use super::*`

In de testmodule staat:

```rust,ignore
use super::*;
```

Hiermee halen we namen uit de bovenliggende module beschikbaar in de testmodule.

Daardoor kunnen we:

```rust,ignore
Inventaris::nieuw()
```

gebruiken.

Je hoeft `super::*` voorlopig niet volledig te doorgronden.

Onthoud vooral:

> Dit is een veelgebruikte manier om de code van een module vanuit de bijbehorende testmodule te testen.

---

## 24. De speler krijgt een inventaris

We kunnen nu onze `Speler` uitbreiden.

In `speler.rs`:

```rust,ignore
use crate::inventaris::Inventaris;

pub struct Speler {
    naam: String,
    gezondheid: i32,
    inventaris: Inventaris,
}
```

De constructor:

```rust,ignore
pub fn nieuw(naam: String) -> Self {
    Self {
        naam,
        gezondheid: 100,
        inventaris: Inventaris::nieuw(),
    }
}
```

En een methode:

```rust,ignore
pub fn voeg_item_toe(&mut self, item: &str) {
    self.inventaris.voeg_item_toe(item);
}
```

---

## 25. Modules kunnen andere modules gebruiken

We hebben nu:

```text id="3q8m0u"
main.rs
   │
   ├── speler
   │      │
   │      └── inventaris
   │
   ├── vijand
   │
   └── gevecht
```

Dat is prima.

Een module kan functies en types uit andere modules gebruiken zolang die toegankelijk zijn.

Bijvoorbeeld:

```rust,ignore
use crate::inventaris::Inventaris;
```

---

## 26. Een mogelijke volledige structuur

Onze RPG begint er nu zo uit te zien:

```text id="0p3j1m"
rpg/
├── Cargo.toml
└── src/
    ├── main.rs
    ├── speler.rs
    ├── vijand.rs
    ├── inventaris.rs
    ├── wapen.rs
    ├── gevecht.rs
    └── savegame.rs
```

We hoeven niet alle bestanden meteen te maken.

Het is beter om een onderdeel pas toe te voegen wanneer we het nodig hebben.

---

## 27. Wat hoort waar?

Een handige eerste vuistregel:

| Bestand | Verantwoordelijkheid |
| --- | --- |
| `main.rs` | programma starten en hoofdverloop |
| `speler.rs` | `Speler` |
| `vijand.rs` | `Vijand` |
| `wapen.rs` | wapens |
| `inventaris.rs` | inventaris |
| `gevecht.rs` | gevechtslogica |
| `savegame.rs` | opslaan en laden |

Deze verdeling is geen harde regel.

Later kan een groter project bijvoorbeeld nog meer modules krijgen.

---

## 28. `main.rs` moet niet alles doen

Een veelgemaakte fout is om `main.rs` alsnog vol te stoppen.

Bijvoorbeeld:

```text id="8s2kq4"
main.rs
├── speler
├── vijand
├── inventaris
├── winkel
├── gevecht
├── savegame
├── menu
├── input
├── honderden regels code
└── tests
```

Dan hebben we technisch wel meerdere bestanden, maar nog steeds geen goede structuur.

`main.rs` moet vooral de onderdelen van het programma **aan elkaar koppelen**.

---

## 29. Een eenvoudige `main`

Uiteindelijk willen we richting iets als:

```rust,ignore
mod gevecht;
mod inventaris;
mod speler;
mod vijand;

use speler::Speler;
use vijand::Vijand;

fn main() {
    let mut speler = Speler::nieuw(String::from("Arin"));

    let mut vijand = Vijand::nieuw(
        String::from("Goblin"),
        30,
        8,
    );

    println!("Een {} verschijnt!", vijand.naam());

    gevecht::speler_valt_aan(&speler, &mut vijand);

    println!(
        "{} heeft nog {} HP.",
        vijand.naam(),
        vijand.gezondheid()
    );
}
```

De details zitten elders.

---

## 30. Eén verantwoordelijkheid per onderdeel

We kunnen een eenvoudige ontwerpregel gebruiken:

> Een module moet een duidelijk onderwerp of een duidelijke verantwoordelijkheid hebben.

Bijvoorbeeld:

```text
speler.rs
    → Wat is een speler?

inventaris.rs
    → Hoe werkt een inventaris?

gevecht.rs
    → Hoe werkt een gevecht?

savegame.rs
    → Hoe slaan we het spel op?
```

Dit betekent niet dat iedere module precies één struct moet bevatten.

Het gaat erom dat de inhoud logisch bij elkaar hoort.

---

## 31. Een veelgemaakte fout: alles `pub`

Een beginnende programmeur kan geneigd zijn overal `pub` voor te zetten:

```rust,ignore
pub struct Speler {
    pub naam: String,
    pub gezondheid: i32,
    pub inventaris: Inventaris,
}
```

Dat werkt.

Maar het betekent ook dat iedere module rechtstreeks alles kan veranderen.

Bijvoorbeeld:

```rust,ignore
speler.gezondheid = -1000;
```

Dat willen we waarschijnlijk niet.

Beter:

```rust,ignore
pub struct Speler {
    naam: String,
    gezondheid: i32,
    inventaris: Inventaris,
}
```

en gecontroleerde methodes aanbieden.

---

## 32. Encapsulatie in de RPG

Stel:

```rust,ignore
pub fn neem_schade(&mut self, schade: i32) {
    self.gezondheid -= schade;

    if self.gezondheid < 0 {
        self.gezondheid = 0;
    }
}
```

Andere modules hoeven niet te weten **hoe** schade wordt verwerkt.

Ze hoeven alleen te weten:

```rust,ignore
speler.neem_schade(20);
```

Dit maakt het later veel gemakkelijker om de interne implementatie te veranderen.

---

## 33. Modules en ownership

Modules veranderen niets aan de ownership-regels van Rust.

Een `Speler` heeft nog steeds één eigenaar.

Een mutable reference blijft een mutable reference.

Bijvoorbeeld:

```rust,ignore
gevecht::speler_valt_aan(&speler, &mut vijand);
```

Hier lenen we:

```text id="9g6g1q"
speler  → immutable
vijand  → mutable
```

De modules veranderen alleen waar de code staat.

---

## 34. Modules en tests

We kunnen tests op verschillende plaatsen zetten.

Bijvoorbeeld:

```text id="1kq0j4"
inventaris.rs
    └── tests
```

Maar we kunnen ook later integratietests gebruiken.

Voorlopig houden we het eenvoudig:

> Code die direct bij een module hoort, mag zijn unit tests in die module hebben.

Zo blijft bijvoorbeeld de testcode voor `Inventaris` bij `Inventaris`.

---

## 35. Een eerste projectarchitectuur

Voor ons RPG-project stellen we voorlopig deze structuur voor:

```text id="8w7c3y"
src/
├── main.rs
│
├── speler.rs
├── vijand.rs
├── wapen.rs
│
├── inventaris.rs
│
├── gevecht.rs
│
└── savegame.rs
```

De afhankelijkheden kunnen ongeveer zo lopen:

```text id="s2t3z9"
             main
              │
      ┌───────┼────────┐
      ↓       ↓        ↓
   speler   vijand   gevecht
      │       │        │
      ↓       │        │
 inventaris   │        │
      │       │        │
      └───────┴────────┘
              │
           savegame
```

Dit is geen exacte technische afhankelijkheidsgrafiek; het is vooral een manier om na te denken over de verantwoordelijkheden.

---

## 36. Rustlings

Maak voor dit hoofdstuk:

```text id="z9tq1c"
exercises/
└── project_structure/
```

De oefeningen worden nu iets groter. Het doel is niet alleen syntax oefenen, maar leren bepalen **waar code hoort**.

## 01 — Eerste module

`01_module.rs`

Maak:

```text id="q4b5c8"
main.rs
groet.rs
```

Laat `groet.rs` een publieke functie bevatten:

```rust,ignore
pub fn zeg_hallo()
```

Roep deze vanuit `main.rs` aan.

---

## 02 — `pub`

`02_pub.rs`

Maak een publieke struct in een aparte module.

Onderzoek wat er gebeurt als `pub` wordt verwijderd.

---

## 03 — `use`

`03_use.rs`

Gebruik een type uit een module met:

```rust,ignore
use
```

en probeer daarna hetzelfde zonder `use`.

---

## 04 — Privé functie

`04_private.rs`

Maak een publieke functie die intern een private functie gebruikt.

Bepaal welke functie vanuit `main.rs` beschikbaar moet zijn.

---

## 05 — Privé velden

`05_private_fields.rs`

Maak:

```rust,ignore
pub struct Speler {
    naam: String,
    gezondheid: i32,
}
```

Maak methodes waarmee de naam en gezondheid kunnen worden gelezen.

---

## 06 — Constructor

`06_constructor.rs`

Maak:

```rust,ignore
Speler::nieuw(...)
```

met `Self`.

---

## 07 — Tweede module

`07_enemy_module.rs`

Maak een `Vijand` in een eigen bestand.

---

## 08 — Twee modules

`08_two_modules.rs`

Gebruik zowel:

```text id="l7x3ck"
Speler
Vijand
```

uit aparte modules.

---

## 09 — `crate::`

`09_crate_path.rs`

Maak een functie in `gevecht.rs` die `Speler` en `Vijand` gebruikt.

Gebruik:

```rust,ignore
crate::
```

om de types te importeren.

---

## 10 — Gevechtsfunctie

`10_combat.rs`

Schrijf:

```rust,ignore
fn speler_valt_aan(
    speler: &Speler,
    vijand: &mut Vijand,
)
```

---

## 11 — Inventaris

`11_inventory_module.rs`

Maak een `Inventaris` in een eigen module met:

```text id="j3k5n8"
nieuw
voeg_item_toe
aantal
```

---

## 12 — Inventaris testen

`12_inventory_tests.rs`

Voeg minimaal drie tests toe.

---

## 13 — Speler gebruikt inventaris

`13_player_inventory.rs`

Laat `Speler` een `Inventaris` bevatten.

---

## 14 — Encapsulatie

`14_encapsulation.rs`

Maak de velden van `Speler` privé.

Gebruik alleen methodes om ze te lezen of wijzigen.

---

## 15 — Compilerfout: ontbrekende `pub`

`15_debug_pub.rs`

Laat bewust een publieke struct zonder `pub` staan.

Lees de compilerfout en los hem op.

---

## 16 — Compilerfout: ontbrekende `mod`

`16_debug_mod.rs`

Maak wel een bestand `vijand.rs`, maar vergeet:

```rust,ignore
mod vijand;
```

Onderzoek de foutmelding.

---

## 17 — Compilerfout: verkeerd pad

`17_debug_path.rs`

Gebruik bewust:

```rust,ignore
use speler::Vijand;
```

terwijl `Vijand` in een andere module staat.

Los de fout op.

---

## 18 — Compilerfout: privé veld

`18_debug_private.rs`

Probeer vanuit `main.rs` een privé veld rechtstreeks te wijzigen.

Gebruik daarna een methode om hetzelfde resultaat veilig te bereiken.

---

## 19 — RPG-structuur

`19_rpg_modules.rs`

Maak minimaal:

```text id="2u0b4p"
main.rs
speler.rs
vijand.rs
inventaris.rs
gevecht.rs
```

Zorg dat alles compileert.

---

## 20 — Eindopdracht

`20_final_structure.rs`

Maak een kleine RPG met de volgende structuur:

```text id="8t4c5v"
src/
├── main.rs
├── speler.rs
├── vijand.rs
├── inventaris.rs
└── gevecht.rs
```

De speler moet:

- een naam hebben
- gezondheid hebben
- een inventaris hebben
- items kunnen krijgen.

De vijand moet:

- een naam hebben
- gezondheid hebben
- schade kunnen veroorzaken.

De gevechtsmodule moet:

- de speler laten aanvallen
- schade aan de vijand toebrengen
- controleren of de vijand nog leeft.

Gebruik encapsulatie en minimaal vijf tests.

---

## Zelfstandig oefenen

## Opdracht 1 — Module opnieuw ontwerpen

Neem een eerdere RPG-oefening waarin alles in `main.rs` staat.

Verdeel de code over:

```text id="l4u0dk"
speler.rs
vijand.rs
gevecht.rs
```

De werking van het programma mag niet veranderen.

---

## Opdracht 2 — Inventaris toevoegen

Voeg:

```text id="e1m7f4"
inventaris.rs
```

toe aan je RPG.

Zorg dat `Speler` een inventaris bezit.

---

## Opdracht 3 — Encapsulatie verbeteren

Zoek alle publieke velden in je RPG.

Vraag jezelf bij ieder veld af:

> Moet iedere andere module dit rechtstreeks kunnen wijzigen?

Als het antwoord nee is, maak het veld privé en bied een methode aan.

---

## Opdracht 4 — Tests verplaatsen

Verplaats tests naar de module waarop ze betrekking hebben.

Bijvoorbeeld:

```text id="u6b0p7"
Inventaris-tests → inventaris.rs
Speler-tests    → speler.rs
Vijand-tests    → vijand.rs
```

---

## Belangrijk ontwerpprincipe

We hebben nu een belangrijke stap gemaakt.

Tot nu toe was de vraag vooral:

> Hoe schrijf ik deze Rust-code?

Vanaf nu wordt een tweede vraag steeds belangrijker:

> **Waar hoort deze code thuis?**

Een goed Rust-programma bestaat niet alleen uit correcte code.

De onderdelen moeten ook logisch georganiseerd zijn.

Voor onze RPG betekent dat bijvoorbeeld:

```text
Speler
  → speler.rs

Vijand
  → vijand.rs

Inventaris
  → inventaris.rs

Gevecht
  → gevecht.rs

Opslaan/laden
  → savegame.rs

Programma besturen
  → main.rs
```

Dat maakt het mogelijk om later nieuwe onderdelen toe te voegen zonder dat één bestand steeds groter wordt.

---

## Vooruitblik naar het eindproject

We hebben nu de belangrijkste technische bouwstenen gehad.

Het volgende hoofdstuk wordt daarom geen nieuw losstaand Rust-concept.

We gaan beginnen met het **ontwerpen van onze uiteindelijke RPG**.

Daarbij bepalen we eerst:

- welke structs we nodig hebben
- welke enums we nodig hebben
- welke traits nuttig zijn
- welke modules we gebruiken
- hoe een gevecht verloopt
- hoe de inventaris werkt
- hoe savegames werken
- hoe de verschillende onderdelen met elkaar communiceren.

Daarna bouwen we het project stap voor stap.

Het doel is niet om één enorm programma in één keer te schrijven.

We bouwen het als een echte softwareontwikkeling:

```text
klein onderdeel
    ↓
compileren
    ↓
testen
    ↓
uitbreiden
    ↓
opnieuw testen
    ↓
volgend onderdeel
```

Compilerfouten en falende tests blijven daarbij gewoon onderdeel van het ontwikkelproces. Ze geven ons informatie over wat er nog moet worden aangepast.

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 27](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_27/).

