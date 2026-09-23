# 10. Structs

## Wat gaan we leren?

Tot nu toe hebben we veel losse variabelen gebruikt.

Bijvoorbeeld:

```rust,ignore
let naam = String::from("Arin");
let gezondheid = 100;
let goud = 50;
let levend = true;
```

Deze variabelen horen eigenlijk allemaal bij dezelfde speler.

In deze les leren we hoe we zulke gegevens kunnen **bundelen in één eigen datatype**.

Dat doen we met een `struct`.

We leren:

- wat een struct is
- waarom structs handig zijn
- hoe je een struct definieert
- hoe je een struct maakt
- hoe je velden uitleest
- hoe je velden verandert
- hoe `mut` bij structs werkt
- hoe je structs aan functies kunt geven
- hoe borrowing met structs werkt
- hoe je structs gebruikt voor onze game.

We leren nog **geen `impl`-blokken en methods**. Die komen in de volgende les.

---

## 1. Wat is een struct?

Een `struct` is een datatype dat we zelf kunnen maken.

Bijvoorbeeld:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
    goud: i32,
}
```

We hebben nu een nieuw datatype gemaakt:

```text
Speler
```

Een `Speler` heeft drie eigenschappen:

```text
naam
gezondheid
goud
```

Deze eigenschappen noemen we **velden**.

Je kunt een struct zien als een formulier:

```text
Speler
├── naam
├── gezondheid
└── goud
```

---

## 2. Waarom hebben we structs nodig?

Zonder struct zouden we bijvoorbeeld dit hebben:

```rust,ignore
let naam = String::from("Arin");
let gezondheid = 100;
let goud = 50;
```

Dat werkt prima zolang we één speler hebben.

Maar wat als we twee spelers hebben?

```rust,ignore
let naam1 = String::from("Arin");
let gezondheid1 = 100;
let goud1 = 50;

let naam2 = String::from("Borin");
let gezondheid2 = 80;
let goud2 = 25;
```

Dat wordt snel onoverzichtelijk.

Met een struct kunnen we schrijven:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
    goud: i32,
}
```

En daarna:

```rust,ignore
let speler1 = Speler {
    naam: String::from("Arin"),
    gezondheid: 100,
    goud: 50,
};

let speler2 = Speler {
    naam: String::from("Borin"),
    gezondheid: 80,
    goud: 25,
};
```

Nu zijn de gegevens netjes gegroepeerd.

---

## 3. Een struct definiëren

Een struct definiëren we met `struct`.

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
    goud: i32,
}
```

De algemene vorm is:

```rust,ignore
struct Naam {
    veld1: Type,
    veld2: Type,
}
```

Bijvoorbeeld:

```rust,ignore
struct Vijand {
    naam: String,
    gezondheid: i32,
    schade: i32,
}
```

Of:

```rust,ignore
struct Wapen {
    naam: String,
    schade: i32,
    waarde: i32,
}
```

We kunnen dus zelf datatypes maken die passen bij ons programma.

---

## 4. Een struct maken

Nadat we een struct hebben gedefinieerd, kunnen we er een waarde van maken.

Bijvoorbeeld:

```rust
struct Speler {
    naam: String,
    gezondheid: i32,
    goud: i32,
}

fn main() {
    let speler = Speler {
        naam: String::from("Arin"),
        gezondheid: 100,
        goud: 50,
    };
}
```

Hier maken we een `Speler`.

De waarden tussen `{` en `}` zijn de velden.

---

## 5. De velden uitlezen

We kunnen bij een veld komen met een punt:

```rust,ignore
speler.naam
```

Bijvoorbeeld:

```rust,ignore
println!("Naam: {}", speler.naam);
println!("HP: {}", speler.gezondheid);
println!("Goud: {}", speler.goud);
```

Of met moderne Rust-interpolatie:

```rust,ignore
println!("Naam: {}", speler.naam);
println!("HP: {}", speler.gezondheid);
println!("Goud: {}", speler.goud);
```

Je kunt het ook direct in een format-string gebruiken wanneer de variabele beschikbaar is, maar voor beginners is de puntnotatie het belangrijkste om te herkennen:

```text
speler.naam
speler.gezondheid
speler.goud
```

---

## 6. Een veld veranderen

We kunnen een veld veranderen als de struct zelf mutable is.

Dus:

```rust,ignore
let mut speler = Speler {
    naam: String::from("Arin"),
    gezondheid: 100,
    goud: 50,
};
```

Daarna:

```rust,ignore
speler.gezondheid = 75;
speler.goud = 100;
```

We kunnen vervolgens printen:

```rust,ignore
println!("HP: {}", speler.gezondheid);
println!("Goud: {}", speler.goud);
```

---

## 7. `mut` geldt voor de hele struct

Dit werkt niet:

```rust,ignore
let speler = Speler {
    naam: String::from("Arin"),
    gezondheid: 100,
    goud: 50,
};

speler.gezondheid = 75;
```

Waarom?

Omdat `speler` niet mutable is.

We hebben nodig:

```rust,ignore
let mut speler = Speler {
    naam: String::from("Arin"),
    gezondheid: 100,
    goud: 50,
};
```

Dan mogen we de velden veranderen.

Een eenvoudige manier om het te onthouden:

> Als je een veld van een struct wilt veranderen, moet de struct-variabele `mut` zijn.

---

## 8. Een struct met verschillende datatypes

Een struct kan verschillende datatypes bevatten.

Bijvoorbeeld:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
    snelheid: f32,
    levend: bool,
}
```

Een speler kan dan bijvoorbeeld zijn:

```rust,ignore
let speler = Speler {
    naam: String::from("Arin"),
    gezondheid: 100,
    snelheid: 4.5,
    levend: true,
};
```

Dit is precies waarvoor structs handig zijn:

> Verschillende stukjes informatie die samen één ding beschrijven.

---

## 9. Structs voor onze game

We kunnen nu beginnen met echte game-objecten.

Een speler:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
    goud: i32,
}
```

Een vijand:

```rust,ignore
struct Vijand {
    naam: String,
    gezondheid: i32,
    schade: i32,
}
```

Een wapen:

```rust,ignore
struct Wapen {
    naam: String,
    schade: i32,
    waarde: i32,
}
```

We kunnen vervolgens objecten maken:

```rust,ignore
let speler = Speler {
    naam: String::from("Arin"),
    gezondheid: 100,
    goud: 50,
};

let vijand = Vijand {
    naam: String::from("Goblin"),
    gezondheid: 40,
    schade: 10,
};

let zwaard = Wapen {
    naam: String::from("IJzeren zwaard"),
    schade: 15,
    waarde: 25,
};
```

Onze gamewereld begint nu vorm te krijgen.

---

## 10. Een struct aan een functie geven

We kunnen een struct aan een functie meegeven.

Bijvoorbeeld:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
}

fn toon_speler(speler: Speler) {
    println!("Naam: {}", speler.naam);
    println!("HP: {}", speler.gezondheid);
}
```

En:

```rust,ignore
let speler = Speler {
    naam: String::from("Arin"),
    gezondheid: 100,
};

toon_speler(speler);
```

Hier gebeurt hetzelfde als bij een `String` in de vorige lessen:

**ownership wordt overgedragen.**

Na:

```rust,ignore
toon_speler(speler);
```

kunnen we `speler` niet zomaar opnieuw gebruiken.

---

## 11. Een struct lenen

Meestal willen we alleen de gegevens bekijken.

Dan gebruiken we borrowing:

```rust,ignore
fn toon_speler(speler: &Speler) {
    println!("Naam: {}", speler.naam);
    println!("HP: {}", speler.gezondheid);
}
```

Aanroepen:

```rust,ignore
let speler = Speler {
    naam: String::from("Arin"),
    gezondheid: 100,
};

toon_speler(&speler);

println!("{}", speler.naam);
```

De functie heeft de speler niet overgenomen.

Hij heeft hem alleen geleend.

Dit is een mooie combinatie van de vorige twee lessen:

- struct
- ownership
- borrowing.

---

## 12. Een struct mutable lenen

We kunnen een struct ook mutable lenen.

Bijvoorbeeld:

```rust,ignore
fn neem_schade(speler: &mut Speler, schade: i32) {
    speler.gezondheid -= schade;
}
```

Dan:

```rust,ignore
let mut speler = Speler {
    naam: String::from("Arin"),
    gezondheid: 100,
};

neem_schade(&mut speler, 25);

println!("HP: {}", speler.gezondheid);
```

Resultaat:

```text
HP: 75
```

De functie heeft de speler niet overgenomen.

Hij mocht de speler alleen tijdelijk veranderen.

---

## 13. Waarom is dit zo handig?

We kunnen nu functies schrijven die met onze game-objecten werken.

Bijvoorbeeld:

```rust,ignore
fn neem_schade(speler: &mut Speler, schade: i32) {
    speler.gezondheid -= schade;
}
```

En:

```rust,ignore
fn toon_speler(speler: &Speler) {
    println!("{} heeft {} HP.", speler.naam, speler.gezondheid);
}
```

Dan:

```rust,ignore
let mut speler = Speler {
    naam: String::from("Arin"),
    gezondheid: 100,
};

toon_speler(&speler);

neem_schade(&mut speler, 30);

toon_speler(&speler);
```

Uitvoer:

```text
Arin heeft 100 HP.
Arin heeft 70 HP.
```

Dit begint al sterk te lijken op een echte game.

---

## 14. Een struct kan andere structs bevatten

Een struct hoeft niet alleen eenvoudige waarden te bevatten.

We kunnen bijvoorbeeld een wapen in een speler stoppen.

```rust,ignore
struct Wapen {
    naam: String,
    schade: i32,
}

struct Speler {
    naam: String,
    gezondheid: i32,
    wapen: Wapen,
}
```

Dan:

```rust,ignore
let zwaard = Wapen {
    naam: String::from("IJzeren zwaard"),
    schade: 15,
};

let speler = Speler {
    naam: String::from("Arin"),
    gezondheid: 100,
    wapen: zwaard,
};
```

Nu kunnen we:

```rust,ignore
println!("Speler: {}", speler.naam);
println!("Wapen: {}", speler.wapen.naam);
println!("Schade: {}", speler.wapen.schade);
```

Je kunt dit zien als:

```text
Speler
├── naam
├── gezondheid
└── wapen
    ├── naam
    └── schade
```

Hiermee kunnen we steeds complexere gamewerelden beschrijven.

---

## 15. Ownership bij structs

Hier komt ownership weer terug.

In:

```rust,ignore
let zwaard = Wapen {
    naam: String::from("IJzeren zwaard"),
    schade: 15,
};

let speler = Speler {
    naam: String::from("Arin"),
    gezondheid: 100,
    wapen: zwaard,
};
```

wordt `zwaard` onderdeel van `speler`.

Het `Wapen`-object is nu eigendom van de `Speler`-waarde.

De oude variabele `zwaard` kan daarna niet zomaar meer worden gebruikt.

Dit is logisch als we nadenken over een game:

> Als een speler een specifiek zwaard bezit, willen we niet dat hetzelfde zwaard tegelijkertijd ergens anders eigenaar is.

Ownership helpt dus ook bij het modelleren van onze gamewereld.

---

## 16. Een struct teruggeven uit een functie

Een functie kan een struct maken en teruggeven.

Bijvoorbeeld:

```rust,ignore
struct Vijand {
    naam: String,
    gezondheid: i32,
}

fn maak_goblin() -> Vijand {
    Vijand {
        naam: String::from("Goblin"),
        gezondheid: 40,
    }
}
```

Dan:

```rust,ignore
let vijand = maak_goblin();

println!("Vijand: {}", vijand.naam);
```

De functie maakt de vijand.

De caller wordt daarna eigenaar.

Dit is een heel gebruikelijk patroon.

---

## 17. Structs met dezelfde datatype

Stel dat we posities in onze game willen bewaren.

Dan kunnen we:

```rust,ignore
struct Positie {
    x: i32,
    y: i32,
}
```

Bijvoorbeeld:

```rust,ignore
let speler_positie = Positie {
    x: 10,
    y: 20,
};
```

En:

```rust,ignore
let vijand_positie = Positie {
    x: 15,
    y: 20,
};
```

Nu hebben we een duidelijk datatype voor een positie.

Dat is beter dan overal losse `x`- en `y`-variabelen gebruiken.

---

## 18. Structs zijn geen classes

Als je eerder met andere programmeertalen hebt gewerkt, kom je misschien het woord **class** tegen.

Een Rust-struct lijkt op sommige manieren op een class, maar is niet hetzelfde concept.

Een `struct` beschrijft vooral:

> Welke gegevens horen bij elkaar?

Later gebruiken we een `impl`-blok om gedrag aan zo'n datatype toe te voegen.

Bijvoorbeeld:

```rust,ignore
impl Speler {
    // gedrag komt hier
}
```

Dat behandelen we in de volgende les.

Voor nu concentreren we ons op de gegevens.

---

## 19. Veelgemaakte fouten

### Fout 1 — een veld vergeten

Als we hebben:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
    goud: i32,
}
```

moeten we bij het maken van een `Speler` alle velden invullen:

```rust,ignore
let speler = Speler {
    naam: String::from("Arin"),
    gezondheid: 100,
    goud: 50,
};
```

---

### Fout 2 — verkeerde datatype

Dit klopt niet:

```rust,ignore
struct Speler {
    gezondheid: i32,
}

let speler = Speler {
    gezondheid: "100",
};
```

`"100"` is tekst.

We hebben een `i32` nodig:

```rust,ignore
gezondheid: 100,
```

---

### Fout 3 — vergeten `mut`

Dit werkt niet:

```rust,ignore
let speler = Speler {
    naam: String::from("Arin"),
    gezondheid: 100,
};

speler.gezondheid = 50;
```

Maak de struct mutable:

```rust,ignore
let mut speler = Speler {
    naam: String::from("Arin"),
    gezondheid: 100,
};

speler.gezondheid = 50;
```

---

### Fout 4 — ownership vergeten

Dit:

```rust,ignore
fn toon_speler(speler: Speler) {
    println!("{}", speler.naam);
}
```

neemt ownership over.

Als we de speler daarna nog nodig hebben, gebruiken we:

```rust,ignore
fn toon_speler(speler: &Speler) {
    println!("{}", speler.naam);
}
```

---

## 20. Zelf proberen

### Opdracht 1 — Eerste struct

Maak een struct:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
}
```

Maak vervolgens een speler.

---

### Opdracht 2 — Velden printen

Print de naam en gezondheid van de speler.

---

### Opdracht 3 — Verander HP

Maak de speler mutable.

Verander de gezondheid.

---

### Opdracht 4 — Vijand

Maak:

```rust,ignore
struct Vijand {
    naam: String,
    gezondheid: i32,
    schade: i32,
}
```

Maak een Goblin met:

- 40 HP
- 10 schade.

---

### Opdracht 5 — Wapen

Maak:

```rust,ignore
struct Wapen {
    naam: String,
    schade: i32,
}
```

Maak een zwaard.

---

### Opdracht 6 — Positie

Maak:

```rust,ignore
struct Positie {
    x: i32,
    y: i32,
}
```

Maak een spelerpositie en print deze.

---

### Opdracht 7 — Functie

Maak:

```rust,ignore
fn toon_speler(speler: &Speler) {
    // ...
}
```

Print de gegevens van de speler.

---

### Opdracht 8 — Schade

Maak:

```rust,ignore
fn neem_schade(speler: &mut Speler, schade: i32) {
    // ...
}
```

Laat een speler schade krijgen.

---

## 21. Eindopdracht — Een echte speler

Maak een `Speler`-struct die minimaal deze informatie bevat:

```text
naam
gezondheid
goud
ervaring
levend
```

Maak vervolgens een speler.

Maak daarnaast deze functies:

```rust,ignore
fn toon_speler(speler: &Speler) {
    // ...
}

fn neem_schade(speler: &mut Speler, schade: i32) {
    // ...
}

fn geef_goud(speler: &mut Speler, hoeveelheid: i32) {
    // ...
}
```

Gebruik ze in een klein programma.

Bijvoorbeeld:

```text
=== Speler ===
Naam: Arin
HP: 100
Goud: 50
XP: 0

De speler krijgt 25 schade.

=== Speler ===
Naam: Arin
HP: 75
Goud: 50
XP: 0

De speler vindt 20 goud.

=== Speler ===
Naam: Arin
HP: 75
Goud: 70
XP: 0
```

Probeer hierbij goed gebruik te maken van borrowing:

- `&Speler` als een functie alleen kijkt
- `&mut Speler` als een functie iets verandert.

---

## Vooruitblik

Onze `Speler` bevat nu gegevens, maar nog geen eigen gedrag.

We moeten functies blijven schrijven zoals:

```rust,ignore
neem_schade(&mut speler, 20);
```

Rust heeft een manier om gedrag direct aan een datatype te koppelen.

Daarvoor gebruiken we een `impl`-blok.

In de volgende les maken we bijvoorbeeld:

```rust,ignore
impl Speler {
    fn neem_schade(&mut self, schade: i32) {
        self.gezondheid -= schade;
    }
}
```

Dan kunnen we schrijven:

```rust,ignore
speler.neem_schade(20);
```

Dat is de volgende stap: **methods en `impl`**.

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 10](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_10/).

---

## Controlelijst

Je bent klaar met dit artikel als je zonder hulp:

- [ ] weet wat een struct is en waarom je structs gebruikt
- [ ] een struct kunt definiëren en instanties ervan kunt aanmaken
- [ ] velden van een struct kunt uitlezen en aanpassen
- [ ] begrijpt hoe `mut` werkt op een complete struct
- [ ] een struct via borrowing (`&Speler` en `&mut Speler`) aan een functie kunt meegeven
- [ ] structs kunt nesten (een struct binnen een andere struct)
- [ ] een nieuwe struct vanuit een functie kunt teruggeven.
