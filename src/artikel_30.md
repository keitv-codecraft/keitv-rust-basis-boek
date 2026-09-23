# Rust 30 — Wapens en inventaris

Onze RPG kan inmiddels een gevecht uitvoeren.

We hebben:

```text
Spel
├── Speler
├── Vijand
└── Gevecht
```

De speler en vijand kunnen elkaar aanvallen en schade toebrengen.

Maar onze speler is nog erg eenvoudig.

Een echte RPG heeft meestal spullen.

De speler kan bijvoorbeeld hebben:

```text
Zwaard
Potion
Brood
Pijlen
Goud
```

In dit hoofdstuk voegen we daarom twee belangrijke onderdelen toe:

```text
Wapens
Inventaris
```

Daarbij gebruiken we veel Rust die we al kennen:

- structs
- enums
- `impl`
- ownership
- borrowing
- `Option`
- `HashMap`
- `Vec`
- iterators
- traits
- tests
- modules.

We leren dus niet alleen nieuwe code. We leren vooral hoe eerder geleerde concepten samen een groter programma vormen.

---

## 1. Wat is een inventaris?

Een inventaris bevat spullen die de speler bezit.

Bijvoorbeeld:

```text
Potion  × 3
Brood   × 2
Pijl    × 15
```

Een `HashMap` is hiervoor geschikt.

We kunnen denken aan:

```text
sleutel       waarde
---------------------
"Potion"  →     3
"Brood"   →     2
"Pijl"    →    15
```

De sleutel is de naam van het item.

De waarde is het aantal.

---

## 2. De `Inventaris` struct

Maak:

```text id="f0lqvs"
src/inventaris.rs
```

Begin met:

```rust,ignore
use std::collections::HashMap;

pub struct Inventaris {
    items: HashMap<String, i32>,
}
```

De `HashMap` is privé.

Dat betekent dat andere modules niet zomaar dit mogen doen:

```rust,ignore
inventaris.items.clear();
```

De inventaris bepaalt zelf hoe items worden toegevoegd en verwijderd.

---

## 3. Een lege inventaris maken

Maak een constructor:

```rust,ignore
impl Inventaris {
    pub fn nieuw() -> Self {
        Self {
            items: HashMap::new(),
        }
    }
}
```

Nu kunnen we:

```rust,ignore
let inventaris = Inventaris::nieuw();
```

gebruiken.

---

## 4. Een item toevoegen

We hebben eerder `entry()` en `or_insert()` geleerd.

Die zijn hier erg handig.

```rust,ignore
impl Inventaris {
    pub fn voeg_item_toe(
        &mut self,
        naam: &str,
        aantal: i32,
    ) {
        let hoeveelheid = self
            .items
            .entry(String::from(naam))
            .or_insert(0);

        *hoeveelheid += aantal;
    }
}
```

Wat gebeurt hier?

Stel dat de inventaris bevat:

```text
Potion → 2
```

Dan:

```rust,ignore
inventaris.voeg_item_toe("Potion", 3);
```

maakt:

```text
Potion → 5
```

Bestaat het item nog niet?

Dan maakt `or_insert(0)` eerst:

```text
Potion → 0
```

en daarna wordt daar 3 bij opgeteld.

Resultaat:

```text
Potion → 3
```

---

## 5. Waarom `&str`?

De parameter is:

```rust,ignore
naam: &str
```

en niet:

```rust,ignore
naam: String
```

Dat is handig omdat de functie de naam alleen hoeft te lezen.

We kunnen bijvoorbeeld beide gebruiken:

```rust,ignore
inventaris.voeg_item_toe(
    "Potion",
    2,
);
```

maar ook:

```rust,ignore
let naam = String::from("Potion");

inventaris.voeg_item_toe(
    &naam,
    2,
);
```

De functie hoeft de `String` niet over te nemen.

---

## 6. Controleren hoeveel items er zijn

Maak:

```rust,ignore
pub fn aantal(
    &self,
    naam: &str,
) -> i32 {
    match self.items.get(naam) {
        Some(aantal) => *aantal,
        None => 0,
    }
}
```

Hier komt `Option` terug.

`HashMap::get()` geeft:

```text
Option<&i32>
```

Dus:

```text
Some(&3)
```

of:

```text
None
```

We vertalen dat naar:

```text
3
```

of:

```text
0
```

---

## 7. Een item gebruiken

Nu willen we bijvoorbeeld een potion gebruiken.

De functie kan zijn:

```rust,ignore
pub fn gebruik_item(
    &mut self,
    naam: &str,
) -> bool {
    // ...
}
```

We willen:

1. controleren of het item bestaat
2. controleren of er minstens één exemplaar is
3. het aantal met één verminderen
4. `true` teruggeven als het gelukt is
5. anders `false`.

Een mogelijke implementatie:

```rust,ignore
pub fn gebruik_item(
    &mut self,
    naam: &str,
) -> bool {
    let aantal = match self.items.get_mut(naam) {
        Some(aantal) => aantal,
        None => return false,
    };

    if *aantal <= 0 {
        return false;
    }

    *aantal -= 1;

    true
}
```

Hier gebruiken we:

```rust,ignore
get_mut()
```

om de waarde in de `HashMap` te veranderen.

---

## 8. Waarom `get_mut()`?

Bij:

```rust,ignore
self.items.get(naam)
```

krijgen we alleen toegang om te lezen.

Bij:

```rust,ignore
self.items.get_mut(naam)
```

krijgen we toegang om de waarde te veranderen.

Bijvoorbeeld:

```rust,ignore
let aantal = self.items.get_mut("Potion");
```

geeft:

```text
Option<&mut i32>
```

Daarom gebruiken we:

```rust,ignore
*aantal
```

om de daadwerkelijke `i32` te veranderen.

---

## 9. Een item verwijderen

Wanneer het aantal nul wordt, willen we het item eventueel helemaal uit de `HashMap` verwijderen.

Een eenvoudige versie:

```rust,ignore
pub fn verwijder_item(
    &mut self,
    naam: &str,
) -> bool {
    self.items.remove(naam).is_some()
}
```

`remove()` geeft:

```text
Some(waarde)
```

als het item bestond.

Anders:

```text
None
```

Daarom kunnen we `.is_some()` gebruiken.

---

## 10. De inventaris tonen

We willen de speler zijn spullen kunnen laten bekijken.

```rust,ignore
pub fn toon(&self) {
    println!("=== INVENTARIS ===");

    for (naam, aantal) in &self.items {
        println!("{} × {}", naam, aantal);
    }
}
```

Hier itereren we over:

```rust,ignore
&self.items
```

De inventaris blijft dus eigenaar van de `HashMap`.

We lenen hem alleen om de inhoud te lezen.

---

## 11. De inventaris testen

Onderaan `inventaris.rs`:

```rust,ignore
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn item_toevoegen() {
        let mut inventaris = Inventaris::nieuw();

        inventaris.voeg_item_toe(
            "Potion",
            3,
        );

        assert_eq!(
            inventaris.aantal("Potion"),
            3
        );
    }
}
```

Nog een test:

```rust,ignore
#[test]
fn onbekend_item_heeft_aantal_nul() {
    let inventaris = Inventaris::nieuw();

    assert_eq!(
        inventaris.aantal("Potion"),
        0
    );
}
```

En:

```rust,ignore
#[test]
fn item_gebruiken() {
    let mut inventaris = Inventaris::nieuw();

    inventaris.voeg_item_toe(
        "Potion",
        2,
    );

    assert!(inventaris.gebruik_item("Potion"));

    assert_eq!(
        inventaris.aantal("Potion"),
        1
    );
}
```

---

## 12. Het eerste wapen

Maak:

```text
src/wapen.rs
```

We maken een eenvoudige `WapenType`:

```rust,ignore
pub enum WapenType {
    Zwaard,
    Bijl,
    Boog,
    Staf,
}
```

En:

```rust,ignore
pub struct Wapen {
    naam: String,
    soort: WapenType,
    schade: i32,
    prijs: i32,
}
```

---

## 13. Een wapen maken

Voeg toe:

```rust,ignore
impl Wapen {
    pub fn nieuw(
        naam: String,
        soort: WapenType,
        schade: i32,
        prijs: i32,
    ) -> Self {
        Self {
            naam,
            soort,
            schade,
            prijs,
        }
    }
}
```

Een zwaard kan dan bijvoorbeeld:

```rust,ignore
let zwaard = Wapen::nieuw(
    String::from("IJzeren zwaard"),
    WapenType::Zwaard,
    15,
    50,
);
```

---

## 14. Wapeninformatie

We willen het wapen kunnen bekijken.

Voeg toe:

```rust,ignore
impl Wapen {
    pub fn schade(&self) -> i32 {
        self.schade
    }

    pub fn prijs(&self) -> i32 {
        self.prijs
    }

    pub fn toon(&self) {
        println!(
            "{} — schade: {}, prijs: {}",
            self.naam,
            self.schade,
            self.prijs
        );
    }
}
```

De velden blijven privé.

De rest van het programma gebruikt methoden.

---

## 15. Wie bezit het wapen?

Nu krijgen we een interessante ontwerpvraag.

Een speler kan een wapen hebben.

Maar misschien heeft een speler helemaal geen wapen.

Dat is precies waarvoor `Option` geschikt is.

In `Speler` kunnen we toevoegen:

```rust,ignore
wapen: Option<Wapen>,
```

De volledige struct wordt bijvoorbeeld:

```rust,ignore
pub struct Speler {
    naam: String,
    gezondheid: i32,
    maximale_gezondheid: i32,
    wapen: Option<Wapen>,
}
```

Een nieuwe speler heeft:

```rust,ignore
wapen: None,
```

---

## 16. Een wapen geven

Voeg aan `Speler` toe:

```rust,ignore
pub fn geef_wapen(
    &mut self,
    wapen: Wapen,
) {
    self.wapen = Some(wapen);
}
```

De speler neemt nu ownership van het wapen over.

Dat is logisch:

```text
Wapen
  ↓
Speler bezit Wapen
```

---

## 17. Het huidige wapen bekijken

We willen misschien het wapen tonen:

```rust,ignore
pub fn toon_wapen(&self) {
    match &self.wapen {
        Some(wapen) => {
            wapen.toon();
        }

        None => {
            println!("Je hebt geen wapen.");
        }
    }
}
```

Let op:

```rust,ignore
match &self.wapen
```

We lenen het wapen alleen.

We willen het niet uit de speler halen.

---

## 18. Waarom niet `match self.wapen`?

Als we schrijven:

```rust,ignore
match self.wapen {
    Some(wapen) => {
        // ...
    }
    None => {
        // ...
    }
}
```

kan het `Wapen` uit de `Speler` worden verplaatst.

Dat willen we hier niet.

We willen alleen kijken.

Daarom:

```rust,ignore
match &self.wapen
```

Dit is een voorbeeld van hoe ownership en borrowing direct invloed hebben op onze programmastructuur.

---

## 19. Aanvalskracht van de speler

De aanvalskracht kan nu afhangen van het wapen.

```rust,ignore
pub fn aanvalskracht(&self) -> i32 {
    match &self.wapen {
        Some(wapen) => wapen.schade(),
        None => 1,
    }
}
```

Zonder wapen doet de speler één schade.

Met een zwaard van 15 schade:

```text
aanvalskracht = 15
```

Dit is een eerste eenvoudige spelregel.

---

## 20. De trait `Aanvaller`

We hadden eerder al een trait:

```rust,ignore
pub trait Aanvaller {
    fn aanvalskracht(&self) -> i32;
}
```

Die kunnen we nu echt gebruiken.

Bijvoorbeeld in `gevecht.rs`:

```rust,ignore
pub trait Aanvaller {
    fn aanvalskracht(&self) -> i32;
}
```

Maar er is een probleem.

Als de trait in `gevecht.rs` staat, moeten andere modules hem via die module gebruiken.

Het is daarom logischer om een eigen module te maken.

Maak:

```text
src/aanvaller.rs
```

met:

```rust,ignore
pub trait Aanvaller {
    fn aanvalskracht(&self) -> i32;
}
```

---

## 21. De speler implementeert `Aanvaller`

In `speler.rs`:

```rust,ignore
use crate::aanvaller::Aanvaller;
```

En:

```rust,ignore
impl Aanvaller for Speler {
    fn aanvalskracht(&self) -> i32 {
        match &self.wapen {
            Some(wapen) => wapen.schade(),
            None => 1,
        }
    }
}
```

Nu heeft de speler een algemene eigenschap:

```text
Speler
  ↓
Aanvaller
```

---

## 22. De vijand implementeert dezelfde trait

In `vijand.rs`:

```rust,ignore
use crate::aanvaller::Aanvaller;
```

En:

```rust,ignore
impl Aanvaller for Vijand {
    fn aanvalskracht(&self) -> i32 {
        self.aanvalskracht
    }
}
```

Nu hebben beide:

```text
Speler ──┐
         ├── Aanvaller
Vijand ──┘
```

---

## 23. Gevecht met `Aanvaller`

We kunnen nu onze gevechtsfunctie algemener maken.

```rust,ignore
use crate::aanvaller::Aanvaller;
use crate::speler::Speler;
use crate::vijand::Vijand;

pub fn speler_valt_aan(
    speler: &Speler,
    vijand: &mut Vijand,
) {
    let schade = speler.aanvalskracht();

    println!(
        "De speler doet {} schade!",
        schade
    );

    vijand.neem_schade(schade);
}
```

De concrete aanvalskracht komt nu uit de speler.

Het wapen bepaalt dus indirect de schade.

---

## 24. Een wapen wisselen

We willen een speler ook een ander wapen kunnen geven.

Bijvoorbeeld:

```rust,ignore
pub fn neem_wapen(
    &mut self,
) -> Option<Wapen> {
    self.wapen.take()
}
```

Hier gebeurt iets interessants.

`take()`:

1. haalt de huidige waarde uit de `Option`
2. zet de `Option` terug op `None`
3. geeft de oude waarde terug.

Bijvoorbeeld:

```text
voor:
wapen = Some(zwaard)

na:
wapen = None

return:
Some(zwaard)
```

Dit is handig wanneer ownership daadwerkelijk moet worden verplaatst.

---

## 25. Een nieuw wapen geven

Nu kunnen we bijvoorbeeld:

```rust,ignore
let oud_wapen = speler.neem_wapen();

speler.geef_wapen(nieuw_wapen);
```

Het oude wapen is dan teruggegeven aan de code die `neem_wapen()` aanriep.

Dat is een mooie combinatie van:

- `Option`
- ownership
- borrowing
- `take()`.

---

## 26. Wapens in de inventaris?

Nu ontstaat een ontwerpvraag.

Moeten wapens onderdeel zijn van:

```text
Inventaris
```

of moeten ze apart worden beheerd?

Er zijn verschillende mogelijkheden.

### Mogelijkheid 1

De inventaris bevat alleen aantallen:

```text
Potion → 3
Brood → 2
```

Wapens staan apart:

```text
Speler
└── wapen: Option<Wapen>
```

### Mogelijkheid 2

De inventaris bevat ook wapens.

Dan wordt het ontwerp ingewikkelder, want een wapen heeft meer eigenschappen dan alleen een naam en aantal.

Voor onze eerste RPG kiezen we:

> Verbruikbare items komen in de `HashMap`; het actieve wapen wordt apart door de speler beheerd.

Dat houdt het model eenvoudig.

---

## 27. Een eerste complete `Speler`

Onze speler bevat nu ongeveer:

```rust,ignore
pub struct Speler {
    naam: String,
    gezondheid: i32,
    maximale_gezondheid: i32,
    wapen: Option<Wapen>,
    inventaris: Inventaris,
}
```

Conceptueel:

```text
Speler
├── naam
├── gezondheid
├── maximale gezondheid
├── wapen
│   └── Option<Wapen>
└── inventaris
    └── HashMap<String, i32>
```

Dit is een veel realistischer model.

---

## 28. De constructor uitbreiden

Onze constructor moet nu ook een inventaris maken:

```rust,ignore
pub fn nieuw(naam: String) -> Self {
    Self {
        naam,
        gezondheid: 100,
        maximale_gezondheid: 100,
        wapen: None,
        inventaris: Inventaris::nieuw(),
    }
}
```

We kunnen daarna bijvoorbeeld in `main`:

```rust,ignore
let mut speler = Speler::nieuw(
    String::from("Arin")
);
```

en vervolgens:

```rust,ignore
speler.geef_wapen(zwaard);
```

---

## 29. Startuitrusting

We kunnen de speler ook direct met een wapen laten beginnen.

Bijvoorbeeld:

```rust,ignore
pub fn nieuw(naam: String) -> Self {
    let mut inventaris = Inventaris::nieuw();

    inventaris.voeg_item_toe(
        "Potion",
        2,
    );

    Self {
        naam,
        gezondheid: 100,
        maximale_gezondheid: 100,
        wapen: Some(
            Wapen::nieuw(
                String::from("Houten zwaard"),
                WapenType::Zwaard,
                5,
                10,
            )
        ),
        inventaris,
    }
}
```

Nu begint iedere speler met:

```text
Houten zwaard
2 Potions
```

---

## 30. Een potion gebruiken

De inventaris kan alleen bijhouden dat de speler een potion heeft.

De `Speler` moet bepalen wat een potion **doet**.

Bijvoorbeeld:

```rust,ignore
pub fn gebruik_potion(&mut self) -> bool {
    if self.inventaris.gebruik_item("Potion") {
        self.gezondheid += 20;

        if self.gezondheid > self.maximale_gezondheid {
            self.gezondheid = self.maximale_gezondheid;
        }

        true
    } else {
        false
    }
}
```

Hier hebben we een belangrijk onderscheid:

```text
Inventaris
    weet:
    "Potion × 3"

Speler
    weet:
    "Een Potion geneest 20 HP"
```

De inventaris beheert de **voorraad**.

De speler beheert de **betekenis van het gebruiken van een item**.

---

## 31. Waarom is dit een goede scheiding?

Stel dat we later een `Ether` toevoegen.

De inventaris hoeft niet te weten wat een Ether doet.

De speler kan bijvoorbeeld bepalen:

```text
Potion → +20 gezondheid
Ether   → +10 mana
Antigif → verwijdert vergiftiging
```

De inventaris hoeft alleen te weten:

```text
Potion → 3
Ether → 2
Antigif → 1
```

Dit maakt onderdelen onafhankelijker.

---

## 32. Inventaris doorzoeken met iterators

We kunnen nu ook zoeken naar items.

Bijvoorbeeld alle items met meer dan vijf exemplaren:

```rust,ignore
pub fn veel_items(&self) -> Vec<&String> {
    self.items
        .iter()
        .filter(|(_, aantal)| **aantal > 5)
        .map(|(naam, _)| naam)
        .collect()
}
```

Hier gebruiken we:

```text
iter()
filter()
map()
collect()
```

Dit is precies waarom we eerder iterators hebben geleerd.

De `HashMap` wordt de bron.

De iterator selecteert en verandert de gegevens.

`collect()` maakt daar een nieuwe `Vec` van.

---

## 33. Let op de types

Bij:

```rust,ignore
.filter(|(_, aantal)| **aantal > 5)
```

is `aantal` niet direct een `i32`.

Omdat we over een geleende `HashMap` itereren, krijgen we referenties.

Daarom zien we meerdere `*`-tekens.

Voor beginners is het belangrijkste:

> Een iterator over een geleende `HashMap` geeft geleende waarden terug.

Je hoeft nog niet alle details van de referentietypen uit je hoofd te kennen.

De compiler kan hier ook helpen.

---

## 34. Alle itemnamen verzamelen

Een eenvoudiger voorbeeld:

```rust,ignore
pub fn item_namen(&self) -> Vec<String> {
    self.items
        .keys()
        .cloned()
        .collect()
}
```

Hier gebruiken we:

```text
keys()
cloned()
collect()
```

Waarom `cloned()`?

Omdat `keys()` geleende sleutels geeft:

```text
&String
```

maar we een nieuwe:

```text
String
```

willen maken.

---

## 35. De inventaris testen met meerdere items

Voeg een test toe:

```rust,ignore
#[test]
fn meerdere_items() {
    let mut inventaris = Inventaris::nieuw();

    inventaris.voeg_item_toe("Potion", 3);
    inventaris.voeg_item_toe("Brood", 2);

    assert_eq!(
        inventaris.aantal("Potion"),
        3
    );

    assert_eq!(
        inventaris.aantal("Brood"),
        2
    );
}
```

En:

```rust,ignore
#[test]
fn hetzelfde_item_wordt_opgeteld() {
    let mut inventaris = Inventaris::nieuw();

    inventaris.voeg_item_toe("Potion", 2);
    inventaris.voeg_item_toe("Potion", 4);

    assert_eq!(
        inventaris.aantal("Potion"),
        6
    );
}
```

---

## 36. Een belangrijke grens: negatieve aantallen

Wat moet er gebeuren bij:

```rust,ignore
inventaris.voeg_item_toe(
    "Potion",
    -5,
);
```

Onze huidige implementatie zou dit accepteren.

Dat is waarschijnlijk geen goede spelregel.

We kunnen daarom bijvoorbeeld weigeren negatieve aantallen toe te voegen.

```rust,ignore
pub fn voeg_item_toe(
    &mut self,
    naam: &str,
    aantal: i32,
) {
    if aantal <= 0 {
        return;
    }

    let hoeveelheid = self
        .items
        .entry(String::from(naam))
        .or_insert(0);

    *hoeveelheid += aantal;
}
```

Dit is een voorbeeld van een belangrijke ontwerpvraag:

> Welke invoer is geldig?

Tests moeten zulke regels vastleggen.

---

## 37. Test de grens

```rust,ignore
#[test]
fn negatieve_hoeveelheid_wordt_genegeerd() {
    let mut inventaris = Inventaris::nieuw();

    inventaris.voeg_item_toe(
        "Potion",
        -5,
    );

    assert_eq!(
        inventaris.aantal("Potion"),
        0
    );
}
```

Nu hebben we een expliciete spelregel.

---

## 38. Het wapen testen

In `wapen.rs`:

```rust,ignore
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn wapen_heeft_juiste_schade() {
        let wapen = Wapen::nieuw(
            String::from("Zwaard"),
            WapenType::Zwaard,
            15,
            50,
        );

        assert_eq!(
            wapen.schade(),
            15
        );
    }
}
```

---

## 39. De speler met wapen testen

In `speler.rs`:

```rust,ignore
#[test]
fn speler_heeft_aanvalskracht_van_wapen() {
    let mut speler = Speler::nieuw(
        String::from("Arin")
    );

    let zwaard = Wapen::nieuw(
        String::from("Zwaard"),
        WapenType::Zwaard,
        15,
        50,
    );

    speler.geef_wapen(zwaard);

    assert_eq!(
        speler.aanvalskracht(),
        15
    );
}
```

Hier testen we niet alleen een los wapen.

We testen de samenwerking tussen:

```text
Speler
+
Wapen
```

---

## 40. Een speler zonder wapen

Test ook:

```rust,ignore
#[test]
fn speler_zonder_wapen_heeft_basisaanval() {
    let speler = Speler::nieuw(
        String::from("Arin")
    );

    assert_eq!(
        speler.aanvalskracht(),
        1
    );
}
```

Hier testen we een `Option::None`-situatie.

---

## 41. Een potion testen

Test:

```rust,ignore
#[test]
fn potion_geneest_speler() {
    let mut speler = Speler::nieuw(
        String::from("Arin")
    );

    speler.neem_schade(30);

    assert!(speler.gebruik_potion());

    assert_eq!(
        speler.gezondheid(),
        90
    );
}
```

Dit test meerdere onderdelen tegelijk:

```text
Speler
 ↓
Inventaris
 ↓
Potion
 ↓
gezondheid
```

---

## 42. Geen potion meer

Test ook:

```rust,ignore
#[test]
fn potion_zonder_voorraad_mislukt() {
    let mut speler = Speler::nieuw(
        String::from("Arin")
    );

    assert!(!speler.gebruik_potion());
}
```

Dit is een voorbeeld van een functie die informatie teruggeeft over succes:

```text
true
```

of:

```text
false
```

Later zouden we dit eventueel kunnen veranderen naar `Result`, maar voor deze eenvoudige actie is `bool` voldoende.

---

## 43. De spelerstatus uitbreiden

Onze status kan nu bijvoorbeeld worden:

```text id="g20i4g"
=== STATUS ===

Arin
Gezondheid: 90/100
Aanvalskracht: 15
Wapen: IJzeren zwaard
Goud: 50
```

We hebben nog geen goud toegevoegd aan onze nieuwste `Speler`, maar dat komt goed van pas voor de winkel.

Voeg daarom toe:

```rust,ignore
goud: i32,
```

---

## 44. Goud beheren

Maak:

```rust,ignore
pub fn goud(&self) -> i32 {
    self.goud
}
```

En:

```rust,ignore
pub fn verdien_goud(
    &mut self,
    hoeveelheid: i32,
) {
    if hoeveelheid > 0 {
        self.goud += hoeveelheid;
    }
}
```

Voor betalen:

```rust,ignore
pub fn betaal(
    &mut self,
    bedrag: i32,
) -> bool {
    if bedrag <= 0 {
        return false;
    }

    if self.goud < bedrag {
        return false;
    }

    self.goud -= bedrag;
    true
}
```

Nu kan een winkel later controleren of een aankoop mogelijk is.

---

## 45. Een winkel

We hebben nu genoeg onderdelen om een heel eenvoudige winkel te maken.

Maak:

```text
src/winkel.rs
```

Een winkel kan bijvoorbeeld wapens bevatten:

```rust,ignore
use crate::wapen::Wapen;

pub struct Winkel {
    wapens: Vec<Wapen>,
}
```

Waarom `Vec`?

Omdat de winkel meerdere wapens kan hebben.

Bijvoorbeeld:

```text
Houten zwaard
IJzeren zwaard
Bijl
Boog
```

---

## 46. Een winkel maken

```rust,ignore
impl Winkel {
    pub fn nieuw() -> Self {
        Self {
            wapens: vec![
                Wapen::nieuw(
                    String::from("Houten zwaard"),
                    WapenType::Zwaard,
                    5,
                    10,
                ),
                Wapen::nieuw(
                    String::from("IJzeren zwaard"),
                    WapenType::Zwaard,
                    15,
                    50,
                ),
            ],
        }
    }
}
```

We gebruiken hier de `vec!`-macro.

Die hebben we al eerder gezien bij `Vec`.

---

## 47. De winkel tonen

```rust,ignore
pub fn toon(&self) {
    println!("=== WINKEL ===");

    for (index, wapen) in self.wapens.iter().enumerate() {
        println!("{}: ", index + 1);
        wapen.toon();
    }
}
```

Hier zien we een nieuwe combinatie:

```text
Vec
+
iter()
+
enumerate()
```

`enumerate()` geeft ons naast het element ook een nummer.

---

## 48. Een wapen kiezen

We kunnen een functie maken:

```rust,ignore
pub fn kies_wapen(
    &self,
    index: usize,
) -> Option<&Wapen> {
    self.wapens.get(index)
}
```

Waarom `Option`?

Omdat de index misschien niet bestaat.

Bijvoorbeeld:

```rust,ignore
let wapen = winkel.kies_wapen(100);
```

kan:

```text
None
```

opleveren.

Dit is een mooie echte toepassing van `Option`.

---

## 49. Een winkel hoeft niet eigenaar te worden van de speler

De winkel hoeft de speler niet te bezitten.

Een aankoopfunctie kan bijvoorbeeld tijdelijk toegang krijgen:

```rust,ignore
pub fn koop(
    &self,
    speler: &mut Speler,
    index: usize,
) -> bool {
    // ...
}
```

De winkel heeft dus:

```text
&self
```

om zijn eigen gegevens te lezen.

De speler krijgt:

```text
&mut Speler
```

om goud en eventueel het wapen te veranderen.

---

## 50. Een probleem: ownership van het gekochte wapen

Hier komen we een interessante ontwerpkwestie tegen.

`kies_wapen()` geeft:

```rust,ignore
Option<&Wapen>
```

maar `Speler::geef_wapen()` verwacht:

```rust,ignore
Wapen
```

Dat zijn verschillende dingen.

We hebben:

```text
&Wapen
```

maar moeten krijgen:

```text
Wapen
```

We kunnen het wapen niet zomaar kopiëren, want een `Wapen` bevat een `String`.

We zouden `Clone` kunnen gebruiken.

---

## 51. `Clone` voor `Wapen`

Voeg toe:

```rust,ignore
#[derive(Debug, Clone)]
pub struct Wapen {
    // ...
}
```

Nu kunnen we een wapen klonen.

Bijvoorbeeld:

```rust,ignore
let nieuw_wapen = match winkel.kies_wapen(index) {
    Some(wapen) => wapen.clone(),
    None => return false,
};
```

Nu hebben we een eigen `Wapen`.

Daarna:

```rust,ignore
speler.geef_wapen(nieuw_wapen);
```

---

## 52. Maar er is nog een probleem

Als de winkel het wapen blijft houden en we het klonen, kan de speler hetzelfde wapen oneindig vaak kopen.

Voor een eerste eenvoudige RPG kan dat prima zijn.

Maar als de winkelvoorraad beperkt moet zijn, is het ontwerp anders.

Dan zou de winkel bijvoorbeeld eigenaar kunnen zijn van een voorraad:

```text
Wapen
+
aantal
```

Dat hoeven we nu nog niet op te lossen.

Dit is juist een goede ontwerpvraag om later te behandelen.

---

## 53. Een belangrijke les over ontwerp

We zien nu iets dat bij grotere programma's vaak gebeurt:

Een kleine beslissing heeft gevolgen verderop.

Bijvoorbeeld:

```text
Wapen in winkel
        ↓
wie bezit het?
        ↓
kan het gekopieerd worden?
        ↓
is de winkelvoorraad onbeperkt?
        ↓
moet een gekocht wapen uit de winkel verdwijnen?
```

Er is niet altijd direct één perfect antwoord.

We kiezen voorlopig voor een eenvoudige oplossing:

> De winkel bevat modellen van wapens en geeft een gekopieerd wapen aan de speler.

Later kunnen we de winkel uitbreiden.

---

## 54. De RPG-structuur

Ons project begint nu ongeveer zo te worden:

```text id="6pmbc4"
src/
├── main.rs
├── spel.rs
├── speler.rs
├── vijand.rs
├── gevecht.rs
├── aanvaller.rs
├── wapen.rs
├── inventaris.rs
└── winkel.rs
```

Dat is inmiddels een redelijk aantal modules.

Maar iedere module heeft een duidelijke taak.

```text
speler.rs
    speler en spelerregels

vijand.rs
    vijanden

wapen.rs
    wapens

inventaris.rs
    voorraad

winkel.rs
    winkel

gevecht.rs
    gevechten

aanvaller.rs
    gemeenschappelijk aanvalsgedrag

spel.rs
    algemene speltoestand
```

---

## 55. De afhankelijkheden

Conceptueel:

```text
                    Spel
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
    Speler        Vijand       Gevecht
       │             │             │
       │             └──────┬──────┘
       │                    │
       ▼                    │
  Inventaris                │
       │                    │
       ▼                    │
    HashMap                 │
                            │
       Wapen ◄──────────────┘
         ▲
         │
      Winkel
```

En:

```text
Speler ──► Aanvaller
Vijand ──► Aanvaller
```

Het schema hoeft niet letterlijk overeen te komen met `use`-regels.

Het helpt ons vooral om de verantwoordelijkheden te begrijpen.

---

## 56. Rustlings — Wapens en inventaris

Maak:

```text
exercises/rpg_items/
```

met:

```text
01_inventaris.rs
02_item_toevoegen.rs
03_item_aantal.rs
04_item_gebruiken.rs
05_item_verwijderen.rs
06_hashmap_test.rs

07_wapentype.rs
08_wapen.rs
09_wapen_nieuw.rs
10_wapen_schade.rs
11_wapen_clone.rs
12_wapen_test.rs

13_speler_option.rs
14_speler_wapen.rs
15_speler_zonder_wapen.rs
16_speler_aanval.rs
17_speler_trait.rs

18_potion.rs
19_potion_test.rs
20_potion_grens.rs

21_inventaris_iterator.rs
22_inventaris_filter.rs
23_inventaris_collect.rs

24_winkel.rs
25_winkel_kiezen.rs
26_winkel_option.rs
27_winkel_kopen.rs

28_debug_ownership.rs
29_debug_borrowing.rs
30_final_items.rs
```

---

## 57. Rustlings 01–06 — Inventaris

## 01 — lege inventaris

Maak een `Inventaris`.

## 02 — item toevoegen

Gebruik `entry()`.

## 03 — aantal opvragen

Gebruik `get()` en `Option`.

## 04 — item gebruiken

Gebruik `get_mut()`.

## 05 — item verwijderen

Gebruik `remove()`.

## 06 — testen

Laat de gegeven tests slagen.

---

## 58. Rustlings 07–12 — Wapens

Maak:

```text
WapenType
Wapen
```

Voeg toe:

```text
nieuw()
schade()
prijs()
clone()
```

Oefening 11 moet bewust een fout bevatten:

De code probeert een `Wapen` te kopiëren alsof het automatisch `Copy` is.

De cursist moet ontdekken waarom `Clone` hier wel kan en `Copy` niet.

---

## 59. Rustlings 13–17 — Wapen en `Option`

Maak:

```rust,ignore
wapen: Option<Wapen>
```

Test:

```text
None
Some(wapen)
```

Oefening 15 moet een speler zonder wapen verwerken.

Oefening 16 maakt `aanvalskracht()` afhankelijk van het huidige wapen.

Oefening 17 voegt de `Aanvaller`-trait toe.

---

## 60. Rustlings 18–20 — Items gebruiken

Maak een potion.

De speler begint bijvoorbeeld met:

```text
2 Potions
```

Een potion:

```text
herstelt 20 HP
```

Maar nooit boven maximale gezondheid.

Test daarom:

```text
80 → 100
95 → 100
40 → 60
```

---

## 61. Rustlings 21–23 — Iterators

Gebruik de inventaris als bron.

### 21

Verzamel alle itemnamen.

### 22

Zoek items waarvan het aantal groter is dan 5.

### 23

Maak daar een `Vec<String>` van.

Gebruik:

```text
iter()
filter()
map()
collect()
```

waar dat logisch is.

---

## 62. Rustlings 24–27 — Winkel

Maak een winkel met meerdere wapens.

De cursist moet:

1. de winkel maken
2. wapens tonen
3. een wapen op index zoeken
4. `Option` correct verwerken
5. een gekocht wapen aan de speler geven.

Een ongeldige index mag niet tot een panic leiden.

Dus liever:

```text
None
```

dan:

```text
panic
```

---

## 63. Rustlings 28 — Ownership-debugging

Geef code zoals:

```rust,ignore
let wapen = Wapen::nieuw(...);

speler.geef_wapen(wapen);

println!("{}", wapen.schade());
```

Laat de cursist de compilerfout onderzoeken.

Wat is er gebeurd?

`geef_wapen()` heeft ownership van het wapen gekregen.

De oorspronkelijke variabele kan daarna niet meer worden gebruikt.

---

## 64. Rustlings 29 — Borrowing-debugging

Geef bijvoorbeeld code waarin tegelijkertijd een immutable en mutable borrow nodig is.

De cursist moet bepalen:

- welke waarde alleen gelezen wordt
- welke waarde veranderd wordt
- hoe lang de borrow nodig is.

Het doel is niet alleen de code repareren.

Het doel is de compiler als hulpmiddel gebruiken om ownership en borrowing beter te begrijpen.

---

## 65. Rustlings 30 — Het volledige item-systeem

De laatste oefening combineert:

```text
Speler
Wapen
Inventaris
HashMap
Option
Trait
Iterator
Tests
```

De speler moet:

```text
een wapen hebben
items kunnen bewaren
een potion kunnen gebruiken
aanvalskracht uit het wapen halen
```

De bestaande gevechtscode moet vervolgens automatisch het nieuwe wapen gebruiken.

---

## 66. Zelfstandige opdracht 1 — Maak een nieuw item

Voeg een item toe:

```text
"Brood"
```

Een brood geneest:

```text
10 HP
```

Maak:

```rust,ignore
pub fn gebruik_brood(
    &mut self,
) -> bool
```

Schrijf tests.

---

## 67. Zelfstandige opdracht 2 — Maak meerdere wapens

Voeg toe:

```text
Houten zwaard   5 schade
IJzeren zwaard 15 schade
Bijl            20 schade
Boog            12 schade
```

Laat de speler verschillende wapens uitproberen.

Controleer met tests dat de aanvalskracht correct verandert.

---

## 68. Zelfstandige opdracht 3 — Wapen wisselen

Laat de speler:

1. een huidig wapen hebben
2. een nieuw wapen krijgen
3. het oude wapen terugkrijgen met `take()`
4. het nieuwe wapen gebruiken.

Hier oefen je specifiek:

```text
Option
take()
ownership
```

---

## 69. Zelfstandige opdracht 4 — Inventaris zoeken

Maak:

```rust,ignore
fn zoek_item(
    &self,
    naam: &str,
) -> Option<i32>
```

Geef:

```text
Some(aantal)
```

als het item bestaat.

En:

```text
None
```

als het niet bestaat.

Gebruik daarna deze functie in een test.

---

## 70. Zelfstandige opdracht 5 — Winkel met betaalfunctie

Maak het mogelijk om een wapen te kopen.

De speler heeft bijvoorbeeld:

```text
50 goud
```

Het ijzeren zwaard kost:

```text
50 goud
```

Na aankoop:

```text
goud = 0
```

De speler heeft het nieuwe wapen.

Test ook:

```text
speler heeft te weinig goud
```

In dat geval:

```text
aankoop mislukt
goud blijft hetzelfde
wapen blijft hetzelfde
```

---

## 71. Zelfstandige opdracht 6 — Volledige status

Maak een methode:

```rust,ignore
pub fn toon_status(&self)
```

die bijvoorbeeld toont:

```text
=== STATUS ===

Naam: Arin
Gezondheid: 85/100
Goud: 35
Aanvalskracht: 15

Wapen:
IJzeren zwaard — 15 schade

Inventaris:
Potion × 2
Brood × 3
```

Gebruik hiervoor de bestaande methoden.

Probeer niet alle interne velden van `Inventaris` vanuit `Speler` rechtstreeks te benaderen.

---

## 72. Ontwerpvraag — waar hoort logica?

We hebben nu drie onderdelen:

```text
Speler
Inventaris
Wapen
```

Stel dat een potion 20 gezondheid geeft.

Waar hoort die regel?

Niet automatisch in `Inventaris`.

De inventaris weet:

```text
Potion × 3
```

De `Speler` weet:

```text
Potion geneest 20 HP
```

Dit soort vragen wordt steeds belangrijker naarmate het programma groeit.

Vraag steeds:

> Welk onderdeel weet het meest over deze regel?

Dat onderdeel is vaak een goede plaats voor de betreffende logica.

---

## 73. Ontwerpvraag — wanneer gebruik je `Option`?

We hebben nu:

```rust,ignore
wapen: Option<Wapen>
```

omdat een speler wel of geen wapen kan hebben.

Dat is anders dan:

```rust,ignore
gezondheid: i32
```

Gezondheid heeft altijd een waarde.

Daarom:

```text
gezondheid → i32
wapen      → Option<Wapen>
```

Een `Option` is dus niet zomaar een ingewikkelde manier om een variabele te maken.

Het beschrijft een echte mogelijkheid:

> Deze waarde kan ontbreken.

---

## 74. Ontwerpvraag — wanneer gebruik je `Vec`?

We gebruiken:

```rust,ignore
Vec<Wapen>
```

voor de wapens in de winkel.

Waarom?

Omdat:

- er meerdere wapens zijn
- de volgorde eventueel relevant kan zijn
- we door alle wapens willen kunnen lopen.

Voor de inventaris gebruiken we:

```rust,ignore
HashMap<String, i32>
```

omdat we juist snel op naam willen zoeken.

Dat zijn verschillende gegevensstructuren voor verschillende behoeften.

---

## 75. Ontwerpvraag — wanneer gebruik je `HashMap`?

Stel dat we de inventaris als `Vec` zouden maken:

```text
Potion
Potion
Potion
Brood
Brood
Pijl
Pijl
Pijl
...
```

Dan moeten we steeds door de lijst zoeken om te bepalen hoeveel Potions er zijn.

Met:

```text
HashMap<String, i32>
```

kunnen we direct het aantal bij een naam bewaren.

Dus:

```text
Vec
    lijst van losse elementen

HashMap
    sleutel → waarde
```

Het juiste type hangt af van wat je met de gegevens wilt doen.

---

## 76. Wat hebben we toegevoegd?

Onze RPG heeft nu een veel interessanter model:

```text
Spel
│
├── Speler
│   ├── gezondheid
│   ├── goud
│   ├── Option<Wapen>
│   └── Inventaris
│       └── HashMap<String, i32>
│
├── Vijanden
│
└── SpelStatus
```

Daarnaast hebben we:

```text
Winkel
    └── Vec<Wapen>

Aanvaller
    ├── Speler
    └── Vijand
```

Dit is een mooi voorbeeld van hoe een groter Rust-programma ontstaat uit kleinere concepten.

---

## 77. Controleer je project

Voer na iedere grote wijziging uit:

```text
cargo check
```

Daarna:

```text
cargo test
```

En wanneer alles werkt:

```text
cargo run
```

Een handige volgorde is dus:

```text
code schrijven
     ↓
cargo check
     ↓
fouten oplossen
     ↓
cargo test
     ↓
tests repareren
     ↓
cargo run
```

Maak er een gewoonte van om niet tientallen wijzigingen te maken voordat je opnieuw compileert.

---

## 78. Eindopdracht

Breid de RPG uit zodat een speler:

1. met een wapen begint
2. een inventaris heeft
3. minimaal twee Potions heeft
4. zijn inventaris kan bekijken
5. zijn huidige wapen kan bekijken
6. aanvalskracht uit het wapen haalt
7. een potion kan gebruiken
8. gezondheid nooit boven het maximum komt
9. verschillende wapens kan gebruiken
10. een wapen uit de winkel kan kopen.

De volgende spelronde moet vervolgens het gekozen wapen gebruiken.

Bijvoorbeeld:

```text
=== AVONTUUR ===

Arin
HP: 100/100
Goud: 50
Wapen: Houten zwaard
Aanvalskracht: 5

Een Goblin verschijnt!

Arin valt aan.
De Goblin krijgt 5 schade.

Goblin valt aan.
Arin krijgt 5 schade.
```

Na het kopen van een beter wapen:

```text
Arin
Wapen: IJzeren zwaard
Aanvalskracht: 15
```

moet hetzelfde gevecht automatisch 15 schade veroorzaken.

De gevechtscode hoeft daarvoor niet speciaal voor het ijzeren zwaard aangepast te worden.

Dat is precies het soort hergebruik dat we willen bereiken.

---

## 79. Vooruitblik

De RPG begint nu echt vorm te krijgen.

We hebben:

```text
speler
vijanden
gevechten
wapens
inventaris
winkel
goud
items
```

De volgende logische stap is om de speler niet langer steeds tegen één vaste vijand te laten vechten.

We gaan de **wereld en het verkennen** toevoegen.

Daarbij kunnen we onder andere gebruiken:

- `enum` voor locaties
- `Vec` voor beschikbare vijanden
- `Option` voor een mogelijke ontmoeting
- iterators om gegevens te doorzoeken
- structs voor locaties
- de bestaande `SpelStatus`
- en uiteindelijk keuzes van de speler.

Zo verandert onze verzameling gevechten langzaam in een echte kleine RPG.

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 30](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_30/).

