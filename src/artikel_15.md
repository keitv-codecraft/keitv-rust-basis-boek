# Rust 15 — Arrays en Vecs: werken met groepen waarden

Tot nu toe hebben we meestal met één waarde tegelijk gewerkt:

```rust,ignore
let gezondheid = 100;
let score = 500;
let naam = String::from("Arin");
```

Maar in een spel hebben we vaak meerdere dingen van hetzelfde soort:

- meerdere vijanden
- meerdere spelers
- meerdere wapens
- meerdere scores
- meerdere items.

Daarvoor kunnen we een **verzameling waarden** gebruiken.

Rust heeft hiervoor onder andere **arrays** en **Vecs**.

---

## 1. Een array

Een array bevat meerdere waarden van hetzelfde type.

```rust,ignore
let scores = [100, 250, 500, 1000];
```

Deze array bevat vier getallen.

We kunnen de waarden bekijken:

```rust,ignore
println!("{}", scores[0]);
println!("{}", scores[1]);
println!("{}", scores[2]);
println!("{}", scores[3]);
```

De uitvoer is:

```text
100
250
500
1000
```

## Let op: tellen begint bij 0

Het eerste element is:

```rust,ignore
scores[0]
```

Het tweede:

```rust,ignore
scores[1]
```

Het derde:

```rust,ignore
scores[2]
```

enzovoort.

Bij vier elementen loopt de index dus van `0` tot en met `3`.

---

## 2. Een array heeft een vaste grootte

Bij het maken van een array bepaalt Rust hoeveel elementen erin zitten.

```rust,ignore
let scores = [100, 250, 500, 1000];
```

Deze array heeft precies vier elementen.

Je kunt er later niet zomaar een vijfde element aan toevoegen.

Dat maakt arrays handig wanneer je vooraf weet hoeveel waarden je hebt.

Bijvoorbeeld:

```rust,ignore
let dagen = ["ma", "di", "wo", "do", "vr", "za", "zo"];
```

Een week heeft altijd zeven dagen.

---

## 3. Het type van een array

We kunnen het type expliciet schrijven:

```rust,ignore
let scores: [i32; 4] = [100, 250, 500, 1000];
```

Hier betekent:

```text
[i32; 4]
```

> een array van vier `i32`-waarden.

De twee onderdelen zijn:

```text
[i32; 4]
  │     │
  │     └── aantal elementen
  └──────── type van ieder element
```

---

## 4. Alle elementen hetzelfde

Een array bevat waarden van hetzelfde type.

Dit werkt:

```rust,ignore
let levens = [100, 80, 50, 25];
```

Dit werkt niet:

```rust,ignore
let waarden = [100, "hallo", true];
```

De elementen hebben hier verschillende types.

Voorlopig houden we het eenvoudig:

> Een array is een groep waarden van hetzelfde type.

---

## 5. Een array doorlopen met een for-lus

We kunnen een array combineren met onze `for`-lus.

```rust,ignore
let scores = [100, 250, 500, 1000];

for score in scores {
    println!("Score: {score}");
}
```

Uitvoer:

```text
Score: 100
Score: 250
Score: 500
Score: 1000
```

Dit is veel handiger dan vier keer zelf `println!` schrijven.

---

## 6. Een groep vijanden

Dit wordt interessant voor ons RPG-spel.

We kunnen bijvoorbeeld een aantal vijanden opslaan:

```rust,ignore
let vijanden = ["Goblin", "Ork", "Trol"];
```

En ze allemaal bekijken:

```rust,ignore
for vijand in vijanden {
    println!("Er staat een {vijand}!");
}
```

Uitvoer:

```text
Er staat een Goblin!
Er staat een Ork!
Er staat een Trol!
```

We kunnen dus een `for`-lus gebruiken om een hele groep te verwerken.

---

## 7. Een array wijzigen

Een array kan `mut` zijn.

```rust,ignore
let mut gezondheid = [100, 80, 50];

gezondheid[1] = 60;
```

Nu zijn de waarden:

```text
100
60
50
```

We hebben dus de gezondheid van de tweede vijand veranderd.

---

## 8. Alle vijanden schade geven

We kunnen dit combineren met een `for`-lus:

```rust,ignore
let mut gezondheid = [100, 80, 50];

for waarde in &mut gezondheid {
    *waarde -= 20;
}
```

Daarna zijn de waarden:

```text
80
60
30
```

Hier zien we iets nieuws:

```rust,ignore
&mut gezondheid
```

en:

```rust,ignore
*waarde
```

Dit zijn onderdelen van borrowing die we eerder al hebben gezien.

De lus krijgt een **mutable reference** naar ieder element.

Daardoor kunnen we het element aanpassen.

---

## 9. Waarom hebben we Vec nodig?

Een array heeft een vaste grootte.

Maar in een RPG weten we vaak niet hoeveel vijanden er zijn.

Misschien zijn er:

```text
3 vijanden
```

maar later:

```text
7 vijanden
```

of:

```text
1 vijand
```

We willen dus een verzameling die kan groeien en kleiner worden.

Daarvoor gebruiken we een `Vec`.

---

## 10. Een Vec maken

Een `Vec` is een verzameling waarvan het aantal elementen kan veranderen.

We kunnen bijvoorbeeld schrijven:

```rust,ignore
let vijanden = vec![
    "Goblin",
    "Ork",
    "Trol",
];
```

We kunnen hem vervolgens net als een array uitlezen:

```rust,ignore
println!("{}", vijanden[0]);
println!("{}", vijanden[1]);
println!("{}", vijanden[2]);
```

---

## 11. Een Vec uitbreiden

Een `Vec` kan groter worden.

Daarvoor gebruiken we `push`.

```rust,ignore
let mut vijanden = vec![
    "Goblin",
    "Ork",
];

vijanden.push("Trol");
```

Nu bevat de `Vec` drie vijanden.

Nog een vijand toevoegen kan ook:

```rust,ignore
vijanden.push("Draak");
```

Nu hebben we:

```text
Goblin
Ork
Trol
Draak
```

Omdat we de `Vec` veranderen, moet hij `mut` zijn:

```rust,ignore
let mut vijanden = ...
```

---

## 12. Hoeveel elementen zitten erin?

Met `.len()` kunnen we het aantal elementen opvragen.

```rust,ignore
let vijanden = vec![
    "Goblin",
    "Ork",
    "Trol",
];

println!("Aantal vijanden: {}", vijanden.len());
```

Uitvoer:

```text
Aantal vijanden: 3
```

---

## 13. Een Vec doorlopen

Net als bij een array kunnen we een `for`-lus gebruiken:

```rust,ignore
let vijanden = vec![
    "Goblin",
    "Ork",
    "Trol",
];

for vijand in &vijanden {
    println!("Vijand: {vijand}");
}
```

We gebruiken hier:

```rust,ignore
&vijanden
```

om de waarden te lenen.

Dat betekent dat we de `Vec` zelf behouden.

---

## 14. Een Vec wijzigen

We kunnen ook ieder element wijzigen.

```rust,ignore
let mut gezondheid = vec![100, 80, 50];

for waarde in &mut gezondheid {
    *waarde -= 20;
}
```

Daarna:

```text
80
60
30
```

Dit patroon is bijzonder nuttig:

```rust,ignore
for waarde in &mut waarden {
    // verander waarde
}
```

We lenen ieder element tijdelijk om het te kunnen aanpassen.

---

## 15. Vec met structs

Een `Vec` wordt pas echt interessant wanneer we hem combineren met onze structs.

We hebben bijvoorbeeld:

```rust,ignore
struct Vijand {
    naam: String,
    gezondheid: i32,
}
```

We kunnen nu een groep vijanden maken:

```rust,ignore
let vijanden = vec![
    Vijand {
        naam: String::from("Goblin"),
        gezondheid: 30,
    },
    Vijand {
        naam: String::from("Ork"),
        gezondheid: 50,
    },
    Vijand {
        naam: String::from("Trol"),
        gezondheid: 100,
    },
];
```

Dit betekent:

```text
Vec<Vijand>
```

oftewel:

> een Vec met `Vijand`-waarden.

---

## 16. Alle vijanden bekijken

We kunnen ze met een `for`-lus bekijken:

```rust,ignore
for vijand in &vijanden {
    println!(
        "{} heeft {} gezondheid.",
        vijand.naam,
        vijand.gezondheid
    );
}
```

Uitvoer:

```text
Goblin heeft 30 gezondheid.
Ork heeft 50 gezondheid.
Trol heeft 100 gezondheid.
```

---

## 17. Alle vijanden schade geven

Nu kunnen we een echte AoE-aanval maken.

AoE staat voor **Area of Effect**: een aanval die meerdere doelen tegelijk raakt.

```rust,ignore
let mut vijanden = vec![
    Vijand {
        naam: String::from("Goblin"),
        gezondheid: 30,
    },
    Vijand {
        naam: String::from("Ork"),
        gezondheid: 50,
    },
    Vijand {
        naam: String::from("Trol"),
        gezondheid: 100,
    },
];

for vijand in &mut vijanden {
    vijand.gezondheid -= 20;
}
```

Iedere vijand krijgt 20 schade.

Daarna:

```text
Goblin → 10
Ork    → 30
Trol   → 80
```

Dit is een mooi voorbeeld van hoe verschillende concepten samenkomen:

- `struct`
- `Vec`
- `for`
- `&mut`
- velden aanpassen

---

## 18. Een Vec kan groeien tijdens het spel

Stel dat er een nieuwe vijand verschijnt:

```rust,ignore
vijanden.push(Vijand {
    naam: String::from("Skelet"),
    gezondheid: 40,
});
```

De groep bevat nu vier vijanden.

We kunnen vervolgens opnieuw door de hele groep lopen.

---

## 19. De eerste vijand bekijken

Net als bij arrays kunnen we een element met een index bekijken:

```rust,ignore
println!("{}", vijanden[0].naam);
```

Hier gebeurt eigenlijk twee keer iets:

```text
vijanden[0]
```

geeft de eerste vijand.

Daarna:

```text
.naam
```

geeft de naam van die vijand.

---

## 20. Voorzichtig met indexen

Deze code:

```rust,ignore
vijanden[0]
```

werkt alleen als er minstens één element is.

Als de `Vec` leeg is:

```rust,ignore
let vijanden: Vec<i32> = vec![];
```

dan is:

```rust,ignore
vijanden[0]
```

ongeldig.

Rust voorkomt dat we zomaar buiten de verzameling lezen en geeft in zo'n situatie een fout tijdens het uitvoeren van het programma.

Voorlopig is het belangrijkste:

> Een index moet naar een bestaand element wijzen.

---

## 21. Een lege Vec

We kunnen ook met een lege `Vec` beginnen.

```rust,ignore
let mut vijanden: Vec<Vijand> = Vec::new();
```

Daarna kunnen we vijanden toevoegen:

```rust,ignore
vijanden.push(Vijand {
    naam: String::from("Goblin"),
    gezondheid: 30,
});
```

De `Vec` begint leeg en groeit tijdens het spel.

---

## 22. Een handig patroon

Dit patroon zul je vaak gebruiken:

```rust,ignore
let mut vijanden = Vec::new();

vijanden.push(...);
vijanden.push(...);
vijanden.push(...);
```

Of wanneer we de beginwaarden al kennen:

```rust,ignore
let mut vijanden = vec![
    ...,
    ...,
    ...,
];
```

Beide manieren maken een `Vec`.

---

## 23. Array of Vec?

Voorlopig kun je deze vuistregel gebruiken:

### Array

Gebruik een array wanneer het aantal elementen vaststaat.

```rust,ignore
let dagen = [
    "ma",
    "di",
    "wo",
    "do",
    "vr",
    "za",
    "zo",
];
```

Er zijn altijd zeven dagen.

### Vec

Gebruik een `Vec` wanneer het aantal elementen kan veranderen.

```rust,ignore
let mut vijanden = vec![
    goblin,
    ork,
];
```

Tijdens het spel kunnen er vijanden bijkomen.

Voor ons RPG-project zullen we daarom **veel vaker `Vec` gebruiken dan arrays**.

---

## 24. Oefeningen

## Oefening 1 — Getallen

Maak een array met vijf getallen.

Print alle getallen met een `for`-lus.

## Oefening 2 — Namen

Maak een array met vijf namen.

Print iedere naam.

## Oefening 3 — Scores

Maak een array met vijf scores.

Bereken hoeveel elementen de array bevat.

## Oefening 4 — Gezondheid

Maak een mutable array:

```text
[100, 80, 60, 40]
```

Verminder iedere waarde met 10.

## Oefening 5 — Eerste Vec

Maak een `Vec` met drie namen.

Voeg daarna met `push` een vierde naam toe.

## Oefening 6 — Aantal vijanden

Maak een `Vec` met vier vijandenamen.

Print het aantal vijanden met `.len()`.

## Oefening 7 — Nieuwe vijand

Begin met twee vijanden.

Voeg daarna drie nieuwe vijanden toe met `push`.

## Oefening 8 — Vijanden

Maak een `struct Vijand` met:

```text
naam
gezondheid
```

Maak vervolgens een `Vec<Vijand>` met drie vijanden.

Print hun namen en gezondheid.

## Oefening 9 — AoE

Geef alle vijanden 10 schade met een `for`-lus.

## Oefening 10 — Genezing

Geef alle vijanden per ongeluk 20 gezondheid terug.

Gebruik daarna dezelfde techniek om dit weer te veranderen in schade.

## Oefening 11 — Nieuwe vijand

Maak een groep vijanden.

Voeg tijdens het programma een nieuwe vijand toe.

Print daarna alle vijanden.

## Oefening 12 — Eindopdracht

Maak een kleine groep van vijf vijanden.

Voer een AoE-aanval uit die iedere vijand 25 schade geeft.

Print daarna de gezondheid van iedere vijand.

---

## 25. Rustlings

Maak:

```text
exercises/vecs/
```

met bijvoorbeeld:

```text
01_array.rs
02_array_index.rs
03_array_for.rs
04_array_mut.rs
05_vec.rs
06_vec_push.rs
07_vec_len.rs
08_vec_for.rs
09_vec_mut.rs
10_vec_empty.rs
11_enemies.rs
12_enemy_damage.rs
13_enemy_heal.rs
14_spawn_enemy.rs
15_aoe.rs
16_game_enemies.rs
17_debug_vec.rs
18_game_battle.rs
```

Bij `17_debug_vec.rs` kan bijvoorbeeld bewust deze fout staan:

```rust,ignore
let mut vijanden = vec![
    "Goblin",
    "Ork",
];

vijanden.push(42);
```

Laat de leerlingen de compiler uitleggen waarom dit niet kan.

Het doel is niet om foutmeldingen te vermijden.

Het doel is leren denken:

> **Wat probeert de compiler mij hier te vertellen?**

---

## 26. Eindopdracht — Het slagveld

Maak een programma met:

```rust,ignore
struct Vijand {
    naam: String,
    gezondheid: i32,
}
```

Maak een `Vec<Vijand>` met minimaal vijf vijanden.

Voer daarna een AoE-aanval uit:

```text
Vuurstorm!
```

Iedere vijand krijgt 25 schade.

Print vervolgens het resultaat:

```text
Vuurstorm!

Goblin heeft nog 5 gezondheid.
Ork heeft nog 25 gezondheid.
Skelet heeft nog 0 gezondheid.
Trol heeft nog 75 gezondheid.
Demon heeft nog 50 gezondheid.
```

Probeer daarna zelf een tweede aanval toe te voegen.

Bijvoorbeeld een genezende spreuk die alle vijanden geneest.

Dat is expres een beetje vreemd: de programmeertechniek is belangrijker dan de spelregels. Later kunnen we natuurlijk voorkomen dat een dode vijand opnieuw wordt genezen.

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 15](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_15/).

