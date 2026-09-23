# 14. Derive-attributen

## Wat gaan we leren?

We hebben inmiddels gezien dat Rust soms extra informatie nodig heeft over een type.

Bijvoorbeeld in het vorige artikel:

```rust,ignore
#[derive(PartialEq)]
enum Wapen {
    Zwaard,
    Boog,
    Staf,
}
```

Daar stond iets voor wat misschien een beetje vreemd leek:

```rust,ignore
#[derive(PartialEq)]
```

Wat doet dat eigenlijk?

`derive` kan Rust vertellen:

> Maak bepaalde standaardfunctionaliteit automatisch voor dit type.

Dat kan veel programmeerwerk schelen.

In dit artikel leren we een paar belangrijke derives:

- `Debug`
- `Clone`
- `Copy`
- `PartialEq`
- `Eq`
- `Default`

We behandelen niet alle mogelijke derives. Het doel is dat je begrijpt **waarom** `derive` bestaat en wanneer je het kunt gebruiken.

---

## 1. Eerst een probleem

Stel dat we deze struct hebben:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
}
```

We willen misschien de inhoud van een `Speler` bekijken tijdens het programmeren.

We zouden kunnen schrijven:

```rust,ignore
println!("Naam: {}", speler.naam);
println!("Gezondheid: {}", speler.gezondheid);
```

Maar soms willen we gewoon zeggen:

> Laat mij deze hele waarde zien.

Daarvoor bestaat `Debug`.

We kunnen schrijven:

```rust,ignore
#[derive(Debug)]
struct Speler {
    naam: String,
    gezondheid: i32,
}
```

Daarna kunnen we:

```rust,ignore
println!("{:?}", speler);
```

gebruiken.

Rust kan dan automatisch een debugweergave van de struct maken.

---

## 2. `Debug`

`Debug` is vooral bedoeld om tijdens het programmeren te kijken wat er in een waarde zit.

Bijvoorbeeld:

```rust,ignore
#[derive(Debug)]
struct Speler {
    naam: String,
    gezondheid: i32,
}
```

Dan:

```rust,ignore
fn main() {
    let speler = Speler {
        naam: String::from("Arin"),
        gezondheid: 100,
    };

    println!("{:?}", speler);
}
```

Je krijgt dan ongeveer:

```text
Speler { naam: "Arin", gezondheid: 100 }
```

De precieze opmaak is door Rust bepaald.

Je hoeft dus niet zelf een speciale `println!` te schrijven voor ieder veld.

---

## 3. `{:?}`

Je hebt eerder gezien:

```rust,ignore
println!("{}", waarde);
```

Voor `Debug` gebruiken we:

```rust,ignore
println!("{:?}", waarde);
```

Het verschil is:

```text
{}    → gewone weergave
{:?}  → debugweergave
```

Je kunt ook:

```rust,ignore
println!("{:#?}", speler);
```

gebruiken.

Dat geeft een uitgebreidere, ingesprongen debugweergave.

Bijvoorbeeld ongeveer:

```text
Speler {
    naam: "Arin",
    gezondheid: 100,
}
```

Dat is vooral handig wanneer een struct veel velden heeft.

---

## 4. Waarom is `Debug` handig?

Stel dat je game opeens vreemd gedrag vertoont.

Je hebt:

```rust,ignore
let mut speler = Speler {
    naam: String::from("Arin"),
    gezondheid: 100,
};
```

Er gebeurt iets:

```rust,ignore
speler.gezondheid -= 30;
```

Je vraagt je af:

> Wat zit er nu precies in `speler`?

Met:

```rust,ignore
println!("{:?}", speler);
```

kun je dat snel bekijken.

Daarom is:

```rust,ignore
#[derive(Debug)]
```

een handige gewoonte voor structs die je tijdens het programmeren wilt kunnen onderzoeken.

---

## 5. `Clone`

We hebben in het artikel over ownership al `clone()` gezien.

Bijvoorbeeld:

```rust,ignore
let naam = String::from("Arin");
let andere_naam = naam.clone();
```

Maar waarom kunnen we `clone()` bij sommige types gebruiken en bij andere niet?

Een type moet aangeven dat het gekopieerd kan worden.

Bijvoorbeeld:

```rust,ignore
#[derive(Clone)]
struct Wapen {
    naam: String,
    schade: i32,
}
```

Nu kunnen we:

```rust,ignore
let zwaard = Wapen {
    naam: String::from("IJzeren zwaard"),
    schade: 20,
};

let ander_zwaard = zwaard.clone();
```

We hebben nu twee afzonderlijke waarden.

---

## 6. Wat gebeurt er bij `clone()`?

Stel:

```rust,ignore
let zwaard = Wapen {
    naam: String::from("IJzeren zwaard"),
    schade: 20,
};

let kopie = zwaard.clone();
```

Dan blijven beide bruikbaar:

```rust,ignore
println!("{}", zwaard.naam);
println!("{}", kopie.naam);
```

De `clone()` maakt bewust een nieuwe waarde.

Dat is anders dan een gewone move:

```rust,ignore
let ander_zwaard = zwaard;
```

Na die regel is `zwaard` niet meer bruikbaar.

Bij:

```rust,ignore
let ander_zwaard = zwaard.clone();
```

blijven beide bruikbaar.

---

## 7. `Clone` moet door de onderdelen ondersteund worden

Kijk naar:

```rust,ignore
#[derive(Clone)]
struct Wapen {
    naam: String,
    schade: i32,
}
```

Rust kan automatisch `Clone` maken omdat:

- `String` clonebaar is
- `i32` clonebaar is.

Rust kan dus voor de hele struct een `clone()` maken.

Als een struct een veld bevat waarvoor geen `Clone` bestaat, kan Rust niet zomaar een `Clone` voor de hele struct maken.

Voor nu hoef je vooral te onthouden:

> `derive(Clone)` laat Rust automatisch een `clone()`-implementatie voor je maken, zolang de onderdelen dat ondersteunen.

---

## 8. `Copy`

We hebben bij ownership al gezien dat sommige eenvoudige waarden gekopieerd worden:

```rust,ignore
let score = 100;
let andere_score = score;

println!("{score}");
```

Dit werkt omdat `i32` `Copy` is.

We kunnen `Copy` ook voor onze eigen eenvoudige structs gebruiken.

Bijvoorbeeld:

```rust,ignore
#[derive(Copy, Clone)]
struct Positie {
    x: i32,
    y: i32,
}
```

Nu:

```rust,ignore
let positie1 = Positie {
    x: 10,
    y: 20,
};

let positie2 = positie1;

println!("{}", positie1.x);
```

Dit werkt.

`positie1` is niet verplaatst op dezelfde manier als een `String`.

---

## 9. Waarom staat daar ook `Clone`?

Je ziet vaak:

```rust,ignore
#[derive(Copy, Clone)]
```

Dat komt doordat `Copy` en `Clone` bij elkaar horen.

Een type dat `Copy` is, moet ook `Clone` zijn.

Dus bij een eenvoudige struct schrijven we vaak:

```rust,ignore
#[derive(Copy, Clone)]
struct Positie {
    x: i32,
    y: i32,
}
```

Een `Positie` bestaat alleen uit `i32`-waarden.

Daarom kan Rust hem eenvoudig kopiëren.

---

## 10. `String` en `Copy`

Dit werkt niet:

```rust,ignore
#[derive(Copy, Clone)]
struct Speler {
    naam: String,
}
```

Waarom?

Omdat `String` geen `Copy` is.

Dat is logisch als je terugdenkt aan ownership.

Een `String` heeft eigen beheerde gegevens. Een simpele kopie van de struct zou niet zomaar betekenen dat er een tweede zelfstandige `String` ontstaat.

`Clone` kan dat wel bewust doen:

```rust,ignore
#[derive(Clone)]
struct Speler {
    naam: String,
}
```

Dus:

```rust,ignore
let speler2 = speler1.clone();
```

kan wel.

---

## 11. `PartialEq`

We hebben deze al gezien.

Met:

```rust,ignore
#[derive(PartialEq)]
```

kunnen we waarden vergelijken met:

```rust,ignore
==
```

en:

```rust,ignore
!=
```

Bijvoorbeeld:

```rust,ignore
#[derive(PartialEq)]
enum Wapen {
    Zwaard,
    Boog,
    Staf,
}
```

Nu:

```rust,ignore
let wapen = Wapen::Zwaard;

if wapen == Wapen::Zwaard {
    println!("Je hebt een zwaard.");
}
```

Zonder `PartialEq` weet Rust niet automatisch hoe het twee `Wapen`-waarden met elkaar moet vergelijken.

---

## 12. `PartialEq` bij structs

`PartialEq` werkt ook bij structs.

Bijvoorbeeld:

```rust,ignore
#[derive(PartialEq)]
struct Positie {
    x: i32,
    y: i32,
}
```

Dan:

```rust,ignore
let positie1 = Positie {
    x: 10,
    y: 20,
};

let positie2 = Positie {
    x: 10,
    y: 20,
};

if positie1 == positie2 {
    println!("De posities zijn hetzelfde.");
}
```

Rust vergelijkt de velden van de struct.

Hier zijn beide:

```text
x = 10
y = 20
```

dus zijn ze gelijk.

---

## 13. `Eq`

Je kunt ook:

```rust,ignore
#[derive(PartialEq, Eq)]
```

zien.

Voor de voorbeelden die we nu gebruiken, kun je `Eq` zien als een uitgebreidere vorm van gelijkheid.

Bij veel eenvoudige types kun je gewoon:

```rust,ignore
#[derive(PartialEq, Eq)]
```

gebruiken.

Voor onze lessen is vooral belangrijk:

```rust,ignore
PartialEq
```

maakt `==` en `!=` mogelijk.

`Eq` komen we later vanzelf weer tegen wanneer we complexere Rust-code behandelen.

Je hoeft de precieze theoretische verschillen tussen `PartialEq` en `Eq` nu nog niet te kennen.

---

## 14. `Default`

Soms willen we een standaardwaarde voor een type.

Bijvoorbeeld een nieuwe speler:

```text
naam: "Onbekend"
gezondheid: 100
goud: 0
```

We kunnen dat zelf programmeren:

```rust,ignore
impl Default for Speler {
    fn default() -> Self {
        Speler {
            naam: String::from("Onbekend"),
            gezondheid: 100,
            goud: 0,
        }
    }
}
```

Maar voor eenvoudige situaties kan Rust dit automatisch doen als de velden zelf een standaardwaarde hebben.

We kunnen bijvoorbeeld:

```rust,ignore
#[derive(Default)]
struct Instellingen {
    geluid: bool,
    volume: i32,
}
```

Dan:

```rust,ignore
let instellingen = Instellingen::default();
```

De standaardwaarden zijn:

```text
geluid = false
volume = 0
```

Dat komt doordat `bool` en `i32` hun eigen standaardwaarde hebben.

---

## 15. Een eigen `Default`-implementatie

Voor een game willen we misschien andere standaardwaarden.

Bijvoorbeeld:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
    goud: i32,
}
```

Dan kunnen we zelf bepalen wat `default()` betekent:

```rust,ignore
impl Default for Speler {
    fn default() -> Self {
        Speler {
            naam: String::from("Nieuwe speler"),
            gezondheid: 100,
            goud: 0,
        }
    }
}
```

Nu:

```rust,ignore
let speler = Speler::default();
```

maakt automatisch:

```text
Naam: Nieuwe speler
Gezondheid: 100
Goud: 0
```

Dit is geen `derive`, maar het is belangrijk om het verschil te zien:

```text
#[derive(Default)]
```

laat Rust de standaardimplementatie automatisch maken.

```text
impl Default for Speler
```

schrijft zelf hoe de standaardwaarde eruitziet.

---

## 16. Meerdere derives tegelijk

Je kunt meerdere derives combineren.

Bijvoorbeeld:

```rust,ignore
#[derive(Debug, Clone, PartialEq)]
struct Wapen {
    naam: String,
    schade: i32,
}
```

Nu kan een `Wapen`:

- met `{:?}` worden afgedrukt
- worden gecloned
- worden vergeleken met `==`.

Bijvoorbeeld:

```rust,ignore
let zwaard = Wapen {
    naam: String::from("Zwaard"),
    schade: 20,
};

let kopie = zwaard.clone();

println!("{:?}", zwaard);

if zwaard == kopie {
    println!("De wapens zijn gelijk.");
}
```

Dit is heel gebruikelijk in Rust.

---

## 17. Een enum met meerdere derives

Ook enums kunnen derives gebruiken.

Bijvoorbeeld:

```rust,ignore
#[derive(Debug, Clone, PartialEq)]
enum Wapen {
    Zwaard,
    Boog,
    Staf,
}
```

Nu kunnen we:

```rust,ignore
let wapen = Wapen::Zwaard;
```

debuggen:

```rust,ignore
println!("{:?}", wapen);
```

vergelijken:

```rust,ignore
if wapen == Wapen::Zwaard {
    println!("Zwaard!");
}
```

en clonen:

```rust,ignore
let ander_wapen = wapen.clone();
```

---

## 18. Een praktisch gamevoorbeeld

We maken een `Item`:

```rust,ignore
#[derive(Debug, Clone, PartialEq)]
struct Item {
    naam: String,
    waarde: i32,
}
```

We maken twee items:

```rust,ignore
let zwaard = Item {
    naam: String::from("Zwaard"),
    waarde: 100,
};

let ander_zwaard = Item {
    naam: String::from("Zwaard"),
    waarde: 100,
};
```

We kunnen ze vergelijken:

```rust,ignore
if zwaard == ander_zwaard {
    println!("Deze items zijn gelijk.");
}
```

We kunnen ze bekijken:

```rust,ignore
println!("{:?}", zwaard);
```

En we kunnen een kopie maken:

```rust,ignore
let kopie = zwaard.clone();
```

Eén korte regel:

```rust,ignore
#[derive(Debug, Clone, PartialEq)]
```

heeft dus behoorlijk veel functionaliteit toegevoegd.

---

## 19. Derive en compilerfouten

`derive` is geen magie die alles kan.

Stel:

```rust,ignore
#[derive(Clone)]
struct Speler {
    naam: String,
    gezondheid: i32,
}
```

Dat werkt.

Maar als Rust een type tegenkomt waarvan `Clone` niet beschikbaar is, kan de derive mislukken.

De compiler zal dan vertellen welk onderdeel het probleem veroorzaakt.

Dat is opnieuw een voorbeeld van iets belangrijks:

> De compiler helpt je begrijpen wat je programma nodig heeft.

---

## 20. Waarom bestaat `derive`?

Zonder `derive` zouden we sommige dingen zelf moeten schrijven.

Bijvoorbeeld een vergelijking:

```rust,ignore
impl PartialEq for Wapen {
    // ingewikkelde implementatie
}
```

Of een debugweergave:

```rust,ignore
impl Debug for Wapen {
    // ingewikkelde implementatie
}
```

Voor veel standaardgedrag is dat helemaal niet nodig.

Rust kan het voor je maken:

```rust,ignore
#[derive(Debug, PartialEq)]
```

Je zegt daarmee eigenlijk:

> Rust, maak de standaardimplementatie hiervoor automatisch.

Dat bespaart veel code.

---

## 21. Zelf proberen

### Opdracht 1 — Debug

Maak een struct:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
}
```

Voeg `Debug` toe.

Print de speler met:

```rust,ignore
println!("{:?}", speler);
```

---

### Opdracht 2 — Mooie debugweergave

Gebruik:

```rust,ignore
println!("{:#?}", speler);
```

Kijk naar het verschil.

---

### Opdracht 3 — Clone

Maak een `Wapen`.

Voeg `Clone` toe.

Maak daarna een kopie met:

```rust,ignore
let kopie = wapen.clone();
```

---

### Opdracht 4 — Vergelijken

Maak een `Wapen` met `PartialEq`.

Maak twee gelijke wapens.

Controleer met:

```rust,ignore
if wapen1 == wapen2 {
    println!("Gelijk!");
}
```

---

### Opdracht 5 — Positie

Maak:

```rust,ignore
#[derive(Debug, Copy, Clone, PartialEq)]
struct Positie {
    x: i32,
    y: i32,
}
```

Maak twee posities.

Vergelijk ze.

Kopieer er één.

Print ze met `Debug`.

---

### Opdracht 6 — Game item

Maak:

```rust,ignore
#[derive(Debug, Clone, PartialEq)]
struct Item {
    naam: String,
    waarde: i32,
}
```

Maak twee items en test alle drie:

```text
Debug
Clone
PartialEq
```

---

## 22. Kleine eindopdracht — een game-item

We bouwen verder aan onze RPG.

Maak:

```rust,ignore
#[derive(Debug, Clone, PartialEq)]
struct Wapen {
    naam: String,
    schade: i32,
    waarde: i32,
}
```

Maak bijvoorbeeld:

```text
IJzeren zwaard
Schade: 20
Waarde: 100
```

Gebruik vervolgens:

### Debug

Print het volledige wapen.

### Clone

Maak een kopie.

### PartialEq

Maak nog een wapen met dezelfde gegevens en controleer of ze gelijk zijn.

Breid het daarna uit met:

```rust,ignore
#[derive(Debug, Clone, PartialEq)]
enum Klasse {
    Krijger,
    Tovenaar,
    Boogschutter,
}
```

Je hebt dan een kleine verzameling gameobjecten waarop je automatisch verschillende nuttige bewerkingen kunt uitvoeren.

---

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 14](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_14/).

---

## Controlelijst

Je bent klaar met dit artikel als je zonder hulp:

- [ ] weet wat het `#[derive(...)]` attribuut doet
- [ ] `Debug` kunt toevoegen en structs met `{:?}` kunt printen
- [ ] weet wat `Clone` en `Copy` doen en waarom `String` geen `Copy` kan zijn
- [ ] `PartialEq` kunt gebruiken om structs te vergelijken met `==`
- [ ] `Default` kunt gebruiken om standaardwaarden in te stellen.
