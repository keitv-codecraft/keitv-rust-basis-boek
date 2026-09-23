# 31. De spelwereld

## Wat gaan we leren?

In dit artikel bouwen we een spelwereld met verbonden locaties waarin de speler kan rondreizen en vijanden kan tegenkomen.

We leren:

- een `Locatie` struct ontwerpen met een naam, beschrijving en verbindingen
- een `Wereld` beheren met een verzameling locaties in een `Vec`
- reizen tussen locaties met veilige foutafhandeling via `Option` en `Result`
- locaties voorzien van optionele vijanden (`Option<Vijand>`)
- een verkenningslus koppelen aan de spelersinvoer.

---

## 1. Een locatie maken

We beginnen eenvoudig.

Een locatie heeft bijvoorbeeld een naam en een beschrijving.

```rust,ignore
struct Locatie {
    naam: String,
    beschrijving: String,
}
```

We kunnen een locatie maken:

```rust,ignore
let bos = Locatie {
    naam: String::from("Het Donkere Bos"),
    beschrijving: String::from("Een dicht bos waar vreemde geluiden vandaan komen."),
};
```

En de informatie tonen:

```rust,ignore
println!("{}", bos.naam);
println!("{}", bos.beschrijving);
```

Net als bij onze andere structs kunnen we hier beter methodes voor maken.

```rust,ignore
impl Locatie {
    fn nieuw(naam: &str, beschrijving: &str) -> Self {
        Self {
            naam: String::from(naam),
            beschrijving: String::from(beschrijving),
        }
    }

    fn toon(&self) {
        println!("=== {} ===", self.naam);
        println!("{}", self.beschrijving);
    }
}
```

Nu kunnen we schrijven:

```rust,ignore
let bos = Locatie::nieuw(
    "Het Donkere Bos",
    "Een dicht bos waar vreemde geluiden vandaan komen.",
);

bos.toon();
```

---

## 2. Locaties herkennen

Een RPG heeft natuurlijk verschillende soorten locaties.

We kunnen bijvoorbeeld onderscheid maken tussen:

- een dorp
- een bos
- een kerker
- een kasteel
- een winkel.

Een `enum` is hiervoor geschikt.

```rust,ignore
enum LocatieType {
    Dorp,
    Bos,
    Kerker,
    Kasteel,
    Winkel,
}
```

Een locatie kan dit type bevatten:

```rust,ignore
struct Locatie {
    naam: String,
    beschrijving: String,
    soort: LocatieType,
}
```

Bijvoorbeeld:

```rust,ignore
let bos = Locatie {
    naam: String::from("Het Donkere Bos"),
    beschrijving: String::from("Een dicht en donker bos."),
    soort: LocatieType::Bos,
};
```

We kunnen vervolgens met `match` reageren op het soort locatie:

```rust,ignore
match bos.soort {
    LocatieType::Dorp => println!("Je bent in een dorp."),
    LocatieType::Bos => println!("Je bent in een bos."),
    LocatieType::Kerker => println!("Je bent in een kerker."),
    LocatieType::Kasteel => println!("Je bent bij een kasteel."),
    LocatieType::Winkel => println!("Je bent in een winkel."),
}
```

---

## 3. Een wereld bestaat uit meerdere locaties

Een wereld bestaat uit meer dan één locatie.

We kunnen daarvoor een `Vec` gebruiken:

```rust,ignore
let wereld = vec![
    Locatie::nieuw(
        "Hasselman",
        "Een klein dorp aan de rivier.",
    ),
    Locatie::nieuw(
        "Dommeldoor",
        "Een groot en donker bos.",
    ),
    Locatie::nieuw(
        "Oude Toren",
        "Een verlaten stenen toren.",
    ),
];
```

We kunnen alle locaties tonen:

```rust,ignore
for locatie in &wereld {
    locatie.toon();
}
```

Hier gebruiken we `&wereld`, omdat we de locaties alleen willen bekijken.

We nemen het eigendom van de locaties dus niet over.

---

## 4. Een locatie kiezen

De speler moet natuurlijk een locatie kunnen kiezen.

Een handige functie is:

```rust,ignore
fn vind_locatie<'a>(
    wereld: &'a [Locatie],
    naam: &str,
) -> Option<&'a Locatie> {
    wereld
        .iter()
        .find(|locatie| locatie.naam == naam)
}
```

Deze functie zoekt een locatie.

De functie kan twee dingen opleveren:

```text
Some(&Locatie)
```

als de locatie bestaat, of:

```text
None
```

als de locatie niet bestaat.

We kunnen dat gebruiken met `match`:

```rust,ignore
match vind_locatie(&wereld, "Dommeldoor") {
    Some(locatie) => locatie.toon(),
    None => println!("Die locatie bestaat niet."),
}
```

---

## 5. Waarom geeft `find` een `Option` terug?

Dit is een belangrijk voorbeeld van waarom `Option` nuttig is.

Stel dat we dit zouden schrijven:

```rust,ignore
let locatie = vind_locatie(&wereld, "Onbekende Stad");
```

Dan bestaat die locatie misschien helemaal niet.

Rust dwingt ons daarom om rekening te houden met beide mogelijkheden:

```rust,ignore
match locatie {
    Some(locatie) => {
        println!("Gevonden:");
        locatie.toon();
    }
    None => {
        println!("Locatie niet gevonden.");
    }
}
```

Dit voorkomt dat ons programma zomaar doet alsof de locatie bestaat.

---

## 6. Verbindingen tussen locaties

Onze wereld wordt interessanter als locaties met elkaar verbonden zijn.

Een eenvoudige manier is om iedere locatie een lijst met bestemmingen te geven.

```rust,ignore
struct Locatie {
    naam: String,
    beschrijving: String,
    verbindingen: Vec<String>,
}
```

Bijvoorbeeld:

```rust,ignore
let hasselman = Locatie {
    naam: String::from("Hasselman"),
    beschrijving: String::from("Een klein dorp aan de rivier."),
    verbindingen: vec![
        String::from("Dommeldoor"),
        String::from("Oude Toren"),
    ],
};
```

Vanuit Hasselman kan de speler dus naar twee andere locaties.

We kunnen de bestemmingen tonen:

```rust,ignore
println!("Je kunt naar:");

for bestemming in &hasselman.verbindingen {
    println!("- {}", bestemming);
}
```

---

## 7. Waarom gebruiken we namen?

Je zou de verbindingen ook kunnen opslaan als getallen:

```rust,ignore
verbindingen: Vec<usize>
```

Dan zou bijvoorbeeld `0` naar de eerste locatie verwijzen.

Dat kan efficiënt zijn, maar voor ons leerproject maakt het de code minder duidelijk.

Voorlopig gebruiken we daarom namen:

```rust,ignore
Vec<String>
```

Later kunnen we leren hoe grotere programma's zulke relaties op andere manieren kunnen modelleren.

---

## 8. Een locatie zoeken vanuit een naam

We kunnen nu een functie maken die controleert of een bepaalde bestemming bestaat.

```rust,ignore
fn heeft_verbinding(locatie: &Locatie, bestemming: &str) -> bool {
    locatie
        .verbindingen
        .iter()
        .any(|naam| naam == bestemming)
}
```

We gebruiken hier de iterator-adaptor `any`.

`any` vraagt:

> Voldoet minstens één element aan deze voorwaarde?

Bijvoorbeeld:

```rust,ignore
if heeft_verbinding(&hasselman, "Dommeldoor") {
    println!("Je kunt naar Dommeldoor.");
}
```

Maar:

```rust,ignore
if heeft_verbinding(&hasselman, "Kasteel") {
    println!("Je kunt naar het kasteel.");
}
```

zal niets tonen als er geen verbinding met het kasteel is.

---

## 9. De speler heeft een huidige locatie

We willen natuurlijk onthouden waar de speler zich bevindt.

Dat kan eenvoudig met een `String`:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
    goud: i32,
    huidige_locatie: String,
}
```

Bijvoorbeeld:

```rust,ignore
let speler = Speler {
    naam: String::from("Arin"),
    gezondheid: 100,
    goud: 50,
    huidige_locatie: String::from("Hasselman"),
};
```

De speler begint dus in Hasselman.

---

## 10. De speler verplaatsen

We kunnen een methode maken:

```rust,ignore
impl Speler {
    fn verplaats_naar(&mut self, locatie: &str) {
        self.huidige_locatie = String::from(locatie);
    }
}
```

Dan:

```rust,ignore
speler.verplaats_naar("Dommeldoor");
```

Maar hiermee zit een probleem.

De methode controleert niet of Dommeldoor daadwerkelijk vanuit de huidige locatie bereikbaar is.

De speler zou dus zomaar kunnen schrijven:

```rust,ignore
speler.verplaats_naar("De Maan");
```

Dat willen we niet.

---

## 11. Verplaatsen moet kunnen mislukken

Hier komt `Result` goed van pas.

Onze functie kan bijvoorbeeld teruggeven:

```rust,ignore
Result<(), String>
```

Dat betekent:

- `Ok(())` → verplaatsen is gelukt
- `Err(...)` → verplaatsen is niet gelukt.

Bijvoorbeeld:

```rust,ignore
fn verplaats(
    speler: &mut Speler,
    bestemming: &str,
    wereld: &[Locatie],
) -> Result<(), String> {
    let huidige = wereld
        .iter()
        .find(|locatie| locatie.naam == speler.huidige_locatie)
        .ok_or(String::from("Huidige locatie bestaat niet."))?;

    if !heeft_verbinding(huidige, bestemming) {
        return Err(format!(
            "Je kunt vanuit {} niet naar {}.",
            huidige.naam,
            bestemming
        ));
    }

    speler.huidige_locatie = String::from(bestemming);

    Ok(())
}
```

Dit voorbeeld combineert meerdere dingen die we al kennen:

- `iter()`
- `find()`
- `Option`
- `ok_or()`
- `?`
- `Result`
- `format!`
- mutable borrowing
- `String`

Dat is precies wat we in een groter Rust-project steeds meer gaan doen: **bestaande onderdelen combineren**.

---

## 12. `ok_or`

We hebben hier iets nieuws gezien:

```rust,ignore
.ok_or(String::from("Huidige locatie bestaat niet."))?
```

We hadden:

```rust,ignore
Option<&Locatie>
```

maar onze functie moet een `Result` teruggeven.

`ok_or` zet daarom:

```text
Some(waarde)
```

om naar:

```text
Ok(waarde)
```

en:

```text
None
```

naar:

```text
Err(fout)
```

Daarna kan `?` de fout automatisch doorgeven.

Dit is vooral handig wanneer een functie verschillende mogelijke fouten heeft.

---

## 13. Verplaatsen gebruiken

We kunnen nu schrijven:

```rust,ignore
match verplaats(&mut speler, "Dommeldoor", &wereld) {
    Ok(()) => println!("Je bent naar Dommeldoor gegaan."),
    Err(fout) => println!("Je kunt niet reizen: {}", fout),
}
```

Als Dommeldoor bereikbaar is:

```text
Je bent naar Dommeldoor gegaan.
```

Als dat niet zo is:

```text
Je kunt niet reizen: Je kunt vanuit Hasselman niet naar Dommeldoor.
```

---

## 14. Ontmoetingen

Nu wordt de wereld interessant.

Een locatie kan bijvoorbeeld een vijand bevatten.

Maar misschien is er helemaal geen vijand.

Dat is opnieuw een goede situatie voor `Option`.

```rust,ignore
struct Locatie {
    naam: String,
    beschrijving: String,
    verbindingen: Vec<String>,
    vijand: Option<Vijand>,
}
```

Een bos kan bijvoorbeeld een vijand hebben:

```rust,ignore
let wolf = Vijand::nieuw("Wolf", 30);

let bos = Locatie {
    naam: String::from("Dommeldoor"),
    beschrijving: String::from("Een donker bos."),
    verbindingen: vec![
        String::from("Hasselman"),
    ],
    vijand: Some(wolf),
};
```

Een dorp heeft misschien geen vijand:

```rust,ignore
let dorp = Locatie {
    naam: String::from("Hasselman"),
    beschrijving: String::from("Een rustig dorp."),
    verbindingen: vec![
        String::from("Dommeldoor"),
    ],
    vijand: None,
};
```

---

## 15. Een vijand bekijken

We kunnen controleren of er een vijand aanwezig is:

```rust,ignore
match &bos.vijand {
    Some(vijand) => {
        println!("Je ziet een vijand!");
        vijand.toon();
    }
    None => {
        println!("Er is hier geen vijand.");
    }
}
```

Let op de `&`:

```rust,ignore
match &bos.vijand
```

We willen de vijand alleen bekijken. We willen hem niet uit de locatie halen.

Dit is een voorbeeld waarbij ownership en borrowing belangrijk worden.

---

## 16. Een vijand verslaan

Stel dat de speler de vijand verslaat.

Dan moeten we de vijand uit de locatie kunnen verwijderen.

Een handige methode is `take()`.

```rust,ignore
if let Some(vijand) = bos.vijand.take() {
    println!("Je hebt {} verslagen!", vijand.naam());
}
```

`take()` doet iets belangrijks:

- de waarde uit de `Option` wordt gehaald
- de oorspronkelijke `Option` wordt `None`
- de waarde wordt aan ons teruggegeven.

Dus:

```text
Some(vijand)
```

wordt:

```text
None
```

en wij krijgen de `Vijand` in handen.

Dit past goed bij een RPG:

```text
Voor het gevecht:
vijand = Some(Wolf)

Na het verslaan:
vijand = None
```

---

## 17. Let op: hiervoor moet de locatie mutable zijn

Omdat `take()` de waarde verandert, hebben we een mutable locatie nodig.

Dus bijvoorbeeld:

```rust,ignore
let mut bos = Locatie {
    // ...
};
```

Daarna:

```rust,ignore
if let Some(vijand) = bos.vijand.take() {
    println!("Je hebt {} verslagen!", vijand.naam());
}
```

De compiler zou ons waarschuwen als we vergeten `mut` te gebruiken.

Dit is precies het soort fout waar Rust ons mee helpt:

> Je probeert iets te veranderen, maar je hebt geen toestemming om het te veranderen.

---

## 18. Willekeurige ontmoetingen

We kunnen later ook willekeurige ontmoetingen toevoegen.

Daarvoor kunnen we de `rand` crate gebruiken die we eerder al hebben gebruikt.

Bijvoorbeeld:

```rust,ignore
let worp = rand::random_range(1..=6);
```

We kunnen daarmee bepalen wat er gebeurt:

```rust,ignore
match worp {
    1 => println!("Je vindt niets."),
    2 => println!("Je vindt een potion."),
    3 => println!("Je vindt goud."),
    4 => println!("Een vijand verschijnt!"),
    5 => println!("Je ontdekt een geheim pad."),
    6 => println!("Je vindt een zeldzaam wapen."),
    _ => unreachable!(),
}
```

De `_` betekent hier:

> Alle andere gevallen.

Omdat we alleen 1 tot en met 6 kunnen gooien, kunnen we die laatste situatie eigenlijk niet bereiken.

---

## 19. Een ontmoeting als enum

Een nog betere manier is om de uitkomst van een ontmoeting als een `enum` te modelleren.

```rust,ignore
enum Ontmoeting {
    Niets,
    Vijand,
    Schat,
    Wapen,
    GeheimPad,
}
```

Een functie kan dan een ontmoeting teruggeven:

```rust,ignore
fn willekeurige_ontmoeting() -> Ontmoeting {
    let worp = rand::random_range(1..=5);

    match worp {
        1 => Ontmoeting::Niets,
        2 => Ontmoeting::Vijand,
        3 => Ontmoeting::Schat,
        4 => Ontmoeting::Wapen,
        5 => Ontmoeting::GeheimPad,
        _ => unreachable!(),
    }
}
```

We kunnen vervolgens:

```rust,ignore
match willekeurige_ontmoeting() {
    Ontmoeting::Niets => println!("Er gebeurt niets."),
    Ontmoeting::Vijand => println!("Een vijand verschijnt!"),
    Ontmoeting::Schat => println!("Je vindt een schat!"),
    Ontmoeting::Wapen => println!("Je vindt een wapen!"),
    Ontmoeting::GeheimPad => println!("Je ontdekt een geheim pad!"),
}
```

Dit is overzichtelijker dan allerlei losse `bool`-waarden.

---

## 20. Een wereld heeft toestand

Tot nu toe hebben we losse gegevens gebruikt.

Een echte RPG heeft echter een toestand die tijdens het spelen verandert.

Bijvoorbeeld:

```rust,ignore
struct Wereld {
    locaties: Vec<Locatie>,
}
```

We kunnen daar later meer informatie aan toevoegen:

```rust,ignore
struct Wereld {
    locaties: Vec<Locatie>,
    dag: u32,
}
```

Of:

```rust,ignore
struct Wereld {
    locaties: Vec<Locatie>,
    dag: u32,
    gebeurtenissen: Vec<String>,
}
```

Een speler heeft ondertussen zijn eigen toestand:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
    goud: i32,
    huidige_locatie: String,
    // ...
}
```

De wereld en de speler zijn dus verschillende onderdelen van het spel.

---

## 21. De spelwereld als module

Ons project begint inmiddels behoorlijk groot te worden.

We voegen daarom een nieuwe module toe:

```text
src/
├── main.rs
├── spel.rs
├── speler.rs
├── vijand.rs
├── gevecht.rs
├── aanvaller.rs
├── wapen.rs
├── inventaris.rs
├── winkel.rs
└── wereld.rs
```

In `main.rs`:

```rust,ignore
mod wereld;
```

In `wereld.rs` kunnen we bijvoorbeeld zetten:

```rust,ignore
pub struct Wereld {
    locaties: Vec<Locatie>,
}
```

en:

```rust,ignore
pub struct Locatie {
    naam: String,
    beschrijving: String,
    verbindingen: Vec<String>,
    vijand: Option<Vijand>,
}
```

Daarmee houden we de details van de wereld bij elkaar.

---

## 22. De verantwoordelijkheden verdelen

Een belangrijk principe bij grotere programma's is:

> Laat ieder onderdeel vooral verantwoordelijk zijn voor zijn eigen taak.

Bijvoorbeeld:

### `speler.rs`

Bevat informatie en gedrag van de speler:

```text
gezondheid
goud
wapen
inventaris
huidige locatie
```

### `vijand.rs`

Bevat vijanden.

### `wapen.rs`

Bevat wapens.

### `inventaris.rs`

Beheert items.

### `winkel.rs`

Beheert winkels.

### `wereld.rs`

Beheert locaties en verbindingen.

### `gevecht.rs`

Beheert gevechten.

### `main.rs`

Start het spel en stuurt de grote stappen aan.

Dit betekent niet dat ieder bestand maar één soort code mag bevatten. Het betekent dat we proberen de verantwoordelijkheden logisch bij elkaar te houden.

---

## 23. De spel-lus

Een RPG heeft uiteindelijk een continue spel-lus.

Bijvoorbeeld:

```rust,ignore
loop {
    println!("Wat wil je doen?");

    // keuze van speler

    // actie uitvoeren

    // toestand aanpassen

    // resultaat tonen
}
```

De speler kan bijvoorbeeld kiezen:

```text
1. Verken locatie
2. Reis
3. Bekijk inventaris
4. Bekijk karakter
5. Stop
```

We hebben alle belangrijke onderdelen hiervoor inmiddels gezien.

Een mogelijke structuur is:

```rust,ignore
loop {
    toon_menu();

    let keuze = lees_keuze();

    match keuze {
        1 => verken(&mut speler, &mut wereld),
        2 => reis(&mut speler, &wereld),
        3 => speler.toon_inventaris(),
        4 => speler.toon(),
        5 => break,
        _ => println!("Ongeldige keuze."),
    }
}
```

De functies `toon_menu`, `lees_keuze`, `verken` en `reis` kunnen we vervolgens afzonderlijk implementeren.

---

## 24. De gebruiker invoer laten geven

Voor een echte spel-lus moeten we invoer van de gebruiker kunnen lezen.

Daarvoor gebruiken we `std::io`.

```rust,ignore
use std::io;
```

Een eenvoudige functie:

```rust,ignore
fn lees_tekst() -> String {
    let mut invoer = String::new();

    io::stdin()
        .read_line(&mut invoer)
        .expect("Kon invoer niet lezen.");

    invoer.trim().to_string()
}
```

Hier zien we verschillende bekende concepten tegelijk:

- `String::new()`
- mutable variabelen
- `&mut`
- `Result`
- `expect`
- `trim`
- `to_string()`

We vragen de gebruiker vervolgens:

```rust,ignore
println!("Wat wil je doen?");
let keuze = lees_tekst();
```

`keuze` bevat bijvoorbeeld:

```text
2
```

We kunnen die tekst naar een getal omzetten:

```rust,ignore
let keuze: u32 = keuze.parse().unwrap_or(0);
```

Daarna:

```rust,ignore
match keuze {
    1 => println!("Je gaat verkennen."),
    2 => println!("Je gaat reizen."),
    3 => println!("Je bekijkt je inventaris."),
    4 => println!("Je bekijkt je karakter."),
    5 => break,
    _ => println!("Ongeldige keuze."),
}
```

---

## 25. Waarom `unwrap_or(0)`?

`parse()` kan mislukken.

Bijvoorbeeld als de speler invoert:

```text
appel
```

Dan kan Rust daar geen `u32` van maken.

`parse()` geeft daarom een `Result` terug.

Met:

```rust,ignore
.unwrap_or(0)
```

zeggen we:

> Als het lukt, gebruik het getal. Als het mislukt, gebruik 0.

Daarna valt de invoer automatisch onder:

```rust,ignore
_ => println!("Ongeldige keuze."),
```

Dit is voor een eenvoudige spelinterface prima.

Later kunnen we de invoer netter afhandelen.

---

## 26. Alles samenbrengen

Een eenvoudige versie van onze RPG kan nu ongeveer dit doen:

```text
=== RPG ===

Je bent in Hasselman.

1. Verken
2. Reis
3. Inventaris
4. Karakter
5. Stop

> 2

Waar wil je naartoe?

1. Dommeldoor
2. Oude Toren

> 1

Je reist naar Dommeldoor.

Je staat aan de rand van een donker bos.

Een wolf verschijnt!

Je gezondheid: 100
Wolf gezondheid: 30

1. Aanvallen
2. Vluchten
```

Dit is geen nieuw concept.

Het is het resultaat van het combineren van de concepten die we al hebben geleerd.

---

## 27. Opzettelijke compilerfouten

Bij enkele oefeningen moet de code expres niet compileren.

Bijvoorbeeld:

```rust,ignore
let bos = Locatie {
    // ...
};

bos.vijand.take();
```

De compiler zal hier klagen omdat `take()` de waarde verandert en `bos` niet mutable is.

Verander:

```rust,ignore
let bos
```

in:

```rust,ignore
let mut bos
```

en probeer opnieuw.

Dit soort fouten zijn geen mislukking.

Ze leren je juist wat Rust van je programma verwacht.

---

## 28. Zelfstandig project

Maak een kleine wereld met minimaal vijf locaties.

Bijvoorbeeld:

```text
Hasselman
   |
   +--- Dommeldoor
   |       |
   |       +--- Oude Toren
   |
   +--- Rivierhaven
           |
           +--- Kasteel
```

De speler begint in Hasselman.

Elke locatie heeft:

- een naam
- een beschrijving
- minimaal één verbinding
- eventueel een vijand.

De speler moet:

1. zijn huidige locatie kunnen bekijken
2. beschikbare bestemmingen kunnen bekijken
3. naar een bestemming kunnen reizen
4. een foutmelding krijgen bij een ongeldige bestemming
5. vijanden kunnen tegenkomen
6. een verslagen vijand uit de locatie kunnen verwijderen.

---

## 29. Eindopdracht — De eerste speelbare wereld

Bouw nu een eerste versie van de RPG waarin de speler daadwerkelijk kan spelen.

De minimale eisen zijn:

### Wereld

- minimaal vijf locaties
- iedere locatie heeft minimaal één verbinding
- de wereld wordt opgeslagen in een `Vec<Locatie>`.

### Speler

De speler heeft:

- een naam
- gezondheid
- goud
- inventaris
- wapen
- huidige locatie.

### Reizen

De speler kan:

- de huidige locatie bekijken
- beschikbare bestemmingen bekijken
- een bestemming kiezen
- alleen naar verbonden locaties reizen.

### Ontmoetingen

Een locatie kan:

```rust,ignore
Option<Vijand>
```

bevatten.

Een vijand kan worden verslagen en daarna uit de locatie verdwijnen.

### Interface

Het spel heeft minimaal:

```text
1. Verken
2. Reis
3. Inventaris
4. Karakter
5. Stop
```

De hoofdloop gebruikt:

```rust,ignore
loop {
    // menu
    // invoer
    // actie
}
```

### Tests

Schrijf minimaal tests voor:

- het vinden van een locatie
- het controleren van een verbinding
- een geldige verplaatsing
- een ongeldige verplaatsing
- een ontmoeting
- het verwijderen van een verslagen vijand.

---

## 30. Wat je hiermee hebt geleerd

In dit hoofdstuk hebben we vooral geleerd hoe bestaande Rust-onderdelen samenkomen in een groter programma.

We hebben onder andere gebruikt:

```text
Vec
Option
Result
enum
struct
match
iter()
find()
any()
for
& en &mut
take()
ok_or()
?
modules
```

Belangrijker nog: we hebben een eerste stap gezet van **losse programmeeroefeningen naar een echt programma**.

Een RPG bestaat niet uit één groot stuk code. Het bestaat uit kleinere onderdelen die samenwerken:

```text
                 ┌─────────────┐
                 │    main     │
                 └──────┬──────┘
                        │
              ┌─────────┴─────────┐
              │                   │
          ┌───▼───┐          ┌────▼────┐
          │ Speler│          │  Wereld  │
          └───┬───┘          └────┬─────┘
              │                   │
       ┌──────┼──────┐       ┌────┴─────┐
       │      │      │       │          │
    Wapen Inventaris Gevecht Locaties Ontmoetingen
```

Vanaf dit punt wordt het project steeds meer een echt spel.

De volgende stap is daarom niet simpelweg nóg een Rust-concept, maar het **uitwerken van de gameplay zelf**: keuzes, gevechten, gebeurtenissen, winnen en verliezen en uiteindelijk een volledige spelronde.

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 31](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_31/).

---

## Controlelijst

Je bent klaar met dit artikel als je zonder hulp:

- [ ] een `Locatie` struct kunt maken met eigenschappen en verbindingen
- [ ] locaties kunt doorzoeken met iterators zoals `find()`
- [ ] de speler veilig kunt laten reizen naar geldige bestemmingen
- [ ] een locatie kunt inspecteren op de aanwezigheid van vijanden en interactie kunt starten.
