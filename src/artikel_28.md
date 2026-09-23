# 28. Een RPG ontwerpen

## Wat gaan we leren?

In dit artikel leren we hoe we een groter softwareproject — onze eigen command-line RPG — methodisch ontwerpen voordat we beginnen met bouwen.

We leren:

- de entiteiten van het spel in kaart brengen (`Speler`, `Vijand`, `Wapen`, `Inventaris`)
- datastructuren en verantwoordelijkheden definiëren
- eigenaarschap (ownership) van gegevens vastleggen
- een modulaire architectuur opzetten met een centrale `SpelStatus`
- een stapsgewijze ontwikkelstrategie hanteren: klein beginnen, testen en geleidelijk uitbreiden.

---

## 1. Wat gaan we bouwen?

Ons spel wordt een eenvoudig tekstgebaseerd RPG.

De speler:

- heeft een naam
- heeft gezondheid
- heeft goud
- heeft een inventaris
- kan wapens gebruiken
- kan vijanden tegenkomen
- kan vechten
- kan items kopen
- kan schade oplopen
- kan sterven
- kan een spel opslaan
- kan een opgeslagen spel laden.

Een mogelijke speelsessie ziet er ongeveer zo uit:

```text
=== AVONTUUR ===

Welkom, Arin!

Gezondheid: 100
Goud: 50

Wat wil je doen?

1. Verken de wereld
2. Bekijk inventaris
3. Winkel
4. Opslaan
5. Stoppen

> 1

Je loopt door het bos...

Een Goblin verschijnt!

Gezondheid Goblin: 30

1. Aanvallen
2. Weglopen

> 1

Je valt aan met je Zwaard.
De Goblin krijgt 15 schade.

De Goblin valt terug aan.
Je krijgt 7 schade.

Je gezondheid: 93
```

Dit hoeft nog geen groot spel te zijn. Het doel van dit project is vooral dat we leren hoe verschillende Rust-concepten samenwerken.

---

## 2. Eerst nadenken: welke dingen bestaan er?

Een handige eerste vraag is:

> Welke dingen bestaan er in ons spel?

We kunnen bijvoorbeeld deze lijst maken:

```text
Speler
Vijand
Wapen
Inventaris
Winkel
Gevecht
Spel
Savegame
```

Niet alles hoeft direct een `struct` te worden.

Sommige dingen zijn gegevens:

```text
Speler
Vijand
Wapen
Inventaris
```

Andere dingen beschrijven gedrag of logica:

```text
Gevecht
Savegame
Spel
```

Dit onderscheid helpt ons om het programma overzichtelijk te houden.

---

## 3. De speler

De speler heeft verschillende eigenschappen.

Bijvoorbeeld:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
    maximaal_gezondheid: i32,
    goud: i32,
    inventaris: Inventaris,
}
```

De speler is eigenaar van zijn inventaris.

Dat betekent dat een `Speler` een `Inventaris` bevat.

Conceptueel:

```text
Speler
├── naam
├── gezondheid
├── maximaal_gezondheid
├── goud
└── inventaris
```

Dit is een belangrijke ontwerpbeslissing.

We willen niet dat `main` zelf alle gegevens van de inventaris moet bijhouden.

De `Speler` heeft een inventaris, en de `Inventaris` zorgt vervolgens zelf voor zijn items.

---

## 4. De inventaris

De inventaris kan bijvoorbeeld een `HashMap` gebruiken:

```rust,ignore
use std::collections::HashMap;

struct Inventaris {
    items: HashMap<String, i32>,
}
```

Bijvoorbeeld:

```text
"Pocion" -> 3
"Brood"  -> 5
"Pijl"   -> 12
```

De inventaris kan vervolgens methoden krijgen zoals:

```rust,ignore
impl Inventaris {
    fn voeg_item_toe(&mut self, naam: &str, aantal: i32) {
        // ...
    }

    fn gebruik_item(&mut self, naam: &str) -> bool {
        // ...
    }

    fn toon(&self) {
        // ...
    }
}
```

De rest van het programma hoeft dan niet te weten hoe de inventaris intern werkt.

Dat is een voorbeeld van **encapsulatie**.

---

## 5. Vijanden

Een vijand lijkt in sommige opzichten op een speler.

Bijvoorbeeld:

```rust,ignore
struct Vijand {
    naam: String,
    gezondheid: i32,
    aanvalskracht: i32,
}
```

Een vijand kan bijvoorbeeld:

```rust,ignore
impl Vijand {
    fn is_levend(&self) -> bool {
        self.gezondheid > 0
    }

    fn neem_schade(&mut self, schade: i32) {
        self.gezondheid -= schade;
    }
}
```

We kunnen later bepalen welke eigenschappen en methoden precies nodig zijn.

---

## 6. Wapens

Een wapen kan bijvoorbeeld bestaan uit:

```rust,ignore
struct Wapen {
    naam: String,
    schade: i32,
    prijs: i32,
}
```

Maar er is nog een interessante vraag:

> Zijn alle wapens hetzelfde soort wapen?

Nee.

We kunnen bijvoorbeeld onderscheid maken tussen:

```text
Zwaard
Boog
Staf
Bijl
```

Daarvoor kunnen we een enum gebruiken:

```rust,ignore
enum WapenType {
    Zwaard,
    Boog,
    Staf,
    Bijl,
}
```

Een wapen kan dan bijvoorbeeld worden:

```rust,ignore
struct Wapen {
    naam: String,
    soort: WapenType,
    schade: i32,
    prijs: i32,
}
```

Hier zien we hoe verschillende Rust-concepten samenwerken:

```text
Wapen
 ├── String
 ├── WapenType
 ├── i32
 └── i32
```

---

## 7. De speler hoeft niet alles zelf te kunnen

We hebben eerder traits geleerd.

Een trait kan bijvoorbeeld beschrijven dat iets kan aanvallen:

```rust,ignore
trait Aanvaller {
    fn aanvalskracht(&self) -> i32;
}
```

We kunnen die trait implementeren voor verschillende typen:

```rust,ignore
impl Aanvaller for Speler {
    fn aanvalskracht(&self) -> i32 {
        // ...
    }
}
```

En:

```rust,ignore
impl Aanvaller for Vijand {
    fn aanvalskracht(&self) -> i32 {
        self.aanvalskracht
    }
}
```

Daardoor kunnen we code schrijven die niet specifiek hoeft te weten of het om een speler of vijand gaat.

Bijvoorbeeld:

```rust,ignore
fn toon_aanval(aanvaller: &impl Aanvaller) {
    println!(
        "Aanvalskracht: {}",
        aanvaller.aanvalskracht()
    );
}
```

We hoeven dit niet overal toe te passen.

Een belangrijk onderdeel van goed ontwerp is juist weten wanneer iets **niet** nodig is.

---

## 8. Wat hoort bij een gevecht?

Nu komen we bij een belangrijk ontwerpbesluit.

Waar moet de gevechtslogica staan?

We zouden bijvoorbeeld kunnen schrijven:

```rust,ignore
fn vecht(speler: &mut Speler, vijand: &mut Vijand) {
    // ...
}
```

Deze functie verandert beide waarden.

Dat past goed bij borrowing:

```text
Speler ──┐
         ├──> vecht()
Vijand ──┘
```

De functie krijgt tijdelijke toegang tot beide objecten.

De functie kan bijvoorbeeld:

1. bepalen hoeveel schade de speler doet
2. schade aan de vijand geven
3. controleren of de vijand nog leeft
4. de vijand laten aanvallen
5. schade aan de speler geven
6. opnieuw controleren of de speler nog leeft.

We kunnen later meer regels toevoegen.

---

## 9. Wie bezit de vijanden?

Stel dat de speler drie vijanden tegenkomt.

Dan hebben we bijvoorbeeld:

```rust,ignore
let mut vijanden: Vec<Vijand> = Vec::new();
```

De `Vec` bevat de vijanden.

Een `Spel` kan deze `Vec` bezitten:

```rust,ignore
struct Spel {
    speler: Speler,
    vijanden: Vec<Vijand>,
}
```

Conceptueel:

```text
Spel
├── Speler
│   └── Inventaris
└── Vec<Vijand>
```

Dit is een belangrijke manier om over ownership na te denken.

Vraag steeds:

> Wie is eigenaar van deze waarde?

In ons ontwerp is het antwoord bijvoorbeeld:

```text
Spel → bezit Speler
Spel → bezit Vec<Vijand>
Speler → bezit Inventaris
Inventaris → bezit HashMap
```

Dat geeft ons een duidelijke structuur.

---

## 10. De wereld

We zouden nog een aparte `Wereld` kunnen maken.

Bijvoorbeeld:

```rust,ignore
struct Wereld {
    vijanden: Vec<Vijand>,
}
```

Maar hebben we die echt nodig?

Nog niet.

We moeten oppassen dat we niet voor ieder idee een nieuwe struct maken.

Voor de eerste versie is dit eenvoudiger:

```rust,ignore
struct Spel {
    speler: Speler,
    vijanden: Vec<Vijand>,
}
```

Later kunnen we `Wereld` toevoegen als het spel daar daadwerkelijk behoefte aan heeft.

Een goed ontwerp mag groeien.

---

## 11. De toestand van het spel

Het spel kan verschillende toestanden hebben.

Bijvoorbeeld:

```text
Startscherm
Spelen
Gevecht
Winkel
Game over
Afgesloten
```

Een enum past hier goed bij:

```rust,ignore
enum SpelStatus {
    Menu,
    Verkennen,
    Gevecht,
    Winkel,
    GameOver,
}
```

De `Spel` kan deze toestand bewaren:

```rust,ignore
struct Spel {
    speler: Speler,
    vijanden: Vec<Vijand>,
    status: SpelStatus,
}
```

Nu kunnen we bijvoorbeeld een hoofdprogramma ontwerpen rond de toestand van het spel.

---

## 12. De hoofdgame-loop

De game-loop wordt ongeveer:

```text
zolang het spel niet afgelopen is:

    kijk naar de huidige toestand

    als Menu:
        toon menu

    als Verkennen:
        laat speler verkennen

    als Gevecht:
        voer gevecht uit

    als Winkel:
        toon winkel

    als GameOver:
        toon eindscherm
```

In Rust kan dit uiteindelijk met `loop` en `match`.

Bijvoorbeeld als eenvoudige schets:

```rust,ignore
loop {
    match spel.status {
        SpelStatus::Menu => {
            // ...
        }
        SpelStatus::Verkennen => {
            // ...
        }
        SpelStatus::Gevecht => {
            // ...
        }
        SpelStatus::Winkel => {
            // ...
        }
        SpelStatus::GameOver => {
            break;
        }
    }
}
```

Dit is nog geen complete implementatie.

Het is een **ontwerp**.

Dat onderscheid is belangrijk.

---

## 13. De modules

Ons project heeft inmiddels verschillende verantwoordelijkheden.

We kunnen die over bestanden verdelen:

```text
src/
├── main.rs
├── spel.rs
├── speler.rs
├── vijand.rs
├── wapen.rs
├── inventaris.rs
├── gevecht.rs
└── savegame.rs
```

Iedere module krijgt een duidelijke taak.

### `main.rs`

Start het programma.

Bij voorkeur staat hier relatief weinig spel-logica.

Bijvoorbeeld:

```rust,ignore
fn main() {
    let mut spel = Spel::nieuw();
    spel.start();
}
```

---

### `spel.rs`

Beheert het algemene spel.

Bijvoorbeeld:

```rust,ignore
struct Spel {
    speler: Speler,
    vijanden: Vec<Vijand>,
    status: SpelStatus,
}
```

En methoden zoals:

```rust,ignore
impl Spel {
    fn nieuw() -> Self {
        // ...
    }

    fn start(&mut self) {
        // ...
    }
}
```

---

### `speler.rs`

Beheert de speler.

Bijvoorbeeld:

```rust,ignore
struct Speler {
    // ...
}
```

Met methoden zoals:

```text
neem_schade()
genees()
is_levend()
verdien_goud()
betaal()
```

---

### `vijand.rs`

Beheert vijanden.

Bijvoorbeeld:

```text
neem_schade()
is_levend()
```

---

### `wapen.rs`

Beheert wapens en `WapenType`.

---

### `inventaris.rs`

Beheert de inventaris en de `HashMap`.

---

### `gevecht.rs`

Beheert de gevechtslogica.

---

### `savegame.rs`

Beheert opslaan en laden.

---

## 14. Een eerste afhankelijkheidsstructuur

We kunnen het project ongeveer zo bekijken:

```text
                    main.rs
                       |
                       v
                    Spel
                  /   |   \
                 /    |    \
                v     v     v
           Speler  Vijanden  Status
             |
             v
        Inventaris
             |
             v
          HashMap

Spel
 |
 +----> Gevecht
 |
 +----> Savegame
 |
 +----> Wapen
```

Dit is geen exacte Rust-syntaxis.

Het is een manier om na te denken over de relaties tussen onderdelen.

---

## 15. Wat moet `Spel` kunnen?

We kunnen alvast een lijst maken.

```text
Spel::nieuw()
Spel::start()
Spel::verken()
Spel::start_gevecht()
Spel::open_winkel()
Spel::opslaan()
Spel::laden()
```

Maar ook hier geldt:

> We hoeven niet alles meteen te programmeren.

We kunnen beginnen met:

```rust,ignore
Spel::nieuw()
Spel::start()
```

Daarna voegen we functionaliteit toe.

---

## 16. Een mogelijke eerste versie

Onze eerste speelbare versie kan heel klein zijn.

### Stap 1

Start het spel:

```text
=== AVONTUUR ===

Welkom!

Druk op Enter om te beginnen.
```

### Stap 2

Maak een speler:

```text
Naam: Arin

Gezondheid: 100
Goud: 50
```

### Stap 3

Laat de speler één vijand tegenkomen:

```text
Een Goblin verschijnt!
```

### Stap 4

Laat de speler aanvallen.

### Stap 5

Laat de Goblin terug aanvallen.

### Stap 6

Stop wanneer één van beide dood is.

Dat is al een echt klein spel.

---

## 17. Daarna uitbreiden

Vervolgens kunnen we steeds één onderdeel toevoegen.

Bijvoorbeeld:

```text
Versie 1
  ↓
één gevecht

Versie 2
  ↓
meerdere vijanden

Versie 3
  ↓
wapens

Versie 4
  ↓
inventaris

Versie 5
  ↓
winkel

Versie 6
  ↓
verkennen

Versie 7
  ↓
meerdere locaties

Versie 8
  ↓
savegame

Versie 9
  ↓
laden

Versie 10
  ↓
uitgebreidere RPG
```

Hierdoor blijft iedere stap begrijpelijk.

---

## 18. Eerst ontwerpen, dan programmeren

Voordat we code schrijven, kunnen we voor ieder onderdeel drie vragen stellen:

### 1. Welke gegevens heeft het nodig?

Bijvoorbeeld:

```text
Speler:
- naam
- gezondheid
- goud
- inventaris
```

### 2. Welke acties kan het uitvoeren?

Bijvoorbeeld:

```text
Speler:
- aanvallen
- schade krijgen
- genezen
- goud verdienen
```

### 3. Wie is eigenaar?

Bijvoorbeeld:

```text
Spel bezit Speler.
Speler bezit Inventaris.
Inventaris bezit items.
```

Deze drie vragen lossen verrassend veel ontwerpvragen op.

---

## 19. Waar gebruiken we de verschillende Rust-concepten?

Ons project wordt uiteindelijk een verzameling van vrijwel alles wat we tot nu toe geleerd hebben.

| Rust-concept | Gebruik in het RPG |
| --- | --- |
| Variabelen | gezondheid, goud, schade |
| Functies | acties en berekeningen |
| `if` | controles en keuzes |
| Lussen | game-loop en herhaling |
| Ownership | bepalen wie gegevens bezit |
| Borrowing | tijdelijk gegevens gebruiken |
| Structs | speler, vijand, wapen |
| `impl` | gedrag van objecten |
| Enums | spelstatus en wapentypen |
| Traits | gemeenschappelijk gedrag |
| `derive` | `Debug`, `Clone`, `PartialEq` enz. |
| `Vec` | lijst met vijanden |
| `HashMap` | inventaris |
| Closures | kleine acties |
| Iterators | gegevens doorzoeken/bewerken |
| Tests | controleren of regels kloppen |
| `String` / `&str` | namen en berichten |
| Modules | project opdelen |
| `Option` | iets dat misschien bestaat |
| `Result` | bewerkingen die kunnen mislukken |
| Generics | herbruikbare functies en typen |
| Bestanden | savegames |

Dat betekent niet dat ieder onderdeel verplicht in iedere functie moet voorkomen.

Het betekent dat we ze nu in een realistisch programma kunnen toepassen.

---

## 20. Wat slaan we op?

We hebben al geleerd hoe we tekstbestanden kunnen schrijven.

Een savegame zou bijvoorbeeld kunnen worden:

```text
naam=Arin
gezondheid=83
goud=125
wapen=Zwaard
item=Potion:3
item=Brood:5
```

Later kunnen we daar meer gegevens aan toevoegen.

Bijvoorbeeld:

```text
status=Verkennen
```

Of:

```text
vijand=Goblin:20:5
vijand=Wolf:35:8
```

We hoeven hiervoor voorlopig geen ingewikkeld bestandsformaat te gebruiken.

Het doel is eerst begrijpen hoe het opslaan en laden werkt.

---

## 21. Wat als het laden mislukt?

Een savegame kan bijvoorbeeld ontbreken.

Of iemand heeft het bestand handmatig aangepast:

```text
gezondheid=abc
```

Daarom gebruiken we `Result`.

Bijvoorbeeld:

```rust,ignore
fn laad_spel(pad: &str) -> Result<Speler, String> {
    // ...
}
```

De aanroeper moet dan beslissen wat er gebeurt.

Bijvoorbeeld:

```text
Savegame laden...

Gelukt!
```

of:

```text
Savegame kon niet worden geladen.

Er is een probleem met het bestand.
```

Dit is een realistische toepassing van foutafhandeling.

---

## 22. Wat als iets niet bestaat?

Een speler kan bijvoorbeeld naar een item zoeken:

```rust,ignore
fn zoek_item(&self, naam: &str) -> Option<&Item> {
    // ...
}
```

Het resultaat kan zijn:

```text
Some(item)
```

of:

```text
None
```

Hetzelfde idee kunnen we gebruiken voor een vijand:

```rust,ignore
fn vind_vijand(&self, naam: &str) -> Option<&Vijand> {
    // ...
}
```

`Option` en `Result` hebben daarmee een duidelijke rol:

```text
Option
    ↓
iets bestaat misschien niet

Result
    ↓
een bewerking kan mislukken
```

---

## 23. Tests worden onderdeel van het ontwerp

Een groter programma wordt gemakkelijker te onderhouden als we belangrijke regels testen.

Bijvoorbeeld:

```rust,ignore
#[test]
fn schade_verlaagt_gezondheid() {
    let mut vijand = Vijand::nieuw("Goblin", 30, 5);

    vijand.neem_schade(10);

    assert_eq!(vijand.gezondheid(), 20);
}
```

Of:

```rust,ignore
#[test]
fn dode_vijand_is_niet_levend() {
    let mut vijand = Vijand::nieuw("Goblin", 30, 5);

    vijand.neem_schade(30);

    assert!(!vijand.is_levend());
}
```

De precieze API kunnen we later bepalen.

Het belangrijke idee is:

> De spelregels die belangrijk zijn, willen we automatisch kunnen controleren.

---

## 24. Een ontwerpregel: houd verantwoordelijkheden klein

Stel dat we deze functie schrijven:

```rust,ignore
fn start_spel() {
    // 500 regels code
}
```

Dat kan misschien werken, maar het wordt snel moeilijk te begrijpen.

We willen liever:

```text
Spel
 ├── start
 ├── menu
 ├── verkennen
 ├── gevecht
 ├── winkel
 └── savegame
```

En ieder onderdeel gebruikt weer zijn eigen structuren en functies.

Het doel is niet om zoveel mogelijk bestanden te maken.

Het doel is dat je bij een probleem snel kunt denken:

> Dit gaat over de inventaris, dus ik kijk eerst in `inventaris.rs`.

---

## 25. Een ontwerpregel: maak gegevens niet onnodig openbaar

We zouden dit kunnen doen:

```rust,ignore
pub struct Speler {
    pub gezondheid: i32,
    pub goud: i32,
}
```

Maar dan kan iedere module zomaar doen:

```rust,ignore
speler.gezondheid = -500;
```

Dat willen we misschien niet.

We kunnen de velden daarom privé houden:

```rust,ignore
pub struct Speler {
    naam: String,
    gezondheid: i32,
    goud: i32,
}
```

En methoden aanbieden:

```rust,ignore
impl Speler {
    pub fn neem_schade(&mut self, schade: i32) {
        // ...
    }
}
```

De methode kan vervolgens regels afdwingen.

Bijvoorbeeld:

```text
gezondheid mag niet lager dan 0 worden
```

Zo houden we de spelregels bij de code die daarvoor verantwoordelijk is.

---

## 26. Een ontwerpregel: begin klein

Een veelgemaakte fout bij een groot programmeerproject is meteen alles willen bouwen.

Bijvoorbeeld:

```text
wereld
quests
NPC's
dialogen
crafting
levels
vaardigheden
magie
handel
savegames
multiplayer
```

Dat doen we niet.

Onze eerste versie heeft:

```text
Speler
Vijand
Gevecht
```

Pas wanneer dat werkt, voegen we meer toe.

Dat heeft nog een voordeel:

Als er iets fout gaat, is de hoeveelheid code waarin we moeten zoeken klein.

---

## 27. Ons eerste concrete ontwerp

Voor de eerste speelbare versie spreken we daarom het volgende af.

## `Speler`

Heeft:

```text
naam
gezondheid
maximale gezondheid
goud
```

## `Vijand`

Heeft:

```text
naam
gezondheid
aanvalskracht
```

## `Spel`

Heeft:

```text
speler
vijanden
status
```

## `SpelStatus`

Heeft:

```text
Menu
Gevecht
GameOver
```

## `Gevecht`

Kan:

```text
speler laten aanvallen
vijand laten aanvallen
controleren of iemand dood is
```

De inventaris en wapens voegen we daarna toe.

---

## 28. De eerste versie tekenen

We kunnen het ontwerp nu eenvoudig tekenen:

```text
                    ┌─────────────┐
                    │    Spel     │
                    └──────┬──────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
              ▼            ▼            ▼
          ┌───────┐    ┌────────┐   ┌──────────┐
          │Speler │    │Vijanden│   │ SpelStatus│
          └───┬───┘    └────────┘   └──────────┘
              │
              │
              ▼
        ┌─────────────┐
        │ Inventaris  │
        └─────────────┘
```

En tijdens een gevecht:

```text
        ┌────────┐
        │ Speler │
        └───┬────┘
            │
            │ valt aan
            ▼
        ┌────────┐
        │ Vijand │
        └───┬────┘
            │
            │ valt terug aan
            ▼
        ┌────────┐
        │ Speler │
        └────────┘
```

Dit is al voldoende om met programmeren te beginnen.

---

## 29. De implementatievolgorde

We gaan het project niet bestand voor bestand volledig afmaken.

We bouwen het in kleine werkende stappen.

### Fase 1 — Basis

```text
Speler
Vijand
Spel
SpelStatus
```

### Fase 2 — Eerste gevecht

```text
aanvallen
schade
sterven
game-over
```

### Fase 3 — Meerdere vijanden

```text
Vec<Vijand>
```

### Fase 4 — Wapens

```text
Wapen
WapenType
```

### Fase 5 — Inventaris

```text
Inventaris
HashMap
```

### Fase 6 — Winkel

```text
prijzen
goud
kopen
```

### Fase 7 — Verkennen

```text
locaties
vijanden
keuzes
```

### Fase 8 — Savegame

```text
opslaan
laden
foutafhandeling
```

### Fase 9 — Tests

```text
spelregels
gevechten
inventaris
savegame
```

Iedere fase moet zelfstandig kunnen werken voordat we doorgaan.

---

## 30. Eerste ontwerp-oefening

Voordat je code schrijft, maak je zelf een klein ontwerp.

Schrijf voor jouw RPG op:

### Speler

Welke vijf eigenschappen heeft de speler?

```text
1. ...
2. ...
3. ...
4. ...
5. ...
```

### Vijand

Welke vier eigenschappen heeft een vijand?

```text
1. ...
2. ...
3. ...
4. ...
```

### Wapen

Welke eigenschappen heeft een wapen?

```text
1. ...
2. ...
3. ...
```

### Spelstatus

Welke toestanden kan het spel hebben?

```text
1. ...
2. ...
3. ...
```

### Inventaris

Welke soorten items wil je kunnen bewaren?

```text
1. ...
2. ...
3. ...
```

Je hoeft nog geen Rust-code te schrijven.

Het doel is leren nadenken over de gegevens die een programma nodig heeft.

---

## 31. Eindopdracht — Ontwerp je eigen RPG

Ontwerp voordat je programmeert jouw eerste versie van het RPG.

Je ontwerp moet minimaal bevatten:

```text
Speler
Vijand
Wapen
Inventaris
Spel
SpelStatus
```

Geef voor iedere struct aan:

- welke velden hij heeft
- welke gegevens hij bezit
- welke methoden hij waarschijnlijk nodig heeft.

Geef voor iedere enum aan:

- welke varianten hij heeft
- waarom een enum hier geschikt is.

Geef voor iedere module aan:

- waarvoor de module verantwoordelijk is.

Maak ten slotte een schema zoals:

```text
Spel
│
├── Speler
│   └── Inventaris
│
├── Vijanden
│
├── Wapens
│
└── SpelStatus
```

Je hoeft het volledige spel nog niet te programmeren.

---

## 32. Wat hebben we hiermee geleerd?

Een groter programma begint niet met honderden regels code.

We beginnen met vragen:

```text
Welke dingen bestaan er?
Welke gegevens hebben ze?
Wie bezit die gegevens?
Welke acties kunnen ze uitvoeren?
Welke onderdelen horen bij elkaar?
Welke verantwoordelijkheden kunnen we scheiden?
```

Daarna kunnen we de antwoorden vertalen naar:

```text
structs
enums
methods
traits
modules
functions
```

En vervolgens kunnen we het programma stap voor stap bouwen.

In de volgende stap gaan we het ontwerp daadwerkelijk omzetten naar een **eerste werkende RPG-versie**. We beginnen bewust klein: een speler, een vijand en een gevecht. Daarna bouwen we de rest er gecontroleerd omheen.

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 28](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_28/).

---

## Controlelijst

Je bent klaar met dit artikel als je zonder hulp:

- [ ] een complex spel kunt ontleden in afzonderlijke structs en enums
- [ ] kunt beargumenteren wie de eigenaar van welke spelgegevens is
- [ ] een overzichtelijk moduleschema kunt tekenen voor een RPG
- [ ] een ontwikkelvolgorde kunt opstellen (eerst klein, dan iteratief uitbreiden).
