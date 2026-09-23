# 5. Commentaar

## Wat gaan we leren?

In dit artikel leren we:

- wat commentaar is
- waarom je commentaar gebruikt
- hoe je commentaar schrijft in Rust
- wat het verschil is tussen `//` en `/* ... */`
- wanneer commentaar nuttig is
- waarom je niet iedere regel code hoeft uit te leggen.

Commentaar wordt door Rust **genegeerd**.

Het is bedoeld voor mensen die de code lezen.

---

## 1. Wat is commentaar?

Commentaar is tekst in je programma die niet wordt uitgevoerd.

Bijvoorbeeld:

```rust
fn main() {
    // Dit is commentaar.
    println!("Hallo!");
}
```

Rust voert alleen de `println!` uit.

Het commentaar is bedoeld voor de programmeur.

Je kunt het bijvoorbeeld gebruiken om uit te leggen wat een stuk code doet.

---

## 2. Commentaar met `//`

De meest gebruikte vorm van commentaar begint met:

```text
//
```

Alles wat daarna op dezelfde regel staat, is commentaar.

Bijvoorbeeld:

```rust
fn main() {
    // Dit wordt niet uitgevoerd.
    println!("Hallo!");
}
```

Je kunt ook commentaar achter een regel zetten:

```rust
let gezondheid = 100; // Begingezondheid van de speler
```

Dat werkt ook.

---

## 3. Commentaar over meerdere regels

Soms wil je een groter stuk tekst als commentaar schrijven.

Daarvoor kun je:

```text
/*
...
*/
```

gebruiken.

Bijvoorbeeld:

```rust
/*
    Dit programma is een klein
    voorbeeld voor onze game.
*/

fn main() {
    println!("Welkom!");
}
```

Alles tussen `/*` en `*/` is commentaar.

Voor korte opmerkingen gebruiken Rust-programmeurs meestal `//`.

---

## 4. Waarom commentaar?

Commentaar kan verschillende doelen hebben.

Bijvoorbeeld uitleggen **waarom** iets gebeurt.

Stel:

```rust,ignore
gezondheid -= 20;
```

Je kunt schrijven:

```rust,ignore
// De speler krijgt schade van de vijand.
gezondheid -= 20;
```

Dat kan nuttig zijn.

Maar dit:

```rust,ignore
// Trek 20 af van gezondheid.
gezondheid -= 20;
```

is minder nuttig.

De code vertelt dat namelijk al.

Een goede vuistregel is:

> Gebruik commentaar vooral om uit te leggen **waarom** iets gebeurt, niet om letterlijk te herhalen **wat** de code doet.

---

## 5. Een voorbeeld uit een game

Vergelijk:

```rust
let mut gezondheid = 100;
gezondheid -= 25;
```

met:

```rust
// De speler begint met volledige gezondheid.
let mut gezondheid = 100;

// De vijand raakt de speler.
gezondheid -= 25;
```

Het tweede voorbeeld kan gemakkelijker te begrijpen zijn.

Maar we hoeven niet iedere regel vol te zetten met commentaar.

Dit is bijvoorbeeld overdreven:

```rust
// Maak een variabele.
let mut gezondheid = 100;

// Trek 25 af.
gezondheid -= 25;

// Print de gezondheid.
println!("{gezondheid}");
```

De code is hier al duidelijk genoeg.

---

## 6. Commentaar kan ook tijdelijk helpen

Tijdens het programmeren kan commentaar handig zijn om iets tijdelijk uit te schakelen.

Bijvoorbeeld:

```rust
fn main() {
    println!("Start game!");

    // println!("Deze tekst zie je nu niet.");

    println!("Game gestart.");
}
```

De tweede `println!` wordt niet uitgevoerd.

Dit wordt soms gebruikt om tijdens het testen snel een stukje code uit te schakelen.

Let wel op: als je een groot programma hebt, kan veel oude uitgecommentarieerde code verwarrend worden.

---

## 7. Commentaar bij functies

Nu we functies hebben geleerd, kunnen we functies documenteren.

Bijvoorbeeld:

```rust
// Berekent de schade van een aanval.
fn bereken_schade(kracht: i32, wapen: i32) -> i32 {
    kracht + wapen
}
```

Dit is nuttiger dan:

```rust
// Functie bereken schade.
fn bereken_schade(kracht: i32, wapen: i32) -> i32 {
    kracht + wapen
}
```

De eerste versie vertelt iets over de bedoeling.

---

## 8. Commentaar is geen vervanging voor goede namen

Stel dat we dit schrijven:

```rust
let x = 100; // gezondheid van de speler
```

Het commentaar maakt duidelijk wat `x` betekent.

Maar nog beter is:

```rust
let gezondheid = 100;
```

Dan hebben we het commentaar misschien helemaal niet nodig.

Dit is een belangrijke programmeerregel:

> **Gebruik duidelijke namen zodat je code zichzelf zoveel mogelijk uitlegt.**

Vergelijk:

```rust
let x = 100;
let y = 20;
let z = x - y;
```

met:

```rust
let gezondheid = 100;
let schade = 20;
let nieuwe_gezondheid = gezondheid - schade;
```

De tweede versie is veel duidelijker.

---

## 9. Commentaar bij onze game

Een klein voorbeeld:

```rust
// Berekent hoeveel schade een aanval doet.
fn bereken_schade(kracht: i32, wapen: i32) -> i32 {
    kracht + wapen
}

fn main() {
    // De speler begint met 100 gezondheid.
    let mut gezondheid = 100;

    let kracht = 10;
    let wapenschade = 7;

    let schade = bereken_schade(kracht, wapenschade);

    // De vijand valt de speler aan.
    gezondheid -= schade;

    println!("Gezondheid: {gezondheid}");
}
```

Hier helpen de commentaren om de bedoeling van het programma te volgen.

---

## 10. Code tijdelijk uitschakelen (uitcommentariëren)

Commentaar kan ook helpen wanneer je een fout probeert te vinden.

Stel dat je dit programma hebt:

```rust
fn main() {
    let mut gezondheid = 100;

    gezondheid -= 20;
    gezondheid -= 30;

    println!("Gezondheid: {gezondheid}");
}
```

Je vermoedt dat de tweede schade de fout veroorzaakt.

Je kunt tijdelijk schrijven:

```rust
fn main() {
    let mut gezondheid = 100;

    gezondheid -= 20;

    // gezondheid -= 30;

    println!("Gezondheid: {gezondheid}");
}
```

Nu kun je testen of het programma zich anders gedraagt.

Dit is een eenvoudige vorm van **debuggen**.

Later leren we betere manieren om fouten systematisch te vinden.

---

## De belangrijkste gedachte

Goede code hoeft niet overal commentaar bij te hebben.

Goede code is in de eerste plaats:

- duidelijk
- logisch opgebouwd
- voorzien van goede namen
- verdeeld over begrijpelijke functies.

Commentaar is er vervolgens om dingen uit te leggen die je niet eenvoudig uit de code zelf kunt aflezen.

Vanaf het volgende artikel wordt dat steeds belangrijker.

Onze programma's gaan namelijk niet meer alleen stap voor stap uitvoeren wat we opschrijven.

Ze gaan **beslissingen nemen**.

```text
Als de speler nog leeft:
    speel verder.

Anders:
    game over.
```

Daarvoor hebben we **control flow** nodig.

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 5](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_5/).

---

## Controlelijst

Je bent klaar met dit artikel als je zonder hulp:

- [ ] weet wat commentaar is
- [ ] begrijpt dat Rust commentaar volledig negeert tijdens het uitvoeren
- [ ] enkellijns commentaar met `//` kunt schrijven
- [ ] meerregelig commentaar met `/* ... */` kunt schrijven
- [ ] weet wanneer commentaar nuttig is (uitleggen *waarom*, niet letterlijk herhalen *wat*)
- [ ] begrijpt dat duidelijke namen voorrang hebben op extra commentaar
- [ ] code tijdelijk kunt uitcommentariëren bij het zoeken naar fouten.
