# Rust 29 — De eerste speelbare RPG bouwen

In het vorige hoofdstuk hebben we ons RPG ontworpen.

We hebben bepaald dat ons spel onder andere bestaat uit:

- een `Spel`
- een `Speler`
- `Vijanden`
- een `SpelStatus`
- later ook wapens en een inventaris.

Nu gaan we dat ontwerp daadwerkelijk programmeren.

We beginnen bewust klein.

Ons doel is aan het einde van dit hoofdstuk een programma te hebben waarin:

1. een spel wordt gestart
2. een speler wordt gemaakt
3. een vijand wordt gemaakt
4. de speler en vijand met elkaar kunnen vechten
5. schade wordt uitgedeeld
6. wordt gecontroleerd of iemand nog leeft
7. het spel eindigt wanneer de speler of vijand dood is.

Daarna breiden we dit programma met oefeningen verder uit.

---

## 1. Begin met een werkende basis

Maak een nieuw Rust-project:

```text
cargo new rpg
cd rpg
cargo run
```

We beginnen met een eenvoudige `main.rs`:

```rust
fn main() {
    println!("=== AVONTUUR ===");
    println!("Welkom bij het RPG!");
}
```

Dit is nog geen spel.

Maar het programma werkt.

Dat is belangrijk.

Bij een groter project willen we regelmatig een werkende versie hebben. We voegen niet eerst honderd regels toe om daarna pas te ontdekken dat het programma niet meer compileert.

---

## 2. De speler

Maak een bestand:

```text
src/speler.rs
```

Daarin:

```rust,ignore
pub struct Speler {
    naam: String,
    gezondheid: i32,
    maximale_gezondheid: i32,
}
```

Omdat `Speler` vanuit `main.rs` gebruikt moet kunnen worden, maken we de struct `pub`.

De velden blijven privé.

Dat is bewust.

Andere modules mogen weten dat er een `Speler` bestaat, maar hoeven niet rechtstreeks de interne gegevens te kunnen veranderen.

---

## 3. Een speler maken

We hebben een constructor nodig.

```rust,ignore
impl Speler {
    pub fn nieuw(naam: String) -> Self {
        Self {
            naam,
            gezondheid: 100,
            maximale_gezondheid: 100,
        }
    }
}
```

Hier gebruiken we `Self`.

Binnen:

```rust,ignore
impl Speler
```

betekent `Self`:

```text
Speler
```

Dus:

```rust,ignore
Self {
    naam,
    gezondheid: 100,
    maximale_gezondheid: 100,
}
```

maakt een nieuwe `Speler`.

---

## 4. De eerste methoden

We willen dat de speler schade kan krijgen.

```rust,ignore
impl Speler {
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

Let op het verschil:

```rust,ignore
&mut self
```

bij `neem_schade`.

De methode verandert de speler.

Bij:

```rust,ignore
&self
```

wordt alleen gelezen.

---

## 5. Informatie over de speler

Omdat de velden privé zijn, kunnen we niet vanuit `main.rs` zomaar schrijven:

```rust,ignore
println!("{}", speler.gezondheid);
```

Dat is juist de bedoeling.

We kunnen een methode maken:

```rust,ignore
impl Speler {
    pub fn toon_status(&self) {
        println!(
            "{} — gezondheid: {}/{}",
            self.naam,
            self.gezondheid,
            self.maximale_gezondheid
        );
    }
}
```

Nu bepaalt `Speler` zelf hoe zijn status wordt weergegeven.

---

## 6. De volledige speler

Onze eerste versie van `speler.rs` ziet er dan zo uit:

```rust,ignore
pub struct Speler {
    naam: String,
    gezondheid: i32,
    maximale_gezondheid: i32,
}

impl Speler {
    pub fn nieuw(naam: String) -> Self {
        Self {
            naam,
            gezondheid: 100,
            maximale_gezondheid: 100,
        }
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

    pub fn toon_status(&self) {
        println!(
            "{} — gezondheid: {}/{}",
            self.naam,
            self.gezondheid,
            self.maximale_gezondheid
        );
    }
}
```

---

## 7. De module gebruiken

In `main.rs`:

```rust,ignore
mod speler;

use speler::Speler;

fn main() {
    println!("=== AVONTUUR ===");

    let speler = Speler::nieuw(String::from("Arin"));

    speler.toon_status();
}
```

Voer uit:

```text
cargo run
```

Je zou iets moeten zien als:

```text
=== AVONTUUR ===
Arin — gezondheid: 100/100
```

We hebben nu al meerdere concepten gecombineerd:

- module
- `struct`
- `impl`
- constructor
- `String`
- ownership
- methoden
- borrowing.

---

## 8. Een vijand

Maak:

```text
src/vijand.rs
```

We beginnen eenvoudig:

```rust,ignore
pub struct Vijand {
    naam: String,
    gezondheid: i32,
    aanvalskracht: i32,
}
```

Maak vervolgens een constructor:

```rust,ignore
impl Vijand {
    pub fn nieuw(
        naam: String,
        gezondheid: i32,
        aanvalskracht: i32,
    ) -> Self {
        Self {
            naam,
            gezondheid,
            aanvalskracht,
        }
    }
}
```

---

## 9. Schade en leven

Voeg dezelfde soort methoden toe:

```rust,ignore
impl Vijand {
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

We willen ook kunnen weten hoeveel schade de vijand doet:

```rust,ignore
impl Vijand {
    pub fn aanvalskracht(&self) -> i32 {
        self.aanvalskracht
    }
}
```

En we willen zijn naam kunnen tonen:

```rust,ignore
impl Vijand {
    pub fn toon_status(&self) {
        println!(
            "{} — gezondheid: {}",
            self.naam,
            self.gezondheid
        );
    }
}
```

---

## 10. De twee modules

`main.rs` wordt:

```rust,ignore
mod speler;
mod vijand;

use speler::Speler;
use vijand::Vijand;

fn main() {
    println!("=== AVONTUUR ===");

    let speler = Speler::nieuw(String::from("Arin"));

    let vijand = Vijand::nieuw(
        String::from("Goblin"),
        30,
        5,
    );

    speler.toon_status();
    vijand.toon_status();
}
```

Hiermee hebben we twee afzonderlijke objecten:

```text
Speler
    naam: Arin
    gezondheid: 100

Vijand
    naam: Goblin
    gezondheid: 30
    aanvalskracht: 5
```

---

## 11. Het eerste gevecht

Nu moeten de speler en vijand elkaar kunnen aanvallen.

Maak:

```text
src/gevecht.rs
```

Daarin:

```rust,ignore
use crate::speler::Speler;
use crate::vijand::Vijand;

pub fn speler_valt_aan(
    speler: &Speler,
    vijand: &mut Vijand,
) {
    println!("De speler valt aan!");

    vijand.neem_schade(10);
}
```

Let goed op de borrowing:

```rust,ignore
speler: &Speler
```

De speler wordt alleen gelezen.

```rust,ignore
vijand: &mut Vijand
```

De vijand wordt veranderd.

Dat is precies wat we nodig hebben.

---

## 12. De vijand valt terug aan

Voeg toe:

```rust,ignore
pub fn vijand_valt_aan(
    vijand: &Vijand,
    speler: &mut Speler,
) {
    let schade = vijand.aanvalskracht();

    println!("De vijand valt aan!");

    speler.neem_schade(schade);
}
```

Hier zien we opnieuw hetzelfde patroon:

```text
&Vijand
    ↓
alleen lezen

&mut Speler
    ↓
veranderen
```

---

## 13. Het gevecht uitvoeren

Pas `main.rs` aan:

```rust,ignore
mod speler;
mod vijand;
mod gevecht;

use speler::Speler;
use vijand::Vijand;

fn main() {
    println!("=== AVONTUUR ===");

    let mut speler = Speler::nieuw(
        String::from("Arin")
    );

    let mut vijand = Vijand::nieuw(
        String::from("Goblin"),
        30,
        5,
    );

    speler.toon_status();
    vijand.toon_status();

    gevecht::speler_valt_aan(
        &speler,
        &mut vijand,
    );

    vijand.toon_status();

    gevecht::vijand_valt_aan(
        &vijand,
        &mut speler,
    );

    speler.toon_status();
}
```

Let op de `mut`:

```rust,ignore
let mut speler
let mut vijand
```

Dat is nodig omdat beide waarden veranderd worden.

---

## 14. Eén aanval is nog geen gevecht

We willen meerdere rondes.

Daarvoor kunnen we een `while`-lus gebruiken:

```rust,ignore
while speler.is_levend() && vijand.is_levend() {
    gevecht::speler_valt_aan(
        &speler,
        &mut vijand,
    );

    if !vijand.is_levend() {
        break;
    }

    gevecht::vijand_valt_aan(
        &vijand,
        &mut speler,
    );
}
```

Hier gebruiken we:

```rust,ignore
&&
```

omdat beide voorwaarden waar moeten zijn.

Het gevecht gaat door zolang:

```text
speler leeft
EN
vijand leeft
```

---

## 15. Het einde van het gevecht

Na de lus:

```rust,ignore
if speler.is_levend() {
    println!("Je hebt gewonnen!");
} else {
    println!("Je bent verslagen.");
}
```

We hebben nu een compleet gevecht.

---

## 16. Een belangrijk ontwerpverschil

Je zou misschien kunnen denken:

> Waarom staat de gevechtslogica niet gewoon in `Speler`?

Bijvoorbeeld:

```rust,ignore
impl Speler {
    fn val_aan(&mut self, vijand: &mut Vijand) {
        // ...
    }
}
```

Dat zou technisch kunnen.

Maar we hebben twee deelnemers aan een gevecht:

```text
Speler
Vijand
```

De gevechtsregels gaan over **beide**.

Daarom is een aparte module logisch:

```text
gevecht.rs
```

Dit is een voorbeeld van een ontwerpkeuze.

Er is niet altijd één magische juiste oplossing.

We willen vooral voorkomen dat één struct verantwoordelijk wordt voor alles.

---

## 17. De spelstatus

Nu voegen we `SpelStatus` toe.

Maak:

```text
src/spel.rs
```

Begin met:

```rust,ignore
pub enum SpelStatus {
    Menu,
    Gevecht,
    GameOver,
}
```

Later kunnen we meer toestanden toevoegen.

---

## 18. De struct `Spel`

In dezelfde module:

```rust,ignore
use crate::speler::Speler;
use crate::vijand::Vijand;

pub struct Spel {
    speler: Speler,
    vijand: Vijand,
    status: SpelStatus,
}
```

We gebruiken hier voorlopig één vijand.

Later vervangen we dat door een `Vec<Vijand>`.

---

## 19. Een nieuw spel

Maak een constructor:

```rust,ignore
impl Spel {
    pub fn nieuw() -> Self {
        Self {
            speler: Speler::nieuw(
                String::from("Arin")
            ),
            vijand: Vijand::nieuw(
                String::from("Goblin"),
                30,
                5,
            ),
            status: SpelStatus::Menu,
        }
    }
}
```

Nu is `Spel` verantwoordelijk voor het maken van de beginsituatie.

---

## 20. Waarom `Spel` eigenaar is

We hebben nu:

```text
Spel
├── Speler
├── Vijand
└── SpelStatus
```

Dat betekent:

```text
Spel bezit de speler.
Spel bezit de vijand.
Spel bezit de status.
```

De `Spel`-structuur bevat dus de volledige toestand van ons programma.

Dat is een belangrijk concept.

De game-loop kan straks de toestand van `Spel` veranderen.

---

## 21. De status gebruiken

We kunnen bijvoorbeeld een methode maken:

```rust,ignore
impl Spel {
    pub fn toon_status(&self) {
        match self.status {
            SpelStatus::Menu => {
                println!("Je bent in het menu.");
            }
            SpelStatus::Gevecht => {
                println!("Je bent in gevecht.");
            }
            SpelStatus::GameOver => {
                println!("Het spel is afgelopen.");
            }
        }
    }
}
```

Hier combineren we:

- `enum`
- `match`
- borrowing
- methoden.

---

## 22. Een eerste game-loop

We kunnen nu een eenvoudige `start`-methode maken.

```rust,ignore
impl Spel {
    pub fn start(&mut self) {
        loop {
            match self.status {
                SpelStatus::Menu => {
                    self.start_gevecht();
                }

                SpelStatus::Gevecht => {
                    self.voer_gevecht_uit();
                }

                SpelStatus::GameOver => {
                    println!("Game over.");
                    break;
                }
            }
        }
    }
}
```

We moeten nu alleen nog de twee methoden maken.

---

## 23. Naar het gevecht gaan

```rust,ignore
impl Spel {
    fn start_gevecht(&mut self) {
        println!("Een Goblin verschijnt!");

        self.status = SpelStatus::Gevecht;
    }
}
```

De status verandert dus van:

```text
Menu
```

naar:

```text
Gevecht
```

---

## 24. Het gevecht uitvoeren

Nu komt een belangrijk Rust-probleem.

We hebben in `self`:

```text
self.speler
self.vijand
```

Maar we willen functies aanroepen die beide tegelijk lenen.

Dat kan bijvoorbeeld zo:

```rust,ignore
fn voer_gevecht_uit(&mut self) {
    while self.speler.is_levend()
        && self.vijand.is_levend()
    {
        gevecht::speler_valt_aan(
            &self.speler,
            &mut self.vijand,
        );

        if !self.vijand.is_levend() {
            break;
        }

        gevecht::vijand_valt_aan(
            &self.vijand,
            &mut self.speler,
        );
    }

    self.status = SpelStatus::GameOver;
}
```

Rust kan hier controleren dat de verschillende delen van `self` veilig worden gebruikt.

---

## 25. De volledige eerste architectuur

We hebben nu:

```text
src/
├── main.rs
├── spel.rs
├── speler.rs
├── vijand.rs
└── gevecht.rs
```

De verantwoordelijkheden zijn:

```text
main.rs
    programma starten

spel.rs
    algemene speltoestand

speler.rs
    speler

vijand.rs
    vijand

gevecht.rs
    gevechtsregels
```

Dit is een echte, zij het kleine, Rust-applicatie.

---

## 26. `main.rs` wordt heel klein

Nu kan `main.rs` uiteindelijk bijna niets meer bevatten:

```rust,ignore
mod speler;
mod vijand;
mod gevecht;
mod spel;

use spel::Spel;

fn main() {
    let mut spel = Spel::nieuw();

    spel.start();
}
```

Dat is precies wat we willen.

`main` start het programma.

De rest van de code zit op de plaats waar die logisch thuishoort.

---

## 27. Voeg tests toe

Nu komt een belangrijk onderdeel.

We hebben functies die spelregels bevatten.

Die willen we testen.

In `speler.rs` kunnen we bijvoorbeeld onderaan toevoegen:

```rust,ignore
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn speler_is_levend_bij_positieve_gezondheid() {
        let speler = Speler::nieuw(
            String::from("Arin")
        );

        assert!(speler.is_levend());
    }
}
```

---

## 28. Schade testen

Voeg nog een test toe:

```rust,ignore
#[test]
fn schade_verlaagt_gezondheid() {
    let mut speler = Speler::nieuw(
        String::from("Arin")
    );

    speler.neem_schade(30);

    assert!(speler.is_levend());
}
```

Dit test nog niet het exacte aantal gezondheidspunten.

Daarvoor zouden we een getter kunnen maken.

Bijvoorbeeld:

```rust,ignore
pub fn gezondheid(&self) -> i32 {
    self.gezondheid
}
```

Dan:

```rust,ignore
#[test]
fn schade_verlaagt_gezondheid() {
    let mut speler = Speler::nieuw(
        String::from("Arin")
    );

    speler.neem_schade(30);

    assert_eq!(speler.gezondheid(), 70);
}
```

Dit is een betere test.

---

## 29. Test de grens

Wat gebeurt er bij enorme schade?

```rust,ignore
#[test]
fn gezondheid_wordt_niet_negatief() {
    let mut speler = Speler::nieuw(
        String::from("Arin")
    );

    speler.neem_schade(500);

    assert_eq!(speler.gezondheid(), 0);
}
```

Dit is een goed voorbeeld van een **grensgeval**.

We testen niet alleen wat normaal gebeurt.

We testen ook wat er aan de rand van de toegestane waarden gebeurt.

---

## 30. Tests voor een vijand

In `vijand.rs` kunnen we hetzelfde doen.

Bijvoorbeeld:

```rust,ignore
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn goblin_is_levend() {
        let goblin = Vijand::nieuw(
            String::from("Goblin"),
            30,
            5,
        );

        assert!(goblin.is_levend());
    }

    #[test]
    fn schade_verlaagt_gezondheid() {
        let mut goblin = Vijand::nieuw(
            String::from("Goblin"),
            30,
            5,
        );

        goblin.neem_schade(10);

        assert_eq!(goblin.gezondheid(), 20);
    }
}
```

Nu kunnen we:

```text
cargo test
```

uitvoeren.

---

## 31. Compiler, tests en runtime

We hebben inmiddels drie verschillende soorten problemen gezien.

### Compilerprobleem

Bijvoorbeeld:

```text
cannot find ...
```

of:

```text
field `goud` does not exist
```

De code kan niet worden gebouwd.

### Testprobleem

Het programma compileert, maar:

```text
assert_eq!(70, 80)
```

faalt.

De code werkt technisch, maar de implementatie voldoet niet aan onze verwachting.

### Runtimeprobleem

Het programma compileert en de tests slagen, maar tijdens het spelen gebeurt iets onverwachts.

Bijvoorbeeld:

```text
De speler krijgt 5 schade.

Gezondheid: -200
```

De compiler kan niet automatisch bepalen dat dit een verkeerde spelregel is.

Daarom zijn tests belangrijk.

---

## 32. De eerste echte RPG-versie

Onze eerste versie heeft nu ongeveer deze structuur:

```text
              ┌─────────────┐
              │    main     │
              └──────┬──────┘
                     │
                     ▼
              ┌─────────────┐
              │    Spel     │
              └──────┬──────┘
                     │
            ┌────────┼────────┐
            │        │        │
            ▼        ▼        ▼
         Speler    Vijand   Status
            │        │
            └────┬───┘
                 │
                 ▼
             Gevecht
```

Dit is nog maar het begin.

Maar de basis is goed genoeg om verder te bouwen.

---

## 33. Waarom bouwen we niet alles tegelijk?

We zouden nu meteen:

- wapens
- inventaris
- winkel
- meerdere vijanden
- locaties
- savegames

kunnen toevoegen.

Maar daarmee zouden we een groot stuk code krijgen voordat we goed weten of de basis werkt.

In plaats daarvan werken we in kleine stappen:

```text
werkende versie
    ↓
kleine uitbreiding
    ↓
tests
    ↓
werkende versie
    ↓
volgende uitbreiding
```

Dit patroon zullen we voor de rest van het project blijven gebruiken.

---

## 34. Oefening 1 — Maak de speler

Maak een `Speler` met:

```text
naam
gezondheid
maximale gezondheid
```

Voeg toe:

```text
nieuw()
neem_schade()
is_levend()
gezondheid()
```

Test minimaal:

- nieuwe speler heeft 100 gezondheid
- 20 schade geeft 80 gezondheid
- 100 schade maakt de speler dood
- gezondheid wordt nooit negatief.

---

## 35. Oefening 2 — Maak de vijand

Maak een `Vijand` met:

```text
naam
gezondheid
aanvalskracht
```

Voeg toe:

```text
nieuw()
neem_schade()
is_levend()
gezondheid()
aanvalskracht()
```

Maak tests voor dezelfde situaties als bij de speler.

---

## 36. Oefening 3 — Maak een gevechtsfunctie

Maak:

```rust,ignore
fn speler_valt_aan(
    speler: &Speler,
    vijand: &mut Vijand,
)
```

De speler moet de vijand schade geven.

Gebruik voorlopig een vaste schade van:

```text
10
```

Maak daarna:

```rust,ignore
fn vijand_valt_aan(
    vijand: &Vijand,
    speler: &mut Speler,
)
```

De vijand gebruikt zijn `aanvalskracht`.

---

## 37. Oefening 4 — Maak een gevechtsronde

Schrijf een functie:

```rust,ignore
fn gevechtsronde(
    speler: &mut Speler,
    vijand: &mut Vijand,
)
```

De functie moet:

1. de speler laten aanvallen
2. controleren of de vijand nog leeft
3. de vijand laten aanvallen als hij nog leeft.

Denk goed na over de volgorde.

De vijand mag niet terug aanvallen wanneer hij door de eerste aanval is gedood.

---

## 38. Oefening 5 — Volledig gevecht

Schrijf:

```rust,ignore
fn vecht(
    speler: &mut Speler,
    vijand: &mut Vijand,
)
```

Gebruik een lus.

Het gevecht moet doorgaan zolang beide leven.

Aan het einde moet je melden:

```text
Je hebt gewonnen!
```

of:

```text
Je bent verslagen.
```

---

## 39. Oefening 6 — `SpelStatus`

Maak:

```rust,ignore
enum SpelStatus {
    Menu,
    Gevecht,
    GameOver,
}
```

Maak een functie die de status toont:

```rust,ignore
fn toon_status(status: &SpelStatus)
```

Gebruik `match`.

---

## 40. Oefening 7 — Maak `Spel`

Maak:

```rust,ignore
struct Spel {
    speler: Speler,
    vijand: Vijand,
    status: SpelStatus,
}
```

Maak:

```rust,ignore
Spel::nieuw()
```

De beginsituatie moet zijn:

```text
speler = Arin
gezondheid = 100

vijand = Goblin
gezondheid = 30
aanvalskracht = 5

status = Menu
```

---

## 41. Oefening 8 — Game-loop

Voeg aan `Spel` een methode toe:

```rust,ignore
fn start(&mut self)
```

Gebruik:

```rust,ignore
loop
```

en:

```rust,ignore
match
```

De game-loop moet minstens deze toestanden verwerken:

```text
Menu
Gevecht
GameOver
```

---

## 42. Oefening 9 — Bewuste compilerfout

Maak expres deze fout:

```rust,ignore
let speler = Speler::nieuw(
    String::from("Arin")
);

speler.neem_schade(10);
```

Bekijk de compilerfout.

Waarom werkt dit niet?

Los het op.

Hint:

De methode gebruikt:

```rust,ignore
&mut self
```

---

## 43. Oefening 10 — Bewuste borrowing-fout

Probeer:

```rust,ignore
fn test(
    speler: &mut Speler,
    vijand: &mut Vijand,
) {
    let naam = speler;

    gevecht::speler_valt_aan(
        speler,
        vijand,
    );
}
```

Compileer.

Bekijk de fout.

Probeer te begrijpen wat Rust je vertelt.

Het doel van deze oefening is niet alleen de fout oplossen.

Het doel is leren lezen **waarom** Rust de code niet accepteert.

---

## 44. Oefening 11 — Meerdere vijanden

Vervang:

```rust,ignore
vijand: Vijand
```

door:

```rust,ignore
vijanden: Vec<Vijand>
```

Maak bijvoorbeeld:

```text
Goblin
Wolf
Ork
```

Laat de speler de vijanden één voor één bevechten.

Je kunt daarvoor een `for`-lus gebruiken.

---

## 45. Oefening 12 — Vijanden zoeken

Maak een functie die controleert of er nog levende vijanden zijn.

Bijvoorbeeld:

```rust,ignore
fn zijn_er_levende_vijanden(
    vijanden: &[Vijand]
) -> bool {
    // ...
}
```

Gebruik een iterator.

Een mogelijke oplossing gebruikt:

```rust,ignore
any()
```

maar probeer eerst zelf na te denken hoe je dit zou kunnen schrijven.

---

## 46. Oefening 13 — Gebruik een trait

We hebben eerder `Aanvaller` geleerd.

Maak:

```rust,ignore
trait Aanvaller {
    fn aanvalskracht(&self) -> i32;
}
```

Implementeer de trait voor:

```text
Speler
Vijand
```

Geef de speler voorlopig een vaste aanvalskracht van:

```text
10
```

Gebruik daarna:

```rust,ignore
fn toon_aanval(aanvaller: &impl Aanvaller)
```

om de aanvalskracht te tonen.

---

## 47. Oefening 14 — Maak de aanval realistischer

Pas:

```rust,ignore
speler_valt_aan()
```

aan zodat de aanvalskracht van de speler wordt gebruikt.

Dus niet meer:

```rust,ignore
vijand.neem_schade(10);
```

maar conceptueel:

```text
schade = speler.aanvalskracht()
vijand krijgt schade
```

Nu wordt de trait onderdeel van de spelregels.

---

## 48. Oefening 15 — Tests voor het gevecht

Schrijf tests die controleren:

### Normale aanval

Een vijand met 30 gezondheid krijgt 10 schade.

Verwachte gezondheid:

```text
20
```

### Dodelijke aanval

Een vijand met 10 gezondheid krijgt 10 schade.

Verwachte toestand:

```text
niet levend
```

### Vijand valt terug aan

Een speler met 100 gezondheid krijgt 5 schade.

Verwachte gezondheid:

```text
95
```

---

## 49. Oefening 16 — Laat het spel eindigen

Het spel moet eindigen wanneer:

```text
de speler dood is
```

of:

```text
alle vijanden dood zijn
```

Gebruik daarvoor de informatie die je al hebt.

Probeer dit eerst zelf te ontwerpen voordat je de game-loop aanpast.

---

## 50. Oefening 17 — Statusmethoden

Voeg aan `Speler` en `Vijand` een methode toe waarmee hun status wordt weergegeven.

Bijvoorbeeld:

```text
Arin — HP: 80/100
Goblin — HP: 10/30
```

Houd de velden zelf privé.

Gebruik dus niet vanuit `main`:

```rust,ignore
speler.gezondheid
```

maar een methode.

---

## 51. Oefening 18 — Verbeter de output

Maak het gevecht leesbaarder.

Bijvoorbeeld:

```text
=== GEVECHT ===

Arin valt aan!
Goblin krijgt 10 schade.

Goblin valt aan!
Arin krijgt 5 schade.

--- Status ---
Arin — HP: 95/100
Goblin — HP: 20/30
```

Probeer de bestaande functies hiervoor te gebruiken.

Maak niet één enorme functie die alles doet.

---

## 52. Oefening 19 — Modulegrenzen controleren

Controleer je project.

Je zou ongeveer moeten hebben:

```text
src/
├── main.rs
├── spel.rs
├── speler.rs
├── vijand.rs
└── gevecht.rs
```

Beantwoord:

1. Waar staat de speler?
2. Waar staat de vijand?
3. Waar staat de gevechtslogica?
4. Waar wordt de game-loop beheerd?
5. Welke structs hebben privévelden?
6. Welke functies zijn `pub` en waarom?

---

## 53. Oefening 20 — Eindopdracht

Maak een volledige eerste RPG-versie.

De speler:

```text
Arin
100 HP
10 aanvalskracht
```

De vijand:

```text
Goblin
30 HP
5 aanvalskracht
```

Het spel moet:

1. starten
2. de speler en vijand maken
3. een gevecht starten
4. beurt voor beurt aanvallen
5. schade tonen
6. gezondheid tonen
7. stoppen wanneer één van beide dood is
8. melden wie gewonnen heeft.

Gebruik minimaal:

```text
modules
structs
impl
methoden
borrowing
Vec of één Vijand
enum
match
loop
if
tests
```

---

## 54. Rustlings — Eerste RPG

Maak:

```text
exercises/rpg/
```

met de volgende bestanden:

```text
01_speler_struct.rs
02_speler_nieuw.rs
03_speler_schade.rs
04_speler_levend.rs
05_speler_test.rs

06_vijand_struct.rs
07_vijand_nieuw.rs
08_vijand_schade.rs
09_vijand_aanval.rs
10_vijand_test.rs

11_gevecht_aanval.rs
12_gevecht_terug.rs
13_gevecht_ronde.rs
14_gevecht_lus.rs
15_gevecht_einde.rs

16_spelstatus.rs
17_spel_struct.rs
18_spel_nieuw.rs
19_spel_loop.rs
20_spel_gevecht.rs

21_module_speler.rs
22_module_vijand.rs
23_module_gevecht.rs
24_module_spel.rs

25_trait_aanvaller.rs
26_trait_speler.rs
27_trait_vijand.rs
28_trait_gevecht.rs

29_tests_gevecht.rs
30_finale_rpg.rs
```

---

## 55. Wat moet iedere Rustlings-oefening doen?

We gebruiken verschillende soorten oefeningen.

Niet alles is een leeg bestand.

Sommige bestanden bevatten bijvoorbeeld:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
}
```

en moeten worden afgemaakt.

Andere bevatten bewust verkeerde code:

```rust,ignore
let speler = Speler::nieuw(
    String::from("Arin")
);

speler.neem_schade(10);
```

De cursist moet de compilerfout begrijpen en oplossen.

Weer andere bevatten een test:

```rust,ignore
#[test]
fn schade_werkt() {
    // ...
}
```

waarbij de implementatie ontbreekt.

Zo oefenen we drie vaardigheden:

```text
code schrijven
+
code lezen
+
fouten oplossen
```

Alle drie zijn belangrijk.

---

## 56. Rustlings 01–05 — Speler

### `01_speler_struct.rs`

Maak de struct.

### `02_speler_nieuw.rs`

Maak `Speler::nieuw()`.

### `03_speler_schade.rs`

Implementeer `neem_schade()`.

### `04_speler_levend.rs`

Implementeer `is_levend()`.

### `05_speler_test.rs`

Laat de test slagen.

---

## 57. Rustlings 06–10 — Vijand

Herhaal dezelfde stappen voor `Vijand`.

Voeg bij oefening 09 een methode toe:

```rust,ignore
aanvalskracht()
```

Bij oefening 10 staat een test die eerst faalt.

De cursist moet de implementatie repareren.

---

## 58. Rustlings 11–15 — Gevecht

Hier combineren we de onderdelen.

### 11

Speler valt aan.

### 12

Vijand valt terug aan.

### 13

Eén gevechtsronde.

### 14

Een volledige gevechtslus.

### 15

Het gevecht stopt correct.

Bij oefening 15 moet bewust een fout in de lus zitten.

De cursist moet ontdekken waarom de lus te lang doorgaat.

---

## 59. Rustlings 16–20 — Spel

Hier bouwen we `SpelStatus` en `Spel`.

De oefeningen leren:

```text
enum
match
struct
ownership
methods
game-loop
```

Bij oefening 19 wordt een bestaande `match` bewust incompleet gemaakt.

Bijvoorbeeld:

```rust,ignore
match self.status {
    SpelStatus::Menu => {
        // ...
    }

    // Gevecht ontbreekt
}
```

De compiler geeft aan dat niet alle mogelijke gevallen zijn afgehandeld.

Dat is nuttige informatie.

---

## 60. Rustlings 21–24 — Modules

Deze oefeningen laten zien hoe de losse bestanden samenwerken.

Een oefening kan bijvoorbeeld `speler.rs` bevatten maar vergeten:

```rust,ignore
mod speler;
```

De cursist moet de fout herkennen.

Een andere oefening maakt `Speler` wel openbaar:

```rust,ignore
pub struct Speler
```

maar vergeet:

```rust,ignore
pub fn nieuw(...)
```

De cursist ziet dan waarom de constructor vanuit een andere module niet gebruikt kan worden.

---

## 61. Rustlings 25–28 — Traits

Maak:

```rust,ignore
trait Aanvaller {
    fn aanvalskracht(&self) -> i32;
}
```

Implementeer hem voor `Speler` en `Vijand`.

Gebruik daarna:

```rust,ignore
fn toon_aanval(aanvaller: &impl Aanvaller)
```

Deze oefeningen verbinden de eerdere lessen over traits en generics met het RPG-project.

---

## 62. Rustlings 29 — Gevechtstests

Maak een serie tests:

```text
speler_doet_schade
vijand_doet_schade
vijand_kan_sterven
speler_kan_sterven
gevecht_eindigt
```

De testcode moet zoveel mogelijk alleen de openbare interface gebruiken.

Dus niet proberen privévelden rechtstreeks te veranderen.

---

## 63. Rustlings 30 — De eerste RPG

De laatste oefening bevat een gedeeltelijk werkende RPG.

De code compileert misschien al, maar meerdere onderdelen ontbreken.

De cursist moet:

- ontbrekende functies implementeren
- compilerfouten oplossen
- tests laten slagen
- het gevecht correct laten eindigen.

Het uiteindelijke programma moet daadwerkelijk kunnen worden gestart met:

```text
cargo run
```

en een volledig gevecht uitvoeren.

---

## 64. Extra opdracht — maak je eigen vijand

Maak een nieuwe vijand.

Bijvoorbeeld:

```text
Wolf
50 HP
8 aanvalskracht
```

of:

```text
Ork
80 HP
12 aanvalskracht
```

De gevechtscode mag hiervoor niet aangepast hoeven worden.

Je moet alleen een andere `Vijand` kunnen maken.

Dat is een eerste kleine oefening in het voordeel van een goed ontworpen model:

> De bestaande gevechtslogica werkt met verschillende vijanden.

---

## 65. Extra opdracht — voeg een moeilijkheidsgraad toe

Maak bijvoorbeeld:

```rust,ignore
enum Moeilijkheid {
    Makkelijk,
    Normaal,
    Moeilijk,
}
```

Bedenk hoe deze invloed heeft op vijanden.

Bijvoorbeeld:

```text
Makkelijk → minder gezondheid
Normaal   → normale waarden
Moeilijk  → meer gezondheid
```

Je hoeft dit nog niet volledig in `Spel` te verwerken.

Maak eerst alleen het ontwerp.

---

## 66. Extra opdracht — ontwerp zonder code

Stel dat we later honderd verschillende vijanden willen hebben.

Beantwoord:

> Welke code zou je absoluut niet voor iedere vijand opnieuw willen schrijven?

Denk aan:

```text
schade ontvangen
controleren of hij leeft
aanvallen
status tonen
```

Dit is een belangrijke stap richting herbruikbare code.

---

## 67. Wat komt hierna?

We hebben nu een minimale RPG die daadwerkelijk kan spelen.

Maar er ontbreekt nog veel.

De volgende uitbreiding wordt **wapens en inventaris**.

Dan krijgen we bijvoorbeeld:

```text
Speler
├── gezondheid
├── goud
├── wapen
└── inventaris
```

en:

```text
Inventaris
└── HashMap<String, i32>
```

Daarmee kunnen we vervolgens:

- items oppakken
- items gebruiken
- wapens wisselen
- schade laten afhangen van het wapen
- goud gebruiken
- een winkel bouwen.

Daarna kunnen we het spel verder uitbreiden met meerdere vijanden, verkennen en uiteindelijk savegames.

Het belangrijkste is dat we vanaf nu niet meer alleen losse Rust-oefeningen maken. **Iedere nieuwe techniek wordt toegevoegd aan hetzelfde RPG-project.**

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 29](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_29/).
