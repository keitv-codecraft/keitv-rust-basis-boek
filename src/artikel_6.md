# 6. Beslissingen met if en else

## Wat gaan we leren?

Tot nu toe deed ons programma meestal precies wat we hadden opgeschreven:

```text
doe dit
doe dat
doe dit
doe dat
```

Maar een game moet beslissingen kunnen nemen.

Bijvoorbeeld:

```text
Als de speler geen gezondheid meer heeft:
    game over.

Anders:
    speel verder.
```

In dit artikel leren we hoe we dat in Rust kunnen doen.

We leren:

- wat **control flow** betekent
- hoe `if` werkt
- voorwaarden schrijven
- vergelijkingen maken
- `else` gebruiken
- meerdere mogelijkheden maken met `else if`
- `if` combineren met functies
- `if` gebruiken in een game
- hoe we compilerfouten bij voorwaarden kunnen oplossen.

---

## 1. Wat is control flow?

**Control flow** betekent letterlijk ongeveer:

> de manier waarop een programma bepaalt welke code wordt uitgevoerd.

Tot nu toe zagen onze programma's er ongeveer zo uit:

```rust
fn main() {
    println!("Stap 1");
    println!("Stap 2");
    println!("Stap 3");
}
```

De computer voert de regels achter elkaar uit:

```text
Stap 1
  ↓
Stap 2
  ↓
Stap 3
```

Maar soms willen we dat een programma een keuze maakt.

Bijvoorbeeld:

```text
gezondheid = 0

Als gezondheid 0 of lager is:
    game over

Anders:
    ga verder
```

Daarvoor gebruiken we `if`.

---

## 2. De eenvoudigste `if`

Een `if` betekent:

> Als deze voorwaarde waar is, voer dan deze code uit.

Bijvoorbeeld:

```rust
fn main() {
    let gezondheid = 100;

    if gezondheid > 0 {
        println!("De speler leeft!");
    }
}
```

De voorwaarde is:

```rust,ignore
gezondheid > 0
```

Rust controleert deze voorwaarde.

In dit geval is:

```text
100 > 0
```

waar.

Dus wordt de code tussen de `{` en `}` uitgevoerd.

De uitvoer is:

```text
De speler leeft!
```

---

## 3. Een `if` heeft een `bool` nodig

Een `if` verwacht een waarde van het type `bool`.

Bijvoorbeeld:

```rust,ignore
true
```

of:

```rust,ignore
false
```

Dit is dus geldig:

```rust,ignore
if true {
    println!("Dit wordt uitgevoerd.");
}
```

En:

```rust,ignore
if false {
    println!("Dit wordt niet uitgevoerd.");
}
```

Maar dit niet:

```rust,ignore
if 10 {
    println!("Hallo!");
}
```

`10` is een getal en geen `bool`.

Rust is hier streng in.

In sommige andere programmeertalen wordt een getal soms automatisch als `true` of `false` geïnterpreteerd.

Rust doet dat niet.

Dat is expres.

---

## 4. Vergelijken

We gebruiken vaak vergelijkingsoperatoren om een `bool` te maken.

De belangrijkste zijn:

| Operator | Betekenis |
| --- | --- |
| `==` | gelijk aan |
| `!=` | niet gelijk aan |
| `>` | groter dan |
| `<` | kleiner dan |
| `>=` | groter dan of gelijk aan |
| `<=` | kleiner dan of gelijk aan |

Bijvoorbeeld:

```rust,ignore
let gezondheid = 50;

gezondheid > 0
```

is:

```text
true
```

Terwijl:

```rust,ignore
gezondheid <= 0
```

is:

```text
false
```

---

## 5. `==` is niet hetzelfde als `=`

Dit is een belangrijke beginnersfout.

Een enkele `=` betekent:

> Geef deze variabele deze waarde.

Bijvoorbeeld:

```rust,ignore
let score = 100;
```

Of:

```rust,ignore
let mut score = 100;
score = 200;
```

Twee `=`-tekens betekenen:

> Vergelijk deze twee waarden.

Bijvoorbeeld:

```rust,ignore
score == 100
```

Dit levert een `bool` op.

Bijvoorbeeld:

```rust,ignore
let score = 100;

if score == 100 {
    println!("Precies 100 punten!");
}
```

---

## 6. `if` met een variabele

Een `if` kan natuurlijk een variabele controleren.

```rust
fn main() {
    let score = 150;

    if score >= 100 {
        println!("Je hebt minstens 100 punten!");
    }
}
```

Als we de score veranderen:

```rust,ignore
let score = 50;
```

wordt de tekst niet afgedrukt.

De voorwaarde:

```text
50 >= 100
```

is namelijk `false`.

---

## 7. `else`

Met alleen `if` kunnen we zeggen:

> Doe iets als de voorwaarde waar is.

Maar vaak willen we ook zeggen:

> Doe iets anders als de voorwaarde niet waar is.

Daarvoor gebruiken we `else`.

```rust
fn main() {
    let gezondheid = 0;

    if gezondheid > 0 {
        println!("De speler leeft!");
    } else {
        println!("Game over!");
    }
}
```

De computer doet nu één van twee dingen:

```text
gezondheid > 0?
       │
   ┌───┴───┐
  ja       nee
  │         │
leeft    game over
```

---

## 8. Een gamevoorbeeld

Dit is natuurlijk precies wat we nodig hebben voor onze game.

```rust
fn main() {
    let gezondheid = 0;

    if gezondheid > 0 {
        println!("Je leeft nog!");
        println!("Ga verder met spelen.");
    } else {
        println!("Je bent verslagen!");
        println!("GAME OVER");
    }
}
```

Als de gezondheid `0` is:

```text
Je bent verslagen!
GAME OVER
```

Als de gezondheid bijvoorbeeld `30` is:

```text
Je leeft nog!
Ga verder met spelen.
```

---

## 9. `else if`

Soms hebben we meer dan twee mogelijkheden.

Stel dat we de gezondheid van een speler willen beoordelen.

We kunnen bijvoorbeeld zeggen:

```text
meer dan 75 → gezond
51 t/m 75   → gewond
1 t/m 50    → ernstig gewond
0 of minder → dood
```

Daarvoor kunnen we `else if` gebruiken.

```rust
fn main() {
    let gezondheid = 40;

    if gezondheid > 75 {
        println!("Je bent gezond.");
    } else if gezondheid > 50 {
        println!("Je bent gewond.");
    } else if gezondheid > 0 {
        println!("Je bent ernstig gewond.");
    } else {
        println!("Je bent dood.");
    }
}
```

Rust controleert de voorwaarden van boven naar beneden.

Bij `gezondheid = 40`:

```text
40 > 75  → false
40 > 50  → false
40 > 0   → true
```

Dus wordt:

```text
Je bent ernstig gewond.
```

uitgevoerd.

Zodra Rust een ware voorwaarde vindt, wordt de rest van deze `if`-keten overgeslagen.

---

## 10. De volgorde is belangrijk

Kijk naar:

```rust,ignore
if gezondheid > 0 {
    println!("Je leeft.");
} else if gezondheid > 50 {
    println!("Je hebt meer dan 50 gezondheid.");
}
```

Dit is fout qua logica.

Waarom?

Als:

```text
gezondheid = 80
```

dan is:

```text
gezondheid > 0
```

al `true`.

Rust voert dus de eerste tak uit.

De tweede voorwaarde wordt nooit bereikt.

Beter is:

```rust,ignore
if gezondheid > 50 {
    println!("Je hebt meer dan 50 gezondheid.");
} else if gezondheid > 0 {
    println!("Je leeft, maar hebt weinig gezondheid.");
} else {
    println!("Je bent dood.");
}
```

**De volgorde van voorwaarden is dus belangrijk.**

---

## 11. Vergelijkingen met getallen

We kunnen alle vergelijkingsoperatoren gebruiken.

### Groter dan

```rust,ignore
if score > 100 {
    println!("Hoge score!");
}
```

### Kleiner dan

```rust,ignore
if gezondheid < 20 {
    println!("Pas op!");
}
```

### Groter dan of gelijk aan

```rust,ignore
if score >= 100 {
    println!("Je hebt minstens 100 punten.");
}
```

### Kleiner dan of gelijk aan

```rust,ignore
if gezondheid <= 0 {
    println!("Game over!");
}
```

### Gelijk aan

```rust,ignore
if score == 100 {
    println!("Precies 100 punten!");
}
```

### Niet gelijk aan

```rust,ignore
if score != 0 {
    println!("Je hebt punten!");
}
```

---

## 12. `if` kan ook tekst vergelijken

We kunnen niet alleen getallen vergelijken.

Bijvoorbeeld:

```rust,ignore
let wapen = "zwaard";

if wapen == "zwaard" {
    println!("Je hebt een zwaard.");
}
```

Dit werkt omdat Rust de twee waarden met elkaar kan vergelijken.

We kunnen ook:

```rust,ignore
if wapen != "boog" {
    println!("Je hebt geen boog.");
}
```

Voorlopig gebruiken we vooral eenvoudige tekstvergelijkingen. Later leren we meer over strings en andere soorten gegevens.

---

## 13. `if` combineren met functies

Onze functies en `if` kunnen samenwerken.

Bijvoorbeeld:

```rust
fn is_levend(gezondheid: i32) -> bool {
    gezondheid > 0
}

fn main() {
    let gezondheid = 50;

    if is_levend(gezondheid) {
        println!("Je kunt verder spelen.");
    } else {
        println!("Game over!");
    }
}
```

Dit is een mooie combinatie van wat we tot nu toe hebben geleerd.

De functie:

```rust,ignore
is_levend()
```

berekent een `bool`.

Daarna gebruikt `if` dat resultaat.

```text
gezondheid
    ↓
is_levend()
    ↓
true / false
    ↓
if
    ↓
actie
```

---

## 14. Een functie kan zelf `if` gebruiken

Het werkt ook andersom.

Een functie kan zelf een beslissing nemen.

Bijvoorbeeld:

```rust,ignore
fn bereken_bonus(score: i32) -> i32 {
    if score >= 1000 {
        500
    } else {
        100
    }
}
```

De functie geeft dus verschillende waarden terug.

```rust,ignore
let bonus = bereken_bonus(1200);
```

geeft:

```text
500
```

Terwijl:

```rust,ignore
let bonus = bereken_bonus(500);
```

geeft:

```text
100
```

Let op dat de laatste tak van de `if`-structuur hier geen `;` heeft:

```rust,ignore
if score >= 1000 {
    500
} else {
    100
}
```

De hele `if` is namelijk een **expressie** die een waarde kan opleveren.

---

## 15. `if` als expressie

Dit is een belangrijk Rust-concept.

We kunnen schrijven:

```rust,ignore
let bonus;

if score >= 1000 {
    bonus = 500;
} else {
    bonus = 100;
}
```

Maar Rust maakt het mogelijk om het korter te schrijven:

```rust,ignore
let bonus = if score >= 1000 {
    500
} else {
    100
};
```

Hier levert de `if` dus een waarde op.

Je kunt het lezen als:

> `bonus` krijgt 500 als score minstens 1000 is, anders 100.

Dit zullen we later vaak gebruiken.

---

## 16. De twee kanten moeten hetzelfde datatype opleveren

Omdat een `if` een waarde kan teruggeven, moeten de mogelijke resultaten bij elkaar passen.

Dit is goed:

```rust,ignore
let bonus = if score >= 1000 {
    500
} else {
    100
};
```

Beide takken leveren een `i32` op.

Maar dit is niet goed:

```rust,ignore
let bonus = if score >= 1000 {
    500
} else {
    "geen bonus"
};
```

De ene kant geeft een getal en de andere kant tekst.

Rust weet dan niet welk datatype `bonus` moet krijgen.

Dit soort beperkingen lijkt soms streng, maar voorkomt veel fouten.

---

## 17. Meerdere voorwaarden combineren

Soms willen we controleren of twee voorwaarden tegelijk waar zijn.

Bijvoorbeeld:

> De speler mag de deur openen als hij een sleutel heeft én minstens 10 goud heeft.

Daarvoor gebruiken we `&&`.

```rust,ignore
let heeft_sleutel = true;
let goud = 25;

if heeft_sleutel && goud >= 10 {
    println!("De deur gaat open.");
}
```

`&&` betekent:

> EN

Beide voorwaarden moeten `true` zijn.

---

## 18. `||` — OF

Soms is één van meerdere voorwaarden genoeg.

Daarvoor gebruiken we `||`.

Bijvoorbeeld:

> Je kunt vluchten als je een paard hebt óf een teleportatie-item.

```rust,ignore
let heeft_paard = false;
let heeft_teleport = true;

if heeft_paard || heeft_teleport {
    println!("Je kunt vluchten!");
}
```

`||` betekent:

> OF

Minstens één van de voorwaarden moet `true` zijn.

---

## 19. `!` — NIET

Met `!` draaien we een `bool` om.

```rust,ignore
let is_vergiftigd = false;

if !is_vergiftigd {
    println!("Je bent niet vergiftigd.");
}
```

`!false` wordt `true`.

En:

```rust,ignore
!true
```

wordt:

```text
false
```

Voorlopig kun je dit onthouden als:

```text
! = niet
```

---

## 20. Een complexere gamevoorwaarde

We kunnen nu bijvoorbeeld schrijven:

```rust,ignore
let gezondheid = 80;
let heeft_sleutel = true;
let goud = 25;

if gezondheid > 0 && heeft_sleutel && goud >= 10 {
    println!("Je kunt de schatkamer binnen.");
} else {
    println!("Je kunt de schatkamer niet binnen.");
}
```

De speler moet dus:

1. leven
2. een sleutel hebben
3. minstens 10 goud hebben.

Alle drie moeten waar zijn.

---

## 21. Haakjes kunnen helpen

Bij ingewikkeldere voorwaarden kunnen haakjes duidelijkheid geven.

Bijvoorbeeld:

```rust,ignore
if gezondheid > 0 && (heeft_sleutel || heeft_teleport) {
    println!("Je kunt ontsnappen.");
}
```

Dit betekent:

> Je moet leven én je moet óf een sleutel óf een teleport hebben.

De haakjes maken duidelijk welke voorwaarden bij elkaar horen.

---

## 22. Een groter gamevoorbeeld

We kunnen nu een eenvoudige gevechtssituatie maken:

```rust
fn bereken_schade(kracht: i32, wapen: i32) -> i32 {
    kracht + wapen
}

fn is_levend(gezondheid: i32) -> bool {
    gezondheid > 0
}

fn main() {
    let speler_kracht = 10;
    let wapen_schade = 8;
    let mut vijand_gezondheid = 25;

    let schade = bereken_schade(speler_kracht, wapen_schade);

    println!("De speler valt aan!");
    println!("Schade: {schade}");

    vijand_gezondheid -= schade;

    println!("Vijandelijke gezondheid: {vijand_gezondheid}");

    if is_levend(vijand_gezondheid) {
        println!("De vijand leeft nog!");
    } else {
        println!("De vijand is verslagen!");
    }
}
```

Hier hebben we nu al verschillende concepten gecombineerd:

- variabelen
- `mut`
- datatypes
- functies
- parameters
- returnwaarden
- `bool`
- `if` en `else`.

Dit begint al behoorlijk op echte game-logica te lijken.

---

## 23. Veelgemaakte fouten

### Fout 1 — `=` gebruiken in plaats van `==`

Fout:

```rust,ignore
if score = 100 {
    println!("100 punten!");
}
```

`=` betekent toewijzen.

Voor vergelijken gebruiken we:

```rust,ignore
if score == 100 {
    println!("100 punten!");
}
```

---

### Fout 2 — Een getal gebruiken als voorwaarde

Fout:

```rust,ignore
if gezondheid {
    println!("Levend!");
}
```

`gezondheid` is een `i32`, geen `bool`.

Gebruik bijvoorbeeld:

```rust,ignore
if gezondheid > 0 {
    println!("Levend!");
}
```

---

### Fout 3 — Een `if` zonder blok

Rust gebruikt `{}`:

```rust,ignore
if gezondheid > 0 {
    println!("Levend!");
}
```

Niet:

```rust,ignore
if gezondheid > 0
    println!("Levend!");
```

---

### Fout 4 — Verkeerde volgorde bij `else if`

Denk goed na over welke voorwaarden eerst gecontroleerd moeten worden.

Bij bereiken van waarden is het vaak logisch om van hoog naar laag te werken.

---

### Fout 5 — `&&` en `||` door elkaar halen

```rust,ignore
if heeft_sleutel && heeft_zwaard {
```

betekent:

> beide nodig.

Terwijl:

```rust,ignore
if heeft_sleutel || heeft_zwaard {
```

betekent:

> één van beide is voldoende.

---

## 24. Kleine eindopdracht — De eerste echte gamebeslissing

Maak een klein programma waarin een speler probeert een schatkamer binnen te gaan.

De speler heeft:

```rust,ignore
let gezondheid = 80;
let goud = 25;
let heeft_sleutel = true;
```

De schatkamer gaat alleen open als:

- de speler leeft
- de speler een sleutel heeft
- de speler minstens 10 goud heeft.

Als alles klopt:

```text
De deur gaat open!
Je vindt een schat!
```

Anders:

```text
De deur blijft gesloten.
```

### Extra uitdaging

Maak een tweede versie waarin je verschillende redenen kunt geven.

Bijvoorbeeld:

```text
Je hebt geen sleutel.
```

of:

```text
Je hebt niet genoeg goud.
```

of:

```text
Je bent verslagen.
```

Daarvoor heb je meerdere `if`/`else if`-takken nodig.

---

## De belangrijkste gedachte

Tot nu toe vertelden we de computer vooral:

> **Doe dit. Doe daarna dat.**

Met `if` kunnen we nu zeggen:

> **Doe dit als dit waar is, en anders doe je dat.**

Dat is een enorme stap.

Onze programma's kunnen vanaf nu reageren op de toestand van de game.

```text
gezondheid
    ↓
  if / else
    ↓
wat gebeurt er?
```

Vanaf hier wordt de game-logica steeds interessanter.

In het volgende artikel leren we hoe we met **lussen** herhaaldelijk code kunnen uitvoeren. Daarmee kunnen we bijvoorbeeld een game-loop nabootsen, meerdere vijanden verwerken en acties herhalen totdat aan een bepaalde voorwaarde is voldaan.

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 6](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_6/).

---

## Controlelijst

Je bent klaar met dit artikel als je zonder hulp:

- [ ] wat control flow betekent
- [ ] wat `if` doet en waarom het een `bool` nodig heeft
- [ ] vergelijkingsoperatoren (`==`, `!=`, `<`, `>`, `<=`, `>=`) kunt gebruiken
- [ ] hoe `else` en `else if` werken
- [ ] waarom de volgorde van voorwaarden belangrijk is
- [ ] logische operatoren (`&&`, `||`, `!`) kunt combineren
- [ ] dat een `if`/`else` als expressie een waarde kan teruggeven
- [ ] `if` kunt combineren met functies en gamebeslissingen.
