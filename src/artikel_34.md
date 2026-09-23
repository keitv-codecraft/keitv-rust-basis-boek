# 34. Ontmoetingen en beloningen

## Wat gaan we leren?

In dit artikel breiden we de spelwereld uit met ontmoetingen, beloningen, ervaringspunten (XP) en level-ups.

We leren:

- een wereld van meerdere locaties modelleren en doorzoeken
- ervaringspunten en goud toekennen na het verslaan van een vijand
- een levelsysteem ontwerpen waarmee de speler sterker wordt bij een level-up
- buit (loot) toevoegen aan de inventaris van de speler
- verslagen vijanden netjes opruimen uit de spellocatie.

---

## 1. De wereld als verzameling locaties

We beginnen eenvoudig.

Een locatie kan bijvoorbeeld een naam en een beschrijving hebben:

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
    beschrijving: String::from("Hoge bomen blokkeren bijna al het licht."),
};
```

En de informatie tonen:

```rust,ignore
println!("{}", bos.naam);
println!("{}", bos.beschrijving);
```

Dit is eigenlijk niets nieuws.

We gebruiken hier een `struct` met twee `String`-velden.

---

## 2. Een constructor maken

Net als bij onze `Speler` en andere structs kunnen we een methode maken waarmee we een locatie maken.

```rust,ignore
impl Locatie {
    fn nieuw(naam: &str, beschrijving: &str) -> Self {
        Self {
            naam: String::from(naam),
            beschrijving: String::from(beschrijving),
        }
    }
}
```

Nu kunnen we schrijven:

```rust,ignore
let bos = Locatie::nieuw(
    "Het Donkere Bos",
    "Hoge bomen blokkeren bijna al het licht.",
);
```

Dit kennen we inmiddels goed:

- `impl` bevat gedrag
- `fn nieuw` is een functie
- `&str` is geschikt voor tekst die we alleen lezen
- `String::from` maakt daarvan een eigen `String`
- `Self` betekent hier `Locatie`.

---

## 3. Een locatie heeft misschien een vijand

Een interessante locatie kan een vijand bevatten.

Maar niet iedere locatie hoeft een vijand te hebben.

Dat is precies het probleem waarvoor we `Option` hebben geleerd.

```rust,ignore
struct Locatie {
    naam: String,
    beschrijving: String,
    vijand: Option<Vijand>,
}
```

Een locatie zonder vijand:

```rust,ignore
let dorp = Locatie {
    naam: String::from("Dorp"),
    beschrijving: String::from("Een klein dorp aan de rand van het bos."),
    vijand: None,
};
```

Een locatie met een vijand:

```rust,ignore
let bos = Locatie {
    naam: String::from("Donker Bos"),
    beschrijving: String::from("Je hoort iets bewegen tussen de bomen."),
    vijand: Some(Vijand::nieuw("Wolf", 40)),
};
```

We hebben hier dus opnieuw:

```text
Option<Vijand>
```

met twee mogelijke situaties:

```text
None
Some(vijand)
```

---

## 4. De locatie bekijken

We kunnen een methode maken om een locatie te bekijken.

```rust,ignore
impl Locatie {
    fn toon(&self) {
        println!();
        println!("== {} ==", self.naam);
        println!("{}", self.beschrijving);

        match &self.vijand {
            Some(vijand) => {
                println!("Een {} staat je in de weg!", vijand.naam);
            }
            None => {
                println!("Het lijkt hier veilig.");
            }
        }
    }
}
```

Let goed op:

```rust,ignore
match &self.vijand
```

We willen de vijand alleen bekijken. We willen hem niet uit de locatie halen.

Daarom lenen we de waarde.

Dit is een mooi voorbeeld van waarom ownership en borrowing belangrijk zijn.

---

## 5. Meerdere locaties

Een RPG heeft natuurlijk meer dan één locatie.

We kunnen een `Vec` gebruiken:

```rust,ignore
let wereld = vec![
    Locatie::nieuw(
        "Dorp",
        "Een rustig dorp met een kleine winkel.",
    ),
    Locatie::nieuw(
        "Bos",
        "Een donker bos vol onbekende geluiden.",
    ),
    Locatie::nieuw(
        "Ruïne",
        "Oude stenen muren staan tussen het gras.",
    ),
];
```

Daarna kunnen we alle locaties tonen:

```rust,ignore
for locatie in &wereld {
    locatie.toon();
}
```

Ook dit is bekende stof.

We combineren hier alleen:

- `Vec`
- `for`
- borrowing
- een methode.

---

## 6. De speler kiest een locatie

We kunnen de speler bijvoorbeeld een nummer laten kiezen.

Stel dat:

```text
1 = Dorp
2 = Bos
3 = Ruïne
```

Dan kunnen we met `match` bepalen wat de speler kiest:

```rust,ignore
let keuze = 2;

match keuze {
    1 => println!("Je gaat naar het dorp."),
    2 => println!("Je gaat naar het bos."),
    3 => println!("Je gaat naar de ruïne."),
    _ => println!("Ongeldige keuze."),
}
```

Dit is een patroon dat we al vaker hebben gebruikt.

Een `match` hoeft niet alleen met enums te werken. Het kan ook met getallen, `String`, `char` en andere waarden werken.

---

## 7. Een functie voor het kiezen van een locatie

We kunnen dit gedrag in een functie stoppen.

```rust,ignore
fn kies_locatie(keuze: usize) -> Option<usize> {
    match keuze {
        1 => Some(0),
        2 => Some(1),
        3 => Some(2),
        _ => None,
    }
}
```

Waarom geven we een `Option` terug?

Omdat de keuze ongeldig kan zijn.

Bijvoorbeeld:

```rust,ignore
match kies_locatie(2) {
    Some(index) => println!("Je koos locatie {}", index),
    None => println!("Ongeldige keuze."),
}
```

Hier zien we een belangrijk patroon:

> Als een functie iets misschien niet kan opleveren, kan `Option` dat expliciet aangeven.

---

## 8. Een betere manier: de locatie direct zoeken

We hoeven niet altijd met een nummer te werken.

Stel dat we een lijst met locaties hebben:

```rust,ignore
let wereld = vec![
    Locatie::nieuw("Dorp", "Een rustig dorp."),
    Locatie::nieuw("Bos", "Een donker bos."),
    Locatie::nieuw("Ruïne", "Een oude ruïne."),
];
```

We kunnen een locatie zoeken op naam.

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

Hier zien we iets dat we al kennen:

```rust,ignore
.iter()
.find(...)
```

`find` geeft een `Option` terug.

Dus:

```rust,ignore
Some(locatie)
```

als de locatie gevonden is, en:

```rust,ignore
None
```

als dat niet zo is.

### Een belangrijke opmerking

In deze functie zien we voor het eerst expliciet een lifetime:

```rust,ignore
'a
```

We hoeven lifetimes op dit moment nog niet volledig te begrijpen.

De functie zegt in feite:

> De gevonden verwijzing blijft geldig zolang de oorspronkelijke `wereld` geldig is.

Voor nu is vooral belangrijk dat je het patroon herkent.

In veel gewone Rust-code zorgt de compiler ervoor dat dit automatisch kan worden afgeleid. Later kunnen we lifetimes uitgebreider behandelen.

---

## 9. Een vijand verslaan geeft ervaring

Nu komen we bij een belangrijk RPG-concept: ervaring.

We kunnen bijvoorbeeld aan `Speler` een veld toevoegen:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
    ervaring: u32,
}
```

Een speler begint bijvoorbeeld met:

```rust,ignore
ervaring: 0,
```

Na een gevecht kunnen we ervaring toevoegen:

```rust,ignore
impl Speler {
    fn geef_ervaring(&mut self, hoeveelheid: u32) {
        self.ervaring += hoeveelheid;
    }
}
```

Gebruik:

```rust,ignore
speler.geef_ervaring(25);
```

Dit is weer een gewone methode met:

```rust,ignore
&mut self
```

omdat de speler verandert.

---

## 10. Ervaring hoort bij een vijand

Het is logisch dat sterkere vijanden meer ervaring geven.

We voegen daarom een veld toe aan `Vijand`:

```rust,ignore
struct Vijand {
    naam: String,
    gezondheid: i32,
    ervaring: u32,
}
```

Bijvoorbeeld:

```rust,ignore
let wolf = Vijand::nieuw("Wolf", 40, 20);
let draak = Vijand::nieuw("Draak", 200, 500);
```

De constructor kan worden:

```rust,ignore
impl Vijand {
    fn nieuw(naam: &str, gezondheid: i32, ervaring: u32) -> Self {
        Self {
            naam: String::from(naam),
            gezondheid,
            ervaring,
        }
    }
}
```

Nu heeft iedere vijand zijn eigen beloning.

---

## 11. Een gevecht geeft de beloning

We hadden in het vorige artikel al een gevechtsfunctie.

We kunnen na een gewonnen gevecht de ervaring toevoegen:

```rust,ignore
fn geef_beloning(speler: &mut Speler, vijand: &Vijand) {
    speler.geef_ervaring(vijand.ervaring);

    println!(
        "{} krijgt {} ervaring!",
        speler.naam,
        vijand.ervaring
    );
}
```

Waarom:

```rust,ignore
&Vijand
```

en niet:

```rust,ignore
Vijand
```

?

Omdat we de vijand alleen hoeven te bekijken.

We nemen geen ownership over.

Dit is een patroon dat je heel vaak zult gebruiken:

```rust,ignore
fn iets(vijand: &Vijand)
```

als een functie informatie van een vijand nodig heeft, maar de vijand niet hoeft te veranderen.

---

## 12. Levelen

Nu hebben we ervaring, maar er gebeurt nog niets mee.

We kunnen een level toevoegen:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
    maximale_gezondheid: i32,
    ervaring: u32,
    level: u32,
}
```

Een nieuwe speler:

```rust,ignore
level: 1,
ervaring: 0,
```

We kunnen een eenvoudige regel kiezen:

```text
100 XP → level 2
200 XP → level 3
300 XP → level 4
```

Een eenvoudige controle:

```rust,ignore
impl Speler {
    fn controleer_level(&mut self) {
        let nieuw_level = self.ervaring / 100 + 1;

        if nieuw_level > self.level {
            self.level = nieuw_level;

            println!(
                "{} is nu level {}!",
                self.naam,
                self.level
            );
        }
    }
}
```

Dit is een mooi voorbeeld van bestaande concepten combineren:

- `&mut self`
- rekenen
- `if`
- velden lezen
- velden wijzigen
- `println!`.

---

## 13. Sterker worden bij een nieuw level

Een level kan ook daadwerkelijk gevolgen hebben.

Bijvoorbeeld:

```rust,ignore
impl Speler {
    fn controleer_level(&mut self) {
        let nieuw_level = self.ervaring / 100 + 1;

        if nieuw_level > self.level {
            self.level = nieuw_level;

            self.maximale_gezondheid += 10;
            self.gezondheid = self.maximale_gezondheid;

            println!("Je bent gestegen naar level {}!", self.level);
            println!("Je maximale gezondheid is verhoogd.");
        }
    }
}
```

Hierdoor krijgt de speler na een level-up:

- een hoger level
- meer maximale gezondheid
- volledige gezondheid.

---

## 14. Meerdere levels overslaan

Stel dat een speler 190 XP krijgt.

Dan gaat hij van level 1 naar:

```text
190 / 100 + 1 = 2
```

Dat is level 2.

Maar stel dat de speler ineens 350 XP krijgt.

Dan:

```text
350 / 100 + 1 = 4
```

De speler springt dus rechtstreeks naar level 4.

Dat kan prima zijn.

Maar misschien wil je dat de speler voor **ieder nieuw level** een beloning krijgt.

Dan kunnen we een `while` gebruiken:

```rust,ignore
impl Speler {
    fn controleer_level(&mut self) {
        let benodigde_ervaring = self.level * 100;

        while self.ervaring >= benodigde_ervaring {
            self.level += 1;

            self.maximale_gezondheid += 10;
            self.gezondheid = self.maximale_gezondheid;

            println!("Level omhoog! Je bent nu level {}.", self.level);
        }
    }
}
```

Hier zit echter een probleem in.

`benodigde_ervaring` verandert niet binnen de lus.

Dat kunnen we verbeteren:

```rust,ignore
impl Speler {
    fn controleer_level(&mut self) {
        while self.ervaring >= self.level * 100 {
            self.level += 1;

            self.maximale_gezondheid += 10;
            self.gezondheid = self.maximale_gezondheid;

            println!("Level omhoog! Je bent nu level {}.", self.level);
        }
    }
}
```

Nu wordt de voorwaarde iedere keer opnieuw berekend.

Dit is een goed voorbeeld van iets wat de compiler niet noodzakelijk voor je kan oplossen.

De code kan syntactisch correct zijn en toch niet doen wat je bedoelt.

Daarom zijn testen zo belangrijk.

---

## 15. Een test voor levelen

We kunnen testen:

```rust,ignore
#[test]
fn speler_stijgt_een_level() {
    let mut speler = Speler::nieuw("Arin");

    speler.geef_ervaring(100);
    speler.controleer_level();

    assert_eq!(speler.level, 2);
}
```

En:

```rust,ignore
#[test]
fn speler_stijgt_meerdere_levels() {
    let mut speler = Speler::nieuw("Arin");

    speler.geef_ervaring(300);
    speler.controleer_level();

    assert_eq!(speler.level, 4);
}
```

De tweede test is vooral interessant.

Je controleert hiermee dat je code niet alleen voor één eenvoudige situatie werkt.

---

## 16. Buit vinden

Naast ervaring kan een vijand ook buit opleveren.

We hebben al een `Inventaris` met een `HashMap`.

Daar kunnen we gebruik van maken.

Stel dat een vijand een aantal goudstukken oplevert:

```rust,ignore
struct Vijand {
    naam: String,
    gezondheid: i32,
    ervaring: u32,
    goud: u32,
}
```

Na een overwinning:

```rust,ignore
fn geef_beloning(speler: &mut Speler, vijand: &Vijand) {
    speler.geef_ervaring(vijand.ervaring);
    speler.geef_goud(vijand.goud);

    println!("Je vindt {} goud.", vijand.goud);
}
```

We gebruiken hier opnieuw bestaande functionaliteit.

---

## 17. Een voorwerp als buit

We kunnen ook een voorwerp vinden.

Bijvoorbeeld:

```rust,ignore
struct Buit {
    naam: String,
    aantal: u32,
}
```

Een vijand kan vervolgens een lijst met buit hebben:

```rust,ignore
struct Vijand {
    naam: String,
    gezondheid: i32,
    ervaring: u32,
    goud: u32,
    buit: Vec<Buit>,
}
```

Bijvoorbeeld:

```rust,ignore
let wolf = Vijand {
    naam: String::from("Wolf"),
    gezondheid: 40,
    ervaring: 20,
    goud: 5,
    buit: vec![
        Buit {
            naam: String::from("Wolvenvacht"),
            aantal: 1,
        },
    ],
};
```

Hier combineren we:

```text
Vijand
 └── Vec<Buit>
       ├── Buit
       └── Buit
```

Dat is een belangrijk idee bij grotere programma's:

> Een struct kan andere structs bevatten.

Dat hebben we eerder al gedaan met bijvoorbeeld `Speler` en `Inventaris`.

---

## 18. Buit toevoegen aan de inventaris

We kunnen door de buit lopen:

```rust,ignore
fn verzamel_buit(speler: &mut Speler, vijand: &Vijand) {
    for buit in &vijand.buit {
        speler.inventaris.voeg_item_toe(
            &buit.naam,
            buit.aantal,
        );

        println!(
            "Je vindt {} x {}.",
            buit.aantal,
            buit.naam
        );
    }
}
```

Let opnieuw op:

```rust,ignore
for buit in &vijand.buit
```

We lenen de buit.

We willen de buit niet uit de vijand verwijderen.

---

## 19. Een ontmoeting

Nu kunnen we verschillende onderdelen combineren.

Een speler komt een vijand tegen:

```rust,ignore
fn ontmoeting(speler: &mut Speler, vijand: &mut Vijand) {
    println!();
    println!("Je ontmoet een {}!", vijand.naam);

    let resultaat = vecht(speler, vijand);

    match resultaat {
        Gevechtsresultaat::Gewonnen => {
            println!("Je hebt gewonnen!");

            geef_beloning(speler, vijand);
            verzamel_buit(speler, vijand);
            speler.controleer_level();
        }

        Gevechtsresultaat::Verloren => {
            println!("Je bent verslagen.");
        }

        Gevechtsresultaat::Gevlucht => {
            println!("Je bent gevlucht.");
        }
    }
}
```

Dit is nu behoorlijk interessante code.

Maar kijk hoeveel ervan we al kennen:

- functie
- `&mut`
- `match`
- enum
- `Vec`
- methodes
- `Option`
- ownership
- borrowing.

We hebben nauwelijks nieuwe Rust-syntax nodig gehad.

Dat is precies de bedoeling.

---

## 20. Een locatie kan een ontmoeting bevatten

Onze locatie bevatte al:

```rust,ignore
vijand: Option<Vijand>
```

Nu kunnen we daar iets mee doen.

Bijvoorbeeld:

```rust,ignore
fn bezoek_locatie(speler: &mut Speler, locatie: &mut Locatie) {
    locatie.toon();

    if let Some(vijand) = &mut locatie.vijand {
        ontmoeting(speler, vijand);
    }
}
```

Hier zien we opnieuw:

```rust,ignore
if let Some(...)
```

We zeggen daarmee:

> Als deze locatie een vijand heeft, geef mij dan een mutable verwijzing naar die vijand.

Heeft de locatie geen vijand?

Dan gebeurt er niets.

---

## 21. Een verslagen vijand verwijderen

Er is echter nog een probleem.

Als we de vijand verslaan, blijft hij in de locatie staan.

De speler kan dan steeds opnieuw tegen dezelfde vijand vechten.

We kunnen na het gevecht controleren of de vijand nog leeft.

Een eenvoudige oplossing is:

```rust,ignore
fn bezoek_locatie(speler: &mut Speler, locatie: &mut Locatie) {
    locatie.toon();

    if let Some(vijand) = &mut locatie.vijand {
        let resultaat = ontmoeting(speler, vijand);

        if matches!(resultaat, Gevechtsresultaat::Gewonnen) {
            locatie.vijand = None;
        }
    }
}
```

`matches!` hebben we eerder gezien.

Het betekent hier ongeveer:

```text
Is resultaat gelijk aan Gevechtsresultaat::Gewonnen?
```

Je kunt dit ook met `match` schrijven:

```rust,ignore
match resultaat {
    Gevechtsresultaat::Gewonnen => {
        locatie.vijand = None;
    }
    _ => {}
}
```

Voor beginners is de tweede versie soms duidelijker.

De eerste is korter.

---

## 22. Waarom kan Rust hier moeilijk doen?

Misschien probeert de compiler bezwaar te maken tegen code waarbij je:

1. een mutable verwijzing naar `locatie.vijand` maakt
2. daarna `locatie.vijand` opnieuw probeert te veranderen.

Dat komt door borrowing.

Je hebt bijvoorbeeld tijdelijk:

```rust,ignore
let vijand = &mut locatie.vijand;
```

en probeert vervolgens tegelijkertijd de hele waarde te veranderen.

Rust voorkomt daarmee dat twee verschillende delen van je programma elkaar onverwacht beïnvloeden.

Een oplossing is vaak om het geleende deel eerst los te laten.

Bijvoorbeeld:

```rust,ignore
fn bezoek_locatie(speler: &mut Speler, locatie: &mut Locatie) {
    locatie.toon();

    let resultaat = if let Some(vijand) = &mut locatie.vijand {
        Some(ontmoeting(speler, vijand))
    } else {
        None
    };

    if let Some(resultaat) = resultaat {
        match resultaat {
            Gevechtsresultaat::Gewonnen => {
                locatie.vijand = None;
            }
            _ => {}
        }
    }
}
```

De tijdelijke borrow van `vijand` is afgelopen voordat we `locatie.vijand` op `None` zetten.

Dit is een uitstekend soort compilerfout om te leren begrijpen.

De compiler zegt hier niet zomaar:

> "Dit mag niet."

Hij beschermt de regels van ownership en borrowing.

---

## 23. Een wereld met meerdere locaties

We kunnen nu een kleine wereld maken:

```rust,ignore
let mut wereld = vec![
    Locatie::nieuw("Dorp", "Een veilig dorp."),
    Locatie::nieuw("Bos", "Een donker bos."),
    Locatie::nieuw("Ruïne", "Een oude ruïne."),
];
```

Stel dat we later vijanden toevoegen:

```rust,ignore
wereld[1].vijand = Some(
    Vijand::nieuw("Wolf", 40, 20, 5)
);
```

En:

```rust,ignore
wereld[2].vijand = Some(
    Vijand::nieuw("Goblin", 60, 40, 12)
);
```

Nu heeft de wereld:

```text
Dorp
 └── geen vijand

Bos
 └── Wolf

Ruïne
 └── Goblin
```

---

## 24. Een eenvoudige spel-lus

We kunnen dit uiteindelijk in een `loop` zetten.

Bijvoorbeeld:

```rust,ignore
loop {
    println!();
    println!("Waar wil je heen?");
    println!("1. Dorp");
    println!("2. Bos");
    println!("3. Ruïne");
    println!("4. Stoppen");

    let keuze = lees_getal();

    match keuze {
        1 => bezoek_locatie(&mut speler, &mut wereld[0]),
        2 => bezoek_locatie(&mut speler, &mut wereld[1]),
        3 => bezoek_locatie(&mut speler, &mut wereld[2]),
        4 => {
            println!("Tot ziens!");
            break;
        }
        _ => {
            println!("Ongeldige keuze.");
        }
    }
}
```

Dit begint al op een echt spel te lijken.

De loop:

1. toont een menu
2. leest de keuze
3. bezoekt een locatie
4. handelt een ontmoeting af
5. gaat terug naar het menu.

En daarna begint de volgende ronde.

---

## 25. Een belangrijke ontwerpkeuze

We kunnen de hele wereld op verschillende manieren ontwerpen.

Bijvoorbeeld:

```text
Wereld
 ├── Dorp
 ├── Bos
 ├── Ruïne
 └── Berg
```

Maar we kunnen ook verbindingen tussen locaties maken.

Bijvoorbeeld:

```text
        Berg
         |
       Ruïne
         |
Dorp --- Bos
```

Dan kan de speler niet zomaar iedere locatie kiezen.

Een locatie zou bijvoorbeeld kunnen bevatten:

```rust,ignore
struct Locatie {
    naam: String,
    beschrijving: String,
    buren: Vec<String>,
    vijand: Option<Vijand>,
}
```

Het veld:

```rust,ignore
buren: Vec<String>
```

bevat dan de namen van locaties waar de speler naartoe kan.

Bijvoorbeeld:

```rust,ignore
buren: vec![
    String::from("Bos"),
    String::from("Ruïne"),
],
```

Dit is opnieuw geen ingewikkelde nieuwe techniek.

Het is vooral een nieuwe toepassing van:

- `Vec`
- `String`
- `struct`
- `Option`.

---

## 26. Waarom ontwerpen belangrijker wordt

We beginnen nu een grens te bereiken.

Bij een klein programma kun je alles vanuit `main` regelen.

Bijvoorbeeld:

```rust
fn main() {
    // alles gebeurt hier
}
```

Maar ons RPG wordt steeds groter.

We hebben inmiddels onderdelen voor:

```text
Speler
Vijand
Wapen
Inventaris
Winkel
Gevecht
Locatie
Wereld
Savegame
```

Daarom moeten we steeds beter nadenken over verantwoordelijkheden.

Een goede vraag bij iedere nieuwe functie is:

> Wie hoort dit te weten?

Bijvoorbeeld:

**Wie moet weten hoeveel ervaring een speler heeft?**

`Speler`.

**Wie moet weten hoeveel ervaring een wolf geeft?**

`Vijand`.

**Wie moet bepalen wat er gebeurt na een gewonnen gevecht?**

Het gevechtssysteem.

**Wie moet weten welke vijanden zich op welke locatie bevinden?**

De wereld of locatie.

Deze manier van nadenken is minstens zo belangrijk als het leren van nieuwe syntax.

---

## 27. Een eerste versie van de architectuur

Onze RPG begint bijvoorbeeld deze vorm te krijgen:

```text
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

Een mogelijke taakverdeling:

### `speler.rs`

Bevat:

- speler
- gezondheid
- level
- ervaring
- goud
- inventaris
- wapen.

### `vijand.rs`

Bevat:

- vijand
- gezondheid
- aanval
- ervaring
- goud
- buit.

### `gevecht.rs`

Bevat:

- aanvallen
- verdedigen
- vluchten
- gevechtsresultaten
- beloningen.

### `locatie.rs`

Bevat:

- naam
- beschrijving
- mogelijke vijand
- eventueel verbindingen.

### `wereld.rs`

Bevat:

- locaties
- reizen
- zoeken naar locaties.

### `main.rs`

Regelt vooral:

- het starten van het spel
- het hoofdmenu
- de hoofd-loop.

Dit is een goed moment om jezelf aan een belangrijke regel te herinneren:

> Een module is geen doel op zichzelf. We gebruiken modules om code begrijpelijk te houden.

---

## 28. Oefening 1 — Een veilige locatie

Maak een struct:

```rust,ignore
struct Locatie {
    naam: String,
    beschrijving: String,
    vijand: Option<Vijand>,
}
```

Schrijf daarna:

```rust,ignore
fn veilige_locatie(...) -> Locatie
```

Deze functie moet een locatie maken zonder vijand.

Gebruik daarna `toon()` om de locatie te bekijken.

### Denk na

Waarom gebruiken we hier:

```rust,ignore
None
```

en niet bijvoorbeeld:

```rust,ignore
gezondheid: 0
```

of:

```rust,ignore
naam: String::from("")
```

---

## 29. Oefening 2 — Ervaring toevoegen

Voeg aan `Speler` een methode toe:

```rust,ignore
fn geef_ervaring(&mut self, hoeveelheid: u32)
```

Test:

- 10 XP toevoegen
- nog 20 XP toevoegen
- controleren dat de speler 30 XP heeft.

Voeg daarna een test toe voor 0 XP.

---

## 30. Oefening 3 — Level omhoog

Schrijf:

```rust,ignore
fn controleer_level(&mut self)
```

Gebruik de regel:

```text
iedere 100 XP = één level
```

Test minimaal:

```text
0 XP   → level 1
99 XP  → level 1
100 XP → level 2
199 XP → level 2
200 XP → level 3
```

Probeer daarna zelf te bedenken wat er bij 500 XP moet gebeuren.

---

## 31. Oefening 4 — Goud als beloning

Geef iedere vijand een hoeveelheid goud.

Maak:

```rust,ignore
fn geef_goud(&mut self, hoeveelheid: u32)
```

en zorg ervoor dat een gewonnen gevecht de speler het goud van de vijand geeft.

Test:

```text
Wolf → 5 goud
Goblin → 12 goud
Draak → 500 goud
```

---

## 32. Oefening 5 — Buit

Maak:

```rust,ignore
struct Buit {
    naam: String,
    aantal: u32,
}
```

Geef een vijand een:

```rust,ignore
Vec<Buit>
```

Laat een gewonnen gevecht alle buit aan de inventaris toevoegen.

Test bijvoorbeeld:

```text
Wolf
 ├── Wolvenvacht × 1
 └── Tand × 2
```

---

## 33. Oefening 6 — Locaties zoeken

Maak een functie waarmee je een locatie op naam kunt zoeken.

Bijvoorbeeld:

```rust,ignore
vind_locatie(&wereld, "Bos")
```

moet:

```rust,ignore
Some(...)
```

opleveren.

Een onbekende naam moet:

```rust,ignore
None
```

opleveren.

Gebruik hiervoor:

```rust,ignore
.iter()
.find(...)
```

Probeer eerst zelf te schrijven hoe de closure voor `find` eruit moet zien.

---

## 34. Oefening 7 — Locatie bezoeken

Maak:

```rust,ignore
fn bezoek_locatie(
    speler: &mut Speler,
    locatie: &mut Locatie,
)
```

De functie moet:

1. de locatie tonen
2. controleren of er een vijand is
3. als er een vijand is, een gevecht starten
4. bij winst ervaring en goud geven
5. bij winst de vijand verwijderen.

Gebruik hierbij zoveel mogelijk bestaande functies.

Probeer niet alles opnieuw te programmeren.

---

## 35. Oefening 8 — Een wereld bouwen

Maak minimaal vijf locaties.

Bijvoorbeeld:

```text
Dorp
Bos
Rivier
Ruïne
Berg
```

Geef minstens drie locaties een vijand.

Geef iedere vijand:

- gezondheid
- aanvalskracht
- XP
- goud
- minimaal één stuk buit.

Daarna moet de speler verschillende locaties kunnen bezoeken.

---

## 36. Oefening 9 — Verbonden locaties

Breid `Locatie` uit met:

```rust,ignore
buren: Vec<String>
```

Maak bijvoorbeeld:

```text
Dorp
 └── Bos

Bos
 ├── Dorp
 └── Ruïne

Ruïne
 ├── Bos
 └── Berg

Berg
 └── Ruïne
```

Laat de speler alleen naar verbonden locaties reizen.

Hierbij moet je dus:

1. de huidige locatie kennen
2. de buren bekijken
3. controleren of de gekozen locatie bestaat
4. naar die locatie gaan.

---

## 37. Oefening 10 — De kleine RPG

Combineer alles.

De speler moet:

1. het spel starten
2. zijn status kunnen bekijken
3. een locatie kunnen kiezen
4. naar een locatie kunnen reizen
5. vijanden kunnen tegenkomen
6. kunnen vechten
7. XP kunnen krijgen
8. levels kunnen stijgen
9. goud kunnen krijgen
10. buit kunnen verzamelen
11. opnieuw kunnen reizen.

Gebruik zoveel mogelijk bestaande code.

Het doel is nadrukkelijk **niet** om zoveel mogelijk nieuwe code te schrijven.

Het doel is om bestaande onderdelen goed met elkaar te verbinden.

---

## 38. Extra uitdaging

Als de basis werkt, kun je zelf uitbreidingen bedenken.

Bijvoorbeeld:

### Willekeurige ontmoetingen

Gebruik `rand` om te bepalen of er een vijand verschijnt.

```rust,ignore
let getal = rand::random_range(1..=100);
```

Bijvoorbeeld:

```text
1–30   → geen vijand
31–70  → normale vijand
71–95  → sterke vijand
96–100 → zeldzame vijand
```

### Meerdere vijanden

Een locatie kan bijvoorbeeld hebben:

```rust,ignore
vijanden: Vec<Vijand>
```

Dan moet de speler meerdere tegenstanders kunnen tegenkomen.

### Ervaringsbonus

Geef extra XP wanneer een speler een sterkere vijand verslaat.

### Zeldzame buit

Gebruik `Option<Buit>` om aan te geven dat een vijand misschien helemaal geen bijzonder voorwerp laat vallen.

### Winkel na een gevecht

Laat de speler met het verdiende goud naar de winkel gaan.

---

## 39. Wat je hier eigenlijk geleerd hebt

Dit artikel introduceerde bewust weinig nieuwe Rust.

Dat is belangrijk.

Je hebt vooral bestaande concepten opnieuw gebruikt:

| Concept | Toepassing |
| --- | --- |
| `struct` | locaties en buit |
| `impl` | gedrag van locaties en spelers |
| `Vec` | wereld, locaties en buit |
| `Option` | mogelijke vijand |
| `match` | keuzes en resultaten |
| `if let` | mogelijke vijand |
| `for` | locaties en buit |
| borrowing | locaties en vijanden bekijken |
| mutable borrowing | speler en vijand wijzigen |
| `iter()` | locaties doorzoeken |
| `find()` | locatie zoeken |
| closures | zoekvoorwaarden |
| `enum` | gevechtsresultaat |
| `HashMap` | inventaris |
| tests | XP, levels en reizen |
| modules | verdeling van het RPG-project |

Dat is precies hoe grotere Rust-programma's ontstaan.

Je hoeft niet voor iedere nieuwe functie een compleet nieuw Rust-concept te leren.

Vaak bestaat nieuwe functionaliteit voornamelijk uit:

```text
bekende struct
+
bekende methode
+
bekende Vec
+
bekende Option
+
bekende match
+
bekende borrowing
```

De uitdaging verschuift daardoor langzaam van:

> "Hoe schrijf ik deze Rust-syntax?"

naar:

> "Welke onderdelen heb ik nodig om dit probleem op te lossen?"

Dat is een belangrijke stap in leren programmeren.

Je weet inmiddels al behoorlijk veel Rust.

De volgende stap is om die kennis steeds zelfstandiger te gaan combineren.

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 34](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_34/).

---

## Controlelijst

Je bent klaar met dit artikel als je zonder hulp:

- [ ] ervaringspunten (XP) en goud kunt toekennen na een gewonnen gevecht
- [ ] een level-up mechanisme kunt programmeren met verhoogde statistieken
- [ ] buit (loot) vanuit een verslagen vijand aan de inventaris kunt toevoegen
- [ ] verslagen vijanden veilig kunt verwijderen met behulp van `Option::take()` of vergelijkbare patronen.
