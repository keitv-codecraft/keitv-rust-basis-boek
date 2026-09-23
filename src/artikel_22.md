# Rust 22 — Iterators en iterator-adaptors

Tot nu toe hebben we met `Vec` al lijsten met waarden gemaakt en met `for` door die lijsten gelopen. We hebben ook closures geleerd, waarmee we kleine stukjes gedrag als waarde kunnen gebruiken.

Rust heeft hier een krachtig systeem voor: **iterators**.

Een iterator geeft ons één voor één waarden uit een verzameling. Vervolgens kunnen we die waarden bijvoorbeeld:

- bekijken
- aanpassen
- selecteren
- omzetten naar andere waarden
- zoeken
- optellen
- verzamelen in een nieuwe `Vec`.

De functies waarmee we een iterator bewerken, zoals `map()` en `filter()`, noemen we **iterator-adaptors**.

We gaan dit vooral gebruiken voor onze RPG.

---

## 1. Van `for` naar een iterator

Stel dat we drie vijanden hebben:

```rust,ignore
let vijanden = vec![
    "Goblin",
    "Ork",
    "Draak",
];
```

We kunnen ze met een `for`-lus bekijken:

```rust,ignore
for vijand in &vijanden {
    println!("{vijand}");
}
```

Dat werkt prima.

Maar we kunnen ook expliciet een iterator maken:

```rust,ignore
let iterator = vijanden.iter();

for vijand in iterator {
    println!("{vijand}");
}
```

`iter()` maakt een iterator die de waarden **leest**.

Belangrijk:

```rust,ignore
vijanden.iter()
```

verandert de `Vec` niet.

De waarden worden geleend.

---

## 2. `iter()` geeft referenties

Dit is belangrijk vanwege ownership en borrowing.

Bij:

```rust,ignore
let namen = vec![
    String::from("Aria"),
    String::from("Borin"),
    String::from("Cedric"),
];

for naam in namen.iter() {
    println!("{naam}");
}
```

blijven de `String`-waarden eigendom van `namen`.

De iterator geeft ons referenties naar de waarden.

Je kunt het conceptueel zien als:

```text
Vec<String>
   |
   +-- String
   +-- String
   +-- String
        ^
        |
   iterator geeft
   referenties
```

Daarom kunnen we na de lus `namen` nog steeds gebruiken:

```rust,ignore
for naam in namen.iter() {
    println!("{naam}");
}

println!("Aantal namen: {}", namen.len());
```

---

## 3. `next()`

Een iterator kan ook handmatig één waarde tegelijk worden uitgelezen.

```rust,ignore
let namen = vec![
    "Aria",
    "Borin",
    "Cedric",
];

let mut iterator = namen.iter();

println!("{:?}", iterator.next());
println!("{:?}", iterator.next());
println!("{:?}", iterator.next());
println!("{:?}", iterator.next());
```

De uitvoer is ongeveer:

```text
Some("Aria")
Some("Borin")
Some("Cedric")
None
```

Hier zien we meteen iets bekends uit het vorige hoofdstuk:

**`next()` geeft een `Option`.**

Zolang er een volgende waarde is:

```rust,ignore
Some(waarde)
```

Als er niets meer is:

```rust,ignore
None
```

We kunnen dit dus met `match` behandelen:

```rust,ignore
match iterator.next() {
    Some(naam) => println!("Volgende naam: {naam}"),
    None => println!("Geen namen meer."),
}
```

In de meeste dagelijkse Rust-code gebruiken we echter gewoon `for`, `map`, `filter` enzovoort.

---

## 4. `map()` — iedere waarde omzetten

Een van de belangrijkste iterator-adaptors is `map()`.

Stel:

```rust,ignore
let getallen = vec![1, 2, 3, 4];
```

We willen van ieder getal het dubbele maken.

Dat kan met een `for`-lus:

```rust,ignore
let mut verdubbeld = Vec::new();

for getal in &getallen {
    verdubbeld.push(getal * 2);
}
```

Maar met `map()` kunnen we hetzelfde korter schrijven:

```rust,ignore
let verdubbeld: Vec<i32> = getallen
    .iter()
    .map(|getal| getal * 2)
    .collect();
```

Hier gebeurt stap voor stap:

```text
getallen
   ↓
iter()
   ↓
1   2   3   4
   ↓
map()
   ↓
2   4   6   8
   ↓
collect()
   ↓
Vec<i32>
```

De closure:

```rust,ignore
|getal| getal * 2
```

wordt voor iedere waarde uitgevoerd.

---

## 5. `map()` met strings

Iterators worden erg handig wanneer we informatie uit structs willen halen.

Stel:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
}
```

En:

```rust,ignore
let spelers = vec![
    Speler {
        naam: String::from("Aria"),
        gezondheid: 100,
    },
    Speler {
        naam: String::from("Borin"),
        gezondheid: 80,
    },
    Speler {
        naam: String::from("Cedric"),
        gezondheid: 120,
    },
];
```

We willen alleen de namen hebben.

Dan kunnen we schrijven:

```rust,ignore
let namen: Vec<String> = spelers
    .iter()
    .map(|speler| speler.naam.clone())
    .collect();
```

Waarom `.clone()`?

Omdat `iter()` de spelers leent.

De `String` in `speler.naam` is eigendom van de oorspronkelijke `Speler`. We willen een nieuwe `Vec<String>` maken die zelf eigenaar is van de namen.

Dit is dus een situatie waarin `clone()` nuttig kan zijn.

---

## 6. `collect()`

`map()` maakt nog geen `Vec`.

Het maakt een nieuwe reeks waarden binnen de iterator.

Met:

```rust,ignore
collect()
```

zeggen we:

> Verzamel de resultaten in een verzameling.

Bijvoorbeeld:

```rust,ignore
let getallen = vec![1, 2, 3];

let verdubbeld: Vec<i32> = getallen
    .iter()
    .map(|getal| getal * 2)
    .collect();
```

De type-aanduiding:

```rust,ignore
Vec<i32>
```

vertelt Rust wat `collect()` moet maken.

Dat maakt deze syntax in het begin vaak duidelijker dan ingewikkelde type-annotaties bij `collect()` zelf.

---

## 7. `filter()` — alleen bepaalde waarden behouden

Met `map()` veranderen we iedere waarde.

Met `filter()` bepalen we welke waarden we willen behouden.

Stel:

```rust,ignore
let getallen = vec![1, 2, 3, 4, 5, 6];
```

We willen alleen even getallen:

```rust,ignore
let even: Vec<&i32> = getallen
    .iter()
    .filter(|getal| **getal % 2 == 0)
    .collect();
```

Hier kan de dubbele `*` vreemd lijken.

De iterator werkt hier met referenties, waardoor de closure uiteindelijk met een extra referentie te maken krijgt.

Voor structs ziet dit er gelukkig veel natuurlijker uit:

```rust,ignore
let levende: Vec<&Vijand> = vijanden
    .iter()
    .filter(|vijand| vijand.gezondheid > 0)
    .collect();
```

Rust kan hier automatisch met de referenties omgaan.

De betekenis is eenvoudig:

> Houd iedere vijand waarvoor de voorwaarde waar is.

---

## 8. `filter()` in onze RPG

Stel:

```rust,ignore
struct Vijand {
    naam: String,
    gezondheid: i32,
}
```

We hebben:

```rust,ignore
let vijanden = vec![
    Vijand {
        naam: String::from("Goblin"),
        gezondheid: 30,
    },
    Vijand {
        naam: String::from("Ork"),
        gezondheid: 0,
    },
    Vijand {
        naam: String::from("Draak"),
        gezondheid: 200,
    },
];
```

We willen alleen levende vijanden:

```rust,ignore
let levende: Vec<&Vijand> = vijanden
    .iter()
    .filter(|vijand| vijand.gezondheid > 0)
    .collect();
```

Daarna kunnen we:

```rust,ignore
for vijand in levende {
    println!("{}", vijand.naam);
}
```

De oorspronkelijke `vijanden` zijn niet veranderd.

---

## 9. `find()` — één waarde zoeken

Soms willen we niet alle passende waarden, maar alleen de eerste.

Daarvoor kunnen we `find()` gebruiken.

```rust,ignore
let getallen = vec![3, 7, 12, 15];

let gevonden = getallen
    .iter()
    .find(|getal| **getal > 10);
```

`gevonden` is een:

```text
Option<&i32>
```

Dus:

```rust,ignore
match gevonden {
    Some(getal) => println!("Gevonden: {getal}"),
    None => println!("Niets gevonden."),
}
```

Dit sluit direct aan op `Option<T>`.

---

## 10. Een item zoeken in een RPG

Stel:

```rust,ignore
struct Item {
    naam: String,
    waarde: i32,
}
```

We kunnen een item zoeken:

```rust,ignore
fn zoek_item<'a>(items: &'a [Item], naam: &str) -> Option<&'a Item> {
    items
        .iter()
        .find(|item| item.naam == naam)
}
```

De lifetime-notatie hierboven laten we voorlopig buiten beschouwing.

Voor ons is vooral belangrijk:

```rust,ignore
items.iter().find(...)
```

geeft:

```text
Some(&Item)
```

als het item gevonden wordt, of:

```text
None
```

als het niet bestaat.

Omdat we hier alleen willen zoeken en geen eigenaar van het item willen worden, is dat precies wat we nodig hebben.

---

## 11. `any()` — bestaat er minstens één?

Met `any()` kunnen we vragen of minstens één waarde aan een voorwaarde voldoet.

Bijvoorbeeld:

```rust,ignore
let gezondheid = vec![100, 80, 0, 50];

let iemand_levend = gezondheid
    .iter()
    .any(|waarde| *waarde > 0);
```

`iemand_levend` is een `bool`.

Het resultaat is:

```text
true
```

Dit is handig in een RPG.

Bijvoorbeeld:

```rust,ignore
let vijand_levend = vijanden
    .iter()
    .any(|vijand| vijand.gezondheid > 0);
```

Dan weten we of er nog minstens één levende vijand is.

---

## 12. `all()` — voldoen ze allemaal?

Het tegenovergestelde is `all()`.

```rust,ignore
let gezondheid = vec![100, 80, 50];

let iedereen_levend = gezondheid
    .iter()
    .all(|waarde| *waarde > 0);
```

Dit geeft:

```text
true
```

Als één waarde niet aan de voorwaarde voldoet, wordt het:

```text
false
```

In een RPG:

```rust,ignore
let iedereen_verslagen = vijanden
    .iter()
    .all(|vijand| vijand.gezondheid <= 0);
```

---

## 13. `sum()` — waarden optellen

Een iterator kan ook waarden optellen.

```rust,ignore
let schade = vec![10, 20, 15];

let totale_schade: i32 = schade
    .iter()
    .sum();
```

Het resultaat is:

```text
45
```

Dit kunnen we bijvoorbeeld gebruiken voor een aanval die meerdere vijanden raakt:

```rust,ignore
let schade_per_vijand = vec![10, 20, 15];

let totale_schade: i32 = schade_per_vijand
    .iter()
    .sum();

println!("Totale schade: {totale_schade}");
```

---

## 14. `iter_mut()` — waarden aanpassen

Tot nu toe gebruikten we:

```rust,ignore
iter()
```

Daarmee kunnen we waarden lezen.

Als we waarden willen aanpassen, gebruiken we:

```rust,ignore
iter_mut()
```

Bijvoorbeeld:

```rust,ignore
let mut gezondheid = vec![100, 80, 50];

for waarde in gezondheid.iter_mut() {
    *waarde -= 10;
}
```

Daarna is de lijst:

```text
90, 70, 40
```

De `*` is nodig omdat `waarde` een mutable referentie is.

Dit sluit aan op wat we eerder over `&mut` en dereferentie hebben geleerd.

---

## 15. Alle vijanden schade geven

Dat kunnen we in onze RPG gebruiken:

```rust,ignore
for vijand in vijanden.iter_mut() {
    vijand.gezondheid -= 10;
}
```

Alle vijanden krijgen 10 schade.

We kunnen ook een closure gebruiken:

```rust,ignore
let geef_schade = |vijand: &mut Vijand| {
    vijand.gezondheid -= 10;
};

for vijand in vijanden.iter_mut() {
    geef_schade(vijand);
}
```

Hier combineren we drie dingen die we al kennen:

- mutable borrowing
- `iter_mut()`
- closures.

---

## 16. `for` of iterator-adaptors?

We hoeven niet iedere `for`-lus te vervangen door iterator-code.

Een `for`-lus is vaak heel duidelijk:

```rust,ignore
for vijand in vijanden.iter() {
    println!("{}", vijand.naam);
}
```

Een iterator-adaptor is vooral handig wanneer we een reeks bewerkingen willen beschrijven.

Bijvoorbeeld:

```rust,ignore
let namen: Vec<String> = vijanden
    .iter()
    .filter(|vijand| vijand.gezondheid > 0)
    .map(|vijand| vijand.naam.clone())
    .collect();
```

Lees dit van boven naar beneden:

```text
vijanden
   ↓
iter()
   ↓
alle vijanden
   ↓
filter()
   ↓
alleen levende vijanden
   ↓
map()
   ↓
hun namen
   ↓
collect()
   ↓
Vec<String>
```

Dat is de kracht van iterator-adaptors.

---

## 17. Iterator-ketens

We kunnen meerdere adaptors achter elkaar zetten.

Bijvoorbeeld:

```rust,ignore
let namen: Vec<String> = vijanden
    .iter()
    .filter(|vijand| vijand.gezondheid > 0)
    .map(|vijand| vijand.naam.clone())
    .collect();
```

Iedere stap geeft een nieuwe iterator door aan de volgende stap.

We kunnen dit daarom zien als een soort lopende bewerking:

```text
iter()
  ↓
filter()
  ↓
map()
  ↓
collect()
```

Een belangrijke gewoonte is om zulke code eerst uit te schrijven en te begrijpen voordat je probeert hem zo kort mogelijk te maken.

---

## 18. Hetzelfde probleem met een `for`-lus

De vorige code kan ook met een gewone lus:

```rust,ignore
let mut namen = Vec::new();

for vijand in &vijanden {
    if vijand.gezondheid > 0 {
        namen.push(vijand.naam.clone());
    }
}
```

Dit is niet fout.

Het is juist nuttig om beide vormen te kunnen lezen.

De iterator-versie:

```rust,ignore
let namen: Vec<String> = vijanden
    .iter()
    .filter(|vijand| vijand.gezondheid > 0)
    .map(|vijand| vijand.naam.clone())
    .collect();
```

beschrijft vooral **wat** we willen:

> Geef mij de namen van alle levende vijanden.

De `for`-lus beschrijft meer **hoe** we dat stap voor stap doen.

Beide zijn waardevolle gereedschappen.

---

## 19. Een belangrijke ownership-waarschuwing

Naast:

```rust,ignore
iter()
```

bestaat ook:

```rust,ignore
into_iter()
```

Deze gaan we voorlopig alleen herkennen.

Het belangrijke verschil is dat `into_iter()` waarden kan **overnemen**.

Bijvoorbeeld:

```rust,ignore
let namen = vec![
    String::from("Aria"),
    String::from("Borin"),
];

for naam in namen.into_iter() {
    println!("{naam}");
}
```

Hier worden de `String`-waarden uit de `Vec` gehaald.

Daarna kunnen we `namen` niet meer gebruiken.

Dit is anders dan:

```rust,ignore
for naam in namen.iter() {
    println!("{naam}");
}
```

waarbij de `Vec` eigenaar blijft.

Voorlopig kun je daarom deze eenvoudige regel gebruiken:

> Gebruik `iter()` wanneer je een verzameling alleen wilt bekijken.
> Gebruik `iter_mut()` wanneer je de waarden wilt aanpassen.
>
> Gebruik `into_iter()` alleen wanneer je bewust het eigendom van de waarden wilt overnemen.

We zullen `into_iter()` en ownership bij iterators later uitgebreider behandelen.

---

## 20. Iterators en tests

Omdat we inmiddels automatische tests kennen, kunnen we iterator-code goed testen.

Bijvoorbeeld:

```rust,ignore
fn levende_vijanden(vijanden: &[Vijand]) -> Vec<&Vijand> {
    vijanden
        .iter()
        .filter(|vijand| vijand.gezondheid > 0)
        .collect()
}
```

Daar kunnen we een test voor schrijven:

```rust,ignore
#[test]
fn vindt_levende_vijanden() {
    let vijanden = vec![
        Vijand {
            naam: String::from("Goblin"),
            gezondheid: 20,
        },
        Vijand {
            naam: String::from("Ork"),
            gezondheid: 0,
        },
    ];

    let levende = levende_vijanden(&vijanden);

    assert_eq!(levende.len(), 1);
    assert_eq!(levende[0].naam, "Goblin");
}
```

Hier testen we niet hoe de iterator intern werkt.

We testen het gedrag van onze functie.

Dat is precies waar automatische tests voor bedoeld zijn.

---

## 21. Veelgebruikte iterator-adaptors

Je hoeft ze nog niet allemaal uit je hoofd te kennen.

Deze zijn voor onze RPG voorlopig het belangrijkst:

| Methode | Betekenis |
| --- | --- |
| `iter()` | waarden lezen |
| `iter_mut()` | waarden aanpassen |
| `next()` | volgende waarde ophalen |
| `map()` | iedere waarde omzetten |
| `filter()` | bepaalde waarden behouden |
| `find()` | eerste passende waarde zoeken |
| `any()` | voldoet minstens één waarde? |
| `all()` | voldoen alle waarden? |
| `sum()` | waarden optellen |
| `collect()` | resultaten verzamelen |

Een handige manier om ze te onthouden:

```text
BRON
 ↓
iter()
 ↓
BEWERKEN
 ↓
map() / filter() / find()
 ↓
RESULTAAT
 ↓
collect() / sum() / any() / all()
```

---

## 22. Veelgemaakte fouten

## Fout 1 — vergeten dat `iter()` referenties geeft

Bij:

```rust,ignore
let getallen = vec![1, 2, 3];

let resultaat = getallen
    .iter()
    .map(|getal| getal * 2)
    .collect();
```

werkt Rust door automatisch derefereren op de juiste plaatsen.

Maar bij complexere bewerkingen kan het verschil tussen:

```rust,ignore
i32
```

en:

```rust,ignore
&i32
```

belangrijk worden.

Als je een compilerfout krijgt, kijk dan eerst naar de types.

---

## Fout 2 — een `Vec` verwachten terwijl er nog een iterator is

Dit:

```rust,ignore
let resultaat = getallen.iter().map(|getal| getal * 2);
```

geeft nog geen `Vec`.

Voor een `Vec` heb je nodig:

```rust,ignore
let resultaat: Vec<i32> = getallen
    .iter()
    .map(|getal| getal * 2)
    .collect();
```

---

## Fout 3 — ownership per ongeluk overnemen

Dit:

```rust,ignore
for naam in namen.into_iter() {
    println!("{naam}");
}
```

kan de waarden uit `namen` overnemen.

Als je `namen` daarna nog nodig hebt, is `iter()` waarschijnlijk wat je zoekt.

---

## Fout 4 — te veel tegelijk proberen

Een lange keten als:

```rust,ignore
let resultaat = iets
    .iter()
    .filter(...)
    .map(...)
    .filter(...)
    .map(...)
    .collect();
```

kan moeilijk te begrijpen zijn.

Begin liever met:

```rust,ignore
let stap1 = ...;
```

en bouw daarna verder.

De compiler is hierbij je hulpmiddel.

---

## 23. Rustlings-oefeningen

Voor dit hoofdstuk gebruiken we:

```text
exercises/iterators/
```

De opdrachten lopen bewust op in moeilijkheid.

## Basis

### 01_iter.rs

Maak een `Vec` met drie getallen en loop er met `iter()` doorheen.

---

### 02_next.rs

Maak een iterator en gebruik `next()` om de eerste waarde op te vragen.

Behandel het resultaat met `match`.

---

### 03_next_none.rs

Lees alle waarden uit een iterator met `next()`.

Controleer wat er gebeurt nadat alle waarden zijn gelezen.

---

### 04_map.rs

Gebruik `map()` om ieder getal met 2 te vermenigvuldigen.

Verzamel het resultaat met `collect()`.

---

### 05_map_names.rs

Maak een `Vec<String>` met drie namen.

Gebruik `map()` om een nieuwe `Vec<String>` te maken waarin iedere naam `"Held: "` ervoor krijgt.

---

### 06_collect.rs

Maak een iterator die de getallen `1` tot en met `5` produceert en verzamel de resultaten in een `Vec<i32>`.

---

## Filteren en zoeken

### 07_filter.rs

Maak een lijst met getallen en gebruik `filter()` om alleen getallen groter dan 10 te behouden.

---

### 08_filter_even.rs

Gebruik `filter()` om alleen even getallen te behouden.

---

### 09_find.rs

Gebruik `find()` om het eerste getal groter dan 20 te vinden.

Behandel het resultaat als `Option`.

---

### 10_find_name.rs

Maak een `Vec<String>` met namen.

Zoek met `find()` naar een specifieke naam.

---

### 11_any.rs

Controleer met `any()` of minstens één getal negatief is.

---

### 12_all.rs

Controleer met `all()` of alle getallen positief zijn.

---

### 13_sum.rs

Bereken met `sum()` de totale waarde van een lijst met getallen.

---

## Mutable iterators

### 14_iter_mut.rs

Maak een `Vec<i32>`.

Gebruik `iter_mut()` om ieder getal met 10 te verhogen.

---

### 15_damage_all.rs

Maak een struct:

```rust,ignore
struct Vijand {
    naam: String,
    gezondheid: i32,
}
```

Gebruik `iter_mut()` om iedere vijand 10 schade te geven.

---

### 16_heal_all.rs

Gebruik `iter_mut()` om alle spelers 5 gezondheid te laten herstellen.

---

## RPG

### 17_living_enemies.rs

Maak een lijst vijanden.

Gebruik:

```text
iter()
filter()
collect()
```

om alle levende vijanden te verzamelen.

---

### 18_enemy_names.rs

Gebruik:

```text
iter()
map()
collect()
```

om de namen van alle vijanden te verzamelen.

Omdat `naam` een `String` is, moet je goed nadenken over ownership en `clone()`.

---

### 19_living_enemy_names.rs

Combineer:

```text
filter()
map()
collect()
```

om alleen de namen van levende vijanden te krijgen.

---

### 20_find_item.rs

Maak een `Item`-struct en een inventaris.

Gebruik `find()` om een item met een bepaalde naam te zoeken.

Geef `Option<&Item>` terug.

---

## Tests

### 21_iterator_test.rs

Schrijf een functie:

```rust,ignore
fn levende_vijanden(vijanden: &[Vijand]) -> Vec<&Vijand>
```

Schrijf minimaal twee tests:

1. er zijn levende vijanden
2. alle vijanden zijn dood.

---

### 22_damage_test.rs

Maak een functie die met `iter_mut()` alle vijanden schade geeft.

Test daarna of alle gezondheid correct is aangepast.

---

### 23_iterator_debug.rs

In deze oefening staat bewust code die niet compileert.

De iterator gebruikt een type dat niet overeenkomt met wat de closure verwacht.

Lees de compilerfout en herstel de code.

---

### 24_collect_debug.rs

De code gebruikt `collect()`, maar Rust weet niet welk type verzameling moet worden gemaakt.

Voeg de ontbrekende type-informatie toe.

---

### 25_borrow_debug.rs

De code probeert tijdens het itereren een verzameling op een manier te wijzigen die niet toegestaan is.

Los het probleem op zonder `clone()` toe te voegen als dat niet nodig is.

---

### 26_ownership_debug.rs

De code gebruikt `into_iter()` en probeert daarna de oorspronkelijke `Vec` nog te gebruiken.

Bepaal of `iter()` of een andere oplossing hier beter past.

---

## Iterator-ketens

### 27_chain.rs

Maak een lijst met vijanden.

Maak daarna één iterator-keten die:

1. alleen levende vijanden selecteert
2. hun namen neemt
3. de namen verzamelt in een `Vec<String>`.

---

### 28_strong_enemies.rs

Selecteer met `filter()` alleen vijanden met minstens 50 gezondheid.

Gebruik daarna `map()` om hun namen te verzamelen.

---

### 29_total_health.rs

Gebruik `iter()` en `map()` om de gezondheid van alle vijanden te verzamelen en gebruik daarna `sum()` om de totale gezondheid te berekenen.

---

### 30_final_battle.rs

Maak een klein gevechtssysteem met:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
}

struct Vijand {
    naam: String,
    gezondheid: i32,
}
```

Gebruik iterators om:

1. alle levende vijanden te vinden
2. hun namen te verzamelen
3. te bepalen of er nog minstens één vijand leeft
4. de totale gezondheid van alle vijanden te berekenen.

Schrijf daarnaast minimaal drie automatische tests.

---

## 24. Zelfstandige opdrachten

Naast Rustlings zijn deze opdrachten geschikt om zelfstandig te maken.

## Opdracht 1 — Inventaris

Maak:

```rust,ignore
struct Item {
    naam: String,
    waarde: i32,
}
```

Maak een inventaris met meerdere items.

Gebruik iterators om:

- alle namen te verzamelen
- items boven een bepaalde waarde te vinden
- een specifiek item te zoeken
- te bepalen of minstens één item meer dan 100 goud waard is.

---

## Opdracht 2 — Gevecht

Maak vijf vijanden.

Gebruik:

```text
iter_mut()
```

om iedere vijand schade te geven.

Gebruik daarna:

```text
filter()
```

om de levende vijanden te vinden.

---

## Opdracht 3 — Ervaringspunten

Geef iedere vijand een hoeveelheid ervaringspunten:

```rust,ignore
struct Vijand {
    naam: String,
    gezondheid: i32,
    ervaring: i32,
}
```

Bereken met `map()` en `sum()` hoeveel ervaring de speler krijgt wanneer alle vijanden verslagen zijn.

---

## Opdracht 4 — Zoek de baas

Maak een lijst met vijanden.

Geef iedere vijand een `naam` en `gezondheid`.

Gebruik `find()` om een vijand met de naam `"Draak"` te vinden.

Behandel de mogelijkheid dat de draak niet bestaat met `Option`.

---

## Opdracht 5 — Battle report

Maak een functie die een tekstuele samenvatting van het gevecht maakt.

Gebruik iterators om bijvoorbeeld:

- het aantal levende vijanden te bepalen
- hun namen te verzamelen
- de totale resterende gezondheid te berekenen.

Schrijf tests voor de functie.

---

## 25. Wat je nu moet kunnen

Na dit hoofdstuk moet je begrijpen:

- wat een iterator is
- waarom `iter()` handig is
- dat `iter()` waarden leent
- hoe `next()` werkt
- dat `next()` een `Option` teruggeeft
- hoe `map()` waarden omzet
- hoe `filter()` waarden selecteert
- hoe `find()` één waarde zoekt
- hoe `any()` en `all()` voorwaarden controleren
- hoe `sum()` waarden optelt
- hoe `collect()` iteratorresultaten verzamelt
- wanneer je `iter_mut()` gebruikt
- waarom `into_iter()` ownership kan overnemen
- hoe meerdere iterator-adaptors achter elkaar kunnen worden gebruikt.

Het belangrijkste patroon om te onthouden is:

```rust,ignore
collectie
    .iter()
    .filter(...)
    .map(...)
    .collect()
```

Maar probeer vooral niet alleen de syntax uit je hoofd te leren.

Begrijp wat iedere stap doet:

```text
iter()
    ↓
geeft waarden één voor één
    ↓
filter()
    ↓
houdt bepaalde waarden over
    ↓
map()
    ↓
zet waarden om
    ↓
collect()
    ↓
maakt er weer een verzameling van
```

Daarmee hebben we de basis gelegd voor een veel krachtigere manier om met lijsten te werken.

In een volgend hoofdstuk kunnen we dieper ingaan op **iterators en adaptors zelf**: lazy evaluation, uitgebreidere combinaties van `map`, `filter`, `flat_map`, `take`, `skip`, `enumerate`, `zip` en de manier waarop iterator-ketens efficiënt worden uitgevoerd.

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 22](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_22/).

