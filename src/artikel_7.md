# Lussen: iets steeds opnieuw doen

## 1. Wat gaan we leren?

In dit artikel leren we hoe we een stukje code **meerdere keren kunnen uitvoeren**.

We leren:

- waarom we lussen nodig hebben
- de `loop`-lus
- een lus stoppen met `break`
- een waarde teruggeven met `break`
- de `while`-lus
- de `for`-lus
- werken met bereiken zoals `1..=10`
- lussen gebruiken in games
- lussen combineren met `if`.

Aan het einde kunnen we bijvoorbeeld een countdown maken, een aantal vijanden aanvallen of een speler meerdere beurten laten spelen.

---

## 2. Waarom hebben we lussen nodig?

Stel dat we drie keer iets willen afdrukken:

```rust,ignore
println!("Aanval!");
println!("Aanval!");
println!("Aanval!");
```

Dat werkt.

Maar wat als we het **100 keer** willen doen?

Dan willen we natuurlijk niet honderd regels schrijven.

Daarom hebben programmeertalen lussen.

Een lus betekent eigenlijk:

> **Doe dit opnieuw totdat we zeggen dat het genoeg is.**

Een eenvoudig voorbeeld:

```rust,ignore
for _ in 0..5 {
    println!("Aanval!");
}
```

Dit geeft vijf keer `Aanval!`.

Lussen zijn overal in games te vinden:

- alle vijanden bekijken
- alle spelers een beurt geven
- een countdown uitvoeren
- een animatie stap voor stap uitvoeren
- een spelronde herhalen
- alle voorwerpen in een inventaris bekijken
- een aantal dobbelstenen gooien.

---

## 3. De eenvoudigste lus: `loop`

Rust heeft een heel eenvoudige lus:

```rust,ignore
loop {
    println!("Hallo!");
}
```

Deze lus gaat **voor altijd door**.

Dat is meestal niet wat we willen.

We moeten de lus daarom ergens stoppen.

Dat doen we met `break`:

```rust,ignore
loop {
    println!("Hallo!");
    break;
}
```

Het programma print `Hallo!` en stopt daarna met de lus.

`break` betekent:

> Stop onmiddellijk met deze lus.

---

## 4. Een lus meerdere keren laten lopen

We kunnen een teller gebruiken:

```rust,ignore
let mut teller = 0;

loop {
    println!("Teller: {teller}");

    teller += 1;

    if teller == 5 {
        break;
    }
}
```

Hier gebeurt het volgende:

1. `teller` begint op `0`.
2. We printen de teller.
3. We verhogen de teller met `1`.
4. We controleren of de teller `5` is.
5. Als dat zo is, stoppen we.

De uitvoer is:

```text
Teller: 0
Teller: 1
Teller: 2
Teller: 3
Teller: 4
```

Let goed op: wanneer `teller` 5 wordt, voeren we de `println!` niet nog een keer uit.

---

## 5. `break` kan ook een waarde teruggeven

Een interessante eigenschap van `loop` is dat we met `break` ook een waarde kunnen teruggeven.

Bijvoorbeeld:

```rust,ignore
let resultaat = loop {
    break 42;
};

println!("Het resultaat is {resultaat}");
```

`resultaat` krijgt de waarde `42`.

Dit is handig als we een lus gebruiken om iets te zoeken.

Bijvoorbeeld:

```rust,ignore
let gevonden = loop {
    let getal = 7;

    if getal == 7 {
        break getal;
    }
};
```

Voorlopig hoef je deze mogelijkheid nog niet veel te gebruiken. Het belangrijkste is:

> `break` stopt een lus.

---

## 6. `while`: zolang iets waar is

Een andere soort lus is `while`.

De vorm is:

```rust,ignore
while voorwaarde {
    // code
}
```

De code wordt uitgevoerd **zolang de voorwaarde `true` is**.

Bijvoorbeeld:

```rust,ignore
let mut gezondheid = 100;

while gezondheid > 0 {
    println!("Gezondheid: {gezondheid}");

    gezondheid -= 25;
}
```

De uitvoer is:

```text
Gezondheid: 100
Gezondheid: 75
Gezondheid: 50
Gezondheid: 25
```

Wanneer `gezondheid` `0` wordt, is:

```rust,ignore
gezondheid > 0
```

niet meer waar.

De lus stopt dus.

---

## 7. `while` in een game

Dit lijkt al behoorlijk op een echte game.

```rust,ignore
let mut gezondheid = 100;

while gezondheid > 0 {
    println!("De vijand valt aan!");

    gezondheid -= 20;

    println!("Je hebt nog {gezondheid} HP.");
}

println!("Game over!");
```

Hiermee hebben we eigenlijk al een heel klein gevechtssysteem.

De speler wordt steeds aangevallen totdat zijn gezondheid op is.

---

## 8. `for`: iets een bepaald aantal keer doen

Vaak weten we vooraf hoeveel keer iets moet gebeuren.

Dan is `for` erg handig.

Bijvoorbeeld:

```rust,ignore
for getal in 1..=5 {
    println!("{getal}");
}
```

Dit geeft:

```text
1
2
3
4
5
```

Hier betekent:

```rust,ignore
1..=5
```

alle getallen van 1 tot en **met** 5.

De `=` achter de twee puntjes is belangrijk.

### `1..5`

```text
1
2
3
4
```

### `1..=5`

```text
1
2
3
4
5
```

Je kunt dit onthouden als:

> `..` stopt vóór het einde.  
> `..=` neemt het einde mee.

---

## 9. Wat betekent `getal in`?

Kijk nog eens naar:

```rust,ignore
for getal in 1..=5 {
    println!("{getal}");
}
```

De variabele `getal` krijgt bij iedere herhaling een nieuwe waarde.

Eerst:

```text
getal = 1
```

Dan:

```text
getal = 2
```

enzovoort.

Je kunt de naam zelf kiezen:

```rust,ignore
for ronde in 1..=5 {
    println!("Ronde {ronde}");
}
```

Hier is `ronde` waarschijnlijk duidelijker dan `getal`.

---

## 10. Een `for`-lus in een game

Stel dat een speler vijf pijlen krijgt:

```rust,ignore
for pijl in 1..=5 {
    println!("Pijl {pijl}!");
}
```

Of we laten een vijand vijf keer schade oplopen:

```rust,ignore
let mut gezondheid = 100;

for aanval in 1..=5 {
    gezondheid -= 10;

    println!("Aanval {aanval}: HP = {gezondheid}");
}
```

Uitvoer:

```text
Aanval 1: HP = 90
Aanval 2: HP = 80
Aanval 3: HP = 70
Aanval 4: HP = 60
Aanval 5: HP = 50
```

---

## 11. Een variabele die we niet gebruiken

Soms willen we iets een bepaald aantal keer doen, maar hebben we het nummer helemaal niet nodig.

Bijvoorbeeld:

```rust,ignore
for _ in 0..3 {
    println!("Kritieke hit!");
}
```

De `_` betekent hier:

> Ik heb deze waarde niet nodig.

We gebruiken `_` vaak bij lussen waarvan we alleen het aantal herhalingen belangrijk vinden.

---

## 12. Lussen en `if`

Lussen worden vaak gecombineerd met beslissingen.

Bijvoorbeeld:

```rust,ignore
for ronde in 1..=10 {
    if ronde == 5 {
        println!("De eindbaas verschijnt!");
    } else {
        println!("Ronde {ronde}");
    }
}
```

Nu kunnen we tijdens een lus beslissingen nemen.

Dat is een belangrijk patroon in games:

```text
herhaal iets
    |
    +-- controleer een voorwaarde
    |
    +-- doe iets
```

---

## 13. Een vijand aanvallen

We kunnen de kennis uit de vorige lessen combineren.

```rust,ignore
let mut vijand_hp = 50;

for aanval in 1..=5 {
    let schade = 12;

    vijand_hp -= schade;

    println!("Aanval {aanval}: vijand heeft {vijand_hp} HP.");

    if vijand_hp <= 0 {
        println!("De vijand is verslagen!");
        break;
    }
}
```

Hier gebruiken we:

- variabelen
- `mut`
- een `for`-lus
- een functie van het type `if`
- een vergelijking
- `break`.

Let op dat we niet zomaar vijf aanvallen uitvoeren.

Zodra de vijand dood is, stoppen we:

```rust,ignore
break;
```

---

## 14. Een countdown

Een klassieke toepassing van een lus is een countdown.

```rust,ignore
for seconde in (1..=5).rev() {
    println!("{seconde}...");
}

println!("START!");
```

Dit geeft:

```text
5...
4...
3...
2...
1...
START!
```

`.rev()` zorgt ervoor dat de waarden achterstevoren worden doorlopen.

Je hoeft nog niet precies te weten hoe `.rev()` intern werkt. Voor nu is het voldoende om te weten:

> `.rev()` draait een bereik om.

Dit is een eerste klein voorbeeld van iets dat we later uitgebreider zullen tegenkomen.

---

## 15. Welke lus gebruik je wanneer?

Er zijn drie belangrijke vormen die we nu kennen.

### `loop`

Gebruik dit wanneer je wilt blijven herhalen totdat je zelf `break` gebruikt.

```rust,ignore
loop {
    // ...
    if klaar {
        break;
    }
}
```

### `while`

Gebruik dit wanneer je wilt herhalen **zolang een voorwaarde waar is**.

```rust,ignore
while gezondheid > 0 {
    // ...
}
```

### `for`

Gebruik dit wanneer je bijvoorbeeld een bereik wilt doorlopen.

```rust,ignore
for ronde in 1..=10 {
    // ...
}
```

Een eenvoudige vuistregel:

| Situatie | Gebruik |
| --- | --- |
| Blijf doorgaan totdat iets gebeurt | `loop` |
| Zolang een voorwaarde waar is | `while` |
| Een aantal waarden doorlopen | `for` |

Er zijn situaties waarin meerdere keuzes mogelijk zijn. Het gaat dus niet om een harde regel.

---

## 16. Veelgemaakte fouten

## Fout 1 — vergeten de teller te veranderen

```rust,ignore
let mut teller = 0;

while teller < 5 {
    println!("{teller}");
}
```

Dit stopt nooit.

We moeten de teller veranderen:

```rust,ignore
let mut teller = 0;

while teller < 5 {
    println!("{teller}");
    teller += 1;
}
```

---

## Fout 2 — `..` en `..=` door elkaar halen

```rust,ignore
for nummer in 1..5 {
    println!("{nummer}");
}
```

Dit geeft 1 tot en met 4.

Wil je 5 ook hebben?

```rust,ignore
for nummer in 1..=5 {
    println!("{nummer}");
}
```

---

## Fout 3 — vergeten `mut` te gebruiken

Dit werkt niet:

```rust,ignore
let gezondheid = 100;

while gezondheid > 0 {
    gezondheid -= 10;
}
```

`gezondheid` mag niet veranderen.

We hebben nodig:

```rust,ignore
let mut gezondheid = 100;
```

---

## Fout 4 — een `loop` zonder `break`

```rust,ignore
loop {
    println!("Nog een keer!");
}
```

Dit blijft doorgaan.

Dat kan soms precies de bedoeling zijn, maar meestal vergeten beginners gewoon de stopvoorwaarde.

---

## 17. Zelf proberen

Probeer eerst zelf deze kleine opdrachten te maken.

### Opdracht 1 — Tel tot vijf

Maak een `for`-lus die de getallen 1 tot en met 5 print.

### Opdracht 2 — Countdown

Print:

```text
5
4
3
2
1
START!
```

### Opdracht 3 — Gezondheid

Begin met:

```rust,ignore
let mut gezondheid = 100;
```

Laat de gezondheid steeds met 20 afnemen totdat de speler dood is.

### Opdracht 4 — Rondes

Maak een programma dat tien rondes print:

```text
Ronde 1
Ronde 2
...
Ronde 10
```

### Opdracht 5 — Eindbaas

Een eindbaas begint met 50 HP.

Elke aanval doet 15 schade.

Gebruik een lus en stop zodra de eindbaas verslagen is.

---

## 18. Rustlings-opgaven

Net als bij de vorige lessen gebruiken we meerdere kleine oefeningen. De bedoeling is dat je niet alleen leest, maar daadwerkelijk programmeert.

Een mogelijke map:

```text
exercises/
└── lussen/
    ├── 01_loop.rs
    ├── 02_loop_break.rs
    ├── 03_teller.rs
    ├── 04_while.rs
    ├── 05_health.rs
    ├── 06_for.rs
    ├── 07_range.rs
    ├── 08_range_inclusive.rs
    ├── 09_countdown.rs
    ├── 10_underscore.rs
    ├── 11_if_in_loop.rs
    ├── 12_break.rs
    ├── 13_game_rounds.rs
    ├── 14_enemy.rs
    ├── 15_boss_fight.rs
    ├── 16_debug_while.rs
    ├── 17_debug_range.rs
    └── 18_debug_loop.rs
```

## ⭐ Niveau 1 — rustig beginnen

### 01 — `loop`

Maak een programma dat één keer:

```text
Hallo!
```

print met behulp van `loop`.

De lus moet daarna stoppen.

---

### 02 — `loop` en `break`

Maak een lus die:

```text
Start!
```

print en daarna stopt.

---

### 03 — Teller

Gebruik een `while`-lus om te tellen van 1 tot en met 5.

---

### 04 — `while`

Begin met:

```rust,ignore
let mut gezondheid = 50;
```

Verminder de gezondheid steeds met 10.

Stop wanneer de gezondheid 0 is.

---

### 05 — Gezondheid

Print bij iedere stap de gezondheid:

```text
HP: 100
HP: 80
HP: 60
HP: 40
HP: 20
```

---

### 06 — `for`

Print de getallen 1 tot en met 10 met een `for`-lus.

---

### 07 — Bereik

Maak een lus die precies deze waarden print:

```text
3
4
5
6
7
```

---

### 08 — Inclusief

Maak een lus die de waarden 1 tot en met 5 print.

Gebruik `..=`.

---

### 09 — Countdown

Maak:

```text
5...
4...
3...
2...
1...
START!
```

---

### 10 — `_`

Print vijf keer:

```text
BAM!
```

Gebruik:

```rust,ignore
for _ in ... {
```

Je hoeft het nummer van de herhaling niet te gebruiken.

---

## ⭐⭐ Niveau 2 — lussen combineren

### 11 — `if` in een lus

Print de getallen 1 tot en met 10.

Print bij getallen groter dan 5:

```text
Groot!
```

en bij de andere getallen:

```text
Klein!
```

---

### 12 — `break`

Maak een lus die getallen blijft tellen.

Stop zodra de teller 7 bereikt.

---

### 13 — Game-rondes

Maak tien spelrondes:

```text
Ronde 1
Ronde 2
...
Ronde 10
```

Bij ronde 5 moet daarnaast verschijnen:

```text
Een nieuwe vijand verschijnt!
```

---

### 14 — Vijand

Een vijand heeft 60 HP.

Elke aanval doet 15 schade.

Print na iedere aanval de resterende HP.

Stop zodra de vijand dood is.

---

### 15 — Eindbaas

Maak een eindbaasgevecht.

De eindbaas begint met 100 HP.

Elke aanval doet 20 schade.

Print bijvoorbeeld:

```text
Aanval 1: 80 HP
Aanval 2: 60 HP
...
```

Wanneer de eindbaas dood is:

```text
De eindbaas is verslagen!
```

---

## ⭐⭐⭐ Niveau 3 — nadenken en combineren

### 16 — Debuggen: oneindige lus

In dit programma zit een fout:

```rust,ignore
let mut teller = 0;

while teller < 5 {
    println!("{teller}");
}
```

Waarom stopt het programma niet?

Maak het programma werkend.

---

### 17 — Debuggen: bereik

In dit programma:

```rust,ignore
for nummer in 1..5 {
    println!("{nummer}");
}
```

moeten de getallen 1 tot en met 5 worden geprint.

Pas het programma aan.

---

### 18 — Debuggen: game over

In dit programma zit een fout:

```rust,ignore
let mut gezondheid = 100;

while gezondheid > 0 {
    println!("HP: {gezondheid}");

    if gezondheid <= 0 {
        println!("Game over!");
        break;
    }
}
```

Het programma moet gezondheid steeds met 25 verminderen en uiteindelijk stoppen.

Zoek uit wat er ontbreekt.

---

## 19. Eindopdracht — Het eerste echte gevecht

Maak een klein gevechtssysteem.

De speler begint met:

```text
100 HP
```

De vijand begint met:

```text
80 HP
```

Elke ronde:

1. doet de speler 20 schade
2. wordt de nieuwe vijandelijke HP getoond
3. controleer je of de vijand dood is
4. als de vijand nog leeft, doet de vijand 10 schade
5. wordt de HP van de speler getoond
6. gaat de volgende ronde beginnen.

Het gevecht stopt zodra één van de twee dood is.

Probeer het programma zo te maken dat de uitvoer bijvoorbeeld lijkt op:

```text
--- Ronde 1 ---
De speler valt aan!
Vijand HP: 60
De vijand valt aan!
Speler HP: 90

--- Ronde 2 ---
De speler valt aan!
Vijand HP: 40
De vijand valt aan!
Speler HP: 80

...
```

Gebruik hierbij alleen dingen die we tot nu toe hebben geleerd:

- variabelen
- `mut`
- functies
- `if` / `else`
- `for` of `while`
- `break`
- `println!`.

Je hebt dus nog **geen structs, ownership of borrowing** nodig.

---

## 20. Wat moet je nu kunnen?

Na deze les zou je moeten begrijpen:

- wat een lus is
- waarvoor `loop` wordt gebruikt
- hoe `break` een lus stopt
- hoe `while` werkt
- hoe `for` werkt
- wat `1..5` betekent
- wat `1..=5` betekent
- waarvoor `_` bij een `for`-lus kan worden gebruikt
- hoe je `if` binnen een lus gebruikt
- hoe je een lus gebruikt voor een gamegevecht
- hoe je een oneindige lus kunt herkennen.

### Checklist

Je kunt nu:

- [ ] een `loop` schrijven
- [ ] een `loop` stoppen met `break`
- [ ] een `while`-lus schrijven
- [ ] een `for`-lus schrijven
- [ ] een bereik gebruiken
- [ ] een countdown maken
- [ ] een lus combineren met `if`
- [ ] een eenvoudig gevecht programmeren.

---

## Vooruitblik

We hebben nu een belangrijke stap gezet: onze programma's kunnen **beslissingen nemen én dingen herhalen**.

Daarna komt een onderwerp dat specifiek voor Rust erg belangrijk is:

**ownership**.

Dat verklaart onder andere waarom Rust soms zegt dat je een variabele niet meer mag gebruiken nadat je hem aan iets anders hebt gegeven. Dat lijkt in eerste instantie vreemd, maar het vormt de basis voor hoe Rust met geheugen omgaat.

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 7](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_7/).

