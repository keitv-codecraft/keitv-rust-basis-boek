# 3. Variabelen en muteerbaarheid

## Wat gaan we leren?

Een programma moet informatie kunnen onthouden.

Een game moet bijvoorbeeld kunnen onthouden:

- hoeveel levens een speler heeft
- hoeveel punten een speler heeft
- hoe snel een speler beweegt
- hoeveel goud een speler heeft.

Daarvoor gebruiken we **variabelen**.

In dit artikel leren we:

- wat een variabele is
- hoe je een variabele maakt
- hoe je een waarde in een variabele bewaart
- waarom Rust variabelen standaard niet laat veranderen
- wat `mut` betekent
- hoe je variabelen gebruikt in een game
- wat het verschil is tussen een variabele veranderen en een nieuwe variabele maken.

---

## 1. Informatie onthouden

Stel dat onze speler drie levens heeft.

We kunnen dat opslaan in een variabele:

```rust
let levens = 3;
```

We kunnen de variabele een naam geven:

```text
levens
```

De waarde van de variabele is:

```text
3
```

Je kunt een variabele zien als een **naam voor een stukje informatie**.

Bijvoorbeeld:

```text
levens → 3
```

Later kan ons programma die informatie gebruiken.

```rust
let levens = 3;

println!("Je hebt {levens} levens.");
```

Het resultaat is:

```text
Je hebt 3 levens.
```

---

## 2. Een variabele maken

We gebruiken `let` om een variabele te maken.

Bijvoorbeeld:

```rust
let score = 100;
```

Hiermee zeggen we tegen Rust:

> Maak een variabele met de naam `score` en bewaar daarin de waarde `100`.

We kunnen de variabele daarna gebruiken:

```rust
let score = 100;

println!("Je score is {score}.");
```

Resultaat:

```text
Je score is 100.
```

---

## 3. Een variabele heeft een naam

De naam van een variabele kiezen we zelf.

Bijvoorbeeld:

```rust
let levens = 3;
let score = 100;
let goud = 25;
```

We kunnen deze variabelen daarna gebruiken:

```rust,ignore
println!("Levens: {levens}");
println!("Score: {score}");
println!("Goud: {goud}");
```

Resultaat:

```text
Levens: 3
Score: 100
Goud: 25
```

Een goede naam maakt code makkelijker te begrijpen.

Vergelijk:

```rust
let x = 3;
```

met:

```rust
let levens = 3;
```

Beide werken.

Maar `levens` vertelt ons meteen wat de waarde betekent.

---

## 4. Een variabele veranderen

Stel dat de speler wordt aangevallen.

De speler heeft eerst:

```text
3 levens
```

Na een aanval heeft de speler:

```text
2 levens
```

Je zou misschien verwachten dat dit werkt:

```rust,ignore
let levens = 3;

levens = 2;
```

Maar Rust geeft een foutmelding.

Waarom?

Omdat een variabele die we met alleen `let` maken standaard **niet muteerbaar** is.

Dat betekent:

> De waarde van deze variabele mag niet worden veranderd.

Dat is een bewuste keuze van Rust.

---

## 5. `mut`

Als we willen dat de waarde van een variabele kan veranderen, gebruiken we `mut`.

Bijvoorbeeld:

```rust
let mut levens = 3;
```

Het woord `mut` betekent dat de variabele **muteerbaar** is.

Nu kunnen we de waarde veranderen:

```rust
let mut levens = 3;

levens = 2;

println!("Je hebt nog {levens} levens.");
```

Resultaat:

```text
Je hebt nog 2 levens.
```

---

## 6. Waarom is `mut` nodig?

Je zou kunnen denken:

> Waarom maakt Rust variabelen niet gewoon altijd veranderbaar?

Dat zou inderdaad eenvoudiger kunnen lijken.

Rust kiest echter bewust voor een andere aanpak.

Als een variabele niet hoort te veranderen, dan kunnen we dat aangeven door `mut` weg te laten.

Bijvoorbeeld:

```rust
let maximale_levens = 3;
```

Iedereen die deze code leest, weet:

> Deze waarde hoort tijdens dit stukje programma niet te veranderen.

Als we vervolgens proberen:

```rust,ignore
maximale_levens = 5;
```

dan vertelt de compiler ons dat dit niet mag.

Dat kan helpen om fouten te voorkomen.

Je kunt daarom onthouden:

```rust
let score = 100;
```

betekent:

> `score` mag niet veranderen.

En:

```rust
let mut score = 100;
```

betekent:

> `score` mag wel veranderen.

---

## 7. Een score bijhouden

Een score is een goed voorbeeld van een muteerbare variabele.

```rust
let mut score = 0;

score = 10;

println!("Score: {score}");
```

Maar we kunnen de score ook stap voor stap verhogen.

```rust
let mut score = 0;

score = score + 10;
score = score + 25;
score = score + 50;

println!("Score: {score}");
```

De uiteindelijke score is:

```text
85
```

We kunnen dit ook korter schrijven:

```rust
let mut score = 0;

score += 10;
score += 25;
score += 50;

println!("Score: {score}");
```

`+=` betekent:

> Tel de rechter waarde op bij de variabele.

Dus:

```rust,ignore
score += 10;
```

is hetzelfde als:

```rust,ignore
score = score + 10;
```

---

## 8. Aftrekken

Hetzelfde werkt met `-=`.

```rust
let mut levens = 3;

levens -= 1;

println!("Je hebt nog {levens} levens.");
```

Resultaat:

```text
Je hebt nog 2 levens.
```

We kunnen dit gebruiken voor schade:

```rust
let mut gezondheid = 100;

gezondheid -= 25;

println!("Gezondheid: {gezondheid}");
```

Resultaat:

```text
Gezondheid: 75
```

---

## 9. Vermenigvuldigen en delen

We hebben ook:

```rust,ignore
*= 
```

en:

```rust,ignore
/=
```

Bijvoorbeeld:

```rust
let mut score = 10;

score *= 2;

println!("Score: {score}");
```

Resultaat:

```text
Score: 20
```

Of:

```rust
let mut goud = 100;

goud /= 2;

println!("Je hebt {goud} goud.");
```

Resultaat:

```text
Je hebt 50 goud.
```

---

## 10. Variabelen kunnen andere waarden gebruiken

We kunnen een nieuwe variabele maken op basis van een andere variabele.

Bijvoorbeeld:

```rust
let levens = 3;
let start_levens = levens;
```

Nu bevatten beide variabelen dezelfde waarde.

We kunnen ook rekenen:

```rust
let basis_schade = 10;
let bonus = 5;

let totale_schade = basis_schade + bonus;

println!("Schade: {totale_schade}");
```

Resultaat:

```text
Schade: 15
```

Dit wordt erg belangrijk bij gameontwikkeling.

Bijvoorbeeld:

```rust
let wapen_schade = 20;
let kracht_bonus = 5;
let totale_schade = wapen_schade + kracht_bonus;
```

De game hoeft dan niet overal `25` te schrijven.

De betekenis van de berekening blijft zichtbaar.

---

## 11. Meerdere variabelen

Een speler heeft natuurlijk meer dan één eigenschap.

Bijvoorbeeld:

```rust
let mut levens = 3;
let mut score = 0;
let mut goud = 50;
```

We kunnen ze samen gebruiken:

```rust,ignore
println!("Levens: {levens}");
println!("Score: {score}");
println!("Goud: {goud}");
```

En later veranderen:

```rust,ignore
levens -= 1;
score += 100;
goud += 25;
```

Bijvoorbeeld na het verslaan van een vijand:

```rust
let mut levens = 3;
let mut score = 0;
let mut goud = 50;

score += 100;
goud += 25;

println!("Je hebt een vijand verslagen!");
println!("Levens: {levens}");
println!("Score: {score}");
println!("Goud: {goud}");
```

---

## 12. Niet alles hoeft `mut` te zijn

Een veelgemaakte fout is om overal `mut` te gebruiken.

Dat hoeft niet.

Kijk bijvoorbeeld naar:

```rust
let maximale_levens = 3;
let mut huidige_levens = 3;
```

De maximale hoeveelheid levens verandert niet.

Daarom hoeft `maximale_levens` niet muteerbaar te zijn.

De huidige levens kunnen wel veranderen.

Daarom gebruiken we `mut`.

Dit maakt het verschil duidelijk:

```text
maximale_levens → blijft 3
huidige_levens  → kan veranderen
```

---

## 13. Een voorbeeld uit een game

Laten we een kleine game-situatie maken.

```rust
fn main() {
    let maximale_gezondheid = 100;
    let mut gezondheid = 100;

    println!("Je begint met {gezondheid} gezondheid.");

    gezondheid -= 30;

    println!("Je bent geraakt!");
    println!("Je hebt nog {gezondheid} gezondheid.");
    println!("Maximale gezondheid: {maximale_gezondheid}");
}
```

We hebben hier twee verschillende soorten informatie:

```text
maximale_gezondheid
```

verandert niet.

Maar:

```text
gezondheid
```

verandert wel.

Dat is een goede reden om alleen `gezondheid` als `mut` te markeren.

---

## 14. Een variabele opnieuw een waarde geven

We kunnen een variabele met `mut` veranderen:

```rust
let mut goud = 10;

goud = 50;
```

Dezelfde variabele bevat daarna `50`.

Je kunt dit zien als:

```text
goud → 10

goud → 50
```

De oude waarde is vervangen.

Dit is iets anders dan twee verschillende variabelen maken.

```rust
let goud = 10;
let nieuw_goud = 50;
```

Hier bestaan twee variabelen:

```text
goud       → 10
nieuw_goud → 50
```

We komen later nog terug op het maken van een nieuwe variabele met dezelfde naam. Dat heet **shadowing**.

Voorlopig hoef je alleen het verschil te kennen tussen:

```rust
let mut goud = 10;
goud = 50;
```

en:

```rust
let goud = 10;
let nieuw_goud = 50;
```

---

## 15. Een naam maakt code begrijpelijk

Stel dat we deze code zien:

```rust
let mut x = 100;
x -= 25;
```

We weten niet direct wat `x` betekent.

Misschien is het gezondheid.

Misschien geld.

Misschien score.

Deze code is duidelijker:

```rust
let mut gezondheid = 100;
gezondheid -= 25;
```

En dit is nog duidelijker:

```rust
let mut gezondheid = 100;
let schade = 25;

gezondheid -= schade;
```

Nu kunnen we bijna als een verhaal lezen wat er gebeurt:

> De speler heeft 100 gezondheid en krijgt 25 schade.

Goede namen maken programmeren veel makkelijker.

---

## 16. Een kleine game

We kunnen alles wat we tot nu toe hebben geleerd combineren.

```rust
fn main() {
    let mut gezondheid = 100;
    let mut score = 0;
    let mut goud = 25;

    println!("Een vijand valt aan!");

    gezondheid -= 20;

    println!("Je hebt nog {gezondheid} gezondheid.");

    println!("Je verslaat de vijand!");

    score += 100;
    goud += 10;

    println!("Score: {score}");
    println!("Goud: {goud}");
}
```

We hebben nu al een klein stukje game-logica.

De speler:

1. begint met gezondheid
2. wordt aangevallen
3. verliest gezondheid
4. verslaat een vijand
5. krijgt punten
6. krijgt goud.

Er is nog geen `if` en er is nog geen echte keuze.

Dat komt later.

---

## 17. Wat heb je geleerd?

Je weet nu:

- wat een variabele is
- hoe `let` werkt
- waarom variabelen standaard niet muteerbaar zijn
- wat `mut` betekent
- hoe je een variabele verandert
- hoe `+=` en `-=` werken
- hoe variabelen kunnen worden gebruikt in berekeningen
- waarom goede variabelenamen belangrijk zijn.

Het belangrijkste verschil om te onthouden is:

```rust
let score = 0;
```

De waarde mag niet veranderen.

En:

```rust
let mut score = 0;
```

De waarde mag wel veranderen.

> Gebruik `mut` alleen wanneer je de waarde daadwerkelijk wilt veranderen.

---

## Rustlings oefeningen

Ga nu verder met de Rustlings oefeningen die bij dit artikel horen. Kom daarna hier weer terug.

---

## Uitdaging: RPG status

Maak een programma dat een speler beschrijft.

Gebruik minimaal deze variabelen:

```text
naam
gezondheid
maximale_gezondheid
score
goud
```

Laat daarna het volgende gebeuren:

1. De speler begint met 100 gezondheid.
2. De speler vindt 20 goud.
3. De speler krijgt 15 schade.
4. De speler krijgt 100 punten.
5. De speler vindt nog 10 goud.

Print na afloop alle informatie.

Bijvoorbeeld:

```text
========================
      SPELER STATUS
========================

Naam: Ralph
Gezondheid: 85 / 100
Score: 100
Goud: 55
```

Probeer de code zo te schrijven dat iemand die jouw programma leest direct begrijpt wat iedere variabele betekent.

---

## Denkvragen

Je hoeft bij deze oefening alleen de vragen voor jezelf te beantwoorden. Je hoeft dus geen code te schrijven.

### 1. Standaard onveranderbaar

Waarom denk je dat Rust ervoor heeft gekozen variabelen standaard niet muteerbaar te maken?

### 2. Het `mut` keyword

Wanneer zou je `mut` gebruiken?

### 3. Wanneer `mut`?

Welke van deze variabelen moeten volgens jou `mut` zijn?

```rust
let maximale_levens = 3;
let huidige_levens = 3;
let speler_naam = "Ralph";
let score = 0;
```

### 4. Duidelijkheid

Wat is volgens jou duidelijker?

```rust
let mut x = 100;
```

of:

```rust
let mut gezondheid = 100;
```

Waarom?

---

## Controlelijst

Je bent klaar met dit artikel als je zonder hulp:

- [ ] weet wat een variabele is
- [ ] `let` kunt gebruiken
- [ ] `mut` kunt gebruiken
- [ ] weet waarom `mut` soms nodig is
- [ ] een variabele kunt veranderen
- [ ] `+=` kunt gebruiken
- [ ] `-=` kunt gebruiken
- [ ] variabelen kunt gebruiken in berekeningen
- [ ] goede namen voor variabelen kunt kiezen
- [ ] een klein stukje game-status met variabelen kunt programmeren.

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 3](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_3/).

