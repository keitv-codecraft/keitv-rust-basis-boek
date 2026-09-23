# 9. Borrowing

## Wat gaan we leren?

In de vorige les leerden we dat een waarde een eigenaar heeft.

Bijvoorbeeld:

```rust,ignore
let naam = String::from("Arin");
```

`naam` is eigenaar van de `String`.

We zagen ook dat we ownership kunnen overdragen:

```rust,ignore
let andere_naam = naam;
```

Maar vaak willen we helemaal geen eigenaar worden.

We willen alleen een waarde **even gebruiken**.

Dat heet **borrowing**.

In deze les leren we:

- wat een reference is
- wat `&` betekent
- wat borrowing betekent
- hoe je een reference aan een functie geeft
- waarom de eigenaar de waarde daarna nog steeds kan gebruiken
- wat een mutable reference is
- wat `&mut` betekent
- de belangrijkste regels voor references
- hoe borrowing in games kan worden gebruikt.

---

## 1. Het probleem uit de vorige les

We hadden bijvoorbeeld:

```rust,ignore
fn toon_naam(naam: String) {
    println!("Welkom, {naam}!");
}
```

En:

```rust,ignore
let naam = String::from("Arin");

toon_naam(naam);

println!("{naam}");
```

Dit werkt niet.

Waarom?

Omdat `naam` aan de functie wordt gegeven.

De functie krijgt ownership.

```text
main
 │
 │ naam
 ▼
"Arin"
 │
 │ ownership
 ▼
toon_naam()
```

Maar stel dat de functie alleen de naam hoeft te **lezen**.

Dan is het zonde om het eigenaarschap over te dragen.

We willen eigenlijk zeggen:

> "Hier is mijn waarde. Je mag hem gebruiken, maar hij blijft van mij."

Daarvoor hebben we een **reference**.

---

## 2. Een reference maken

Een reference maak je met `&`.

Bijvoorbeeld:

```rust,ignore
let naam = String::from("Arin");

let verwijzing = &naam;
```

Hier wordt geen nieuwe String gemaakt.

`verwijzing` verwijst naar de bestaande String.

Je kunt het ongeveer zo voorstellen:

```text
naam
 │
 ▼
"Arin"
 ▲
 │
verwijzing
```

De belangrijke eigenschap:

> `verwijzing` is niet de eigenaar van de String.

De eigenaar blijft `naam`.

---

## 3. Een reference gebruiken

We kunnen de reference gewoon gebruiken:

```rust,ignore
let naam = String::from("Arin");
let verwijzing = &naam;

println!("{verwijzing}");
```

Dit print:

```text
Arin
```

De originele variabele bestaat nog steeds:

```rust,ignore
let naam = String::from("Arin");
let verwijzing = &naam;

println!("{verwijzing}");
println!("{naam}");
```

Beide werken.

---

## 4. Borrowing

Wanneer we een reference gebruiken om tijdelijk toegang tot een waarde te geven, noemen we dat **borrowing**.

In gewone woorden:

```text
ownership:
"Hier, het is nu van jou."

borrowing:
"Je mag het gebruiken, maar het blijft van mij."
```

Dat verschil is de kern van borrowing.

---

## 5. Borrowing bij functies

Dit is een van de belangrijkste toepassingen.

We kunnen onze functie veranderen van:

```rust,ignore
fn toon_naam(naam: String) {
    println!("Welkom, {naam}!");
}
```

naar:

```rust,ignore
fn toon_naam(naam: &String) {
    println!("Welkom, {naam}!");
}
```

Nu geven we geen `String` aan de functie.

We geven een reference naar een `String`.

Bij het aanroepen schrijven we:

```rust,ignore
let naam = String::from("Arin");

toon_naam(&naam);

println!("{naam}");
```

Nu blijft `naam` gewoon bruikbaar.

---

## 6. Wat gebeurt er precies?

We hebben:

```rust,ignore
let naam = String::from("Arin");

toon_naam(&naam);
```

Je kunt dit zien als:

```text
naam
 │
 ▼
"Arin"
 ▲
 │
reference
 │
 ▼
toon_naam()
```

De functie kan de waarde bekijken.

Maar de functie wordt geen eigenaar.

Wanneer de functie klaar is, verdwijnt de reference.

De oorspronkelijke String blijft bestaan.

---

## 7. Ownership versus borrowing

Vergelijk deze twee functies.

### Ownership overdragen

```rust,ignore
fn toon_naam(naam: String) {
    println!("{naam}");
}
```

Aanroepen:

```rust,ignore
toon_naam(naam);
```

Na de aanroep kan `naam` niet meer worden gebruikt.

### Borrowing

```rust,ignore
fn toon_naam(naam: &String) {
    println!("{naam}");
}
```

Aanroepen:

```rust,ignore
toon_naam(&naam);
```

Na de aanroep kan `naam` nog steeds worden gebruikt.

Dit is een van de belangrijkste verschillen die je in Rust moet leren herkennen.

---

## 8. Waarom heet het `&`?

Het symbool:

```rust,ignore
&
```

kun je voorlopig lezen als:

> "een verwijzing naar"

Dus:

```rust,ignore
&naam
```

betekent ongeveer:

> "een verwijzing naar `naam`"

En:

```rust,ignore
&String
```

betekent:

> "een reference naar een `String`"

We zullen later preciezer leren wat een reference technisch is.

Voor nu is dit model voldoende.

---

## 9. Een reference verandert de waarde niet

Kijk naar:

```rust,ignore
let naam = String::from("Arin");

let verwijzing = &naam;
```

We hebben de String niet veranderd.

We hebben alleen toegang gekregen tot de waarde.

Daarom noemen we dit een **immutable reference**.

Dat betekent:

> Een reference waarmee je de waarde kunt bekijken, maar niet veranderen.

Bijvoorbeeld:

```rust,ignore
fn toon_item(item: &String) {
    println!("Item: {item}");
}
```

De functie kan het item bekijken.

Maar niet zomaar veranderen.

---

## 10. Meerdere references

Een van de handige eigenschappen van borrowing is dat meerdere delen van een programma dezelfde waarde kunnen bekijken.

Bijvoorbeeld:

```rust,ignore
let naam = String::from("Arin");

let eerste = &naam;
let tweede = &naam;
let derde = &naam;

println!("{eerste}");
println!("{tweede}");
println!("{derde}");
```

Dit is toegestaan.

Je kunt je voorstellen:

```text
        ┌── eerste
        │
naam ───┼── tweede
        │
        └── derde
```

Geen van deze references is eigenaar.

`naam` blijft eigenaar.

---

## 11. Borrowing in een game

Stel dat we een speler hebben:

```rust,ignore
let speler_naam = String::from("Arin");
```

We willen de naam op verschillende plaatsen gebruiken.

Bijvoorbeeld:

```rust,ignore
fn toon_welkom(naam: &String) {
    println!("Welkom, {naam}!");
}

fn toon_status(naam: &String) {
    println!("Status van {naam}");
}
```

Dan kunnen we:

```rust,ignore
let speler_naam = String::from("Arin");

toon_welkom(&speler_naam);
toon_status(&speler_naam);

println!("Speler: {speler_naam}");
```

We hebben de naam drie keer gebruikt zonder hem te kopiëren.

Dat is precies waarvoor borrowing erg nuttig is.

---

## 12. Een reference kan ook naar een andere variabele worden verwezen

We kunnen ook dit doen:

```rust,ignore
let naam = String::from("Arin");

let verwijzing = &naam;

println!("{verwijzing}");
```

Maar let op:

```rust,ignore
let verwijzing = &naam;
```

maakt geen nieuwe `String`.

Er is nog steeds maar één String.

De reference geeft alleen toegang tot die bestaande waarde.

---

## 13. Mutable borrowing

Tot nu toe konden we alleen kijken.

Maar soms willen we een functie toestaan om een waarde te veranderen.

Bijvoorbeeld:

```rust,ignore
let mut naam = String::from("Arin");
```

We kunnen een **mutable reference** maken:

```rust,ignore
let verwijzing = &mut naam;
```

`&mut` betekent:

> een mutable reference

Daarmee kan de waarde via de reference worden aangepast.

---

## 14. Een mutable reference gebruiken

Bijvoorbeeld:

```rust,ignore
let mut naam = String::from("Arin");

let verwijzing = &mut naam;

verwijzing.push_str(" de dappere");

println!("{naam}");
```

De uitvoer is:

```text
Arin de dappere
```

De reference heeft de String veranderd.

Maar de eigenaar blijft `naam`.

```text
naam
 │
 ▼
"Arin de dappere"
 ▲
 │
verwijzing
```

---

## 15. Mutable borrowing bij functies

Dit is vooral nuttig bij functies.

Bijvoorbeeld:

```rust,ignore
fn genees(gezondheid: &mut i32) {
    *gezondheid += 20;
}
```

En:

```rust,ignore
let mut gezondheid = 50;

genees(&mut gezondheid);

println!("Gezondheid: {gezondheid}");
```

De uitvoer:

```text
Gezondheid: 70
```

Hier gebeurt iets nieuws:

```rust,ignore
&mut gezondheid
```

geeft de functie tijdelijk toestemming om de waarde te veranderen.

---

## 16. Waarom staat er een `*`?

In:

```rust,ignore
*gezondheid += 20;
```

zie je een `*`.

Dat betekent hier dat we de waarde gebruiken **waar de reference naar verwijst**.

Je kunt voorlopig onthouden:

```rust,ignore
&mut gezondheid
```

betekent:

> geef een mutable reference naar `gezondheid`

En:

```rust,ignore
*gezondheid
```

betekent:

> gebruik de waarde waar deze reference naar verwijst

Dit noemen we **dereferencing**.

We gaan hier voorlopig niet veel dieper op in.

---

## 17. Een gamevoorbeeld met healing

Dit is een mooi voorbeeld van borrowing.

```rust
fn genees(gezondheid: &mut i32) {
    *gezondheid += 25;
}

fn main() {
    let mut gezondheid = 50;

    genees(&mut gezondheid);

    println!("Gezondheid: {gezondheid}");
}
```

De functie hoeft de variabele niet te bezitten.

Hij mag hem alleen tijdelijk aanpassen.

Dat is precies wat mutable borrowing betekent.

---

## 18. Borrowing en `mut`

Let goed op:

```rust,ignore
let mut gezondheid = 50;
```

is nodig.

Waarom?

Omdat de waarde veranderd wordt.

Dit werkt dus niet:

```rust,ignore
let gezondheid = 50;

genees(&mut gezondheid);
```

De compiler zal aangeven dat `gezondheid` niet mutable is.

Je hebt nodig:

```rust,ignore
let mut gezondheid = 50;
```

Dus:

> `mut` op de variabele betekent dat de waarde mag veranderen.

En:

> `&mut` betekent dat je een mutable reference maakt waarmee je die waarde kunt veranderen.

---

## 19. De belangrijkste borrowing-regels

Rust heeft hier strenge regels.

Voor nu zijn deze regels het belangrijkst.

### Regel 1 — Een gewone reference verandert niets

```rust,ignore
let naam = String::from("Arin");
let verwijzing = &naam;
```

Je kunt de String bekijken, maar niet via deze reference veranderen.

---

### Regel 2 — Een mutable reference kan veranderen

```rust,ignore
let mut naam = String::from("Arin");
let verwijzing = &mut naam;

verwijzing.push_str("!");
```

---

### Regel 3 — De eigenaar blijft eigenaar

Bij:

```rust,ignore
let naam = String::from("Arin");
let verwijzing = &naam;
```

blijft `naam` eigenaar.

---

### Regel 4 — Een reference neemt geen ownership over

Daarom kunnen we:

```rust,ignore
let naam = String::from("Arin");

toon_naam(&naam);

println!("{naam}");
```

blijven gebruiken.

---

## 20. Niet tegelijk veranderen én bekijken

Rust heeft een belangrijke regel:

> Je kunt niet tegelijkertijd een mutable reference en gewone references naar dezelfde waarde hebben.

Bijvoorbeeld:

```rust,ignore
let mut naam = String::from("Arin");

let a = &naam;
let b = &mut naam;
```

Dit geeft een compilerfout.

Waarom?

Omdat Rust wil voorkomen dat het ene deel van het programma een waarde verandert terwijl een ander deel ervan uitgaat dat de waarde hetzelfde blijft.

Dat kan bijvoorbeeld tot verwarrend gedrag leiden.

Rust voorkomt het daarom al tijdens het compileren.

---

## 21. Meerdere immutable references mogen wel

Dit mag:

```rust,ignore
let naam = String::from("Arin");

let a = &naam;
let b = &naam;
let c = &naam;

println!("{a}");
println!("{b}");
println!("{c}");
```

Dat is veilig omdat niemand de waarde verandert.

Een eenvoudige manier om de regel te onthouden:

> **Veel lezers óf één schrijver.**

Dus:

```text
lezen + lezen + lezen
        ✓
```

maar:

```text
lezen + schrijven
        ✗
```

en ook:

```text
schrijven + schrijven
        ✗
```

wanneer die references tegelijkertijd actief zijn.

---

## 22. Een mutable reference doorgeven

We kunnen dit gebruiken voor gamefuncties.

```rust,ignore
fn neem_schade(gezondheid: &mut i32, schade: i32) {
    *gezondheid -= schade;
}
```

Dan:

```rust,ignore
let mut gezondheid = 100;

neem_schade(&mut gezondheid, 30);

println!("HP: {gezondheid}");
```

Resultaat:

```text
HP: 70
```

We hebben hier mooi verschillende onderwerpen gecombineerd:

- functie
- parameters
- `mut`
- reference
- mutable reference
- rekenen.

---

## 23. Ownership, borrowing en `clone`

We kunnen nu drie verschillende manieren vergelijken.

### Ownership verplaatsen

```rust,ignore
toon_naam(naam);
```

De functie krijgt ownership.

### Borrowing

```rust,ignore
toon_naam(&naam);
```

De functie krijgt tijdelijk toegang.

### Kopiëren

```rust,ignore
toon_naam(naam.clone());
```

De functie krijgt een nieuwe kopie.

Deze drie manieren hebben verschillende betekenissen.

```text
ownership:
"Het is nu van jou."

borrowing:
"Je mag het gebruiken."

clone:
"Hier is een aparte kopie."
```

Dit onderscheid ga je in Rust heel vaak tegenkomen.

---

## 24. Borrowing in ons RPG-project

Stel dat onze speler 100 HP heeft.

We kunnen een functie maken voor schade:

```rust,ignore
fn neem_schade(gezondheid: &mut i32, schade: i32) {
    *gezondheid -= schade;
}
```

En een functie om de status te bekijken:

```rust,ignore
fn toon_gezondheid(gezondheid: &i32) {
    println!("Gezondheid: {gezondheid}");
}
```

Dan:

```rust,ignore
let mut gezondheid = 100;

toon_gezondheid(&gezondheid);

neem_schade(&mut gezondheid, 25);

toon_gezondheid(&gezondheid);
```

Resultaat:

```text
Gezondheid: 100
Gezondheid: 75
```

De functies worden geen eigenaar van `gezondheid`.

Ze gebruiken de waarde alleen tijdelijk.

---

## 25. Waarom is borrowing zo belangrijk?

Zonder borrowing zouden we vaak moeten kiezen tussen:

1. ownership overdragen
2. alles kopiëren.

Dat zou onhandig zijn.

Stel dat een gamewereld een enorme hoeveelheid gegevens bevat.

We willen niet voor iedere functie de hele wereld kopiëren.

En we willen ook niet dat iedere functie eigenaar wordt van de wereld.

Borrowing geeft ons een derde mogelijkheid:

> **Gebruik dezelfde gegevens tijdelijk zonder ze te kopiëren en zonder ownership over te dragen.**

Dit is een van de redenen waarom ownership en borrowing zo'n centrale rol spelen in Rust.

---

## 26. Veelgemaakte fouten

### Fout 1 — `&` vergeten

Dit:

```rust,ignore
fn toon_naam(naam: &String) {
    println!("{naam}");
}
```

verwacht een reference.

Dus dit:

```rust,ignore
toon_naam(naam);
```

is niet correct.

Je moet schrijven:

```rust,ignore
toon_naam(&naam);
```

---

### Fout 2 — `&mut` vergeten

Als een functie een mutable reference verwacht:

```rust,ignore
fn genees(gezondheid: &mut i32) {
    *gezondheid += 20;
}
```

moet je:

```rust,ignore
genees(&mut gezondheid);
```

gebruiken.

---

### Fout 3 — `mut` vergeten

Dit werkt niet:

```rust,ignore
let gezondheid = 50;

genees(&mut gezondheid);
```

Maak de variabele mutable:

```rust,ignore
let mut gezondheid = 50;
```

---

### Fout 4 — denken dat borrowing ownership verandert

Bij:

```rust,ignore
toon_naam(&naam);
```

blijft `naam` eigenaar.

De functie leent de waarde alleen.

---

## 27. Zelf proberen

### Opdracht 1 — Eerste reference

Maak een String.

Maak er een reference naar.

Print beide.

---

### Opdracht 2 — Functie met reference

Maak:

```rust,ignore
fn toon_naam(naam: &String) {
    println!("Naam: {naam}");
}
```

Roep deze functie aan zonder ownership over te dragen.

---

### Opdracht 3 — Meerdere references

Maak drie references naar dezelfde String.

Print ze alle drie.

---

### Opdracht 4 — Mutable reference

Maak:

```rust,ignore
let mut score = 100;
```

Maak een mutable reference en verhoog de score met 50.

---

### Opdracht 5 — Genezen

Maak een functie:

```rust,ignore
fn genees(gezondheid: &mut i32) {
    // ...
}
```

Laat de functie 20 HP toevoegen.

---

### Opdracht 6 — Schade

Maak:

```rust,ignore
fn neem_schade(gezondheid: &mut i32, schade: i32) {
    // ...
}
```

Gebruik de functie om een speler schade toe te brengen.

---

## 28. Eindopdracht — Het gevechtssysteem

Bouw voort op het gevecht uit de vorige lessen.

Maak minimaal deze functies:

```rust,ignore
fn neem_schade(gezondheid: &mut i32, schade: i32) {
    // ...
}

fn genees(gezondheid: &mut i32, hoeveelheid: i32) {
    // ...
}

fn toon_gezondheid(gezondheid: &i32) {
    // ...
}
```

Begin met:

```text
Speler HP: 100
Vijand HP: 80
```

Laat vervolgens gebeuren:

```text
De vijand valt aan!
Speler HP: 90

De speler valt aan!
Vijand HP: 60

De speler gebruikt een drankje!
Speler HP: 100
```

Gebruik **borrowing** om de gezondheid te bekijken en te veranderen.

Probeer daarbij geen `clone()` te gebruiken.

Het doel van deze opdracht is vooral dat je begrijpt waarom we references gebruiken.

---

### De belangrijkste gedachte

Als je deze drie regels uit elkaar kunt houden, heb je de kern te pakken:

```text
waarde verplaatsen
→ ownership

waarde tijdelijk bekijken
→ borrowing

waarde tijdelijk mogen veranderen
→ mutable borrowing
```

Of nog korter:

```text
String
  │
  ├── naam        → eigenaar
  │
  ├── &naam       → lenen om te lezen
  │
  └── &mut naam   → lenen om te veranderen
```

---

## Vooruitblik

We hebben nu de belangrijkste basis van Rust achter de rug:

- variabelen
- datatypes
- functies
- beslissingen
- lussen
- ownership
- borrowing.

We kunnen nu eindelijk gegevens op een nette manier gaan **bundelen**.

Tot nu toe hebben we bijvoorbeeld:

```rust,ignore
let naam = String::from("Arin");
let gezondheid = 100;
let goud = 50;
let levend = true;
```

Maar dit zijn vier losse variabelen.

In de volgende les maken we daar één logisch geheel van met een **`struct`**:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
    goud: i32,
    levend: bool,
}
```

Daarmee kunnen we voor het eerst echt beginnen met het modelleren van onze gamewereld.

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 9](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_9/).

---

## Controlelijst

Je bent klaar met dit artikel als je zonder hulp:

- [ ] kunt uitleggen wat een reference is en wat `&` betekent
- [ ] begrijpt dat borrowing geen ownership overdraagt
- [ ] een `&String` of ander leentype aan een functie kunt meegeven
- [ ] weet dat meerdere immutable references (`&`) tegelijk mogen bestaan
- [ ] weet wat `&mut` betekent en waarom de variabele zelf dan ook `mut` moet zijn
- [ ] begrijpt waarom je niet tegelijkertijd mag lezen en schrijven naar dezelfde waarde
- [ ] weet hoe je een geleende waarde kunt aanpassen (met `*`).
