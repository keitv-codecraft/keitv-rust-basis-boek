# Rust 36 — Consolideren: van veel losse onderdelen naar één geheel

Ons RPG-project is inmiddels behoorlijk groot geworden.

We hebben inmiddels onder andere:

- een speler
- vijanden
- wapens
- een inventaris
- een winkel
- gevechten
- locaties
- een wereld
- NPC's
- dialogen
- ontmoetingen
- XP en levels
- goud en buit
- savegames
- een speltoestand.

Dat is mooi.

Maar er ontstaat ook een nieuw probleem.

Als we alles gewoon achter elkaar blijven toevoegen, wordt het programma steeds moeilijker te begrijpen.

Misschien staat er straks in `main.rs`:

```text
- speler maken
- wereld maken
- menu tonen
- invoer lezen
- reizen
- NPC zoeken
- gesprek voeren
- vijand kiezen
- gevecht starten
- XP geven
- level controleren
- buit geven
- winkel openen
- spel opslaan
- spel laden
- ...
```

Alles werkt misschien nog.

Maar het wordt steeds moeilijker om te beantwoorden:

> Waar hoort deze code eigenlijk thuis?

Dat is precies het probleem dat we in dit artikel gaan aanpakken.

We gaan het programma **consolideren**.

Daarmee bedoelen we:

> We brengen de bestaande onderdelen terug tot een duidelijke structuur waarin ieder onderdeel een herkenbare verantwoordelijkheid heeft.

Dit is de voorbereiding op **refactoring**.

---

## 1. Eerst: werkende code is waardevol

Voordat we gaan opruimen is één regel belangrijk:

> Maak werkende code niet zomaar kapot omdat je hem mooier wilt maken.

We hebben al veel werkende onderdelen.

Dat is juist een voordeel.

We kunnen ze stap voor stap verbeteren.

Daarom gaan we niet meteen alles herschrijven.

We gebruiken een veilige aanpak:

```text
1. Zorg dat de code werkt.
2. Zorg dat tests werken.
3. Verander één klein onderdeel.
4. Test opnieuw.
5. Ga verder.
```

Als een test na een verandering niet meer werkt, weten we ongeveer waar we moeten zoeken.

---

## 2. Waarom wordt `main.rs` een probleem?

Stel dat we beginnen met:

```rust,ignore
fn main() {
    println!("Welkom bij mijn RPG!");
}
```

Dat is overzichtelijk.

Later wordt het:

```rust,ignore
fn main() {
    let mut speler = Speler::nieuw("Arin");

    let mut wereld = maak_wereld();

    loop {
        println!("Wat wil je doen?");
        println!("1. Reizen");
        println!("2. Status");
        println!("3. Inventaris");
        println!("4. Winkel");
        println!("5. Opslaan");
        println!("6. Stoppen");

        let keuze = lees_getal();

        match keuze {
            1 => {
                // tientallen regels reizen
            }

            2 => {
                // status
            }

            3 => {
                // inventaris
            }

            4 => {
                // winkel
            }

            5 => {
                // savegame
            }

            6 => {
                break;
            }

            _ => {
                println!("Ongeldige keuze.");
            }
        }
    }
}
```

En later misschien:

```rust
fn main() {
    // honderden regels
}
```

Het probleem is niet dat `main` technisch te lang is.

Het probleem is dat `main` nu **te veel weet**.

---

## 3. Een functie moet een duidelijke taak hebben

Kijk naar:

```rust,ignore
fn main() {
    // wereld maken
    // speler maken
    // menu tonen
    // invoer lezen
    // reizen
    // gevecht
    // winkel
    // opslaan
}
```

We kunnen dat opdelen:

```rust,ignore
fn main() {
    let mut speler = maak_speler();
    let mut wereld = maak_wereld();

    speel_spel(&mut speler, &mut wereld);
}
```

Dat ziet er veel eenvoudiger uit.

De details zijn niet verdwenen.

Ze zijn verplaatst naar functies die daar beter voor geschikt zijn.

---

## 4. `main` als startpunt

Een goed doel voor `main` is bijvoorbeeld:

```rust,ignore
fn main() {
    let mut speler = Speler::nieuw("Arin");
    let mut wereld = maak_wereld();

    speel_spel(&mut speler, &mut wereld);
}
```

Je kunt dit bijna lezen als een samenvatting:

> Maak een speler.  
> Maak de wereld.  
> Speel het spel.

Dat is een goed teken.

`main` hoeft niet te weten **hoe** een wereld wordt gemaakt.

En ook niet hoe een gevecht werkt.

---

## 5. Functies verplaatsen zonder gedrag te veranderen

Stel dat we dit hebben:

```rust,ignore
fn main() {
    println!("1. Reizen");
    println!("2. Status");
    println!("3. Inventaris");
    println!("4. Stoppen");

    let keuze = lees_getal();

    match keuze {
        1 => println!("Reizen"),
        2 => println!("Status"),
        3 => println!("Inventaris"),
        4 => println!("Stoppen"),
        _ => println!("Ongeldige keuze"),
    }
}
```

We kunnen het menu verplaatsen:

```rust,ignore
fn toon_menu() {
    println!("1. Reizen");
    println!("2. Status");
    println!("3. Inventaris");
    println!("4. Stoppen");
}
```

En `main`:

```rust,ignore
fn main() {
    toon_menu();

    let keuze = lees_getal();

    match keuze {
        1 => println!("Reizen"),
        2 => println!("Status"),
        3 => println!("Inventaris"),
        4 => println!("Stoppen"),
        _ => println!("Ongeldige keuze"),
    }
}
```

We hebben niets aan het gedrag veranderd.

We hebben alleen de code georganiseerd.

Dit is een eenvoudige vorm van refactoring.

---

## 6. Refactoring betekent niet: alles opnieuw schrijven

Het woord **refactoring** klinkt soms alsof we een compleet programma opnieuw moeten bouwen.

Dat is niet de bedoeling.

Bij refactoring veranderen we vooral:

> de structuur van de code

zonder dat we bewust veranderen:

> wat het programma doet.

Bijvoorbeeld:

```text
Voor:

main()
 ├── menu
 ├── reizen
 ├── gevecht
 ├── winkel
 └── savegame
```

wordt:

```text
Na:

main()
 └── speel_spel()
      ├── menu
      ├── reizen
      ├── gevecht
      ├── winkel
      └── savegame
```

Het gedrag kan hetzelfde blijven.

De structuur wordt duidelijker.

---

## 7. Een goede naam is belangrijk

Vergelijk:

```rust,ignore
fn doe_ding() {
    // ...
}
```

met:

```rust,ignore
fn toon_speler_status() {
    // ...
}
```

De tweede naam vertelt meteen wat de functie doet.

Nog een voorbeeld:

```rust,ignore
fn verwerk() {
    // ...
}
```

tegen:

```rust,ignore
fn geef_gevechtsbeloning(
    speler: &mut Speler,
    vijand: &Vijand,
) {
    // ...
}
```

Een goede naam kan veel uitleg overbodig maken.

---

## 8. Functies hoeven niet klein om klein te zijn

We moeten echter ook niet doorslaan.

Dit:

```rust,ignore
fn print_een() {
    println!("1");
}

fn print_twee() {
    println!("2");
}

fn print_drie() {
    println!("3");
}
```

is niet automatisch beter dan:

```rust,ignore
fn toon_menu() {
    println!("1. Reizen");
    println!("2. Status");
    println!("3. Inventaris");
}
```

Een functie moet een **begrijpelijke taak** hebben.

Niet iedere regel hoeft zijn eigen functie te krijgen.

---

## 9. Verantwoordelijkheden

We gaan daarom nadenken over verantwoordelijkheden.

Neem de speler.

`Speler` moet bijvoorbeeld weten:

- hoeveel gezondheid hij heeft
- hoeveel goud hij heeft
- welk wapen hij gebruikt
- hoeveel XP hij heeft
- wat er in zijn inventaris zit.

Maar `Speler` hoeft niet te weten:

- welke knop de speler op het toetsenbord indrukt
- hoe het hoofdmenu wordt weergegeven
- hoe de wereld wordt opgeslagen
- hoe een willekeurige ontmoeting wordt gekozen.

Dat zijn andere verantwoordelijkheden.

---

## 10. Een eenvoudige regel

Een nuttige vraag is:

> Als ik deze code verander, welk onderdeel van het programma zou daar logisch gezien verantwoordelijk voor moeten zijn?

Bijvoorbeeld:

### Gezondheid veranderen

```rust,ignore
speler.neem_schade(20);
```

Dat hoort bij `Speler`.

### Een vijand aanvallen

```rust,ignore
vecht(&mut speler, &mut vijand);
```

Dat hoort bij het gevechtssysteem.

### Een locatie kiezen

```rust,ignore
wereld.kies_locatie(...);
```

Dat hoort bij de wereld.

### Tekst van het hoofdmenu tonen

```rust,ignore
toon_menu();
```

Dat hoort bij de interface van het spel.

---

## 11. De modules opnieuw bekijken

We hadden eerder bijvoorbeeld:

```text id="l3o9cs"
src/
├── main.rs
├── speler.rs
├── vijand.rs
├── wapen.rs
├── inventaris.rs
├── aanvaller.rs
├── gevecht.rs
├── locatie.rs
├── wereld.rs
├── winkel.rs
└── savegame.rs
```

Dat is al een behoorlijke structuur.

Maar we moeten nu controleren:

> Zit de code ook daadwerkelijk op de juiste plaats?

Een module moet niet alleen bestaan.

De inhoud moet er ook bij passen.

---

## 12. `speler.rs`

In `speler.rs` willen we bijvoorbeeld:

```rust,ignore
pub struct Speler {
    naam: String,
    gezondheid: i32,
    maximale_gezondheid: i32,
    level: u32,
    ervaring: u32,
    goud: u32,
}
```

En methodes:

```rust,ignore
impl Speler {
    pub fn nieuw(naam: &str) -> Self {
        // ...
    }

    pub fn neem_schade(&mut self, schade: i32) {
        // ...
    }

    pub fn genees(&mut self, hoeveelheid: i32) {
        // ...
    }

    pub fn geef_ervaring(&mut self, hoeveelheid: u32) {
        // ...
    }

    pub fn geef_goud(&mut self, hoeveelheid: u32) {
        // ...
    }
}
```

De details van de speler blijven daarmee bij de speler.

---

## 13. Private velden zijn nuttig

We hebben eerder geleerd:

```rust,ignore
pub struct Speler {
    naam: String,
    gezondheid: i32,
}
```

De struct is publiek.

Maar de velden zijn privé.

Een andere module kan dus niet zomaar:

```rust,ignore
speler.gezondheid = -5000;
```

doen.

In plaats daarvan gebruiken we:

```rust,ignore
speler.neem_schade(20);
```

Dat geeft `Speler` controle over zijn eigen toestand.

---

## 14. Waarom is dat handig?

Stel dat we willen voorkomen dat gezondheid onder nul komt.

Dan kunnen we:

```rust,ignore
pub fn neem_schade(&mut self, schade: i32) {
    self.gezondheid -= schade;

    if self.gezondheid < 0 {
        self.gezondheid = 0;
    }
}
```

Als `gezondheid` publiek was, zou iedere andere module de waarde rechtstreeks kunnen veranderen.

Dan zouden we dezelfde controle overal moeten uitvoeren.

Door de wijziging via een methode te laten lopen, houden we de regel op één plaats.

---

## 15. `vijand.rs`

Hetzelfde geldt voor `Vijand`.

Bijvoorbeeld:

```rust,ignore
pub struct Vijand {
    naam: String,
    gezondheid: i32,
    aanvalskracht: i32,
    ervaring: u32,
    goud: u32,
}
```

Met methodes:

```rust,ignore
impl Vijand {
    pub fn nieuw(
        naam: &str,
        gezondheid: i32,
        aanvalskracht: i32,
        ervaring: u32,
        goud: u32,
    ) -> Self {
        // ...
    }

    pub fn neem_schade(&mut self, schade: i32) {
        // ...
    }

    pub fn is_levend(&self) -> bool {
        self.gezondheid > 0
    }
}
```

Ook hier:

> De struct beheert zijn eigen toestand.

---

## 16. `gevecht.rs`

Het gevechtssysteem hoeft niet te weten hoe een `Vijand` intern zijn gezondheid opslaat.

Het hoeft alleen de publieke interface te gebruiken:

```rust,ignore
vijand.neem_schade(schade);
```

en:

```rust,ignore
vijand.is_levend()
```

Bijvoorbeeld:

```rust,ignore
pub fn val_aan(
    speler: &mut Speler,
    vijand: &mut Vijand,
) {
    let schade = speler.aanvalskracht();

    vijand.neem_schade(schade);
}
```

Dat maakt de code overzichtelijker.

---

## 17. Een module hoeft niet alles te weten

Dit is een belangrijk ontwerpprincipe.

`gevecht.rs` hoeft bijvoorbeeld niet te weten:

- hoe een speler op het scherm wordt weergegeven
- hoe een savegame wordt opgeslagen
- hoe een NPC praat
- hoe de speler naar een andere locatie reist.

Het gevechtssysteem heeft alleen de informatie nodig die relevant is voor gevechten.

---

## 18. De wereldmodule

De `wereld` kan verantwoordelijk zijn voor locaties.

Bijvoorbeeld:

```rust,ignore
pub struct Wereld {
    locaties: Vec<Locatie>,
    huidige_locatie: usize,
}
```

Een constructor:

```rust,ignore
impl Wereld {
    pub fn nieuw(locaties: Vec<Locatie>) -> Self {
        Self {
            locaties,
            huidige_locatie: 0,
        }
    }
}
```

Nu hebben we één object dat weet:

> Welke locaties bestaan er en waar is de speler?

---

## 19. Huidige locatie opvragen

We kunnen een methode maken:

```rust,ignore
impl Wereld {
    pub fn huidige_locatie(&self) -> &Locatie {
        &self.locaties[self.huidige_locatie]
    }
}
```

Gebruik:

```rust,ignore
println!(
    "Je bent in {}.",
    wereld.huidige_locatie().naam()
);
```

Hier moeten we natuurlijk zorgen dat `Locatie` een geschikte publieke methode heeft, bijvoorbeeld:

```rust,ignore
pub fn naam(&self) -> &str {
    &self.naam
}
```

We laten de daadwerkelijke `String` dus privé.

---

## 20. Waarom niet gewoon `wereld.locaties[0]`?

Dat zou misschien eenvoudiger lijken.

Maar daarmee laat je andere code direct in de interne structuur van `Wereld` kijken.

Stel dat we later besluiten dat we locaties anders willen opslaan.

Bijvoorbeeld:

```text id="y8x8g1"
Vec<Locatie>
```

wordt:

```text id="g4v4u7"
HashMap<String, Locatie>
```

Als overal in ons programma dit staat:

```rust,ignore
wereld.locaties[0]
```

hebben we veel code die aangepast moet worden.

Als we overal:

```rust,ignore
wereld.huidige_locatie()
```

gebruiken, kunnen we de interne implementatie makkelijker veranderen.

---

## 21. Encapsulatie

Dit noemen we **encapsulatie**.

Het betekent hier vooral:

> De interne gegevens van een onderdeel worden afgeschermd en andere code gebruikt een duidelijke interface.

Bijvoorbeeld:

```text id="x1c4pf"
Andere module
     │
     ▼
wereld.huidige_locatie()
     │
     ▼
Wereld
 ┌─────────────────┐
 │ locaties        │
 │ huidige_locatie │
 └─────────────────┘
```

De andere module hoeft niet te weten hoe `Wereld` dat intern organiseert.

---

## 22. Een goede interface

Een interface bestaat in ons geval uit publieke functies en methodes.

Bijvoorbeeld:

```rust,ignore
pub fn reis_naar(&mut self, naam: &str) -> bool
```

De gebruiker van `Wereld` hoeft niet te weten:

- hoe locaties worden opgezocht
- hoe de huidige index wordt veranderd
- hoe verbindingen worden gecontroleerd.

Hij hoeft alleen te weten:

```rust,ignore
if wereld.reis_naar("Bos") {
    println!("Je reist naar het bos.");
}
```

Dat is een duidelijke interface.

---

## 23. Een slechte interface

Een slecht ontwerp zou bijvoorbeeld zijn:

```rust,ignore
pub struct Wereld {
    pub locaties: Vec<Locatie>,
    pub huidige_locatie: usize,
}
```

Dan kan overal in het programma:

```rust,ignore
wereld.huidige_locatie = 999;
```

worden geschreven.

Of:

```rust,ignore
wereld.locaties.clear();
```

De compiler kan dat toestaan.

Maar het ontwerp van `Wereld` wordt daarmee kwetsbaar.

---

## 24. `main.rs` wordt nu klein

Als onze modules goed zijn opgebouwd, kan `main.rs` bijvoorbeeld zo worden:

```rust,ignore
mod speler;
mod vijand;
mod gevecht;
mod wereld;
mod locatie;
mod inventaris;
mod wapen;
mod winkel;
mod savegame;

use speler::Speler;
use wereld::Wereld;

fn main() {
    let mut speler = Speler::nieuw("Arin");
    let mut wereld = Wereld::nieuw();

    speel_spel(&mut speler, &mut wereld);
}
```

Dat is overzichtelijk.

Maar er ontbreekt nog één onderdeel:

```rust,ignore
speel_spel()
```

---

## 25. Een aparte spelmodule

We kunnen een module maken:

```text id="g9g9m5"
src/
├── main.rs
├── spel.rs
├── speler.rs
├── vijand.rs
├── gevecht.rs
├── wereld.rs
├── locatie.rs
├── inventaris.rs
├── wapen.rs
├── winkel.rs
└── savegame.rs
```

`spel.rs` wordt verantwoordelijk voor de algemene spelbesturing.

Bijvoorbeeld:

```rust,ignore
pub fn speel_spel(
    speler: &mut Speler,
    wereld: &mut Wereld,
) {
    loop {
        toon_menu();

        let keuze = lees_getal();

        match keuze {
            1 => reis(speler, wereld),
            2 => toon_status(speler),
            3 => toon_inventaris(speler),
            4 => break,
            _ => println!("Ongeldige keuze."),
        }
    }
}
```

Nu krijgt iedere laag een duidelijkere taak.

---

## 26. De lagen van ons programma

We kunnen onze RPG ongeveer zo bekijken:

```text id="i3d2z1"
┌───────────────────────────────┐
│           main.rs             │
│       programma starten       │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│            spel.rs             │
│        spelbesturing           │
└───────┬────────┬────────┬─────┘
        │        │        │
        ▼        ▼        ▼
     Wereld   Gevecht   Winkel
        │        │
        ▼        ▼
    Locaties   Speler
        │        │
        ▼        ▼
       NPC    Inventaris
                 │
                 ▼
               Wapen
```

Dit is geen harde Rust-regel.

Het is een manier om over het programma na te denken.

---

## 27. De spelmodule moet niet alles zelf doen

`spel.rs` moet bijvoorbeeld niet zelf de gezondheid van de vijand aanpassen.

Dus liever niet:

```rust,ignore
vijand.gezondheid -= 20;
```

maar:

```rust,ignore
vijand.neem_schade(20);
```

Waarom?

Omdat `Speler` en `Vijand` verantwoordelijk zijn voor hun eigen toestand.

De spelmodule stuurt alleen aan.

---

## 28. De hoofdregel

Een handige vuistregel:

> Een module gebruikt het gedrag van andere modules, maar beheert niet zomaar hun interne gegevens.

Dus:

```rust,ignore
wereld.reis_naar("Bos");
```

is beter dan:

```rust,ignore
wereld.locaties[3].bezocht = true;
wereld.huidige_locatie = 3;
```

De eerste versie zegt wat we willen.

De tweede versie zegt precies hoe het intern moet gebeuren.

---

## 29. Een tweede voorbeeld: inventaris

Stel dat `Inventaris` intern een:

```rust,ignore
HashMap<String, u32>
```

gebruikt.

Dan hoeft `Speler` niet te weten hoe dat werkt.

Hij kan gewoon:

```rust,ignore
speler.inventaris.voeg_item_toe("Potion", 1);
```

of misschien nog beter:

```rust,ignore
speler.voeg_item_toe("Potion", 1);
```

Dan kan `Speler` zelf bepalen hoe de inventaris wordt gebruikt.

---

## 30. Wanneer maak je een methode?

Een nuttige vraag:

> Is dit gedrag logisch verbonden met één bepaald object?

Bijvoorbeeld:

```rust,ignore
speler.neem_schade(20);
```

Ja.

Schade ontvangen is gedrag van een speler.

Maar:

```rust,ignore
vecht(&mut speler, &mut vijand);
```

is iets anders.

Een gevecht heeft meerdere deelnemers.

Daarom past het beter bij een apart gevechtssysteem.

---

## 31. Wanneer maak je een vrije functie?

Een vrije functie is bijvoorbeeld:

```rust,ignore
fn bereken_schade(
    aanval: i32,
    verdediging: i32,
) -> i32 {
    // ...
}
```

Deze functie hoort niet duidelijk bij één specifieke struct.

Het is een algemene berekening.

Dan is een vrije functie vaak logisch.

---

## 32. Wanneer maak je een module?

Als een aantal functies en structs samen één duidelijk onderdeel vormen.

Bijvoorbeeld:

```text id="p4f2r7"
gevecht.rs
    ├── Gevechtsresultaat
    ├── Aanvalsresultaat
    ├── vecht()
    ├── val_aan()
    └── geef_beloning()
```

Of:

```text id="m3c6t8"
savegame.rs
    ├── opslaan()
    ├── laden()
    └── ...
```

Een module geeft daarmee een duidelijke grens aan.

---

## 33. Een module is geen willekeurige map

Je kunt dus niet zomaar zeggen:

```text
"Deze functie staat in deze module omdat er toevallig nog ruimte was."
```

Vraag liever:

> Welke verantwoordelijkheid delen deze onderdelen?

Bijvoorbeeld:

```text
inventaris.rs
```

bevat alles wat direct met de inventaris te maken heeft.

Niet:

```text
inventaris.rs
    ├── inventaris
    ├── savegame
    ├── gevecht
    └── dialoog
```

Dat zou snel onduidelijk worden.

---

## 34. Een eerste concrete structuur

Een mogelijke eindstructuur voor onze huidige RPG:

```text id="2r6b7s"
src/
├── main.rs
│
├── spel.rs
│
├── speler.rs
├── vijand.rs
├── npc.rs
│
├── wereld.rs
├── locatie.rs
│
├── gevecht.rs
├── ontmoeting.rs
│
├── inventaris.rs
├── wapen.rs
├── winkel.rs
│
└── savegame.rs
```

De verantwoordelijkheden:

| Module | Verantwoordelijkheid |
| --- | --- |
| `main.rs` | programma starten |
| `spel.rs` | hoofdspel en menu |
| `speler.rs` | speler |
| `vijand.rs` | vijanden |
| `npc.rs` | NPC's |
| `wereld.rs` | wereld en reizen |
| `locatie.rs` | locaties |
| `gevecht.rs` | gevechten |
| `ontmoeting.rs` | ontmoetingen |
| `inventaris.rs` | inventaris |
| `wapen.rs` | wapens |
| `winkel.rs` | winkel |
| `savegame.rs` | opslaan/laden |

Dit is een **voorstel**, geen absolute waarheid.

Een ander ontwerp kan ook goed zijn.

---

## 35. Niet te vroeg opsplitsen

Er is ook een andere valkuil.

Stel dat we dit hebben:

```rust,ignore
fn bereken_schade(aanval: i32, verdediging: i32) -> i32 {
    aanval - verdediging
}
```

Daar hoeft niet meteen een aparte module voor te komen.

Een kleine functie kan prima in `gevecht.rs` staan.

We moeten dus niet proberen zoveel mogelijk bestanden te maken.

Het doel is:

> Duidelijke verantwoordelijkheden.

Niet:

> Zo veel mogelijk modules.

---

## 36. Code dupliceren is een signaal

Stel dat we op drie plaatsen dit hebben:

```rust,ignore
if speler.gezondheid() <= 0 {
    println!("Je bent dood.");
}
```

Dan kan dat een teken zijn dat we iets kunnen verbeteren.

Misschien bestaat er al:

```rust,ignore
speler.is_levend()
```

Dan kunnen we gebruiken:

```rust,ignore
if !speler.is_levend() {
    println!("Je bent dood.");
}
```

We hebben daarmee niet alleen code korter gemaakt.

We hebben de regel gecentraliseerd.

---

## 37. Nog een voorbeeld van duplicatie

Stel dat we op drie plaatsen:

```rust,ignore
if gezondheid > 0 {
    // ...
}
```

schrijven.

Later verandert de definitie van "levend" misschien.

Bijvoorbeeld:

```text
gezondheid > 0
en
niet verdoofd
```

Dan moeten we drie plaatsen aanpassen.

Als we één methode hebben:

```rust,ignore
fn is_levend(&self) -> bool {
    self.gezondheid > 0
}
```

hoeven we maar één plek aan te passen.

---

## 38. Hergebruik is een belangrijk doel

We willen dus uiteindelijk code krijgen zoals:

```rust,ignore
if speler.is_levend() {
    // ...
}
```

in plaats van overal opnieuw te bedenken wat "levend" betekent.

Dit noemen we vaak **hergebruik**.

Een goede functie of methode kan op veel plaatsen gebruikt worden.

---

## 39. De compiler helpt bij refactoring

Stel dat we:

```rust,ignore
fn geef_goud(&mut self, hoeveelheid: u32)
```

veranderen in:

```rust,ignore
fn voeg_goud_toe(&mut self, hoeveelheid: u32)
```

Dan kunnen oude aanroepen:

```rust,ignore
speler.geef_goud(50);
```

compilerfouten geven.

Dat is eigenlijk nuttig.

De compiler vertelt ons:

> Hier gebruikt nog code de oude naam.

We kunnen alle fouten één voor één oplossen.

Daarom is de compiler tijdens refactoring een belangrijke hulp.

---

## 40. Tests zijn nog belangrijker

De compiler kan controleren:

> Is dit geldige Rust?

Maar niet:

> Werkt mijn RPG nog hetzelfde?

Daarvoor hebben we tests.

Stel dat:

```rust,ignore
#[test]
fn schade_verlaagt_gezondheid() {
    let mut speler = Speler::nieuw("Arin");

    speler.neem_schade(20);

    assert_eq!(speler.gezondheid(), 80);
}
```

Als we de interne structuur van `Speler` veranderen, moet deze test nog steeds slagen.

Dat geeft vertrouwen.

---

## 41. Refactoring met een veiligheidsnet

Een goede werkwijze is:

```text id="d5r6s1"
                Code werkt
                    │
                    ▼
              cargo test
                    │
                    ▼
             kleine wijziging
                    │
                    ▼
              cargo test
                    │
              ┌─────┴─────┐
              │           │
           geslaagd     mislukt
              │           │
              ▼           ▼
         verdergaan    probleem
```

Zo maak je steeds kleine stappen.

---

## 42. Een praktische refactoring-oefening

Neem jullie huidige RPG-project.

Doe nog niets aan de code.

Maak eerst een lijst van alle onderdelen:

```text
Speler
Vijand
Wapen
Inventaris
Winkel
Gevecht
Locatie
Wereld
NPC
Ontmoeting
Savegame
Speltoestand
```

Schrijf achter ieder onderdeel:

> Wat is zijn belangrijkste verantwoordelijkheid?

Als je bij één onderdeel tien verschillende verantwoordelijkheden opschrijft, is dat een aanwijzing dat het misschien te veel doet.

---

## 43. Oefening: zoek verantwoordelijkheden

Neem bijvoorbeeld deze functie:

```rust,ignore
fn speel_spel() {
    // speler tonen
    // wereld tonen
    // NPC zoeken
    // gesprek
    // vijand zoeken
    // gevecht
    // inventaris tonen
    // winkel
    // savegame
}
```

Vraag per onderdeel:

> Hoort dit werkelijk bij `speel_spel()`?

Waarschijnlijk niet.

`spel.rs` kan deze onderdelen **aanroepen**, maar hoeft ze niet zelf te implementeren.

---

## 44. Oefening: maak `main` kleiner

Begin bijvoorbeeld met:

```rust,ignore
fn main() {
    // heel veel code
}
```

Maak hiervan:

```rust,ignore
fn main() {
    let mut speler = maak_speler();
    let mut wereld = maak_wereld();

    speel_spel(&mut speler, &mut wereld);
}
```

De functies mogen in eerste instantie nog in hetzelfde bestand staan.

Pas daarna verplaats je ze naar modules.

Dat maakt de verandering overzichtelijk.

---

## 45. Oefening: verplaats de speler

Maak:

```text id="d7y4p8"
speler.rs
```

Verplaats daar:

- `struct Speler`
- de constructor
- methodes van `Speler`
- tests van `Speler`.

Laat `main.rs` daarna alleen nog:

```rust,ignore
mod speler;

use speler::Speler;
```

gebruiken.

---

## 46. Oefening: verplaats het gevecht

Doe hetzelfde voor het gevecht.

Maak:

```text id="h8z3n6"
gevecht.rs
```

Verplaats:

- gevechtsresultaten
- aanval
- verdediging
- schadeberekening
- gevechtsfuncties
- relevante tests.

Controleer daarna:

```text id="g4k6t1"
cargo check
cargo test
```

---

## 47. Oefening: maak `Wereld`

Maak een echte struct:

```rust,ignore
pub struct Wereld {
    // ...
}
```

Laat `Wereld` verantwoordelijk zijn voor:

- locaties
- huidige locatie
- reizen
- zoeken.

Probeer te voorkomen dat `main.rs` rechtstreeks bij de interne velden van `Wereld` kan.

---

## 48. Oefening: maak `Spel`

Maak:

```text id="p3v8m2"
spel.rs
```

met bijvoorbeeld:

```rust,ignore
pub fn speel_spel(
    speler: &mut Speler,
    wereld: &mut Wereld,
) {
    // ...
}
```

`main.rs` moet nu ongeveer alleen nog het programma starten.

---

## 49. Oefening: vind dubbele code

Zoek in het hele project naar stukken code die hetzelfde probleem oplossen.

Bijvoorbeeld:

```text
Hoe wordt gecontroleerd of een speler leeft?
Hoe wordt goud toegevoegd?
Hoe wordt een vijand gevonden?
Hoe wordt een locatie gevonden?
Hoe wordt schade toegepast?
```

Als hetzelfde probleem op meerdere plaatsen wordt opgelost, vraag dan:

> Kunnen we hier één functie of methode van maken?

---

## 50. Oefening: verbeter namen

Zoek namen zoals:

```rust,ignore
doe_ding()
verwerk()
check()
data()
temp()
x()
```

Vraag bij iedere naam:

> Kan ik zonder de functie te openen begrijpen wat deze doet?

Verbeter bijvoorbeeld:

```rust,ignore
verwerk()
```

naar:

```rust,ignore
verwerk_gevechtsresultaat()
```

of:

```rust,ignore
geef_gevechtsbeloning()
```

afhankelijk van wat de functie werkelijk doet.

---

## 51. Rustlings

Maak:

```text id="b8k2r6"
exercises/consolideren/
```

## Functies

### `01_toon_menu.rs`

Verplaats een menu naar een aparte functie.

### `02_speler_status.rs`

Maak een functie die de status toont.

### `03_reis.rs`

Maak een aparte functie voor reizen.

### `04_speel_spel.rs`

Combineer meerdere functies in `speel_spel`.

---

## Structuren

### `05_speler_module.rs`

Zet `Speler` in een module.

### `06_vijand_module.rs`

Zet `Vijand` in een module.

### `07_gevecht_module.rs`

Zet gevechtscode in een module.

### `08_wereld_module.rs`

Maak een `Wereld`-module.

### `09_private_velden.rs`

Maak velden privé en voeg publieke methodes toe.

### `10_interface.rs`

Gebruik de methodes in plaats van rechtstreeks velden te wijzigen.

---

## Refactoring

### `11_verplaats_functie.rs`

Verplaats een functie naar een logischere module.

### `12_verwijder_duplicatie.rs`

Twee stukken code doen hetzelfde. Maak er één functie van.

### `13_betere_naam.rs`

Geef een slecht benoemde functie een duidelijke naam.

### `14_te_grote_functie.rs`

Splits een grote functie op.

### `15_main_kleiner.rs`

Maak `main` kleiner.

---

## Ownership en modules

### `16_debug_private.rs`

Een andere module probeert een privéveld rechtstreeks te wijzigen.

Los dit op met een methode.

### `17_debug_move.rs`

Een waarde wordt verplaatst terwijl een andere functie hem nog nodig heeft.

Gebruik borrowing.

### `18_debug_mut.rs`

Een functie moet een waarde wijzigen maar krijgt geen `&mut`.

Los de functie-aanroep op.

### `19_debug_borrow.rs`

Een mutable borrow duurt te lang.

Beperk de scope zodat de waarde daarna opnieuw gebruikt kan worden.

---

## Tests

### `20_test_refactor.rs`

Refactor een functie zonder het gedrag te veranderen.

De bestaande tests moeten blijven slagen.

---

## 52. Eindopdracht — Consolideer het RPG

Neem het volledige RPG-project van de vorige artikelen.

Voer de volgende stappen uit.

### Stap 1 — Maak een inventaris

Schrijf alle structs, enums, functies en modules op.

---

### Stap 2 — Bepaal verantwoordelijkheden

Schrijf bij ieder onderdeel:

```text
Wat beheert dit?
Wat mag het wijzigen?
Welke informatie heeft het nodig?
Welke informatie hoeft het niet te kennen?
```

---

### Stap 3 — Maak `main.rs` klein

Streef naar iets in deze richting:

```rust,ignore
fn main() {
    let mut speler = Speler::nieuw("Arin");
    let mut wereld = Wereld::nieuw();

    spel::speel_spel(&mut speler, &mut wereld);
}
```

---

### Stap 4 — Maak duidelijke modules

Gebruik bijvoorbeeld:

```text id="c8m5x2"
src/
├── main.rs
├── spel.rs
├── speler.rs
├── vijand.rs
├── npc.rs
├── wereld.rs
├── locatie.rs
├── gevecht.rs
├── ontmoeting.rs
├── inventaris.rs
├── wapen.rs
├── winkel.rs
└── savegame.rs
```

Niet ieder project hoeft exact deze structuur te gebruiken.

Leg bij je eigen structuur uit waarom je voor deze indeling kiest.

---

### Stap 5 — Bescherm interne gegevens

Maak belangrijke velden privé.

Gebruik methodes om gegevens te lezen of wijzigen.

---

### Stap 6 — Verwijder duplicatie

Zoek minstens drie stukken dubbele logica.

Maak daar gedeelde functies of methodes van.

---

### Stap 7 — Verbeter namen

Verbeter minstens vijf namen die niet duidelijk zijn.

---

### Stap 8 — Test

Voer uit:

```text id="j4v8q2"
cargo check
cargo test
cargo run
```

Alle drie zijn belangrijk.

`cargo check` controleert de code.

`cargo test` controleert het gedrag.

`cargo run` laat zien of het spel daadwerkelijk werkt.

---

## 53. Een belangrijke vraag: wanneer stop je met refactoren?

Ook refactoring kan te ver gaan.

Je kunt eindeloos bezig blijven met:

```text
"Deze functie kan misschien nog ergens anders heen."
```

of:

```text
"Misschien moet deze module nog worden opgesplitst."
```

Dat hoeft niet.

Een goede structuur is niet:

> maximaal abstract.

Een goede structuur is:

> begrijpelijk, testbaar en makkelijk genoeg om aan te passen.

Als de leerlingen naar een module kunnen kijken en redelijk snel begrijpen wat die module doet, hebben ze al veel bereikt.

---

## 54. De drie vragen

Bij toekomstige code kunnen we daarom steeds drie vragen stellen.

### 1. Wat doet deze code?

Als dat niet duidelijk is:

- betere naam
- kleinere functie
- duidelijkere structuur.

### 2. Waar hoort deze code thuis?

Als dat niet duidelijk is:

- verantwoordelijkheid bepalen
- functie of methode kiezen
- module kiezen.

### 3. Kan ik deze code veilig veranderen?

Als dat niet duidelijk is:

- tests schrijven
- kleine stappen zetten
- `cargo check`
- `cargo test`.

Deze drie vragen vormen een goede basis voor refactoring.

---

## 55. Refactoring is een programmeervaardigheid

Tot nu toe lag de nadruk vooral op:

> Hoe schrijf ik nieuwe Rust-code?

Nu komt er een tweede vaardigheid bij:

> Hoe zorg ik dat bestaande code begrijpelijk blijft?

Dat is minstens zo belangrijk.

Een groot programma wordt namelijk niet alleen groot doordat er veel code bijkomt.

Het wordt ook groot doordat bestaande code steeds opnieuw wordt aangepast.

Zonder enige structuur kan dat snel veranderen in een programma waarin niemand meer precies weet:

> Waar moet ik dit aanpassen?

Met een goede structuur kunnen we juist zeggen:

```text
"Dit gaat over de speler.
→ speler.rs"

"Dit gaat over reizen.
→ wereld.rs"

"Dit gaat over vechten.
→ gevecht.rs"

"Dit gaat over het hoofdmenu.
→ spel.rs"
```

Dat is precies de richting waarin we met ons RPG-project willen gaan.

---

## 56. Vooruitblik: echte refactoring

In het volgende artikel kunnen we daadwerkelijk gaan refactoren.

Dan nemen we bijvoorbeeld een wat rommelige versie van ons RPG en verbeteren die stap voor stap.

We kunnen daarbij bewust beginnen met code die:

- te veel verantwoordelijkheden heeft
- dubbele code bevat
- onduidelijke namen gebruikt
- te veel `pub` gebruikt
- te veel logica in `main.rs` heeft
- functies heeft die te veel parameters krijgen.

Daarna verbeteren we die code **zonder het gedrag van het spel te veranderen**.

Dat wordt een mooie eerste echte refactoring-oefening:

```text
rommelige maar werkende code
            ↓
       tests schrijven
            ↓
       kleine wijziging
            ↓
          testen
            ↓
       volgende wijziging
            ↓
          testen
            ↓
      duidelijke structuur
```

Daarmee maken we de belangrijke stap van:

**"Ik kan Rust-code schrijven."**

naar:

**"Ik kan een bestaand Rust-programma begrijpen en veilig verbeteren."**

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 36](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_36/).

