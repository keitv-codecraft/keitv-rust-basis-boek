# 23. Iterators verder uitgediept

## Wat gaan we leren?

In dit artikel gaan we dieper in op de werking van iterators en ontdekken we de kracht van lazy evaluation en geavanceerde adaptors.

We leren:

- wat lazy evaluation betekent en waarom iterators pas werken als je een consumer aanroept
- het verschil tussen adaptors (zoals `map`, `filter`, `take`) en consumers (zoals `collect`, `count`, `sum`)
- indexen toevoegen met `enumerate()`
- delen selecteren met `take()` en `skip()`
- twee stromen samenvoegen met `zip()`
- geneste structuren afvlakken met `flat_map()` en `filter_map()`.

---

## 1. Wat is een iterator eigenlijk?

Een iterator kun je zien als een object dat weet:

> "Wat is de volgende waarde?"

Bijvoorbeeld:

```rust,ignore
let getallen = vec![10, 20, 30];

let mut iterator = getallen.iter();
```

De iterator staat eerst vóór de eerste waarde.

Als we vragen:

```rust,ignore
iterator.next()
```

krijgen we:

```text
Some(10)
```

De iterator is nu verder gegaan.

Nog een keer:

```rust,ignore
iterator.next()
```

geeft:

```text
Some(20)
```

En daarna:

```rust,ignore
iterator.next()
```

geeft:

```text
Some(30)
```

Daarna:

```rust,ignore
iterator.next()
```

geeft:

```text
None
```

Je kunt het zien als:

```text
Vec
┌────┬────┬────┐
│ 10 │ 20 │ 30 │
└────┴────┴────┘
  ↑
  |
iterator
```

Na `next()`:

```text
┌────┬────┬────┐
│ 10 │ 20 │ 30 │
└────┴────┴────┘
       ↑
       |
    iterator
```

De iterator houdt dus bij waar hij gebleven is.

---

## 2. `for` gebruikt een iterator

We hoeven meestal niet zelf `next()` te schrijven.

Als we:

```rust,ignore
for getal in getallen.iter() {
    println!("{getal}");
}
```

schrijven, zorgt Rust er onder andere voor dat de volgende waarden van de iterator worden opgehaald.

Conceptueel lijkt dat op:

```rust,ignore
loop {
    match iterator.next() {
        Some(getal) => {
            println!("{getal}");
        }
        None => {
            break;
        }
    }
}
```

Je hoeft dit normaal gesproken niet zelf zo te schrijven.

Het is vooral belangrijk om te begrijpen dat een `for`-lus en een iterator nauw met elkaar verbonden zijn.

---

## 3. Een iterator is niet hetzelfde als een `Vec`

Een `Vec` is een verzameling waarden.

Bijvoorbeeld:

```rust,ignore
let vijanden = vec![
    "Goblin",
    "Ork",
    "Draak",
];
```

Een iterator is iets anders.

```rust,ignore
let iterator = vijanden.iter();
```

De `Vec` bevat de waarden.

De iterator weet hoe hij die waarden één voor één kan leveren.

Dus:

```text
Vec
 ↓
bevat waarden

Iterator
 ↓
levert waarden één voor één
```

Dat verschil wordt belangrijk bij iterator-adaptors.

---

## 4. Wat is een iterator-adaptor?

Een **iterator-adaptor** is een methode die een bestaande iterator gebruikt en daar een nieuwe iterator van maakt.

Bijvoorbeeld:

```rust,ignore
let iterator = getallen.iter();

let nieuwe_iterator = iterator.map(|getal| getal * 2);
```

`map()` neemt dus een iterator en maakt een andere iterator.

Conceptueel:

```text
oude iterator
      ↓
    map()
      ↓
nieuwe iterator
```

Hetzelfde geldt voor:

```rust,ignore
filter()
```

en bijvoorbeeld:

```rust,ignore
take()
```

Een adaptor verandert dus de manier waarop de iterator zijn waarden levert.

---

## 5. Het belangrijke idee: `map()` voert nog niets uit

Dit is een van de belangrijkste eigenschappen van Rust-iterators.

Kijk naar:

```rust,ignore
let getallen = vec![1, 2, 3];

let resultaat = getallen
    .iter()
    .map(|getal| {
        println!("map wordt uitgevoerd");
        getal * 2
    });
```

Misschien verwacht je dat:

```text
map wordt uitgevoerd
map wordt uitgevoerd
map wordt uitgevoerd
```

op het scherm verschijnt.

Maar dat gebeurt niet.

Waarom?

Omdat `map()` de waarden nog niet heeft verwerkt.

We hebben alleen een nieuwe iterator gemaakt.

---

## 6. Lazy evaluation

Dit gedrag noemen we **lazy evaluation**.

Dat betekent hier:

> Een iterator voert zijn werk pas uit wanneer iemand daadwerkelijk om een waarde vraagt.

Bijvoorbeeld:

```rust,ignore
let getallen = vec![1, 2, 3];

let resultaat = getallen
    .iter()
    .map(|getal| {
        println!("getal verwerken: {getal}");
        getal * 2
    });
```

Tot hier gebeurt er nog niets.

Pas wanneer we bijvoorbeeld:

```rust,ignore
for getal in resultaat {
    println!("resultaat: {getal}");
}
```

doen, worden de waarden verwerkt.

De uitvoer wordt dan ongeveer:

```text
getal verwerken: 1
resultaat: 2
getal verwerken: 2
resultaat: 4
getal verwerken: 3
resultaat: 6
```

---

## 7. Waarom is dat handig?

Dit betekent dat Rust niet noodzakelijk eerst een hele nieuwe lijst hoeft te maken.

Stel:

```text
1.000.000 getallen
       ↓
filter
       ↓
map
       ↓
resultaat
```

De bewerkingen kunnen één voor één worden uitgevoerd.

Conceptueel:

```text
getal 1
 ↓
filter
 ↓
map
 ↓
resultaat

getal 2
 ↓
filter
 ↓
map
 ↓
resultaat

getal 3
 ↓
filter
 ↓
map
 ↓
resultaat
```

Er hoeft dus niet automatisch voor iedere tussenstap een nieuwe `Vec` te worden gemaakt.

Dat is een belangrijke reden waarom iterator-ketens efficiënt kunnen zijn.

---

## 8. `collect()` voert de iterator uit

Nu komen we terug bij:

```rust,ignore
collect()
```

Bijvoorbeeld:

```rust,ignore
let resultaat: Vec<i32> = getallen
    .iter()
    .map(|getal| getal * 2)
    .collect();
```

`collect()` vraagt de iterator om zijn waarden.

Daardoor worden de vorige stappen daadwerkelijk uitgevoerd.

Conceptueel:

```text
iter()
 ↓
map()
 ↓
collect()
 ↓
waarden worden opgevraagd
 ↓
map wordt uitgevoerd
 ↓
Vec wordt gevuld
```

Daarom noemen we `collect()` vaak een **consumer** van de iterator.

Een consumer gebruikt de iterator en vraagt de resultaten op.

---

## 9. Adaptors en consumers

We kunnen de belangrijkste methodes voorlopig in twee groepen verdelen.

### Adaptors

Deze maken een nieuwe iterator:

```text
map()
filter()
take()
skip()
enumerate()
zip()
```

Bijvoorbeeld:

```rust,ignore
iterator
    .filter(...)
    .map(...)
```

### Consumers

Deze vragen daadwerkelijk waarden op:

```text
collect()
sum()
find()
any()
all()
```

Bijvoorbeeld:

```rust,ignore
iterator
    .filter(...)
    .count()
```

Hier zorgt `count()` ervoor dat de iterator daadwerkelijk wordt uitgevoerd.

Het onderscheid is belangrijk:

```text
ADAPTOR
maakt een nieuwe iterator

CONSUMER
vraagt waarden van de iterator
```

---

## 10. `count()`

Een eenvoudige consumer die we nog niet uitgebreid hebben besproken is:

```rust,ignore
count()
```

Bijvoorbeeld:

```rust,ignore
let getallen = vec![10, 20, 30, 40];

let aantal = getallen.iter().count();

println!("{aantal}");
```

Dit geeft:

```text
4
```

We kunnen ook eerst filteren:

```rust,ignore
let aantal = getallen
    .iter()
    .filter(|getal| **getal > 20)
    .count();
```

Het resultaat is:

```text
2
```

Omdat alleen `30` en `40` groter zijn dan 20.

---

## 11. `enumerate()` — de positie erbij

Soms willen we niet alleen de waarde hebben, maar ook de positie.

Bijvoorbeeld:

```rust,ignore
let namen = vec![
    "Aria",
    "Borin",
    "Cedric",
];
```

We kunnen schrijven:

```rust,ignore
for (index, naam) in namen.iter().enumerate() {
    println!("{index}: {naam}");
}
```

De uitvoer:

```text
0: Aria
1: Borin
2: Cedric
```

`enumerate()` geeft dus paren:

```text
(index, waarde)
```

De index begint bij `0`.

---

## 12. `enumerate()` in een RPG

Stel dat we een groep vijanden tonen:

```rust,ignore
for (index, vijand) in vijanden.iter().enumerate() {
    println!(
        "{}. {} ({} HP)",
        index + 1,
        vijand.naam,
        vijand.gezondheid
    );
}
```

We gebruiken `index + 1` omdat spelers meestal liever vijand 1, 2 en 3 zien dan vijand 0, 1 en 2.

Bijvoorbeeld:

```text
1. Goblin (30 HP)
2. Ork (80 HP)
3. Draak (200 HP)
```

---

## 13. `enumerate()` en `map()`

We kunnen `enumerate()` ook combineren met `map()`.

```rust,ignore
let namen: Vec<String> = vijanden
    .iter()
    .enumerate()
    .map(|(index, vijand)| {
        format!("{}. {}", index + 1, vijand.naam)
    })
    .collect();
```

Hier gebeurt:

```text
iter()
 ↓
(index, vijand)
 ↓
map()
 ↓
String
 ↓
collect()
 ↓
Vec<String>
```

Dit is een goed voorbeeld van een iterator-keten die steeds een andere vorm krijgt.

---

## 14. `take()` — neem de eerste waarden

Met:

```rust,ignore
take()
```

kunnen we aangeven hoeveel waarden we willen gebruiken.

Bijvoorbeeld:

```rust,ignore
let getallen = vec![10, 20, 30, 40, 50];

let eerste_drie: Vec<&i32> = getallen
    .iter()
    .take(3)
    .collect();
```

Het resultaat bevat:

```text
10
20
30
```

De overige waarden worden niet gebruikt.

---

## 15. `take()` in een RPG

Stel dat we maximaal drie vijanden op het scherm willen tonen:

```rust,ignore
for vijand in vijanden.iter().take(3) {
    println!("{}", vijand.naam);
}
```

Als er tien vijanden zijn, worden er maar drie bekeken.

Dit is een mooi voorbeeld van lazy evaluation.

Rust hoeft niet eerst alle tien vijanden door de hele keten te laten lopen.

---

## 16. `skip()` — sla waarden over

Het tegenovergestelde is:

```rust,ignore
skip()
```

Bij:

```rust,ignore
let getallen = vec![10, 20, 30, 40, 50];

let resultaat: Vec<&i32> = getallen
    .iter()
    .skip(2)
    .collect();
```

worden de eerste twee waarden overgeslagen.

Het resultaat is:

```text
30
40
50
```

We kunnen `skip()` en `take()` combineren:

```rust,ignore
let resultaat: Vec<&i32> = getallen
    .iter()
    .skip(2)
    .take(2)
    .collect();
```

Resultaat:

```text
30
40
```

Dit lijkt op:

```text
10  20  30  40  50
     ↑
   skip 2
         ↓
       30  40
       ↑
     take 2
```

---

## 17. `zip()` — twee iterators combineren

Stel dat we twee lijsten hebben:

```rust,ignore
let namen = vec![
    "Aria",
    "Borin",
    "Cedric",
];

let gezondheid = vec![
    100,
    80,
    120,
];
```

We willen naam en gezondheid samen gebruiken.

Dat kan met:

```rust,ignore
for (naam, hp) in namen.iter().zip(gezondheid.iter()) {
    println!("{naam}: {hp} HP");
}
```

Resultaat:

```text
Aria: 100 HP
Borin: 80 HP
Cedric: 120 HP
```

`zip()` maakt paren:

```text
Aria    + 100
Borin   + 80
Cedric  + 120
```

---

## 18. `zip()` stopt bij de kortste iterator

Wat gebeurt er als de lijsten verschillende lengtes hebben?

```rust,ignore
let namen = vec![
    "Aria",
    "Borin",
    "Cedric",
];

let gezondheid = vec![
    100,
    80,
];
```

Dan levert:

```rust,ignore
namen.iter().zip(gezondheid.iter())
```

slechts twee paren op.

```text
Aria    100
Borin   80
```

`Cedric` heeft geen bijpassende gezondheid en wordt dus niet gebruikt.

Dit is belangrijk om te onthouden.

---

## 19. `zip()` met structgegevens

In een echte RPG zou je normaal gesproken naam en gezondheid gewoon in één `Vijand`-struct bewaren.

Dus liever:

```rust,ignore
struct Vijand {
    naam: String,
    gezondheid: i32,
}
```

dan twee losse lijsten.

Toch is `zip()` een nuttig gereedschap wanneer twee verschillende gegevensreeksen bewust bij elkaar horen.

---

## 20. `flat_map()` — meerdere waarden uit één waarde

Nu komen we bij een iets moeilijker concept.

Stel dat iedere speler meerdere items heeft.

```rust,ignore
let inventarissen = vec![
    vec!["Zwaard", "Schild"],
    vec!["Boog"],
    vec!["Drank", "Sleutel"],
];
```

We hebben dus:

```text
speler 1 → Zwaard, Schild
speler 2 → Boog
speler 3 → Drank, Sleutel
```

Dit is een `Vec` met daarin andere `Vec`s.

We kunnen met `map()` een nieuwe verzameling van verzamelingen maken.

```rust,ignore
let alle_inventarissen = inventarissen
    .iter()
    .map(|inventaris| inventaris);
```

Maar we willen eigenlijk één reeks:

```text
Zwaard
Schild
Boog
Drank
Sleutel
```

Daarvoor kunnen we `flat_map()` gebruiken:

```rust,ignore
let alle_items: Vec<&&str> = inventarissen
    .iter()
    .flat_map(|inventaris| inventaris.iter())
    .collect();
```

Het idee is:

```text
map()
 ↓
één waarde → één waarde

flat_map()
 ↓
één waarde → meerdere waarden
```

---

## 21. Waarom heet het `flat_map()`?

Stel dat `map()` dit maakt:

```text
[
    [A, B],
    [C],
    [D, E]
]
```

Dat zijn drie lijsten.

`flat_map()` maakt daar conceptueel van:

```text
[A, B, C, D, E]
```

De geneste structuur wordt als het ware plat gemaakt.

Daarom:

```text
flat_map
   ↓
flat + map
```

Voor beginners is vooral dit voldoende:

> Gebruik `flat_map()` wanneer één invoerwaarde meerdere uitvoerwaarden kan opleveren en je die allemaal in één iterator wilt krijgen.

---

## 22. `flat_map()` met een RPG

Stel dat we kamers hebben en iedere kamer meerdere vijanden bevat:

```rust,ignore
let kamers = vec![
    vec!["Goblin", "Goblin"],
    vec!["Ork"],
    vec!["Draak", "Slang"],
];
```

Met:

```rust,ignore
let vijanden: Vec<&&str> = kamers
    .iter()
    .flat_map(|kamer| kamer.iter())
    .collect();
```

krijgen we één reeks vijanden.

We kunnen daarna bijvoorbeeld filteren:

```rust,ignore
let vijanden: Vec<&&str> = kamers
    .iter()
    .flat_map(|kamer| kamer.iter())
    .filter(|naam| **naam != "Goblin")
    .collect();
```

Nu worden Goblins eruit gehaald.

---

## 23. Iterator-ketens lezen

Een lange iterator-keten kan in het begin intimiderend zijn.

Neem:

```rust,ignore
let namen: Vec<String> = vijanden
    .iter()
    .filter(|vijand| vijand.gezondheid > 0)
    .enumerate()
    .map(|(index, vijand)| {
        format!("{}. {}", index + 1, vijand.naam)
    })
    .take(3)
    .collect();
```

Lees hem niet in één keer.

Lees hem stap voor stap:

```text
vijanden
   ↓
iter()
   ↓
vijanden lenen
   ↓
filter()
   ↓
alleen levende vijanden
   ↓
enumerate()
   ↓
nummer + vijand
   ↓
map()
   ↓
tekst maken
   ↓
take(3)
   ↓
maximaal drie
   ↓
collect()
   ↓
Vec<String>
```

Dit is een belangrijke programmeervaardigheid:

> Breek ingewikkelde code mentaal op in kleine stappen.

---

## 24. De keten kan ook in losse stappen

Als een keten te ingewikkeld wordt, kun je hem opsplitsen.

Bijvoorbeeld:

```rust,ignore
let levende = vijanden
    .iter()
    .filter(|vijand| vijand.gezondheid > 0);

let genummerd = levende.enumerate();

let namen = genummerd.map(|(index, vijand)| {
    format!("{}. {}", index + 1, vijand.naam)
});

let namen: Vec<String> = namen
    .take(3)
    .collect();
```

Dit doet hetzelfde.

Voor beginners is deze vorm soms veel gemakkelijker te begrijpen.

---

## 25. Lazy evaluation wordt nu duidelijker

Stel:

```rust,ignore
let resultaat = vijanden
    .iter()
    .filter(|vijand| vijand.gezondheid > 0)
    .map(|vijand| {
        println!("Map: {}", vijand.naam);
        vijand.naam.clone()
    });
```

Er is nog niets uitgevoerd.

Pas wanneer we:

```rust,ignore
for naam in resultaat {
    println!("Naam: {naam}");
}
```

doen, worden de stappen uitgevoerd.

En niet eerst:

```text
alle filter-resultaten
```

en daarna:

```text
alle map-resultaten
```

maar conceptueel per element:

```text
Vijand 1
 ↓
filter
 ↓
map
 ↓
resultaat

Vijand 2
 ↓
filter
 ↓
map
 ↓
resultaat

Vijand 3
 ↓
filter
 ↓
map
 ↓
resultaat
```

Dit is een belangrijk verschil met het idee dat iedere methode meteen een nieuwe `Vec` maakt.

---

## 26. Lazy evaluation kan werk besparen

Stel dat we:

```rust,ignore
let vijand = vijanden
    .iter()
    .filter(|vijand| vijand.gezondheid > 0)
    .find(|vijand| vijand.naam == "Draak");
```

gebruiken.

`find()` hoeft niet noodzakelijk alle vijanden te bekijken.

Zodra de Draak gevonden is, is het resultaat bekend.

Conceptueel:

```text
Goblin
 ↓
filter → ja
 ↓
find → nee

Ork
 ↓
filter → ja
 ↓
find → nee

Draak
 ↓
filter → ja
 ↓
find → JA!
 ↓
stop
```

De iterator hoeft daarna niet verder te gaan.

Dit is een van de voordelen van lazy iterators.

---

## 27. `take()` en lazy evaluation

Hetzelfde geldt voor:

```rust,ignore
let eerste_drie = getallen
    .iter()
    .take(3);
```

Als de oorspronkelijke lijst een miljoen waarden bevat, vraagt `take(3)` maar om de eerste drie.

Er wordt dus niet automatisch een miljoen waarden verwerkt.

Dit soort gedrag wordt vooral interessant bij grote hoeveelheden gegevens.

---

## 28. `filter()` en closures

We hebben eerder geleerd dat een closure gedrag als waarde kan bevatten.

Bij:

```rust,ignore
.filter(|vijand| vijand.gezondheid > 0)
```

geven we aan `filter()` een closure.

Die closure beantwoordt eigenlijk de vraag:

> Moet deze waarde behouden blijven?

Bijvoorbeeld:

```rust,ignore
|vijand| vijand.gezondheid > 0
```

geeft:

```text
true  → behouden
false → verwijderen
```

Bij:

```rust,ignore
.map(|vijand| vijand.naam.clone())
```

beantwoordt de closure een andere vraag:

> Wat moet deze waarde worden?

Dat geeft een nuttige mentale scheiding:

```text
filter → behouden of niet?
map    → wat wordt de nieuwe waarde?
```

---

## 29. `filter_map()`

Er bestaat ook een adaptor die deze ideeën combineert:

```rust,ignore
filter_map()
```

Deze is iets gevorderder, maar past goed bij wat we al kennen van `Option`.

Stel dat we alleen positieve getallen willen en die vervolgens willen omzetten.

```rust,ignore
let getallen = vec![-2, 4, -1, 8];
```

We kunnen bijvoorbeeld schrijven:

```rust,ignore
let positieve: Vec<i32> = getallen
    .iter()
    .filter_map(|getal| {
        if *getal > 0 {
            Some(*getal * 2)
        } else {
            None
        }
    })
    .collect();
```

Het resultaat:

```text
8
16
```

De closure geeft:

```text
Some(waarde)
```

als we een waarde willen behouden.

En:

```text
None
```

als we niets willen produceren.

Dit combineert dus:

```text
filter
+
map
```

in één stap.

Je hoeft `filter_map()` nog niet uit je hoofd te kennen. Het belangrijkste is dat je begrijpt wat het doet.

---

## 30. `flat_map()` en `Option`

Omdat `Option` en iterators goed samenwerken, kunnen we later ook patronen tegenkomen zoals:

```rust,ignore
items.iter().filter_map(...)
```

Daarmee kunnen we bijvoorbeeld alleen bestaande optionele waarden meenemen.

Dit is een belangrijk idee in Rust:

> Veel Rust-typen zijn ontworpen om goed met iterators samen te werken.

Daarom komen `Option`, `Result`, closures en iterators in echte programma's vaak samen.

`Result` en iterators zullen we later uitgebreider combineren.

---

## 31. Ownership bij iterator-adaptors

Ownership blijft belangrijk.

Stel:

```rust,ignore
let namen = vec![
    String::from("Aria"),
    String::from("Borin"),
];
```

Met:

```rust,ignore
let lengte: Vec<usize> = namen
    .iter()
    .map(|naam| naam.len())
    .collect();
```

blijft `namen` gewoon bestaan.

We lenen de strings.

Maar met:

```rust,ignore
let lengte: Vec<usize> = namen
    .into_iter()
    .map(|naam| naam.len())
    .collect();
```

worden de `String`-waarden uit de `Vec` gehaald.

Daarna is `namen` niet meer beschikbaar.

Het verschil is dus:

```text
iter()
 ↓
lenen

into_iter()
 ↓
overnemen
```

Dit is precies waarom ownership zo belangrijk blijft wanneer we met iterators werken.

---

## 32. `iter_mut()` in een keten

We kunnen ook mutable iterators combineren met adaptors.

Bijvoorbeeld:

```rust,ignore
vijanden
    .iter_mut()
    .for_each(|vijand| {
        vijand.gezondheid -= 10;
    });
```

Hier zien we een nieuwe methode:

```rust,ignore
for_each()
```

`for_each()` is een consumer.

Hij voert een closure uit voor iedere waarde.

Toch is een gewone `for`-lus hier vaak duidelijker:

```rust,ignore
for vijand in vijanden.iter_mut() {
    vijand.gezondheid -= 10;
}
```

Gebruik iterator-adaptors dus niet alleen omdat het korter kan.

Gebruik de vorm die de bedoeling duidelijk maakt.

---

## 33. Een praktische regel

Voor deze cursus kun je voorlopig ongeveer deze regels gebruiken:

### Alleen lezen

```rust,ignore
iter()
```

### Waarden aanpassen

```rust,ignore
iter_mut()
```

### Ownership overnemen

```rust,ignore
into_iter()
```

### Iedere waarde veranderen

```rust,ignore
map()
```

### Alleen sommige waarden behouden

```rust,ignore
filter()
```

### Eerste passende waarde zoeken

```rust,ignore
find()
```

### Nummer toevoegen

```rust,ignore
enumerate()
```

### Eerste aantal nemen

```rust,ignore
take()
```

### Begin overslaan

```rust,ignore
skip()
```

### Twee reeksen combineren

```rust,ignore
zip()
```

### Geneste reeksen plat maken

```rust,ignore
flat_map()
```

### Iterator uitvoeren

```rust,ignore
collect()
sum()
find()
any()
all()
count()
```

---

## 34. Een compleet RPG-voorbeeld

We kunnen nu veel van deze kennis combineren.

Stel:

```rust,ignore
struct Vijand {
    naam: String,
    gezondheid: i32,
}
```

En:

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
        naam: String::from("Trol"),
        gezondheid: 100,
    },
    Vijand {
        naam: String::from("Draak"),
        gezondheid: 250,
    },
];
```

We willen de eerste drie levende vijanden tonen:

```rust,ignore
let namen: Vec<String> = vijanden
    .iter()
    .filter(|vijand| vijand.gezondheid > 0)
    .take(3)
    .enumerate()
    .map(|(index, vijand)| {
        format!("{}. {}", index + 1, vijand.naam)
    })
    .collect();
```

De keten:

```text
vijanden
   ↓
iter()
   ↓
filter()
   ↓
alleen levende vijanden
   ↓
take(3)
   ↓
maximaal drie
   ↓
enumerate()
   ↓
nummer toevoegen
   ↓
map()
   ↓
tekst maken
   ↓
collect()
   ↓
Vec<String>
```

Dit is precies het soort code dat we in grotere Rust-programma's veel zullen tegenkomen.

---

## 35. Maar schrijf niet altijd de kortste code

De vorige code is krachtig, maar voor een beginner kan dit ook prima zijn:

```rust,ignore
let mut namen = Vec::new();
let mut nummer = 1;

for vijand in &vijanden {
    if vijand.gezondheid > 0 {
        namen.push(format!("{}. {}", nummer, vijand.naam));
        nummer += 1;

        if nummer > 3 {
            break;
        }
    }
}
```

Deze code is langer, maar misschien gemakkelijker te begrijpen.

Iterator-code is geen doel op zichzelf.

Het doel is duidelijke en betrouwbare code.

---

## 36. Wat gebeurt er onder de motorkap?

We hoeven dit nog niet technisch volledig te begrijpen.

Maar conceptueel kunnen we zeggen:

```text
Vec
 ↓
iter()
 ↓
Iterator
 ↓
Adaptor
 ↓
nieuwe Iterator
 ↓
Adaptor
 ↓
nieuwe Iterator
 ↓
Consumer
 ↓
resultaat
```

Bijvoorbeeld:

```rust,ignore
let resultaat: Vec<i32> = getallen
    .iter()
    .filter(|getal| **getal > 10)
    .map(|getal| *getal * 2)
    .collect();
```

Conceptueel:

```text
Vec
 ↓
iter()
 ↓
filter
 ↓
map
 ↓
collect
```

Maar `filter()` en `map()` maken onderweg niet automatisch allerlei `Vec`s.

Ze bouwen een beschrijving van hoe de waarden moeten worden verwerkt.

Pas wanneer een consumer waarden opvraagt, begint de iterator daadwerkelijk te lopen.

---

## 37. Waarom dit later belangrijk wordt

We hebben nu meerdere belangrijke Rust-concepten gecombineerd:

```text
ownership
borrowing
structs
closures
Option
Vec
iterators
tests
```

Dat is precies waarom iterators een belangrijk onderdeel van Rust zijn.

Later komen daar:

```text
generics
traits
Result
```

bij.

Dan zullen veel Rust-functies ineens minder vreemd lijken.

Je zult bijvoorbeeld functies tegenkomen die conceptueel zeggen:

> Geef mij een willekeurige verzameling waar ik doorheen kan lopen.

Dat wordt mogelijk gemaakt door traits en generics.

We hoeven die syntax nu nog niet volledig te beheersen.

---

## 38. Zelfstandige opdrachten

### Opdracht 1 — Vijandenlijst

Maak een functie:

```rust,ignore
fn vijand_lijst(vijanden: &[Vijand]) -> Vec<String>
```

De functie moet een lijst maken zoals:

```text
1. Goblin - 30 HP
2. Ork - 80 HP
3. Draak - 200 HP
```

Gebruik:

```text
iter()
enumerate()
map()
collect()
```

---

### Opdracht 2 — Levende vijanden

Maak:

```rust,ignore
fn levende_vijanden(vijanden: &[Vijand]) -> Vec<&Vijand>
```

Gebruik `filter()`.

Schrijf tests.

---

### Opdracht 3 — Eerste drie

Maak een functie die maximaal drie levende vijanden teruggeeft.

Gebruik:

```text
filter()
take()
collect()
```

---

### Opdracht 4 — Totaal gevecht

Bereken de totale resterende gezondheid van alle levende vijanden.

Gebruik:

```text
filter()
map()
sum()
```

---

### Opdracht 5 — Kamers

Maak een spelwereld met kamers:

```text
kamer 1 → Goblin, Goblin
kamer 2 → Ork
kamer 3 → Draak, Slang
```

Gebruik `flat_map()` om alle vijanden in één reeks te krijgen.

Filter daarna alle Goblins eruit.

---

## 39. Belangrijkste begrippen

Je moet na dit hoofdstuk vooral deze begrippen herkennen:

| Begrip | Betekenis |
| --- | --- |
| iterator | levert waarden één voor één |
| `next()` | vraagt de volgende waarde |
| lazy | werk wordt pas uitgevoerd wanneer nodig |
| adaptor | maakt een nieuwe iterator op basis van een iterator |
| consumer | gebruikt de iterator daadwerkelijk |
| `map()` | iedere waarde omzetten |
| `filter()` | bepaalde waarden behouden |
| `enumerate()` | index toevoegen |
| `take()` | eerste aantal waarden nemen |
| `skip()` | eerste aantal waarden overslaan |
| `zip()` | twee iterators combineren |
| `flat_map()` | meerdere resultaten per invoer combineren |
| `filter_map()` | filteren en omzetten met `Option` |
| `collect()` | resultaten verzamelen |
| `sum()` | waarden optellen |
| `find()` | eerste passende waarde zoeken |
| `count()` | aantal waarden tellen |

Het belangrijkste mentale model is:

```text
                 ITERATOR
                    │
          ┌─────────┴─────────┐
          ↓                   ↓
       adaptor             adaptor
       filter()             map()
          │                   │
          └─────────┬─────────┘
                    ↓
              nieuwe iterator
                    │
                    ↓
                consumer
                    │
                    ↓
                resultaat
```

En vooral:

> **Een adaptor voert zijn werk niet meteen uit. Hij beschrijft hoe de volgende waarden moeten worden verwerkt.**

Pas een consumer zoals `collect()`, `sum()`, `find()`, `count()`, `any()` of `all()` vraagt daadwerkelijk om resultaten.

Dat verklaart waarom iterator-ketens zowel krachtig als efficiënt kunnen zijn.

---

## 40. Vooruitblik

We hebben nu veel van de praktische kant van iterators gezien.

Wat we nog bewust niet hebben behandeld, zijn vragen als:

- Wat is precies de `Iterator`-trait?
- Waarom werkt `map()` met allerlei verschillende typen?
- Wat betekenen de generics in iterator-methodes?
- Wat zijn `Item` en `Self::Item`?
- Wat betekenen `Fn`, `FnMut` en `FnOnce`?
- Hoe kun je zelf een iterator maken?

Daarvoor hebben we **traits, closures en generics** nodig.

Die onderwerpen komen daarom later in de cursus terug.

Voorlopig is het belangrijker dat je iterator-code kunt **lezen, schrijven, testen en debuggen**.

Als je deze code begrijpt:

```rust,ignore
let namen: Vec<String> = vijanden
    .iter()
    .filter(|vijand| vijand.gezondheid > 0)
    .take(3)
    .enumerate()
    .map(|(index, vijand)| {
        format!("{}. {}", index + 1, vijand.naam)
    })
    .collect();
```

en stap voor stap kunt uitleggen wat iedere regel doet, dan beheers je de belangrijkste praktische basis van iterator-adaptors.

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 23](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_23/).

---

## Controlelijst

Je bent klaar met dit artikel als je zonder hulp:

- [ ] kunt uitleggen wat lazy evaluation is en waarom iterators lui zijn
- [ ] het verschil kent tussen adaptors en consumers
- [ ] `enumerate()` kunt gebruiken om een element met zijn volgnummer te koppelen
- [ ] `take()` en `skip()` kunt gebruiken om bereiken te selecteren
- [ ] `zip()` kunt gebruiken om twee lijsten te combineren
- [ ] `flat_map()` of `filter_map()` kunt inzetten om data te filteren en transformeren.
