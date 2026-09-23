# Rust 19 — Modules en zichtbaarheid

Tot nu toe hebben we onze programma's meestal in één bestand geschreven.

Dat is handig wanneer een programma klein is:

```text
src/
└── main.rs
```

Maar ons RPG-project begint inmiddels behoorlijk groot te worden.

We hebben bijvoorbeeld:

- spelers
- vijanden
- wapens
- gevechten
- functies
- traits
- tests
- verschillende soorten game-logica.

Als alles in één bestand staat, wordt het steeds moeilijker om te vinden waar iets staat.

Rust heeft daarom **modules**.

Met modules kunnen we code logisch opdelen.

---

## 1. Waarom modules?

Stel dat we dit allemaal in `main.rs` zetten:

```rust,ignore
struct Speler {
    // ...
}

struct Vijand {
    // ...
}

struct Wapen {
    // ...
}

fn bereken_schade() {
    // ...
}

fn genees_speler() {
    // ...
}

fn laad_speler() {
    // ...
}
```

Dat kan technisch prima.

Maar naarmate het programma groeit, wordt het bestand steeds moeilijker te begrijpen.

We kunnen de onderdelen bijvoorbeeld verdelen:

```text
src/
├── main.rs
├── speler.rs
├── vijand.rs
├── wapen.rs
└── gevecht.rs
```

Dan staat de code voor spelers in `speler.rs`, de code voor vijanden in `vijand.rs`, enzovoort.

---

## 2. Een eerste module

We kunnen in `main.rs` een module definiëren:

```rust,ignore
mod speler;
```

Dit vertelt Rust dat er een module met de naam `speler` bestaat.

Daarbij hoort een bestand:

```text
src/
├── main.rs
└── speler.rs
```

In `speler.rs` kunnen we bijvoorbeeld schrijven:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
}
```

De module bestaat nu, maar `Speler` is nog niet automatisch overal beschikbaar.

Daarvoor moeten we iets begrijpen dat **zichtbaarheid** heet.

---

## 3. `pub`

Standaard is iets binnen een module niet zomaar van buiten de module bereikbaar.

We kunnen iets openbaar maken met `pub`.

In `speler.rs`:

```rust,ignore
pub struct Speler {
    pub naam: String,
    pub gezondheid: i32,
}
```

Nu mogen andere modules `Speler` gebruiken.

Ook de velden hebben afzonderlijk `pub` nodig als we ze rechtstreeks van buitenaf willen gebruiken.

Dit zijn dus twee verschillende dingen:

```rust,ignore
pub struct Speler {
    pub naam: String,
}
```

De eerste `pub` maakt de struct openbaar.

De tweede `pub` maakt het veld openbaar.

---

## 4. Een functie openbaar maken

Hetzelfde geldt voor functies.

In `speler.rs`:

```rust,ignore
pub fn begroet() {
    println!("Welkom!");
}
```

Zonder `pub`:

```rust,ignore
fn begroet() {
    println!("Welkom!");
}
```

kan de functie alleen binnen de module worden gebruikt.

Met:

```rust,ignore
pub fn begroet() {
    println!("Welkom!");
}
```

kunnen we haar ook vanuit een andere module gebruiken.

---

## 5. Een module gebruiken

Stel dat `speler.rs` bevat:

```rust,ignore
pub struct Speler {
    pub naam: String,
    pub gezondheid: i32,
}
```

Dan kunnen we in `main.rs` schrijven:

```rust,ignore
mod speler;

fn main() {
    let speler = speler::Speler {
        naam: String::from("Arin"),
        gezondheid: 100,
    };

    println!("{}", speler.naam);
}
```

Het `::` betekent hier dat we iets binnen de module `speler` willen gebruiken.

```text
speler::Speler
```

betekent:

> Gebruik `Speler` uit de module `speler`.

---

## 6. `use`

Wanneer we een naam vaak gebruiken, kunnen we die naar de huidige scope halen met `use`.

Bijvoorbeeld:

```rust,ignore
mod speler;

use speler::Speler;

fn main() {
    let speler = Speler {
        naam: String::from("Arin"),
        gezondheid: 100,
    };
}
```

Zonder `use`:

```rust,ignore
let speler = speler::Speler {
    // ...
};
```

Met `use`:

```rust,ignore
let speler = Speler {
    // ...
};
```

`use` maakt niets openbaar.

Het zorgt alleen dat we een naam gemakkelijker kunnen gebruiken.

---

## 7. Een module kan ook functies bevatten

Een module hoeft niet alleen structs te bevatten.

Bijvoorbeeld:

```rust,ignore
pub fn nieuwe_speler(naam: &str) -> Speler {
    Speler {
        naam: String::from(naam),
        gezondheid: 100,
    }
}
```

We kunnen dan in `main.rs` schrijven:

```rust,ignore
mod speler;

use speler::nieuwe_speler;

fn main() {
    let speler = nieuwe_speler("Arin");

    println!("{}", speler.naam);
}
```

Dit begint al meer op een echt project te lijken.

---

## 8. Een RPG opdelen

Ons project kan bijvoorbeeld deze structuur krijgen:

```text
src/
├── main.rs
├── speler.rs
├── vijand.rs
├── wapen.rs
└── gevecht.rs
```

### `speler.rs`

```rust,ignore
pub struct Speler {
    pub naam: String,
    pub gezondheid: i32,
}
```

### `vijand.rs`

```rust,ignore
pub struct Vijand {
    pub naam: String,
    pub gezondheid: i32,
}
```

### `wapen.rs`

```rust,ignore
pub struct Wapen {
    pub naam: String,
    pub schade: i32,
}
```

### `gevecht.rs`

```rust,ignore
pub fn bereken_schade(kracht: i32, wapenschade: i32) -> i32 {
    kracht + wapenschade
}
```

### `main.rs`

```rust,ignore
mod speler;
mod vijand;
mod wapen;
mod gevecht;

use speler::Speler;
use vijand::Vijand;
use wapen::Wapen;

fn main() {
    let speler = Speler {
        naam: String::from("Arin"),
        gezondheid: 100,
    };

    let goblin = Vijand {
        naam: String::from("Goblin"),
        gezondheid: 50,
    };

    let zwaard = Wapen {
        naam: String::from("Zwaard"),
        schade: 20,
    };

    println!("Speler: {}", speler.naam);
    println!("Vijand: {}", goblin.naam);
    println!("Wapen: {}", zwaard.naam);
}
```

Dit is veel overzichtelijker dan één groot bestand.

---

## 9. Zichtbaarheid van velden

Een belangrijk detail:

```rust,ignore
pub struct Speler {
    pub naam: String,
    gezondheid: i32,
}
```

Hier is `naam` openbaar, maar `gezondheid` niet.

Dit werkt:

```rust,ignore
println!("{}", speler.naam);
```

Maar dit niet vanuit een andere module:

```rust,ignore
println!("{}", speler.gezondheid);
```

De compiler geeft dan een fout.

Dit is expres.

Een module kan bepalen welke onderdelen andere delen van het programma rechtstreeks mogen gebruiken.

---

## 10. Waarom niet alles `pub` maken?

Je zou kunnen denken:

> Dan zet ik gewoon overal `pub` voor.

Dat kan, maar daarmee geef je andere delen van het programma veel vrijheid.

Stel:

```rust,ignore
pub struct Speler {
    pub gezondheid: i32,
}
```

Dan kan iedere code die toegang heeft tot de speler dit doen:

```rust,ignore
speler.gezondheid = -500;
```

Misschien willen we dat helemaal niet.

We kunnen het veld daarom privé houden:

```rust,ignore
pub struct Speler {
    gezondheid: i32,
}
```

En gecontroleerde methoden aanbieden:

```rust,ignore
impl Speler {
    pub fn gezondheid(&self) -> i32 {
        self.gezondheid
    }

    pub fn neem_schade(&mut self, schade: i32) {
        self.gezondheid -= schade;
    }
}
```

Andere code kan dan:

```rust,ignore
speler.neem_schade(20);
```

maar niet zomaar:

```rust,ignore
speler.gezondheid = -500;
```

Modules en zichtbaarheid helpen dus om te bepalen **welke delen van je programma andere code mag gebruiken**.

---

## 11. `main.rs` als startpunt

Een handige manier om naar ons project te kijken:

```text
main.rs
   │
   ├── speler
   ├── vijand
   ├── wapen
   └── gevecht
```

`main.rs` kan de verschillende onderdelen van het spel samenbrengen.

De modules bevatten vervolgens de specifieke logica.

Je hoeft niet alles vanuit `main.rs` te laten gebeuren.

---

## 12. Modules kunnen functies en structs combineren

Een volledige `speler.rs` kan bijvoorbeeld zijn:

```rust,ignore
pub struct Speler {
    pub naam: String,
    gezondheid: i32,
}

impl Speler {
    pub fn new(naam: &str) -> Self {
        Self {
            naam: String::from(naam),
            gezondheid: 100,
        }
    }

    pub fn gezondheid(&self) -> i32 {
        self.gezondheid
    }

    pub fn neem_schade(&mut self, schade: i32) {
        self.gezondheid -= schade;
    }
}
```

Daarna:

```rust,ignore
mod speler;

use speler::Speler;

fn main() {
    let mut speler = Speler::new("Arin");

    speler.neem_schade(20);

    println!("Gezondheid: {}", speler.gezondheid());
}
```

Hier gebruiken we veel van wat we al kennen:

- structs
- `impl`
- methods
- borrowing
- `String`
- `&str`
- `Self`
- modules
- `pub`.

Dat is precies de bedoeling: nieuwe onderwerpen moeten voortbouwen op eerdere kennis.

---

## 13. Een module is geen aparte applicatie

Het is belangrijk dat je modules niet ziet als losse programma's.

Dit:

```text
src/
├── main.rs
├── speler.rs
└── vijand.rs
```

is nog steeds **één Rust-programma**.

De bestanden zijn alleen logisch verdeeld.

Rust compileert de verschillende onderdelen samen tot één programma.

---

## 14. Modules en tests

Modules zijn ook handig voor tests.

Bijvoorbeeld in `speler.rs`:

```rust,ignore
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn nieuwe_speler_heeft_100_gezondheid() {
        let speler = Speler::new("Arin");

        assert_eq!(speler.gezondheid(), 100);
    }
}
```

De test staat dan dicht bij de code die hij test.

Dat sluit direct aan op ons vorige artikel over automatische tests.

---

## 15. Een grotere RPG-structuur

Wanneer ons project verder groeit, kunnen we bijvoorbeeld eindigen met:

```text
src/
├── main.rs
├── speler.rs
├── vijand.rs
├── wapen.rs
├── gevecht.rs
├── item.rs
└── inventory.rs
```

Later kunnen we zelfs modules binnen modules gebruiken.

Maar voor deze cursus hoeven we dat nog niet uitgebreid te behandelen.

Eerst moeten de basisprincipes duidelijk zijn:

```text
mod   → een module opnemen
pub   → iets openbaar maken
use   → een naam gemakkelijker gebruiken
::    → iets binnen een module benoemen
```

---

## 16. Veelgemaakte fouten

### `pub` vergeten

```rust,ignore
struct Speler {
    // ...
}
```

terwijl `main.rs` probeert:

```rust,ignore
use speler::Speler;
```

De compiler vertelt dat `Speler` niet openbaar is.

---

### Een veld vergeten

```rust,ignore
pub struct Speler {
    naam: String,
}
```

De struct is openbaar, maar `naam` niet.

Dit:

```rust,ignore
let speler = Speler {
    naam: String::from("Arin"),
};
```

kan dan vanuit een andere module niet zomaar.

---

### `mod` vergeten

Als `speler.rs` bestaat maar `main.rs` heeft niet:

```rust,ignore
mod speler;
```

dan weet de moduleboom niet dat `speler` onderdeel van het programma is.

---

### `use` vergeten

Als je dit hebt:

```rust,ignore
mod speler;
```

kun je schrijven:

```rust,ignore
let speler = speler::Speler {
    // ...
};
```

maar niet zomaar:

```rust,ignore
let speler = Speler {
    // ...
};
```

Daarvoor heb je:

```rust,ignore
use speler::Speler;
```

nodig.

---

## 17. Rustlings

Maak:

```text
exercises/modules/
```

## 01_module.rs

Maak een module:

```rust,ignore
mod speler {
    // ...
}
```

Maak binnen de module een functie die `"Welkom!"` afdrukt.

Roep de functie vanuit `main` aan.

---

## 02_public_function.rs

Maak een module met een openbare functie:

```rust,ignore
pub fn begroet() {
    // ...
}
```

Roep deze vanuit `main` aan.

---

## 03_private_function.rs

Maak een module met een privéfunctie.

Probeer deze vanuit `main` aan te roepen.

De code moet expres niet compileren.

Lees de compilerfout en leg uit waarom.

Maak daarna een tweede versie waarin het programma wel compileert.

---

## 04_public_struct.rs

Maak:

```rust,ignore
pub struct Speler {
    pub naam: String,
    pub gezondheid: i32,
}
```

Maak vanuit `main` een speler.

---

## 05_private_field.rs

Maak:

```rust,ignore
pub struct Speler {
    pub naam: String,
    gezondheid: i32,
}
```

Probeer vanuit `main` `gezondheid` rechtstreeks te veranderen.

Onderzoek de compilerfout.

---

## 06_use.rs

Gebruik:

```rust,ignore
mod speler;
```

en:

```rust,ignore
use speler::Speler;
```

Maak daarna een speler zonder `speler::Speler` te schrijven.

---

## 07_module_path.rs

Gebruik bewust géén `use`.

Maak een speler met:

```rust,ignore
speler::Speler
```

zodat je ziet hoe een modulepad werkt.

---

## 08_two_modules.rs

Maak twee modules:

```text
speler
vijand
```

Elke module bevat een eigen struct.

Maak in `main` één speler en één vijand.

---

## 09_game_modules.rs

Maak drie modules:

```text
speler
wapen
gevecht
```

Verdeel de bijbehorende structs en functies over deze modules.

---

## 10_constructor.rs

Maak in `speler.rs`:

```rust,ignore
impl Speler {
    pub fn new(naam: &str) -> Self {
        // ...
    }
}
```

Gebruik `Speler::new()` vanuit `main`.

---

## 11_private_health.rs

Maak `gezondheid` privé.

Voeg toe:

```rust,ignore
pub fn gezondheid(&self) -> i32
```

en:

```rust,ignore
pub fn neem_schade(&mut self, schade: i32)
```

Test beide vanuit `main`.

---

## 12_module_test.rs

Voeg tests toe aan `speler.rs`.

Test dat een nieuwe speler 100 gezondheid heeft.

Gebruik:

```rust,ignore
#[cfg(test)]
mod tests {
    use super::*;
}
```

---

## 13_debug_missing_pub.rs

De volgende code is expres fout:

```rust,ignore
mod speler {
    struct Speler {
        pub naam: String,
    }
}

fn main() {
    let speler = speler::Speler {
        naam: String::from("Arin"),
    };

    println!("{}", speler.naam);
}
```

Gebruik de compiler om te ontdekken wat ontbreekt.

---

## 14_debug_field_visibility.rs

Maak de struct openbaar, maar laat één veld privé.

Probeer beide velden vanuit `main` te gebruiken.

Bepaal aan de hand van de compilerfout welk veld niet toegankelijk is.

---

## 15_rpg_modules.rs

Maak:

```text
src/
├── main.rs
├── speler.rs
├── vijand.rs
└── wapen.rs
```

Maak in ieder bestand de bijbehorende struct.

Gebruik ze allemaal vanuit `main`.

---

## 16_rpg_battle.rs

Voeg een module `gevecht` toe.

Maak daarin:

```rust,ignore
pub fn bereken_schade(kracht: i32, wapenschade: i32) -> i32
```

Gebruik deze vanuit `main`.

---

## 17_module_tests.rs

Voeg tests toe aan minimaal twee modules.

Test bijvoorbeeld:

- de standaardgezondheid van een speler
- de schade van een wapen
- de schadeberekening van een aanval.

---

## 18_debug_module.rs

Maak expres een programma waarin:

- een module ontbreekt
- een `pub` ontbreekt
- een `use` ontbreekt.

Laat de leerlingen de problemen één voor één oplossen.

---

## 19_complete_rpg.rs

Bouw een klein RPG-project met:

```text
speler.rs
vijand.rs
wapen.rs
gevecht.rs
main.rs
```

Het programma moet:

1. een speler maken
2. een vijand maken
3. een wapen maken
4. schade berekenen
5. de vijand beschadigen
6. de gezondheid tonen.

Gebruik minimaal één private struct-field en een publieke methode om die waarde te lezen of wijzigen.

---

## 20_final_modules.rs

Maak een kleine, nette RPG-structuur waarin:

- `Speler` in `speler.rs` staat
- `Vijand` in `vijand.rs` staat
- `Wapen` in `wapen.rs` staat
- gevechtslogica in `gevecht.rs` staat
- `main.rs` alleen het spel opstart
- iedere module minimaal één test bevat.

Probeer `main.rs` zo klein mogelijk te houden.

---

## 18. Oefeningen zonder Rustlings

Naast de Rustlings-bestanden zijn de volgende oefeningen bedoeld om zelfstandig te ontwerpen.

### Oefening 1 — Verplaats bestaande code

Neem een eerder RPG-programma dat alles in `main.rs` heeft staan.

Verplaats:

- `Speler` naar `speler.rs`
- `Vijand` naar `vijand.rs`
- `Wapen` naar `wapen.rs`.

Zorg dat het programma nog steeds werkt.

---

### Oefening 2 — Maak gezondheid privé

Maak `gezondheid` privé.

Maak vervolgens methoden waarmee andere code:

- de gezondheid kan bekijken
- schade kan toebrengen
- de speler kan genezen.

---

### Oefening 3 — Scheid de gevechtslogica

Verplaats alle functies die met gevechten te maken hebben naar `gevecht.rs`.

`main.rs` mag daarna alleen nog het gevecht starten.

---

### Oefening 4 — Voeg tests toe

Voeg voor iedere module minimaal één test toe.

Laat minstens één test expres falen en gebruik de testuitvoer om de fout te vinden.

---

### Oefening 5 — Ontwerp je eigen module

Kies een onderdeel van het RPG dat nog niet als module bestaat.

Bijvoorbeeld:

- inventory
- quests
- items
- character creation.

Maak hiervoor zelf een module met minimaal één struct, één publieke functie of methode en één test.

---

## 19. Wat moet je na dit artikel kunnen?

Je moet nu begrijpen:

- waarom we code in modules opdelen
- wat een module is
- hoe `.rs`-bestanden bij modules horen
- wat `pub` doet
- waarom structvelden afzonderlijk openbaar of privé kunnen zijn
- hoe `use` werkt
- hoe `::` in modulepaden werkt
- hoe modules tests kunnen bevatten
- hoe je een groter RPG-project overzichtelijk kunt structureren.

De belangrijkste begrippen zijn:

```text
mod
pub
use
::
```

En vooral:

> Een module helpt je programma overzichtelijk te organiseren en bepaalt welke onderdelen van die code van buitenaf gebruikt mogen worden.

In het volgende artikel gaan we een ander veelvoorkomend probleem oplossen:

> Wat doe je wanneer een waarde er misschien wel is, maar misschien ook niet?

Daarvoor hebben we `Option<T>`.

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 19](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_19/).

