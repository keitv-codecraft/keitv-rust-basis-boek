# Rust 12 — Enums: één van meerdere mogelijkheden

## 1. Wat gaan we leren?

Tot nu toe hebben we geleerd hoe we gegevens kunnen bewaren met variabelen en structs.

Bijvoorbeeld:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
}
```

Maar soms willen we niet zomaar een waarde bewaren.

Soms kan iets **één van een aantal verschillende mogelijkheden** zijn.

Een speler kan bijvoorbeeld een actie uitvoeren:

- aanvallen
- verdedigen
- genezen
- vluchten.

Een wapen kan bijvoorbeeld een:

- zwaard
- boog
- staf

zijn.

Een spel kan bijvoorbeeld in één van deze toestanden zijn:

- menu
- spelen
- gepauzeerd
- game over.

Voor zulke situaties is een **enum** erg handig.

Na dit artikel kun je:

- een enum maken
- waarden van een enum gebruiken
- controleren welke variant je hebt
- enums gebruiken met `if` en `match`
- enums gebruiken als onderdeel van een struct
- enums maken met extra gegevens
- enums gebruiken in een game.

---

## 2. Waarom hebben we enums nodig?

Stel dat we willen onthouden welk wapen een speler heeft.

Een eenvoudige oplossing zou kunnen zijn:

```rust,ignore
let wapen = "zwaard";
```

Maar dan is `"zwaard"` gewoon tekst.

We zouden ook kunnen schrijven:

```rust,ignore
let wapen = 1;
```

waarbij bijvoorbeeld:

```text
1 = zwaard
2 = boog
3 = staf
```

Maar dat is niet erg duidelijk.

Wat betekent `2`?

Een enum maakt de mogelijkheden expliciet.

We kunnen schrijven:

```rust,ignore
enum Wapen {
    Zwaard,
    Boog,
    Staf,
}
```

Nu weten we precies welke soorten wapens bestaan.

Een speler kan bijvoorbeeld een zwaard hebben:

```rust,ignore
let wapen = Wapen::Zwaard;
```

Of een boog:

```rust,ignore
let wapen = Wapen::Boog;
```

Of een staf:

```rust,ignore
let wapen = Wapen::Staf;
```

---

## 3. Wat is een enum?

Een enum beschrijft een aantal mogelijke **varianten**.

Bijvoorbeeld:

```rust,ignore
enum Wapen {
    Zwaard,
    Boog,
    Staf,
}
```

Hier hebben we één enum:

```text
Wapen
```

met drie varianten:

```text
Zwaard
Boog
Staf
```

Je kunt het zien als:

```text
Wapen
├── Zwaard
├── Boog
└── Staf
```

Een variabele van het type `Wapen` kan één van deze drie waarden bevatten.

Bijvoorbeeld:

```rust,ignore
let wapen = Wapen::Zwaard;
```

Maar niet:

```rust,ignore
let wapen = Wapen::Laser;
```

want `Laser` bestaat niet in onze enum.

De compiler kan ons daar dus mee helpen.

---

## 4. De naam van een enum-variant

Je gebruikt een variant met `::`.

Bijvoorbeeld:

```rust,ignore
Wapen::Zwaard
```

De eerste naam:

```text
Wapen
```

is de enum.

De tweede naam:

```text
Zwaard
```

is de variant.

Dus:

```rust,ignore
let wapen = Wapen::Zwaard;
```

betekent:

> Maak een waarde van het type `Wapen`, met de variant `Zwaard`.

---

## 5. Een eerste eenvoudig voorbeeld

We maken een enum voor de richting waarin een speler kan lopen:

```rust,ignore
enum Richting {
    Omhoog,
    Omlaag,
    Links,
    Rechts,
}
```

We kunnen vervolgens een richting kiezen:

```rust,ignore
let richting = Richting::Omhoog;
```

Of:

```rust,ignore
let richting = Richting::Links;
```

De variabele `richting` kan dus verschillende waarden hebben, maar altijd van het type `Richting`.

---

## 6. Een enum gebruiken met `if`

We kunnen controleren welke waarde we hebben.

Daarvoor gebruiken we `==`...

Maar er is een probleem.

Rust weet niet automatisch dat twee enum-waarden met `==` vergeleken kunnen worden.

We zouden bijvoorbeeld willen schrijven:

```rust,ignore
if richting == Richting::Omhoog {
    println!("Je loopt omhoog!");
}
```

Om dit mogelijk te maken, moeten we de enum eerst voorzien van een extra eigenschap:

```rust,ignore
#[derive(PartialEq)]
enum Richting {
    Omhoog,
    Omlaag,
    Links,
    Rechts,
}
```

`#[derive(PartialEq)]` zorgt ervoor dat Rust weet hoe waarden van deze enum met `==` en `!=` kunnen worden vergeleken.

Dit is nuttig, maar het is nog niet nodig om precies te begrijpen **hoe** Rust dit doet.

We zullen `derive` later uitgebreider behandelen.

Voorlopig is het belangrijkste:

```rust,ignore
#[derive(PartialEq)]
```

maakt vergelijken mogelijk.

Daarna:

```rust,ignore
let richting = Richting::Omhoog;

if richting == Richting::Omhoog {
    println!("Je loopt omhoog!");
}
```

---

## 7. `match`: kijken welke variant we hebben

Voor enums is er een nog belangrijker hulpmiddel:

```rust,ignore
match
```

Met `match` kunnen we verschillende mogelijkheden afzonderlijk behandelen.

Bijvoorbeeld:

```rust
enum Richting {
    Omhoog,
    Omlaag,
    Links,
    Rechts,
}

fn main() {
    let richting = Richting::Links;

    match richting {
        Richting::Omhoog => println!("Omhoog!"),
        Richting::Omlaag => println!("Omlaag!"),
        Richting::Links => println!("Links!"),
        Richting::Rechts => println!("Rechts!"),
    }
}
```

Hier gebeurt het volgende:

```text
Is het Omhoog?
    → doe dit

Is het Omlaag?
    → doe dat

Is het Links?
    → doe dat

Is het Rechts?
    → doe dat
```

Elke mogelijkheid krijgt zijn eigen code.

---

## 8. `match` moet alle mogelijkheden behandelen

Dit is belangrijk.

Onze enum heeft vier varianten:

```rust,ignore
enum Richting {
    Omhoog,
    Omlaag,
    Links,
    Rechts,
}
```

Als we schrijven:

```rust,ignore
match richting {
    Richting::Omhoog => println!("Omhoog!"),
    Richting::Omlaag => println!("Omlaag!"),
}
```

dan missen we:

```text
Links
Rechts
```

Rust zal een foutmelding geven.

Dat is juist handig.

Rust zegt eigenlijk:

> Je hebt niet verteld wat er moet gebeuren bij alle mogelijke situaties.

Bij een enum wil Rust graag dat je daar bewust over nadenkt.

---

## 9. `_`: alle andere mogelijkheden

Soms willen we niet iedere mogelijkheid afzonderlijk behandelen.

Dan kunnen we `_` gebruiken.

Bijvoorbeeld:

```rust,ignore
match richting {
    Richting::Omhoog => println!("Omhoog!"),
    _ => println!("Een andere richting."),
}
```

De `_` betekent hier:

> Alles wat hierboven nog niet behandeld is.

Dus:

```text
Omhoog → "Omhoog!"
Omlaag → "Een andere richting."
Links  → "Een andere richting."
Rechts → "Een andere richting."
```

Dit is handig wanneer verschillende situaties hetzelfde gedrag hebben.

---

## 10. Een enum in een functie

Een enum kan gewoon als parameter worden gebruikt.

```rust,ignore
enum Richting {
    Omhoog,
    Omlaag,
    Links,
    Rechts,
}

fn beweeg(richting: Richting) {
    match richting {
        Richting::Omhoog => println!("Je gaat omhoog."),
        Richting::Omlaag => println!("Je gaat omlaag."),
        Richting::Links => println!("Je gaat naar links."),
        Richting::Rechts => println!("Je gaat naar rechts."),
    }
}
```

We kunnen dan schrijven:

```rust,ignore
beweeg(Richting::Links);
```

Of:

```rust,ignore
beweeg(Richting::Omhoog);
```

---

## 11. Een enum in een struct

Enums worden vooral interessant wanneer we ze combineren met structs.

Stel:

```rust,ignore
enum Klasse {
    Krijger,
    Tovenaar,
    Boogschutter,
}
```

We kunnen dit gebruiken in een speler:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
    klasse: Klasse,
}
```

Dan kunnen we bijvoorbeeld maken:

```rust,ignore
let speler = Speler {
    naam: String::from("Arin"),
    gezondheid: 100,
    klasse: Klasse::Krijger,
};
```

De speler heeft nu een `klasse`.

Die klasse kan alleen één van de drie mogelijkheden zijn.

---

## 12. Een method gebruiken met een enum

We kunnen een method maken die naar de klasse kijkt:

```rust,ignore
impl Speler {
    fn toon_klasse(&self) {
        match self.klasse {
            Klasse::Krijger => println!("Je bent een krijger."),
            Klasse::Tovenaar => println!("Je bent een tovenaar."),
            Klasse::Boogschutter => println!("Je bent een boogschutter."),
        }
    }
}
```

Gebruik:

```rust,ignore
speler.toon_klasse();
```

Dit is een mooi voorbeeld van hoe onze eerdere onderwerpen steeds samenkomen:

```text
struct
    ↓
impl
    ↓
method
    ↓
match
    ↓
enum
```

---

## 13. Enums kunnen extra gegevens bevatten

Dit is één van de krachtigste eigenschappen van enums.

Een variant hoeft niet alleen een naam te hebben.

Een variant kan ook gegevens bevatten.

Bijvoorbeeld:

```rust,ignore
enum Aanval {
    Zwaard,
    Boog,
    Magie(i32),
}
```

Hier zijn:

```text
Zwaard
Boog
Magie(i32)
```

drie varianten.

De eerste twee hebben geen extra gegevens.

`Magie` heeft wel een extra waarde:

```text
i32
```

We kunnen bijvoorbeeld schrijven:

```rust,ignore
let aanval = Aanval::Magie(50);
```

Hierbij is `50` de kracht van de magie.

---

## 14. `match` met gegevens

Nu kunnen we die waarde uitlezen:

```rust,ignore
match aanval {
    Aanval::Zwaard => println!("Je valt aan met een zwaard!"),
    Aanval::Boog => println!("Je schiet een pijl!"),
    Aanval::Magie(schade) => {
        println!("Je gebruikt magie voor {schade} schade!");
    }
}
```

Let op:

```rust,ignore
Aanval::Magie(schade)
```

Hier krijgt de waarde die in de enum zit tijdelijk de naam:

```text
schade
```

Als we hadden:

```rust,ignore
let aanval = Aanval::Magie(50);
```

dan is binnen deze tak:

```text
schade = 50
```

---

## 15. Meerdere gegevens in een variant

Een variant kan zelfs meerdere waarden bevatten.

Bijvoorbeeld:

```rust,ignore
enum Aanval {
    Zwaard,
    Boog,
    Magie(i32, i32),
}
```

De magie heeft nu bijvoorbeeld:

```text
schade
mana
```

We kunnen schrijven:

```rust,ignore
let aanval = Aanval::Magie(50, 20);
```

En:

```rust,ignore
match aanval {
    Aanval::Zwaard => {
        println!("Zwaardaanval!");
    }

    Aanval::Boog => {
        println!("Boogaanval!");
    }

    Aanval::Magie(schade, mana) => {
        println!("Magie doet {schade} schade.");
        println!("Je gebruikt {mana} mana.");
    }
}
```

---

## 16. Verschillende soorten gegevens

Een enum kan ook varianten hebben met verschillende soorten gegevens.

Bijvoorbeeld:

```rust,ignore
enum SpelEvent {
    Schade(i32),
    Genezing(i32),
    Bericht(String),
}
```

We kunnen dan drie verschillende gebeurtenissen maken:

```rust,ignore
let event1 = SpelEvent::Schade(20);
let event2 = SpelEvent::Genezing(10);
let event3 = SpelEvent::Bericht(String::from("Een vijand verschijnt!"));
```

Met `match` kunnen we ze verwerken:

```rust,ignore
match event1 {
    SpelEvent::Schade(hoeveelheid) => {
        println!("Je krijgt {hoeveelheid} schade.");
    }

    SpelEvent::Genezing(hoeveelheid) => {
        println!("Je geneest {hoeveelheid} HP.");
    }

    SpelEvent::Bericht(tekst) => {
        println!("{tekst}");
    }
}
```

Dit is een patroon dat we later in echte programma's veel zullen tegenkomen.

---

## 17. Enums zonder extra gegevens

Niet iedere enum hoeft ingewikkeld te zijn.

Een eenvoudige enum is soms precies wat we nodig hebben.

Bijvoorbeeld de toestand van een game:

```rust,ignore
enum SpelStatus {
    Menu,
    Spelen,
    Gepauzeerd,
    GameOver,
}
```

We kunnen bijvoorbeeld hebben:

```rust,ignore
let status = SpelStatus::Menu;
```

En:

```rust,ignore
match status {
    SpelStatus::Menu => println!("Hoofdmenu"),
    SpelStatus::Spelen => println!("Het spel loopt."),
    SpelStatus::Gepauzeerd => println!("Gepauzeerd."),
    SpelStatus::GameOver => println!("Game over!"),
}
```

Dit is een heel nuttig patroon voor games.

---

## 18. Enums en `if`

Je kunt enums ook met `if` gebruiken.

Daarvoor moet vergelijken mogelijk zijn:

```rust,ignore
#[derive(PartialEq)]
enum SpelStatus {
    Menu,
    Spelen,
    Gepauzeerd,
    GameOver,
}
```

Dan:

```rust,ignore
let status = SpelStatus::Spelen;

if status == SpelStatus::Spelen {
    println!("Het spel is bezig.");
}
```

Maar als we veel verschillende varianten moeten behandelen, is `match` meestal duidelijker:

```rust,ignore
match status {
    SpelStatus::Menu => println!("Menu"),
    SpelStatus::Spelen => println!("Spelen"),
    SpelStatus::Gepauzeerd => println!("Pauze"),
    SpelStatus::GameOver => println!("Game over"),
}
```

Voorlopig kun je als vuistregel onthouden:

> Eén eenvoudige controle? `if` kan handig zijn.  
> Verschillende varianten van een enum behandelen? Denk aan `match`.

---

## 19. Een gamevoorbeeld

We maken een eenvoudig RPG-gevecht.

```rust,ignore
enum Actie {
    Aanvallen,
    Verdedigen,
    Genezen,
    Vluchten,
}
```

Een speler kan een actie kiezen:

```rust,ignore
let actie = Actie::Aanvallen;
```

We verwerken de actie:

```rust,ignore
match actie {
    Actie::Aanvallen => {
        println!("Je valt de vijand aan!");
    }

    Actie::Verdedigen => {
        println!("Je gaat in de verdediging.");
    }

    Actie::Genezen => {
        println!("Je gebruikt een genezingsdrankje.");
    }

    Actie::Vluchten => {
        println!("Je probeert te vluchten!");
    }
}
```

We hebben nu een duidelijke manier om verschillende acties te beschrijven.

---

## 20. Een enum met een game-effect

We kunnen de acties ook gegevens meegeven:

```rust,ignore
enum Actie {
    Aanvallen(i32),
    Genezen(i32),
    Vluchten,
}
```

Bijvoorbeeld:

```rust,ignore
let actie = Actie::Aanvallen(25);
```

Nu betekent dit:

> Val aan met 25 schade.

We kunnen het verwerken:

```rust,ignore
match actie {
    Actie::Aanvallen(schade) => {
        println!("Je doet {schade} schade!");
    }

    Actie::Genezen(hoeveelheid) => {
        println!("Je geneest {hoeveelheid} HP!");
    }

    Actie::Vluchten => {
        println!("Je vlucht!");
    }
}
```

---

## 21. Een enum als returnwaarde

Een functie kan ook een enum teruggeven.

Bijvoorbeeld:

```rust,ignore
enum Richting {
    Omhoog,
    Omlaag,
    Links,
    Rechts,
}

fn volgende_richting() -> Richting {
    Richting::Rechts
}
```

We kunnen daarna:

```rust,ignore
let richting = volgende_richting();
```

en:

```rust,ignore
match richting {
    Richting::Omhoog => println!("Omhoog"),
    Richting::Omlaag => println!("Omlaag"),
    Richting::Links => println!("Links"),
    Richting::Rechts => println!("Rechts"),
}
```

Dit wordt later erg belangrijk wanneer functies verschillende mogelijke resultaten kunnen hebben.

---

## 22. Veelgemaakte fouten

## Fout 1 — Een variant gebruiken die niet bestaat

Als je hebt:

```rust,ignore
enum Wapen {
    Zwaard,
    Boog,
}
```

dan bestaat dit niet:

```rust,ignore
Wapen::Staf
```

De compiler vertelt je dat.

---

## Fout 2 — `match` niet compleet maken

Bij:

```rust,ignore
enum Richting {
    Omhoog,
    Omlaag,
    Links,
    Rechts,
}
```

moet `match` alle mogelijkheden behandelen.

Dus:

```rust,ignore
match richting {
    Richting::Omhoog => println!("Omhoog"),
    Richting::Omlaag => println!("Omlaag"),
    Richting::Links => println!("Links"),
    Richting::Rechts => println!("Rechts"),
}
```

Of gebruik `_` wanneer dat passend is.

---

## Fout 3 — Vergeten dat een variant gegevens bevat

Bij:

```rust,ignore
enum Aanval {
    Zwaard,
    Magie(i32),
}
```

is:

```rust,ignore
Aanval::Magie(50)
```

correct.

Maar:

```rust,ignore
Aanval::Magie
```

niet.

`Magie` heeft immers een `i32` nodig.

---

## Fout 4 — Verkeerde soort gegevens

Als we hebben:

```rust,ignore
enum Aanval {
    Magie(i32),
}
```

dan moet:

```rust,ignore
Aanval::Magie(50)
```

een `i32` bevatten.

Niet bijvoorbeeld:

```rust,ignore
Aanval::Magie("vuur")
```

---

## 23. Zelf proberen

## Opdracht 1 — Richting

Maak:

```rust,ignore
enum Richting {
    Omhoog,
    Omlaag,
    Links,
    Rechts,
}
```

Maak een variabele met `Richting::Rechts`.

Gebruik `match` om:

```text
Je gaat naar rechts!
```

af te drukken.

---

## Opdracht 2 — Wapen

Maak:

```rust,ignore
enum Wapen {
    Zwaard,
    Boog,
    Staf,
}
```

Maak een variabele voor ieder soort wapen.

---

## Opdracht 3 — Wapen kiezen

Gebruik `match` om verschillende teksten te printen:

```text
Je gebruikt een zwaard.
Je gebruikt een boog.
Je gebruikt een staf.
```

---

## Opdracht 4 — Spelstatus

Maak:

```rust,ignore
enum SpelStatus {
    Menu,
    Spelen,
    Gepauzeerd,
    GameOver,
}
```

Gebruik `match` om voor iedere status een passende tekst te printen.

---

## Opdracht 5 — Klasse

Maak:

```rust,ignore
enum Klasse {
    Krijger,
    Tovenaar,
    Boogschutter,
}
```

Maak daarna:

```rust,ignore
struct Speler {
    naam: String,
    klasse: Klasse,
}
```

Maak een speler en gebruik `match` om zijn klasse te tonen.

---

## Opdracht 6 — Aanval

Maak:

```rust,ignore
enum Aanval {
    Zwaard,
    Boog,
    Magie(i32),
}
```

Maak een aanval en gebruik `match` om de aanval te beschrijven.

---

## Opdracht 7 — Genezing

Maak:

```rust,ignore
enum Actie {
    Aanvallen(i32),
    Genezen(i32),
}
```

Maak een functie die de actie afdrukt.

---

## Opdracht 8 — Event

Maak:

```rust,ignore
enum Event {
    Schade(i32),
    Genezing(i32),
    Bericht(String),
}
```

Maak van iedere variant een voorbeeld.

---

## 24. Rustlings-opgaven

Maak voor dit artikel de volgende map:

```text
exercises/
└── enums/
    ├── 01_simple_enum.rs
    ├── 02_weapon.rs
    ├── 03_direction.rs
    ├── 04_match.rs
    ├── 05_match_all.rs
    ├── 06_match_default.rs
    ├── 07_game_status.rs
    ├── 08_player_class.rs
    ├── 09_enum_struct.rs
    ├── 10_enum_function.rs
    ├── 11_enum_return.rs
    ├── 12_enum_data.rs
    ├── 13_attack.rs
    ├── 14_game_event.rs
    ├── 15_debug_variant.rs
    ├── 16_debug_match.rs
    ├── 17_debug_data.rs
    └── 18_game_action.rs
```

## ⭐ Niveau 1 — eenvoudige enums

### 1. Eerste enum

Maak:

```rust,ignore
enum Kleur {
    Rood,
    Groen,
    Blauw,
}
```

Maak een variabele met de waarde `Kleur::Groen`.

---

### 2. Wapen

Maak een enum:

```text
Zwaard
Boog
Staf
```

Maak een variabele met `Zwaard`.

---

### 3. Richting

Maak een enum met vier richtingen.

Maak een variabele met `Links`.

---

### 4. Eerste `match`

Gebruik `match` om de gekozen richting te printen.

---

### 5. Alle varianten

Maak een `match` die alle vier richtingen behandelt.

---

### 6. Standaardtak

Gebruik `_` om alle richtingen behalve `Omhoog` hetzelfde te behandelen.

---

### 7. Game status

Maak de enum:

```text
Menu
Spelen
Gepauzeerd
GameOver
```

Gebruik `match`.

---

## ⭐⭐ Niveau 2 — enums combineren

### 8. Klasse

Maak:

```text
Krijger
Tovenaar
Boogschutter
```

Gebruik de enum in een `Speler`.

---

### 9. Struct met enum

Maak:

```rust,ignore
struct Speler {
    naam: String,
    klasse: Klasse,
}
```

Maak een speler en print zijn klasse.

---

### 10. Enum als parameter

Maak:

```rust,ignore
fn toon_wapen(wapen: Wapen)
```

Gebruik `match` in deze functie.

---

### 11. Enum als returnwaarde

Maak:

```rust,ignore
fn kies_wapen() -> Wapen
```

Laat de functie bijvoorbeeld:

```rust,ignore
Wapen::Zwaard
```

teruggeven.

Gebruik daarna `match`.

---

### 12. Enum met gegevens

Maak:

```rust,ignore
enum Aanval {
    Zwaard,
    Boog,
    Magie(i32),
}
```

Gebruik `match` en print de hoeveelheid schade bij magie.

---

### 13. Aanvallen

Maak verschillende aanvallen:

```text
Zwaard
Boog
Magie(50)
```

Laat iedere aanval een andere tekst produceren.

---

### 14. Game events

Maak:

```rust,ignore
enum GameEvent {
    Schade(i32),
    Genezing(i32),
    Bericht(String),
}
```

Verwerk alle drie soorten events met `match`.

---

## ⭐⭐⭐ Niveau 3 — combineren met eerdere kennis

### 15. Debug — verkeerde variant

Je krijgt:

```rust,ignore
enum Wapen {
    Zwaard,
    Boog,
}

let wapen = Wapen::Staf;
```

Zoek de fout en los hem op.

---

### 16. Debug — incomplete `match`

Je krijgt een enum met vier varianten, maar de `match` behandelt er maar drie.

Laat de compilerfout je helpen om de ontbrekende variant te vinden.

---

### 17. Debug — gegevens

Je krijgt:

```rust,ignore
enum Aanval {
    Magie(i32),
}

let aanval = Aanval::Magie("vuur");
```

Zoek uit waarom dit niet werkt.

---

### 18. Eindopdracht — actiesysteem

Maak:

```rust,ignore
enum Actie {
    Aanvallen(i32),
    Genezen(i32),
    Verdedigen,
    Vluchten,
}
```

Maak vervolgens een functie die een `Actie` ontvangt en beschrijft wat de speler doet.

Bijvoorbeeld:

```text
Je valt aan voor 25 schade!
```

of:

```text
Je verdedigt jezelf.
```

---

## 25. Kleine eindopdracht — het gevechtssysteem

Maak een klein programma waarin een speler verschillende acties kan uitvoeren.

Gebruik:

```rust,ignore
enum Actie {
    Aanvallen(i32),
    Genezen(i32),
    Verdedigen,
    Vluchten,
}
```

Maak bijvoorbeeld:

```rust,ignore
let actie = Actie::Aanvallen(25);
```

Gebruik `match` om te bepalen wat er gebeurt.

Breid het daarna uit met:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
}
```

en gebruik een `impl`-blok.

Het doel is dat je inmiddels verschillende dingen samen kunt gebruiken:

```text
enum
  ↓
struct
  ↓
impl
  ↓
method
  ↓
match
```

Je hoeft nog geen groot spel te maken. Het gaat erom dat je begrijpt hoe deze onderdelen samenwerken.

---

## 26. Wat moet je nu kunnen?

Na dit artikel zou je moeten kunnen:

- [ ] uitleggen wat een enum is
- [ ] een enum met meerdere varianten maken
- [ ] een enumwaarde maken met `::`
- [ ] een enum gebruiken in een struct
- [ ] een enum als parameter gebruiken
- [ ] een enum als returnwaarde gebruiken
- [ ] met `match` verschillende varianten behandelen
- [ ] begrijpen waarom `match` alle mogelijkheden moet behandelen
- [ ] `_` gebruiken voor overige mogelijkheden
- [ ] gegevens aan een enum-variant meegeven
- [ ] die gegevens binnen een `match` gebruiken
- [ ] enums combineren met structs en methods.

Het belangrijkste idee van dit artikel is eenvoudig:

> **Een enum beschrijft verschillende mogelijkheden waarvan er steeds één gekozen is.**

Bijvoorbeeld:

```rust,ignore
enum Wapen {
    Zwaard,
    Boog,
    Staf,
}
```

Een `Wapen` is dus niet tegelijk een zwaard én een boog. Het is **één van de mogelijke varianten**.

En met `match` kunnen we vervolgens netjes bepalen wat we met iedere variant doen.

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 12](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_12/).

