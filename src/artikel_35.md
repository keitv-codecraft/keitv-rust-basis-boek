# Rust 35 — Ontmoetingen, NPC's en de spelwereld interactief maken

In het vorige artikel hebben we onze RPG-wereld uitgebreid.

De speler kon:

- locaties bezoeken
- vijanden tegenkomen
- vechten
- ervaring krijgen
- een level omhoog gaan
- goud krijgen
- buit verzamelen
- tussen verbonden locaties reizen.

Onze wereld begint daarmee op een echt spel te lijken.

Toch is er nog een belangrijk verschil met een echte RPG:

> De wereld reageert nog maar weinig op de speler.

In dit artikel brengen we daar verandering in.

We voegen onder andere toe:

- meerdere vijanden op een locatie
- willekeurige ontmoetingen
- NPC's
- eenvoudige gesprekken
- verschillende soorten ontmoetingen
- een duidelijkere spel-lus
- tests voor deze onderdelen.

Daarbij gebruiken we vooral Rust die we al kennen.

---

## 1. Een locatie kan meerdere dingen bevatten

Tot nu toe had een locatie bijvoorbeeld:

```rust,ignore
struct Locatie {
    naam: String,
    beschrijving: String,
    vijand: Option<Vijand>,
}
```

Dat werkt voor één vijand.

Maar stel dat het bos drie wolven bevat.

Dan zouden we bijvoorbeeld kunnen schrijven:

```rust,ignore
struct Locatie {
    naam: String,
    beschrijving: String,
    vijanden: Vec<Vijand>,
}
```

Nu kan een locatie nul, één of honderd vijanden hebben.

Een leeg gebied:

```rust,ignore
let dorp = Locatie {
    naam: String::from("Dorp"),
    beschrijving: String::from("Een rustig dorp."),
    vijanden: Vec::new(),
};
```

Een bos:

```rust,ignore
let bos = Locatie {
    naam: String::from("Donker Bos"),
    beschrijving: String::from("Je hoort wolven tussen de bomen."),
    vijanden: vec![
        Vijand::nieuw("Wolf", 40, 20, 5),
        Vijand::nieuw("Wolf", 40, 20, 5),
        Vijand::nieuw("Grote Wolf", 70, 40, 10),
    ],
};
```

Hier gebruiken we alleen bekende concepten:

- `struct`
- `Vec`
- `String`
- een constructor
- `Vec::new`
- `vec!`.

---

## 2. Alle vijanden tonen

We kunnen de vijanden van een locatie tonen:

```rust,ignore
impl Locatie {
    fn toon_vijanden(&self) {
        for vijand in &self.vijanden {
            println!(
                "{} — {} gezondheid",
                vijand.naam,
                vijand.gezondheid
            );
        }
    }
}
```

Waarom gebruiken we:

```rust,ignore
&self.vijanden
```

?

Omdat we de vijanden alleen bekijken.

We willen ze niet uit de `Vec` halen.

---

## 3. Alleen levende vijanden tonen

Een verslagen vijand kan bijvoorbeeld nog in de `Vec` staan.

We kunnen dan filteren:

```rust,ignore
impl Locatie {
    fn toon_levende_vijanden(&self) {
        for vijand in self.vijanden.iter().filter(|v| v.is_levend()) {
            println!("{}", vijand.naam);
        }
    }
}
```

Hier gebruiken we bekende iteratorfunctionaliteit:

```rust,ignore
iter()
filter()
```

en een closure:

```rust,ignore
|v| v.is_levend()
```

We hoeven dus geen aparte `if` in de `for`-lus te schrijven.

Maar deze versie is ook prima:

```rust,ignore
impl Locatie {
    fn toon_levende_vijanden(&self) {
        for vijand in &self.vijanden {
            if vijand.is_levend() {
                println!("{}", vijand.naam);
            }
        }
    }
}
```

Voor beginners is deze tweede versie vaak makkelijker te lezen.

Dat is een belangrijk punt:

> Kortere code is niet automatisch betere code.

Gebruik een iterator wanneer die de bedoeling duidelijk maakt.

---

## 4. Een vijand zoeken

We kunnen een specifieke vijand zoeken.

```rust,ignore
fn vind_vijand(
    vijanden: &[Vijand],
    naam: &str,
) -> Option<&Vijand> {
    vijanden
        .iter()
        .find(|vijand| vijand.naam == naam)
}
```

De functie geeft dus:

```text
Some(&Vijand)
```

als de vijand bestaat.

En:

```text
None
```

als hij niet bestaat.

We zien hier opnieuw een patroon dat inmiddels bekend moet worden:

```text
zoeken
    ↓
misschien gevonden
    ↓
Option
```

---

## 5. Een willekeurige vijand kiezen

We kunnen ook een willekeurige vijand kiezen.

Stel dat een locatie drie vijanden heeft.

Dan kunnen we een willekeurige index kiezen:

```rust,ignore
let index = rand::random_range(0..vijanden.len());
```

Daarna:

```rust,ignore
let vijand = &vijanden[index];
```

Er zit echter een probleem in.

Wat gebeurt er als:

```rust,ignore
vijanden.len()
```

gelijk is aan `0`?

Dan proberen we een willekeurige waarde uit:

```text
0..0
```

te kiezen.

Dat levert geen bruikbare index op.

Daarom moeten we eerst controleren:

```rust,ignore
if vijanden.is_empty() {
    println!("Er zijn geen vijanden.");
} else {
    let index = rand::random_range(0..vijanden.len());
    let vijand = &vijanden[index];

    println!("Je ontmoet een {}!", vijand.naam);
}
```

Dit is een belangrijk programmeerpatroon:

> Controleer eerst of een waarde geldig is voordat je haar gebruikt.

---

## 6. Een functie voor een willekeurige vijand

We kunnen dit gedrag in een functie stoppen.

```rust,ignore
fn willekeurige_vijand(vijanden: &[Vijand]) -> Option<&Vijand> {
    if vijanden.is_empty() {
        return None;
    }

    let index = rand::random_range(0..vijanden.len());

    Some(&vijanden[index])
}
```

Nu kunnen we schrijven:

```rust,ignore
match willekeurige_vijand(&bos.vijanden) {
    Some(vijand) => println!("Je ontmoet een {}!", vijand.naam),
    None => println!("Hier zijn geen vijanden."),
}
```

Let op hoeveel bestaande concepten hierin samenkomen:

- slice `&[Vijand]`
- `Option`
- `if`
- `return`
- `rand`
- indexeren
- borrowing.

---

## 7. Een ontmoeting is niet altijd een gevecht

Tot nu toe betekende een ontmoeting meestal:

> Er is een vijand → vechten.

Maar in een RPG kan er van alles gebeuren.

Bijvoorbeeld:

- een vijand
- een handelaar
- een vriendelijke reiziger
- een schatkist
- niets.

We kunnen dat modelleren met een enum.

```rust,ignore
enum Ontmoeting {
    Vijand(Vijand),
    Handelaar,
    Reiziger,
    Schatkist,
    Niets,
}
```

Een belangrijke eigenschap van enums is dat een variant zelf gegevens kan bevatten.

Bijvoorbeeld:

```rust,ignore
Ontmoeting::Vijand(vijand)
```

bevat een `Vijand`.

Terwijl:

```rust,ignore
Ontmoeting::Handelaar
```

geen extra gegevens bevat.

---

## 8. Een ontmoeting afhandelen

We kunnen nu `match` gebruiken:

```rust,ignore
fn handel_ontmoeting(
    speler: &mut Speler,
    ontmoeting: Ontmoeting,
) {
    match ontmoeting {
        Ontmoeting::Vijand(mut vijand) => {
            println!("Een {} valt je aan!", vijand.naam);
            vecht(speler, &mut vijand);
        }

        Ontmoeting::Handelaar => {
            println!("Je hebt een handelaar gevonden.");
        }

        Ontmoeting::Reiziger => {
            println!("Een reiziger begroet je.");
        }

        Ontmoeting::Schatkist => {
            println!("Je vindt een schatkist!");
        }

        Ontmoeting::Niets => {
            println!("Er gebeurt niets bijzonders.");
        }
    }
}
```

Dit is een belangrijke toepassing van enums.

De enum beschrijft:

> Welke soort gebeurtenis is er gebeurd?

De `match` bepaalt:

> Wat moeten we doen?

---

## 9. Een NPC

Een NPC is een personage dat niet door de speler wordt bestuurd.

We kunnen beginnen met een eenvoudige struct:

```rust,ignore
struct Npc {
    naam: String,
    beschrijving: String,
}
```

Bijvoorbeeld:

```rust,ignore
let herbergier = Npc {
    naam: String::from("Mira"),
    beschrijving: String::from(
        "De eigenaar van de dorpsherberg."
    ),
};
```

We kunnen een constructor maken:

```rust,ignore
impl Npc {
    fn nieuw(naam: &str, beschrijving: &str) -> Self {
        Self {
            naam: String::from(naam),
            beschrijving: String::from(beschrijving),
        }
    }
}
```

---

## 10. Gesprekken

Een eenvoudige NPC kan een lijst met zinnen hebben:

```rust,ignore
struct Npc {
    naam: String,
    beschrijving: String,
    dialogen: Vec<String>,
}
```

Bijvoorbeeld:

```rust,ignore
let herbergier = Npc {
    naam: String::from("Mira"),
    beschrijving: String::from(
        "De eigenaar van de dorpsherberg."
    ),
    dialogen: vec![
        String::from("Welkom in mijn herberg."),
        String::from("Pas op voor de wolven in het bos."),
        String::from("Er gaan vreemde verhalen rond over de ruïne."),
    ],
};
```

We kunnen alle dialogen tonen:

```rust,ignore
impl Npc {
    fn praat(&self) {
        for zin in &self.dialogen {
            println!("{}: {}", self.naam, zin);
        }
    }
}
```

Dat is een heel eenvoudige vorm van dialoog.

---

## 11. Een gesprek met één zin

We kunnen ook één zin tegelijk tonen.

```rust,ignore
impl Npc {
    fn zeg(&self, nummer: usize) {
        if nummer < self.dialogen.len() {
            println!(
                "{}: {}",
                self.naam,
                self.dialogen[nummer]
            );
        }
    }
}
```

Waarom controleren we:

```rust,ignore
nummer < self.dialogen.len()
```

?

Omdat dit anders fout kan gaan:

```rust,ignore
self.dialogen[nummer]
```

Als de `Vec` bijvoorbeeld drie zinnen bevat, zijn de geldige indexen:

```text
0
1
2
```

Niet:

```text
3
```

Dit is een voorbeeld van een fout die niet door de compiler hoeft te worden gevonden.

De code kan geldig Rust zijn, maar tijdens het uitvoeren kan een ongeldige index een probleem veroorzaken.

---

## 12. Een betere interface

We kunnen ook `Option` gebruiken.

```rust,ignore
impl Npc {
    fn dialoog(&self, nummer: usize) -> Option<&str> {
        self.dialogen.get(nummer).map(|tekst| tekst.as_str())
    }
}
```

Nu krijgen we:

```rust,ignore
Some("Welkom in mijn herberg.")
```

of:

```rust,ignore
None
```

Dit is een mooi voorbeeld van hoe meerdere bestaande concepten samenwerken:

```text
Vec
 ↓
get()
 ↓
Option
 ↓
map()
```

De functie kan daardoor niet zomaar buiten de `Vec` lezen.

---

## 13. NPC's op een locatie

Een locatie kan NPC's bevatten:

```rust,ignore
struct Locatie {
    naam: String,
    beschrijving: String,
    vijanden: Vec<Vijand>,
    npcs: Vec<Npc>,
}
```

Een dorp kan bijvoorbeeld hebben:

```rust,ignore
let dorp = Locatie {
    naam: String::from("Dorp"),
    beschrijving: String::from(
        "Een klein dorp aan de rand van het bos."
    ),
    vijanden: Vec::new(),
    npcs: vec![
        Npc::nieuw(
            "Mira",
            "De eigenaar van de herberg."
        ),
        Npc::nieuw(
            "Bram",
            "Een oude jager."
        ),
    ],
};
```

We kunnen ze tonen:

```rust,ignore
impl Locatie {
    fn toon_npcs(&self) {
        for npc in &self.npcs {
            println!("{} — {}", npc.naam, npc.beschrijving);
        }
    }
}
```

---

## 14. Een NPC zoeken

We kunnen dezelfde techniek gebruiken als bij locaties en vijanden.

```rust,ignore
fn vind_npc<'a>(
    npcs: &'a [Npc],
    naam: &str,
) -> Option<&'a Npc> {
    npcs
        .iter()
        .find(|npc| npc.naam == naam)
}
```

De functie zegt:

> Zoek in deze lijst een NPC met deze naam.

Als hij bestaat:

```text
Some(&Npc)
```

anders:

```text
None
```

We zien opnieuw dat dezelfde programmeertechniek op verschillende soorten gegevens werkt.

---

## 15. Waarom `Vec` hier handig is

Een dorp kan:

```text
0 NPC's
```

hebben.

Of:

```text
1 NPC
```

Of:

```text
20 NPC's
```

Daarom is:

```rust,ignore
Vec<Npc>
```

een logische keuze.

Hetzelfde geldt voor vijanden:

```rust,ignore
Vec<Vijand>
```

en voor dialogen:

```rust,ignore
Vec<String>
```

Een `Vec` is dus niet alleen "een lijst met getallen".

Het kan bijna iedere verzameling waarden bevatten.

---

## 16. Willekeurige ontmoetingen

We kunnen nu een eenvoudige willekeurige ontmoeting maken.

Bijvoorbeeld:

```rust,ignore
fn willekeurige_ontmoeting() -> Ontmoeting {
    let getal = rand::random_range(1..=100);

    match getal {
        1..=50 => Ontmoeting::Niets,
        51..=75 => Ontmoeting::Reiziger,
        76..=90 => Ontmoeting::Schatkist,
        91..=100 => Ontmoeting::Handelaar,
        _ => Ontmoeting::Niets,
    }
}
```

Hier gebruiken we ranges in een `match`.

Bijvoorbeeld:

```rust,ignore
51..=75
```

betekent:

```text
51 tot en met 75
```

De kansen zijn hier:

```text
1–50    niets
51–75   reiziger
76–90   schatkist
91–100  handelaar
```

Dit is slechts een voorbeeld.

De getallen zijn onderdeel van het spelontwerp.

---

## 17. Een vijand toevoegen aan de willekeurige ontmoeting

We kunnen ook een vijand meegeven.

```rust,ignore
fn willekeurige_ontmoeting(
    vijanden: &[Vijand],
) -> Ontmoeting {
    let getal = rand::random_range(1..=100);

    match getal {
        1..=50 => Ontmoeting::Niets,

        51..=70 => Ontmoeting::Reiziger,

        71..=85 => Ontmoeting::Schatkist,

        86..=95 => {
            if let Some(vijand) = willekeurige_vijand(vijanden) {
                Ontmoeting::Vijand(vijand.clone())
            } else {
                Ontmoeting::Niets
            }
        }

        96..=100 => Ontmoeting::Handelaar,

        _ => Ontmoeting::Niets,
    }
}
```

Hier zien we een interessant ownership-vraagstuk.

`willekeurige_vijand()` geeft:

```rust,ignore
Option<&Vijand>
```

terug.

Maar `Ontmoeting::Vijand(...)` moet een echte `Vijand` bevatten.

Daarom gebruiken we:

```rust,ignore
vijand.clone()
```

We maken dus een kopie van de vijand.

Dit is hier bewust eenvoudig gehouden.

Later kunnen we nadenken over een ontwerp waarbij ontmoetingen verwijzen naar bestaande vijanden in plaats van ze te kopiëren.

---

## 18. Niet alles hoeft willekeurig te zijn

Een veelgemaakte beginnersfout is denken:

> Een RPG moet alles random maken.

Dat hoeft helemaal niet.

Sommige dingen kunnen vaststaan.

Bijvoorbeeld:

```text
Dorp:
    Mira is altijd aanwezig.

Bos:
    meestal een willekeurige vijand.

Ruïne:
    altijd een oude schatkist.

Berg:
    aan het einde wacht een draak.
```

Een goede game gebruikt vaak een combinatie van:

- vaste gebeurtenissen
- willekeurige gebeurtenissen
- gebeurtenissen die afhangen van de keuzes van de speler.

---

## 19. Een gebeurtenis als enum

We kunnen verschillende soorten gebeurtenissen nog duidelijker modelleren:

```rust,ignore
enum Gebeurtenis {
    Gevecht(Vijand),
    Gesprek(Npc),
    Schat(u32),
    Handel,
    Niets,
}
```

Hiermee kunnen we bijvoorbeeld zeggen:

```rust,ignore
Gebeurtenis::Schat(50)
```

Dat betekent:

> De speler heeft een schat gevonden met 50 goud.

Of:

```rust,ignore
Gebeurtenis::Gesprek(npc)
```

Dat betekent:

> De speler ontmoet deze NPC.

---

## 20. Gebeurtenissen uitvoeren

Een centrale functie kan bepalen wat er moet gebeuren:

```rust,ignore
fn voer_gebeurtenis_uit(
    speler: &mut Speler,
    gebeurtenis: Gebeurtenis,
) {
    match gebeurtenis {
        Gebeurtenis::Gevecht(mut vijand) => {
            println!("Een {} verschijnt!", vijand.naam);
            vecht(speler, &mut vijand);
        }

        Gebeurtenis::Gesprek(npc) => {
            println!("Je spreekt met {}.", npc.naam);
            npc.praat();
        }

        Gebeurtenis::Schat(goud) => {
            println!("Je vindt {} goud!", goud);
            speler.geef_goud(goud);
        }

        Gebeurtenis::Handel => {
            println!("Je vindt een handelaar.");
        }

        Gebeurtenis::Niets => {
            println!("Er gebeurt niets.");
        }
    }
}
```

Dit is een heel nuttig patroon voor grotere programma's.

Een enum beschrijft:

> Wat voor gebeurtenis is dit?

Een functie die `match` gebruikt bepaalt:

> Wat doen we met die gebeurtenis?

---

## 21. Een speltoestand

We kunnen hetzelfde idee gebruiken voor de toestand van het spel.

Bijvoorbeeld:

```rust,ignore
enum Speltoestand {
    Spelen,
    Gevecht,
    Gesprek,
    GameOver,
    Gewonnen,
}
```

De hoofdloop kan dan afhankelijk zijn van de toestand.

Bijvoorbeeld:

```rust,ignore
let mut toestand = Speltoestand::Spelen;

loop {
    match toestand {
        Speltoestand::Spelen => {
            println!("Je kunt reizen.");
        }

        Speltoestand::Gevecht => {
            println!("Je bent in gevecht.");
        }

        Speltoestand::Gesprek => {
            println!("Je bent in gesprek.");
        }

        Speltoestand::GameOver => {
            println!("Het spel is afgelopen.");
            break;
        }

        Speltoestand::Gewonnen => {
            println!("Je hebt gewonnen!");
            break;
        }
    }
}
```

Hiermee krijgen we een eenvoudig **state machine**-achtig systeem.

Het programma bevindt zich steeds in één toestand.

---

## 22. Toestanden veranderen

Een toestand kan veranderen.

Bijvoorbeeld:

```rust,ignore
toestand = Speltoestand::Gevecht;
```

Daarna zal de volgende keer door de `loop` een andere tak van de `match` worden uitgevoerd.

Dit idee is erg belangrijk voor spellen.

Een spel kan bijvoorbeeld deze toestanden hebben:

```text
Spelen
  ↓
Gevecht
  ↓
Spelen
```

Of:

```text
Spelen
  ↓
Gesprek
  ↓
Spelen
```

Of:

```text
Spelen
  ↓
Gevecht
  ↓
GameOver
```

---

## 23. Waarom een enum beter is dan losse booleans

Je zou kunnen proberen dit te maken:

```rust,ignore
let in_gevecht = false;
let in_gesprek = false;
let game_over = false;
```

Maar dan kunnen ongeldige combinaties ontstaan:

```text
in_gevecht = true
in_gesprek = true
game_over = true
```

Wat betekent dat?

Bij een enum:

```rust,ignore
enum Speltoestand {
    Spelen,
    Gevecht,
    Gesprek,
    GameOver,
}
```

kan de speler maar één toestand tegelijk hebben.

Dat maakt het ontwerp duidelijker.

---

## 24. De hoofdloop

We kunnen nu een eenvoudige hoofdloop ontwerpen:

```rust,ignore
let mut toestand = Speltoestand::Spelen;

loop {
    match toestand {
        Speltoestand::Spelen => {
            println!("1. Reizen");
            println!("2. Status");
            println!("3. Stoppen");

            let keuze = lees_getal();

            match keuze {
                1 => {
                    // reizen
                }

                2 => {
                    // status
                }

                3 => {
                    toestand = Speltoestand::GameOver;
                }

                _ => {
                    println!("Ongeldige keuze.");
                }
            }
        }

        Speltoestand::Gevecht => {
            // gevecht afhandelen
        }

        Speltoestand::Gesprek => {
            // gesprek afhandelen
        }

        Speltoestand::GameOver => {
            println!("Tot ziens!");
            break;
        }

        Speltoestand::Gewonnen => {
            println!("Je hebt het spel uitgespeeld!");
            break;
        }
    }
}
```

Dit is nog geen volledig spel.

Dat hoeft ook niet.

We zijn eerst het ontwerp aan het opbouwen.

---

## 25. Een belangrijke ontwerpverbetering

In bovenstaande code staat veel logica in `main`.

Dat willen we uiteindelijk vermijden.

In plaats daarvan kunnen we functies maken:

```rust,ignore
fn speel_ronde(
    speler: &mut Speler,
    wereld: &mut Wereld,
) -> Speltoestand {
    // ...
}
```

De functie kan bijvoorbeeld teruggeven:

```rust,ignore
Speltoestand::Spelen
```

of:

```rust,ignore
Speltoestand::Gevecht
```

of:

```rust,ignore
Speltoestand::GameOver
```

De hoofdloop wordt dan veel eenvoudiger:

```rust,ignore
let mut toestand = Speltoestand::Spelen;

loop {
    toestand = match toestand {
        Speltoestand::Spelen => speel_ronde(
            &mut speler,
            &mut wereld,
        ),

        Speltoestand::Gevecht => {
            speel_gevecht(&mut speler, &mut wereld)
        }

        Speltoestand::Gesprek => {
            speel_gesprek(&mut speler, &mut wereld)
        }

        Speltoestand::GameOver => {
            break;
        }

        Speltoestand::Gewonnen => {
            break;
        }
    };
}
```

Dit is een eerste voorbeeld van een programma waarin `main` vooral de onderdelen aanstuurt.

Dat is een goed doel voor ons RPG.

---

## 26. Tests voor willekeurige code

Willekeurige code is lastig rechtstreeks te testen.

Stel dat:

```rust,ignore
willekeurige_ontmoeting()
```

soms een vijand geeft en soms een schatkist.

Dan kan deze test niet betrouwbaar zijn:

```rust,ignore
assert_eq!(
    willekeurige_ontmoeting(),
    Ontmoeting::Schat
);
```

De uitkomst is immers willekeurig.

Daarom testen we liever eigenschappen.

Bijvoorbeeld:

```text
Een willekeurige ontmoeting moet altijd één geldige variant opleveren.
```

Of we testen de onderdelen zonder willekeur.

Bijvoorbeeld:

```rust,ignore
fn maak_schat(goud: u32) -> Gebeurtenis {
    Gebeurtenis::Schat(goud)
}
```

Daar kunnen we wel goed een test voor schrijven.

---

## 27. Testen van NPC-dialogen

Een eenvoudige test:

```rust,ignore
#[test]
fn npc_heeft_dialogen() {
    let npc = Npc {
        naam: String::from("Mira"),
        beschrijving: String::from("Herbergier"),
        dialogen: vec![
            String::from("Welkom."),
            String::from("Pas op."),
        ],
    };

    assert_eq!(
        npc.dialogen.len(),
        2
    );
}
```

En:

```rust,ignore
#[test]
fn npc_eerste_dialoog() {
    let npc = Npc {
        naam: String::from("Mira"),
        beschrijving: String::from("Herbergier"),
        dialogen: vec![
            String::from("Welkom."),
        ],
    };

    assert_eq!(
        npc.dialoog(0),
        Some("Welkom.")
    );
}
```

We testen hier niet alleen code.

We testen ook of ons **ontwerp** doet wat we verwachten.

---

## 28. Testen van ongeldige dialogen

We moeten ook de grens testen:

```rust,ignore
#[test]
fn npc_ongeldige_dialoog() {
    let npc = Npc {
        naam: String::from("Mira"),
        beschrijving: String::from("Herbergier"),
        dialogen: vec![
            String::from("Welkom."),
        ],
    };

    assert_eq!(
        npc.dialoog(99),
        None
    );
}
```

Dit is een goede test omdat `99` duidelijk geen geldige index is.

---

## 29. Een nuttige gewoonte: denk in situaties

Wanneer je een functie schrijft, vraag jezelf dan af:

> Welke situaties bestaan er?

Bijvoorbeeld:

```rust,ignore
fn vind_npc(...)
```

Mogelijke situaties:

```text
NPC bestaat
NPC bestaat niet
```

Daarom:

```rust,ignore
Option<&Npc>
```

Bij:

```rust,ignore
fn willekeurige_vijand(...)
```

zijn er:

```text
vijanden aanwezig
geen vijanden aanwezig
```

Daarom:

```rust,ignore
Option<&Vijand>
```

Bij:

```rust,ignore
fn voer_gebeurtenis_uit(...)
```

zijn er meerdere soorten gebeurtenissen.

Daarom:

```rust,ignore
enum Gebeurtenis
```

Dit is een belangrijke programmeervaardigheid:

> Denk eerst na over de mogelijke situaties en kies daarna een datatype dat die situaties goed kan beschrijven.

---

## 30. Oefening 1 — Meerdere vijanden

Pas `Locatie` aan zodat deze:

```rust,ignore
Vec<Vijand>
```

bevat.

Maak een bos met:

- 2 wolven
- 1 goblin
- 1 sterke wolf.

Maak een methode:

```rust,ignore
toon_vijanden()
```

die ze allemaal toont.

---

## 31. Oefening 2 — Alleen levende vijanden

Maak:

```rust,ignore
toon_levende_vijanden()
```

Gebruik eerst een gewone `for`-lus en `if`.

Maak daarna een tweede versie met:

```rust,ignore
filter()
```

Vergelijk beide versies.

Vraag jezelf af:

> Welke versie vind ik makkelijker te lezen?

---

## 32. Oefening 3 — Willekeurige vijand

Schrijf:

```rust,ignore
fn willekeurige_vijand(
    vijanden: &[Vijand]
) -> Option<&Vijand>
```

Test:

- een lege lijst
- een lijst met één vijand
- een lijst met meerdere vijanden.

Let vooral op het geval:

```text
vijanden.len() == 0
```

---

## 33. Oefening 4 — Ontmoeting enum

Maak:

```rust,ignore
enum Ontmoeting {
    Vijand(Vijand),
    Handelaar,
    Reiziger,
    Schatkist,
    Niets,
}
```

Schrijf daarna een functie die iedere variant een andere tekst laat tonen.

Gebruik `match`.

---

## 34. Oefening 5 — NPC

Maak:

```rust,ignore
struct Npc {
    naam: String,
    beschrijving: String,
    dialogen: Vec<String>,
}
```

Maak:

```rust,ignore
Npc::nieuw(...)
```

en:

```rust,ignore
npc.praat()
```

---

## 35. Oefening 6 — Dialoog kiezen

Maak:

```rust,ignore
fn dialoog(&self, nummer: usize) -> Option<&str>
```

Test:

```text
0 → eerste zin
1 → tweede zin
geldige laatste index → laatste zin
te hoge index → None
```

---

## 36. Oefening 7 — NPC zoeken

Maak:

```rust,ignore
fn vind_npc(
    npcs: &[Npc],
    naam: &str,
) -> Option<&Npc>
```

Gebruik:

```rust,ignore
iter()
find()
```

Test zowel een bestaande als een onbekende NPC.

---

## 37. Oefening 8 — Gebeurtenissen

Maak:

```rust,ignore
enum Gebeurtenis {
    Gevecht(Vijand),
    Gesprek(Npc),
    Schat(u32),
    Handel,
    Niets,
}
```

Schrijf:

```rust,ignore
voer_gebeurtenis_uit(...)
```

en zorg dat iedere variant iets anders doet.

---

## 38. Oefening 9 — Speltoestand

Maak:

```rust,ignore
enum Speltoestand {
    Spelen,
    Gevecht,
    Gesprek,
    GameOver,
    Gewonnen,
}
```

Schrijf een kleine `loop` waarin de speler tussen toestanden kan wisselen.

Bijvoorbeeld:

```text
Spelen
  ↓
Gesprek
  ↓
Spelen
```

en:

```text
Spelen
  ↓
GameOver
```

---

## 39. Oefening 10 — Willekeurige ontmoetingen

Maak een functie:

```rust,ignore
fn willekeurige_ontmoeting() -> Gebeurtenis
```

Gebruik `rand::random_range`.

Bedenk zelf de kansen.

Bijvoorbeeld:

```text
50% niets
20% reiziger
15% schat
10% vijand
5% handelaar
```

Probeer vervolgens de functie honderd keer uit te voeren en te tellen wat er gebeurt.

---

## 40. Oefening 11 — Een levend dorp

Maak een dorp met minstens drie NPC's.

Iedere NPC heeft:

- een naam
- een beschrijving
- minstens drie dialogen.

Laat de speler:

1. de NPC's bekijken
2. een NPC kiezen
3. een gesprek voeren
4. terugkeren naar het hoofdmenu.

---

## 41. Oefening 12 — Een levende wereld

Breid de wereld uit met:

- minstens vijf locaties
- minstens vijf NPC's
- meerdere vijanden
- minstens één winkel
- minstens één locatie met willekeurige ontmoetingen.

De speler moet vrij kunnen rondreizen.

---

## 42. Rustlings

Maak:

```text
exercises/ontmoetingen/
```

## Meerdere vijanden

### `01_locatie_vijanden.rs`

Voeg `Vec<Vijand>` toe aan `Locatie`.

### `02_meerdere_vijanden.rs`

Maak een locatie met meerdere vijanden.

### `03_toon_vijanden.rs`

Loop door de vijanden.

### `04_levende_vijanden.rs`

Toon alleen levende vijanden.

### `05_filter_vijanden.rs`

Gebruik `filter`.

### `06_vind_vijand.rs`

Gebruik `find`.

---

## Willekeur

### `07_leeg_vijanden.rs`

Behandel een lege lijst veilig.

### `08_random_index.rs`

Kies een geldige willekeurige index.

### `09_random_vijand.rs`

Maak:

```rust,ignore
willekeurige_vijand()
```

### `10_random_debug.rs`

De code kan crashen wanneer de lijst leeg is.

Zoek uit waarom.

Los het op met een controle.

---

## Enums

### `11_ontmoeting_enum.rs`

Maak de enum `Ontmoeting`.

### `12_ontmoeting_match.rs`

Gebruik `match`.

### `13_ontmoeting_data.rs`

Gebruik een enumvariant die een `Vijand` bevat.

### `14_ontmoeting_resultaat.rs`

Voeg een nieuwe variant toe en pas alle `match`-expressies aan.

---

## NPC's

### `15_npc.rs`

Maak de struct `Npc`.

### `16_npc_nieuw.rs`

Maak de constructor.

### `17_npc_praat.rs`

Laat alle dialogen zien.

### `18_npc_dialoog.rs`

Geef één dialoogzin terug.

### `19_npc_ongeldig.rs`

Laat een ongeldige index `None` opleveren.

### `20_npc_vinden.rs`

Zoek een NPC met `find`.

---

## Gebeurtenissen

### `21_gebeurtenis.rs`

Maak de enum `Gebeurtenis`.

### `22_schat.rs`

Verwerk een schat met een hoeveelheid goud.

### `23_gesprek.rs`

Verwerk een gesprek met een NPC.

### `24_gevecht.rs`

Verwerk een gevecht.

### `25_gebeurtenis_match.rs`

Verwerk alle varianten met `match`.

---

## Speltoestand

### `26_speltoestand.rs`

Maak `Speltoestand`.

### `27_toestand_match.rs`

Gebruik een `match` om verschillende toestanden te verwerken.

### `28_toestand_wijzigen.rs`

Laat een toestand veranderen.

### `29_spel_loop.rs`

Maak een eenvoudige spel-lus.

### `30_spel_stoppen.rs`

Laat de loop eindigen met `break`.

---

## Tests

### `31_test_npc.rs`

Test het aantal dialogen.

### `32_test_dialoog.rs`

Test een geldige dialoog.

### `33_test_ongeldige_dialoog.rs`

Test `None`.

### `34_test_vijanden.rs`

Test een locatie met meerdere vijanden.

### `35_test_lege_locatie.rs`

Test een locatie zonder vijanden.

---

## Debug-opgaven

### `36_debug_index.rs`

De code gebruikt een index die buiten de `Vec` kan vallen.

Zoek het probleem.

### `37_debug_borrow.rs`

De code probeert een geleende waarde tegelijk te gebruiken en te wijzigen.

Los de borrowing-fout op.

### `38_debug_move_npc.rs`

Een `Npc` wordt verplaatst en daarna opnieuw gebruikt.

Los dit op met borrowing of `clone()`.

### `39_debug_option.rs`

De code behandelt `Option` alsof er altijd een waarde aanwezig is.

Gebruik `match` of `if let`.

---

## Eindopgave

### `40_final_ontmoetingen.rs`

Bouw een klein interactief RPG-gedeelte waarin:

- de wereld meerdere locaties heeft
- locaties meerdere vijanden kunnen hebben
- locaties NPC's kunnen hebben
- NPC's dialogen hebben
- de speler NPC's kan aanspreken
- de speler vijanden kan tegenkomen
- er willekeurige ontmoetingen kunnen plaatsvinden
- gebeurtenissen met een enum worden beschreven
- de speltoestand met een enum wordt bijgehouden
- gevechten gebruikmaken van het bestaande gevechtssysteem
- de speler XP, goud en buit kan krijgen
- het hoofdprogramma een duidelijke spel-lus heeft.

Schrijf minimaal **acht tests**.

---

## 43. Extra uitdaging — keuzes in dialogen

We kunnen dialogen interessanter maken.

Een NPC zou bijvoorbeeld kunnen zeggen:

```text
Mira:
"Welkom in mijn herberg."

1. "Wat weet je over het bos?"
2. "Waar is de winkel?"
3. "Tot ziens."
```

We kunnen keuzes modelleren met een enum:

```rust,ignore
enum DialoogKeuze {
    Bos,
    Winkel,
    Vertrekken,
}
```

Daarna:

```rust,ignore
match keuze {
    DialoogKeuze::Bos => {
        println!("Mira: Pas op voor de wolven.");
    }

    DialoogKeuze::Winkel => {
        println!("Mira: De winkel is naast de put.");
    }

    DialoogKeuze::Vertrekken => {
        println!("Mira: Tot ziens.");
    }
}
```

Ook hier gebruiken we bijna uitsluitend bekende concepten.

---

## 44. Extra uitdaging — opdrachten van NPC's

Een NPC kan de speler een opdracht geven.

Bijvoorbeeld:

> "Versla drie wolven."

We kunnen een eenvoudige struct maken:

```rust,ignore
struct Opdracht {
    beschrijving: String,
    doel: u32,
    voortgang: u32,
    beloning: u32,
}
```

Bijvoorbeeld:

```rust,ignore
let opdracht = Opdracht {
    beschrijving: String::from("Versla drie wolven."),
    doel: 3,
    voortgang: 0,
    beloning: 100,
};
```

Na een overwinning:

```rust,ignore
opdracht.voortgang += 1;
```

En:

```rust,ignore
if opdracht.voortgang >= opdracht.doel {
    println!("Opdracht voltooid!");
}
```

Dit is opnieuw vooral een combinatie van bestaande technieken.

---

## 45. Extra uitdaging — opdrachten met een enum

Je kunt het nog flexibeler maken.

Bijvoorbeeld:

```rust,ignore
enum OpdrachtType {
    VerslaVijanden,
    VindItem,
    BezoekLocatie,
}
```

Een opdracht kan dan bevatten:

```rust,ignore
struct Opdracht {
    beschrijving: String,
    soort: OpdrachtType,
    doel: u32,
    voortgang: u32,
    beloning: u32,
}
```

Nu kan hetzelfde systeem verschillende soorten opdrachten ondersteunen.

Een belangrijke ontwerpgedachte ontstaat hier:

> Gebruik een `enum` wanneer er verschillende soorten van hetzelfde concept bestaan.

---

## 46. Wat hebben we deze keer vooral geoefend?

We hebben weinig nieuwe Rust geleerd.

We hebben vooral bestaande kennis opnieuw gebruikt.

| Bekend concept | Nieuwe toepassing |
| --- | --- |
| `struct` | NPC, gebeurtenisgegevens |
| `enum` | ontmoetingen en speltoestanden |
| `Vec` | NPC's, vijanden en dialogen |
| `Option` | zoeken en mogelijke waarden |
| `match` | gebeurtenissen en toestanden |
| `if let` | optionele vijanden |
| `loop` | hoofdspel |
| `break` | spel beëindigen |
| borrowing | NPC's en vijanden bekijken |
| mutable borrowing | gevechten en speltoestand |
| closures | `find` en `filter` |
| iterators | zoeken en filteren |
| `HashMap` | inventaris en andere bestaande systemen |
| tests | grenzen en gedrag |
| modules | onderdelen van het RPG |

Het belangrijkste nieuwe idee is eigenlijk niet een stukje syntax.

Het is **modelleren**.

We vragen ons steeds af:

> Welke dingen bestaan er in mijn programma?

Bijvoorbeeld:

```text
Speler
Vijand
NPC
Locatie
Wereld
Gebeurtenis
Opdracht
Speltoestand
```

Daarna vragen we:

> Welke informatie hoort bij zo'n ding?

En:

> Welk gedrag hoort erbij?

En tenslotte:

> Welke situaties kan dit ding hebben?

Daar komen de Rust-datatypen vanzelf steeds beter bij passen.

---

## 47. De volgende stap

Ons RPG heeft inmiddels behoorlijk veel onderdelen.

We hebben:

```text
                    ┌─────────────┐
                    │   Speler    │
                    └──────┬──────┘
                           │
             ┌─────────────┼─────────────┐
             │             │             │
         Inventaris      Wapen          XP
             │                           │
             │                         Level
             │
             ▼
         ┌────────┐
         │ Wereld │
         └───┬────┘
             │
        ┌────┼────┐
        ▼    ▼    ▼
      Dorp  Bos  Ruïne
       │     │     │
      NPC   Vijand Vijand
       │     │
    Dialoog Gevecht
              │
        ┌─────┼─────┐
        ▼     ▼     ▼
       XP    Goud   Buit
```

We beginnen daarmee tegen een echte grens aan te lopen:

> Hoe zorgen we ervoor dat al deze onderdelen netjes met elkaar samenwerken zonder dat `main.rs` een enorme hoeveelheid code wordt?

Dat wordt de volgende belangrijke stap.

We gaan dan niet zomaar nóg meer functionaliteit toevoegen, maar kijken naar **het ontwerp van het complete RPG**, verantwoordelijkheden tussen modules en hoe we de bestaande onderdelen samenbrengen tot één coherent programma.

Dat is een belangrijke overgang: van losse oefeningen naar het daadwerkelijk bouwen van een groter Rust-programma.

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 35](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_35/).

