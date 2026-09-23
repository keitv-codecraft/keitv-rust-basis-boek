# Ownership: wie is de eigenaar?

## 1. Wat gaan we leren?

Ownership is een van de belangrijkste ideeën van Rust.

In deze les leren we:

- wat ownership betekent
- waarom Rust ownership heeft
- wat een eigenaar is
- wat er gebeurt wanneer een variabele een waarde krijgt
- wat een **move** is
- waarom `String` anders werkt dan `i32`
- wat een scope is
- wat er gebeurt wanneer een eigenaar uit scope verdwijnt
- hoe je met `clone()` bewust een kopie kunt maken
- hoe ownership samenwerkt met functies.

We leren **nog niet** hoe we references (`&`) gebruiken. Dat komt in de volgende les.

---

## 2. Waarom heeft Rust ownership?

Programma's gebruiken geheugen.

Wanneer we bijvoorbeeld schrijven:

```rust,ignore
let score = 100;
```

moet de computer ergens ruimte reserveren om `100` te bewaren.

Bij eenvoudige programma's lijkt dat allemaal vanzelf te gaan.

Maar in grotere programma's ontstaan belangrijke vragen:

- Wanneer is geheugen niet meer nodig?
- Wie moet het opruimen?
- Wat gebeurt er als twee delen van een programma hetzelfde geheugen proberen te gebruiken?
- Wat gebeurt er als geheugen twee keer wordt opgeruimd?

Veel programmeertalen lossen dit grotendeels op met een **garbage collector**.

Rust kiest een andere aanpak.

Rust gebruikt onder andere **ownership-regels** om tijdens het compileren te kunnen bepalen wanneer bepaalde waarden niet meer nodig zijn.

Daardoor kan Rust veel geheugenproblemen voorkomen zonder een garbage collector te gebruiken.

Dat klinkt ingewikkeld, maar het basisidee is eigenlijk eenvoudig:

> **Elke waarde heeft een eigenaar.**

---

## 3. Een eerste voorbeeld

Kijk naar:

```rust,ignore
let naam = String::from("Arin");
```

We hebben een `String` gemaakt.

De variabele `naam` is nu de eigenaar van deze `String`.

Je kunt het voorlopig zo voorstellen:

```text
naam
 │
 ▼
"Arin"
```

Zolang `naam` eigenaar is, kunnen we de `String` gebruiken.

```rust,ignore
let naam = String::from("Arin");

println!("{naam}");
```

Dat werkt gewoon.

---

## 4. Wat gebeurt er met de eigenaar?

Kijk naar:

```rust,ignore
let naam = String::from("Arin");

{
    println!("{naam}");
}
```

De extra `{ ... }` maken een **scope**.

Een scope is simpel gezegd:

> Het gebied waarin een variabele bestaat en gebruikt kan worden.

Bijvoorbeeld:

```rust,ignore
{
    let naam = String::from("Arin");

    println!("{naam}");
}
```

Buiten deze scope bestaat `naam` niet meer:

```rust,ignore
{
    let naam = String::from("Arin");
}

println!("{naam}"); // fout
```

Rust weet wanneer de waarde niet meer nodig is.

Aan het einde van de scope wordt de waarde opgeruimd.

Je hoeft daar normaal gesproken niets voor te doen.

---

## 5. Waarom is `String` interessant?

Tot nu toe hebben we vaak gewerkt met:

```rust,ignore
let score = 100;
```

Een `i32` is een relatief eenvoudige waarde.

Maar een `String` kan bijvoorbeeld veel tekst bevatten:

```rust,ignore
let naam = String::from("Arin");
```

Een `String` beheert geheugen dat tijdens het programma kan veranderen.

Bijvoorbeeld:

```rust,ignore
let mut naam = String::from("Arin");

naam.push_str(" de dappere");

println!("{naam}");
```

De `String` heeft dus geheugen dat Rust moet beheren.

Daarom wordt ownership bij `String` goed zichtbaar.

---

## 6. Wat gebeurt er als we een `String` kopiëren?

Kijk naar:

```rust,ignore
let naam = String::from("Arin");
let andere_naam = naam;
```

Misschien verwacht je dat er nu twee strings bestaan:

```text
naam         → "Arin"
andere_naam  → "Arin"
```

Maar zo werkt Rust niet.

Bij deze code:

```rust,ignore
let andere_naam = naam;
```

wordt het **eigenaarschap verplaatst**.

Dit noemen we een **move**.

Na de move is `andere_naam` de eigenaar:

```text
naam         → niet meer geldig
                    │
                    ▼
andere_naam  → "Arin"
```

Daarom werkt dit niet:

```rust,ignore
let naam = String::from("Arin");
let andere_naam = naam;

println!("{naam}");
```

De compiler geeft een foutmelding.

Rust zegt in feite:

> `naam` is niet meer de eigenaar van deze waarde.

---

## 7. Waarom doet Rust dit?

Dit is een belangrijke vraag.

Stel dat beide variabelen eigenaar zouden zijn:

```text
naam        ─┐
             ├──> "Arin"
andere_naam ─┘
```

Wat gebeurt er wanneer beide variabelen verdwijnen?

Dan zouden beide kunnen proberen dezelfde waarde op te ruimen.

Dat zou ernstige geheugenproblemen kunnen veroorzaken.

Rust voorkomt dit door duidelijke ownership-regels te gebruiken:

> Er is maar één eigenaar van een bepaalde waarde.

Bij een move verhuist het eigenaarschap.

---

## 8. Een move is geen gewone kopie

Dit:

```rust,ignore
let naam = String::from("Arin");
let andere_naam = naam;
```

betekent dus niet:

> Maak een volledig nieuwe String.

Het betekent:

> Geef het eigenaarschap van deze String aan `andere_naam`.

Daarom is `naam` daarna niet meer bruikbaar.

---

## 9. Maar waarom werkt dit dan wel?

Kijk naar:

```rust,ignore
let score = 100;
let andere_score = score;

println!("{score}");
println!("{andere_score}");
```

Dit werkt wel.

Waarom?

Omdat eenvoudige types zoals `i32` **Copy** zijn.

Rust kan zulke waarden eenvoudig kopiëren.

Je kunt je voorstellen:

```text
score         → 100
andere_score  → 100
```

Er is geen ingewikkeld stuk beheerd geheugen dat door beide variabelen wordt gedeeld.

Voor nu hoef je `Copy` nog niet als technisch Rust-concept te begrijpen.

Onthoud alleen:

> Sommige eenvoudige types kunnen automatisch gekopieerd worden.

Voorbeelden zijn onder andere:

```rust,ignore
let a = 10;
let b = a;
```

en:

```rust,ignore
let levend = true;
let ander_levend = levend;
```

Daarna zijn beide variabelen nog bruikbaar.

---

## 10. `String` versus `i32`

Dit verschil is belangrijk:

```rust,ignore
let a = 10;
let b = a;

println!("{a}"); // werkt
```

maar:

```rust,ignore
let naam = String::from("Arin");
let andere_naam = naam;

println!("{naam}"); // fout
```

De reden is niet dat `String` "slecht" is.

Het verschil zit in hoe deze types hun gegevens beheren.

Een `i32` bevat de waarde zelf.

Een `String` beheert daarnaast geheugen waarin de tekst staat.

Rust moet daar heel zorgvuldig mee omgaan.

---

## 11. Een echte kopie maken met `clone`

Soms willen we wél twee zelfstandige strings.

Dan kunnen we `clone()` gebruiken:

```rust,ignore
let naam = String::from("Arin");
let andere_naam = naam.clone();

println!("{naam}");
println!("{andere_naam}");
```

Nu hebben we daadwerkelijk twee strings.

Je kunt het zo voorstellen:

```text
naam         → "Arin"

andere_naam  → "Arin"
```

Het zijn twee afzonderlijke waarden.

---

## 12. Waarom niet altijd `clone()` gebruiken?

Je zou misschien denken:

> Dan gebruik ik gewoon overal `.clone()`.

Dat kan, maar dat is niet altijd verstandig.

Een clone kan betekenen dat Rust daadwerkelijk nieuwe gegevens moet maken en kopiëren.

Bij kleine strings is dat meestal geen probleem.

Maar bij grote hoeveelheden gegevens kan kopiëren duur zijn.

Bijvoorbeeld:

```rust,ignore
let grote_kaart = ...;
let andere_kaart = grote_kaart.clone();
```

Als de kaart veel gegevens bevat, kan dat behoorlijk wat geheugen en rekentijd kosten.

Daarom heeft Rust references en borrowing.

Daarmee kunnen we gegevens gebruiken zonder het eigenaarschap over te dragen of alles te kopiëren.

**Dat behandelen we in de volgende les.**

---

## 13. Ownership en functies

Ownership wordt vooral interessant wanneer we waarden aan functies geven.

Kijk naar:

```rust,ignore
fn toon_naam(naam: String) {
    println!("{naam}");
}

fn main() {
    let naam = String::from("Arin");

    toon_naam(naam);

    println!("{naam}");
}
```

Dit werkt niet.

Waarom?

Omdat we hier:

```rust,ignore
toon_naam(naam);
```

het eigenaarschap van de `String` aan de functie geven.

De functie krijgt de waarde.

De oorspronkelijke variabele `naam` is daarna niet meer eigenaar.

---

## 14. De functie neemt ownership over

Je kunt dit zien als:

```text
main
 │
 │ naam
 ▼
"Arin"
 │
 │ ownership gaat naar functie
 ▼
toon_naam()
```

Binnen de functie is `naam` nu de eigenaar.

Wanneer de functie klaar is, verdwijnt die lokale variabele.

De `String` kan daarna worden opgeruimd.

---

## 15. Ownership teruggeven

Een functie kan ownership ook weer teruggeven.

Bijvoorbeeld:

```rust
fn maak_naam() -> String {
    String::from("Arin")
}

fn main() {
    let naam = maak_naam();

    println!("{naam}");
}
```

De functie maakt een `String`.

Die waarde wordt teruggegeven.

Daarna wordt `naam` de eigenaar.

Je kunt ownership dus verplaatsen:

```text
functie
   │
   │ return
   ▼
main
```

---

## 16. Ownership in een game

Ownership wordt veel interessanter als we aan game-objecten denken.

Stel dat we een voorwerp hebben:

```rust,ignore
let zwaard = String::from("IJzeren zwaard");
```

De speler bezit het zwaard.

Nu geven we het zwaard aan een handelaar:

```rust,ignore
fn geef_aan_handelaar(item: String) {
    println!("De handelaar krijgt: {item}");
}
```

En:

```rust,ignore
let zwaard = String::from("IJzeren zwaard");

geef_aan_handelaar(zwaard);
```

Het zwaard is nu niet meer van de oorspronkelijke variabele.

De functie heeft het eigenaarschap gekregen.

Dit sluit mooi aan bij een echte game:

```text
speler
  │
  └── zwaard

       ↓ verkopen

handelaar
  │
  └── zwaard
```

Ownership kan dus helpen om na te denken over:

> Wie bezit dit object op dit moment?

---

## 17. Nog een gamevoorbeeld: een naam

Stel dat we een spelernaam hebben:

```rust,ignore
let speler = String::from("Arin");
```

We kunnen de naam aan een functie geven:

```rust,ignore
fn start_game(naam: String) {
    println!("Welkom, {naam}!");
}
```

Dan:

```rust,ignore
start_game(speler);
```

De functie neemt ownership over.

Dat betekent dat `speler` daarna niet meer gebruikt kan worden.

Dat lijkt misschien lastig.

En precies daarom bestaat borrowing.

---

## 18. Waarom borrowing nodig is

Stel dat we alleen een naam willen **bekijken**.

We willen helemaal niet dat een functie de naam bezit.

We willen eigenlijk zeggen:

> "Hier, kijk maar even naar deze String. Je mag hem gebruiken, maar hij blijft van mij."

Dat kan met een **reference**.

Bijvoorbeeld:

```rust,ignore
fn toon_naam(naam: &String) {
    println!("{naam}");
}
```

En:

```rust,ignore
let naam = String::from("Arin");

toon_naam(&naam);

println!("{naam}");
```

Nu blijft `naam` bruikbaar.

De functie heeft namelijk niet het eigenaarschap gekregen.

Ze heeft alleen tijdelijk toegang gekregen.

Dit noemen we **borrowing**.

---

## 19. References komen in de volgende les

We hebben hier expres alvast één voorbeeld laten zien:

```rust,ignore
&naam
```

en:

```rust,ignore
&String
```

Maar we gaan dit **nog niet uitgebreid behandelen**.

De belangrijke overgang is:

### Ownership

```rust,ignore
fn toon_naam(naam: String)
```

De functie krijgt ownership.

### Borrowing

```rust,ignore
fn toon_naam(naam: &String)
```

De functie krijgt geen ownership, maar mag de waarde gebruiken.

In de volgende les gaan we precies leren:

- wat `&` betekent
- wat een reference is
- wat borrowing betekent
- waarom meerdere references mogelijk zijn
- wanneer je wel en niet mag veranderen wat je leent
- wat `&mut` betekent.

---

## 20. De belangrijkste regels

Voor nu zijn deze regels voldoende.

### Regel 1

Elke waarde heeft een eigenaar.

### Regel 2

Er is één eigenaar van een waarde.

### Regel 3

Wanneer ownership wordt verplaatst, is de oude variabele niet meer bruikbaar.

Bijvoorbeeld:

```rust,ignore
let a = String::from("Hallo");
let b = a;

// a is niet meer bruikbaar
```

### Regel 4

Sommige eenvoudige types worden automatisch gekopieerd.

Bijvoorbeeld:

```rust,ignore
let a = 10;
let b = a;

// a is nog steeds bruikbaar
```

### Regel 5

Met `clone()` kun je bewust een zelfstandige kopie maken.

```rust,ignore
let a = String::from("Hallo");
let b = a.clone();
```

### Regel 6

Een waarde wordt automatisch opgeruimd wanneer de eigenaar uit scope verdwijnt.

---

## 21. Veelgemaakte fouten

## Fout 1 — denken dat `=` altijd kopieert

Bij:

```rust,ignore
let b = a;
```

wordt niet altijd een nieuwe kopie gemaakt.

Bij een `String` wordt ownership verplaatst.

---

## Fout 2 — een moved value opnieuw gebruiken

Dit:

```rust,ignore
let naam = String::from("Arin");
let andere_naam = naam;

println!("{naam}");
```

geeft een compilerfout.

De compiler helpt je hier juist.

Hij voorkomt dat je programma onduidelijk of onveilig met geheugen omgaat.

---

## Fout 3 — overal `clone()` gebruiken

`clone()` is een nuttig hulpmiddel, maar het is niet automatisch de beste oplossing.

Als je alleen tijdelijk toegang tot een waarde nodig hebt, is borrowing vaak beter.

Dat leren we straks.

---

## 22. Zelf proberen

### Opdracht 1 — eigenaar

Maak een `String` met je naam.

Print de naam.

---

### Opdracht 2 — move

Maak:

```rust,ignore
let naam = String::from("Arin");
let andere_naam = naam;
```

Probeer daarna beide variabelen te printen.

Wat zegt de compiler?

---

### Opdracht 3 — `i32`

Doe hetzelfde met een `i32`.

```rust,ignore
let score = 100;
let andere_score = score;
```

Print beide.

Waarom werkt dit wel?

---

### Opdracht 4 — clone

Maak een String en gebruik `clone()` om een tweede String te maken.

Print beide.

---

### Opdracht 5 — scope

Maak een String binnen een blok:

```rust,ignore
{
    let schat = String::from("Gouden kroon");
}
```

Probeer de schat daarna buiten het blok te gebruiken.

---

### Opdracht 6 — functie

Maak een functie:

```rust,ignore
fn toon_item(item: String) {
    println!("Item: {item}");
}
```

Geef er een String aan.

Probeer daarna de oorspronkelijke variabele opnieuw te gebruiken.

---

## 23. Rustlings-opgaven

We maken opnieuw veel kleine oefeningen.

```text
exercises/
└── ownership/
    ├── 01_scope.rs
    ├── 02_string_move.rs
    ├── 03_use_after_move.rs
    ├── 04_integer_copy.rs
    ├── 05_boolean_copy.rs
    ├── 06_clone.rs
    ├── 07_string_scope.rs
    ├── 08_move_function.rs
    ├── 09_return_string.rs
    ├── 10_game_item.rs
    ├── 11_game_transfer.rs
    ├── 12_player_name.rs
    ├── 13_debug_move.rs
    ├── 14_debug_clone.rs
    └── 15_game_inventory.rs
```

## ⭐ Niveau 1

### 01 — Scope

Maak een String binnen een blok en print hem daar.

---

### 02 — String move

Maak:

```rust,ignore
let naam = String::from("Arin");
```

Verplaats de String naar een tweede variabele.

Print daarna alleen de tweede variabele.

---

### 03 — Use after move

In deze oefening staat expres een fout:

```rust,ignore
let naam = String::from("Arin");
let andere_naam = naam;

println!("{naam}");
```

Laat de code compileren.

Je moet dus begrijpen waarom de eerste `println!` niet mag en de code aanpassen.

---

### 04 — Integer copy

Maak:

```rust,ignore
let score = 100;
let andere_score = score;
```

Print beide scores.

---

### 05 — Boolean copy

Doe hetzelfde met een `bool`.

Bijvoorbeeld:

```rust,ignore
let levend = true;
```

---

### 06 — Clone

Maak twee zelfstandige Strings met `clone()`.

Print beide.

---

### 07 — Scope

Maak binnen een scope een game-item:

```rust,ignore
let item = String::from("Gouden sleutel");
```

Print het item binnen de scope.

Zorg ervoor dat je het item buiten de scope niet probeert te gebruiken.

---

## ⭐⭐ Niveau 2

### 08 — Ownership naar een functie

Maak:

```rust,ignore
fn gebruik_item(item: String) {
    println!("Je gebruikt: {item}");
}
```

Geef een String aan deze functie.

---

### 09 — String teruggeven

Maak een functie die een String teruggeeft:

```rust,ignore
fn maak_item() -> String {
    // ...
}
```

Gebruik de teruggegeven String in `main`.

---

### 10 — Game item

Maak een programma waarin:

1. een zwaard wordt gemaakt
2. het zwaard ownership heeft in `main`
3. het zwaard aan een functie wordt gegeven
4. de functie het zwaard print.

---

### 11 — Item overdragen

Maak een functie:

```rust,ignore
fn geef_item(item: String) {
    // ...
}
```

Laat een speler een item aan een handelaar geven.

Het oorspronkelijke item mag daarna niet meer gebruikt worden.

---

### 12 — Spelernaam

Maak een String met de naam van de speler.

Geef de naam aan een functie die een welkomstbericht print.

Bedenk vervolgens waarom je de naam daarna niet meer kunt gebruiken.

---

## ⭐⭐⭐ Niveau 3

### 13 — Debug: move

Dit programma bevat een ownership-fout:

```rust,ignore
fn main() {
    let wapen = String::from("Zwaard");
    let ander_wapen = wapen;

    println!("Wapen: {wapen}");
    println!("Ander wapen: {ander_wapen}");
}
```

Maak het programma correct.

Probeer eerst op te lossen zonder `clone()` te gebruiken.

---

### 14 — Debug: clone

Dit programma moet beide namen printen:

```rust,ignore
fn main() {
    let naam = String::from("Arin");
    let andere_naam = naam;

    println!("{naam}");
    println!("{andere_naam}");
}
```

Zorg ervoor dat beide variabelen bruikbaar zijn.

---

### 15 — Game inventory

Maak een klein programma waarin een speler een item bezit.

Bijvoorbeeld:

```text
Speler bezit: IJzeren zwaard
```

Geef vervolgens het zwaard aan een functie die het overdraagt aan een handelaar.

Probeer daarna het zwaard opnieuw vanuit `main` te gebruiken.

De compiler moet je vertellen waarom dat niet mag.

Pas daarna het programma aan zodat het gedrag logisch is.

---

## 24. Eindopdracht — De eigenaar van het zwaard

Maak een klein programma met een speler en een zwaard.

Begin bijvoorbeeld met:

```rust,ignore
let zwaard = String::from("Drakenzwaard");
```

Maak een functie:

```rust,ignore
fn verkoop_item(item: String) {
    println!("Verkocht: {item}");
}
```

Geef het zwaard aan deze functie.

Het programma moet uiteindelijk ongeveer dit doen:

```text
Speler vindt: Drakenzwaard
Verkocht: Drakenzwaard
```

Probeer daarna het programma uit te breiden met een tweede item.

Denk steeds na:

> **Wie is op dit moment de eigenaar van deze waarde?**

Je hoeft hiervoor nog geen references te gebruiken.

---

## 25. Wat moet je nu kunnen?

Na deze les zou je moeten begrijpen:

- [ ] wat ownership betekent
- [ ] wat een eigenaar is
- [ ] wat een scope is
- [ ] waarom `String` anders werkt dan `i32`
- [ ] wat een move is
- [ ] waarom een variabele na een move niet meer gebruikt kan worden
- [ ] waarom eenvoudige `Copy`-types anders werken
- [ ] hoe `clone()` een zelfstandige kopie maakt
- [ ] hoe ownership naar een functie kan worden verplaatst
- [ ] hoe een functie ownership kan teruggeven
- [ ] waarom borrowing handig kan zijn.

Het belangrijkste dat je uit deze les moet onthouden is:

> **Als je een waarde aan iets anders geeft, vraag jezelf dan af: wie is nu de eigenaar?**

---

## Vooruitblik: borrowing

We hebben nu gezien dat dit:

```rust,ignore
fn toon_naam(naam: String)
```

ownership overdraagt.

Maar vaak willen we helemaal geen ownership overdragen.

We willen alleen zeggen:

> "Je mag deze waarde even bekijken of gebruiken, maar hij blijft van mij."

Daarvoor gebruikt Rust **references**:

```rust,ignore
fn toon_naam(naam: &String)
```

Dat is het onderwerp van de volgende les: **references en borrowing**.

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 8](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_8/).

