# 25. HashMap

## Wat gaan we leren?

In dit artikel leren we hoe we gegevens koppelen aan sleutels met behulp van de collectie `HashMap`.

We leren:

- een `HashMap` aanmaken en importeren (`use std::collections::HashMap`)
- sleutel-waardeparen toevoegen en bijwerken met `insert()`
- waarden veilig opzoeken met `get()` en `contains_key()` (levert `Option`)
- waarden aanpassen met de handige `entry(...).or_insert(...)` api
- itereren over sleutels, waarden of paren met `for`
- een inventarissysteem met aantallen bouwen voor een RPG.

---

## 1. Een HashMap maken

`HashMap` staat in de standaardbibliotheek van Rust. We moeten hem daarom eerst importeren:

```rust,ignore
use std::collections::HashMap;
```

Daarna kunnen we een lege `HashMap` maken:

```rust,ignore
let mut inventaris: HashMap<String, i32> = HashMap::new();
```

Hiermee zeggen we:

> Maak een `HashMap` waarin een `String` als sleutel wordt gebruikt en een `i32` als waarde.

We kunnen vervolgens waarden toevoegen:

```rust,ignore
inventaris.insert(String::from("Potion"), 3);
inventaris.insert(String::from("Zwaard"), 1);
inventaris.insert(String::from("Brood"), 5);
```

De inhoud is dan conceptueel:

```text
Potion → 3
Zwaard → 1
Brood  → 5
```

Een `HashMap` heeft dus altijd een type voor de **sleutel** en een type voor de **waarde**.

```rust,ignore
HashMap<Sleutel, Waarde>
```

Bijvoorbeeld:

```rust,ignore
HashMap<String, i32>
HashMap<String, bool>
HashMap<i32, String>
```

---

## 2. Waarden opzoeken met `get`

Een van de belangrijkste functies van `HashMap` is `get`.

```rust,ignore
let aantal = inventaris.get("Potion");
```

Het belangrijke hier is dat `get` geen gewone waarde teruggeeft.

Het resultaat is:

```rust,ignore
Option<&i32>
```

Dat komt doordat de sleutel misschien helemaal niet bestaat.

Bijvoorbeeld:

```rust,ignore
let aantal = inventaris.get("Potion");

match aantal {
    Some(aantal) => println!("Je hebt {aantal} potions."),
    None => println!("Je hebt geen potions."),
}
```

Dit sluit direct aan bij wat we eerder hebben geleerd over `Option`.

Een `HashMap` kan namelijk niet garanderen dat een bepaalde sleutel aanwezig is.

---

## 3. Een bestaande waarde uitlezen

We kunnen ook `if let` gebruiken:

```rust,ignore
if let Some(aantal) = inventaris.get("Potion") {
    println!("Je hebt {aantal} potions.");
}
```

Als `"Potion"` niet bestaat, gebeurt er niets.

We kunnen ook een alternatief geven:

```rust,ignore
let aantal = inventaris.get("Potion").unwrap_or(&0);

println!("Aantal potions: {aantal}");
```

Hiermee zeggen we:

> Als `"Potion"` bestaat, gebruik de gevonden waarde. Anders gebruik `0`.

Let op dat `get` een referentie teruggeeft.

Daarom is het resultaat bijvoorbeeld:

```text
Option<&i32>
```

en niet:

```text
Option<i32>
```

Dit is een goed voorbeeld van hoe `HashMap`, `Option` en borrowing samenkomen.

---

## 4. Controleren of een sleutel bestaat

Soms willen we alleen weten of een item aanwezig is.

Daarvoor bestaat:

```rust,ignore
contains_key
```

Bijvoorbeeld:

```rust,ignore
if inventaris.contains_key("Zwaard") {
    println!("Je hebt een zwaard.");
}
```

Of:

```rust,ignore
if !inventaris.contains_key("Boog") {
    println!("Je hebt geen boog.");
}
```

`contains_key` geeft een `bool` terug:

```rust,ignore
true
```

of:

```rust,ignore
false
```

---

## 5. Een waarde vervangen

Als een sleutel al bestaat, kunnen we met `insert` de bijbehorende waarde vervangen.

```rust,ignore
inventaris.insert(String::from("Potion"), 3);
inventaris.insert(String::from("Potion"), 7);
```

Na de tweede `insert` is het aantal:

```text
Potion → 7
```

De oude waarde is dus vervangen.

Dit is anders dan een `Vec`, waar `push` een nieuwe waarde toevoegt.

Bij een `HashMap` hoort iedere sleutel bij maximaal één waarde.

---

## 6. Waarden aanpassen

Stel dat de speler een potion gebruikt.

We willen:

```text
Potion → 3
```

veranderen in:

```text
Potion → 2
```

We kunnen de waarde via `get_mut` aanpassen.

```rust,ignore
if let Some(aantal) = inventaris.get_mut("Potion") {
    *aantal -= 1;
}
```

Hier gebeurt iets belangrijks.

`get_mut` geeft een mutable reference terug:

```rust,ignore
Option<&mut i32>
```

Daarom moeten we de waarde derefereren met `*`:

```rust,ignore
*aantal -= 1;
```

Het sterretje betekent hier:

> Werk met de waarde waar deze referentie naar verwijst.

Na deze code is de inventaris:

```text
Potion → 2
```

---

## 7. Een item verwijderen

Met `remove` kunnen we een sleutel en de bijbehorende waarde verwijderen:

```rust,ignore
inventaris.remove("Zwaard");
```

Daarna staat `"Zwaard"` niet meer in de `HashMap`.

`remove` geeft bovendien de verwijderde waarde terug.

Bijvoorbeeld:

```rust,ignore
if let Some(aantal) = inventaris.remove("Zwaard") {
    println!("Er zijn {aantal} zwaarden verwijderd.");
}
```

Als `"Zwaard"` niet bestond, krijgen we:

```rust,ignore
None
```

Ook hier zien we dus weer `Option`.

---

## 8. De lengte van een HashMap

Met `len()` kunnen we zien hoeveel sleutel-waardeparen erin zitten:

```rust,ignore
println!("Aantal verschillende items: {}", inventaris.len());
```

Let op:

```rust,ignore
Potion → 3
Brood  → 5
```

heeft een lengte van:

```text
2
```

Niet `8`.

`len()` telt het aantal **verschillende sleutels**, niet de som van de waarden.

---

## 9. Een HashMap doorlopen

We kunnen met een `for`-lus over een `HashMap` lopen.

```rust,ignore
for (item, aantal) in &inventaris {
    println!("{item}: {aantal}");
}
```

Hier gebruiken we `&inventaris`.

Dat betekent dat we de `HashMap` lenen.

We nemen de waarden dus niet over.

Een belangrijk detail is dat de volgorde van de elementen van een `HashMap` niet gegarandeerd is.

Je moet dus niet verwachten dat:

```text
Potion
Zwaard
Brood
```

altijd in precies die volgorde wordt afgedrukt.

---

## 10. Een HashMap aanpassen tijdens het doorlopen

Net zoals bij een `Vec` kunnen we een `HashMap` mutable lenen:

```rust,ignore
for (_, aantal) in &mut inventaris {
    *aantal += 1;
}
```

Hiermee verhogen we ieder aantal met één.

Stel:

```text
Potion → 3
Brood  → 5
```

Dan wordt het:

```text
Potion → 4
Brood  → 6
```

De `_` betekent hier dat we de sleutel niet nodig hebben.

---

## 11. Alleen de sleutels

Met `keys()` kunnen we over de sleutels lopen:

```rust,ignore
for item in inventaris.keys() {
    println!("Item: {item}");
}
```

`keys()` geeft referenties naar de sleutels.

Dit is handig als we bijvoorbeeld alleen de namen van de items willen tonen.

---

## 12. Alleen de waarden

Met `values()` kunnen we alleen de waarden bekijken:

```rust,ignore
for aantal in inventaris.values() {
    println!("Aantal: {aantal}");
}
```

Bijvoorbeeld:

```text
3
1
5
```

De bijbehorende sleutels zijn hier niet nodig.

---

## 13. Sleutels én waarden

De methode `iter()` geeft zowel sleutel als waarde:

```rust,ignore
for (item, aantal) in inventaris.iter() {
    println!("{item}: {aantal}");
}
```

Dit is functioneel vergelijkbaar met:

```rust,ignore
for (item, aantal) in &inventaris {
    println!("{item}: {aantal}");
}
```

Voorlopig is de tweede vorm vaak prettiger om te lezen.

---

## 14. Een inventaris met aantallen

Een `HashMap` is bijzonder geschikt voor een inventaris.

```rust
use std::collections::HashMap;

fn main() {
    let mut inventaris: HashMap<String, i32> = HashMap::new();

    inventaris.insert(String::from("Potion"), 3);
    inventaris.insert(String::from("Brood"), 5);
    inventaris.insert(String::from("Zwaard"), 1);

    for (item, aantal) in &inventaris {
        println!("{item}: {aantal}");
    }
}
```

Dit model is handig als we alleen willen weten hoeveel exemplaren van ieder item de speler heeft.

---

## 15. Een probleem met `insert`

Stel dat een speler een potion vindt.

We zouden misschien dit schrijven:

```rust,ignore
inventaris.insert(String::from("Potion"), 1);
```

Maar als de speler al drie potions had, worden die drie vervangen.

We krijgen dan:

```text
Potion → 1
```

terwijl we eigenlijk willen:

```text
Potion → 4
```

We moeten dus eerst controleren of de sleutel bestaat.

Dat kan, maar Rust biedt hiervoor een handigere oplossing.

---

## 16. `entry`

Met `entry` kunnen we een sleutel opzoeken en vervolgens iets doen als deze nog niet bestaat.

Bijvoorbeeld:

```rust,ignore
inventaris
    .entry(String::from("Potion"))
    .or_insert(0);
```

Dit betekent ongeveer:

> Zoek `"Potion"` op. Als deze nog niet bestaat, voeg hem toe met waarde `0`.

Stel dat we daarna een potion vinden:

```rust,ignore
let aantal = inventaris
    .entry(String::from("Potion"))
    .or_insert(0);

*aantal += 1;
```

Als er nog geen potion was:

```text
Potion → 1
```

Als er al drie waren:

```text
Potion → 4
```

Dit patroon is bijzonder nuttig.

---

## 17. Waarom `or_insert` een referentie geeft

Deze code:

```rust,ignore
let aantal = inventaris
    .entry(String::from("Potion"))
    .or_insert(0);
```

geeft een mutable reference terug naar de waarde in de `HashMap`.

Daarom kunnen we:

```rust,ignore
*aantal += 1;
```

schrijven.

Het is vergelijkbaar met:

```rust,ignore
if let Some(aantal) = inventaris.get_mut("Potion") {
    *aantal += 1;
} else {
    inventaris.insert(String::from("Potion"), 1);
}
```

`entry(...).or_insert(...)` maakt dit patroon alleen veel compacter.

---

## 18. Een functie om een item toe te voegen

We kunnen dit gedrag in een functie zetten:

```rust,ignore
use std::collections::HashMap;

fn voeg_item_toe(inventaris: &mut HashMap<String, i32>, item: &str) {
    let aantal = inventaris
        .entry(String::from(item))
        .or_insert(0);

    *aantal += 1;
}
```

We kunnen hem dan gebruiken:

```rust,ignore
fn main() {
    let mut inventaris = HashMap::new();

    voeg_item_toe(&mut inventaris, "Potion");
    voeg_item_toe(&mut inventaris, "Potion");
    voeg_item_toe(&mut inventaris, "Brood");

    for (item, aantal) in &inventaris {
        println!("{item}: {aantal}");
    }
}
```

De uitkomst bevat dan:

```text
Potion: 2
Brood: 1
```

De volgorde kan verschillen.

---

## 19. Ownership en HashMap

`HashMap` werkt natuurlijk ook samen met ownership.

Kijk naar:

```rust,ignore
let naam = String::from("Arin");

let mut levels = HashMap::new();

levels.insert(naam, 10);
```

De `String` wordt hier eigendom van de `HashMap`.

We kunnen daarna niet zomaar meer:

```rust,ignore
println!("{naam}");
```

doen.

De `String` is namelijk verplaatst naar de `HashMap`.

Dit is hetzelfde ownership-principe dat we eerder hebben geleerd.

---

## 20. Een HashMap met `String`-waarden

Hetzelfde geldt voor waarden:

```rust,ignore
let mut wapens = HashMap::new();

let naam = String::from("Vlamzwaard");
let beschrijving = String::from("Een zwaard dat vuurschade veroorzaakt.");

wapens.insert(naam, beschrijving);
```

De `HashMap` wordt eigenaar van beide `String`-waarden.

We kunnen de waarden later via de `HashMap` benaderen:

```rust,ignore
if let Some(beschrijving) = wapens.get("Vlamzwaard") {
    println!("{beschrijving}");
}
```

---

## 21. HashMap met integers als sleutel

Een sleutel hoeft geen `String` te zijn.

We kunnen bijvoorbeeld spelers een nummer geven:

```rust,ignore
let mut spelers = HashMap::new();

spelers.insert(1, "Arin");
spelers.insert(2, "Bram");
spelers.insert(3, "Cora");
```

Nu kunnen we een speler opzoeken met zijn nummer:

```rust,ignore
if let Some(naam) = spelers.get(&2) {
    println!("Speler: {naam}");
}
```

Hier gebruiken we `&2` omdat `get` een referentie naar de sleutel verwacht.

---

## 22. HashMap met structs

Een `HashMap` kan ook structs als waarden bevatten.

Bijvoorbeeld:

```rust,ignore
#[derive(Debug)]
struct Speler {
    naam: String,
    gezondheid: i32,
    level: i32,
}
```

Daarna:

```rust,ignore
let mut spelers: HashMap<String, Speler> = HashMap::new();

spelers.insert(
    String::from("Arin"),
    Speler {
        naam: String::from("Arin"),
        gezondheid: 100,
        level: 5,
    },
);
```

We kunnen de speler vervolgens ophalen:

```rust,ignore
if let Some(speler) = spelers.get("Arin") {
    println!("Level: {}", speler.level);
}
```

Omdat `get` een referentie geeft, lenen we de `Speler`.

We nemen hem niet over.

---

## 23. Een struct aanpassen via `get_mut`

We kunnen ook een speler aanpassen:

```rust,ignore
if let Some(speler) = spelers.get_mut("Arin") {
    speler.gezondheid -= 20;
}
```

Nu heeft Arin:

```text
gezondheid: 80
```

De `Speler` blijft eigendom van de `HashMap`.

We lenen hem alleen tijdelijk mutable.

---

## 24. HashMap en iterators

Omdat we inmiddels iterators kennen, kunnen we ook iterator-methodes gebruiken.

Bijvoorbeeld:

```rust,ignore
let sterke_spelers: Vec<&Speler> = spelers
    .values()
    .filter(|speler| speler.level >= 5)
    .collect();
```

Hier gebeurt het volgende:

1. `values()` geeft de spelers.
2. `filter()` houdt alleen spelers met level 5 of hoger.
3. `collect()` maakt daar een `Vec` van.

We hebben hiermee verschillende onderwerpen gecombineerd:

- `HashMap`
- references
- iterators
- closures
- `filter`
- `Vec`.

Dat is precies het soort combinatie dat in het uiteindelijke RPG-project nuttig wordt.

---

## 25. Een prijslijst

Een ander RPG-voorbeeld is een winkel:

```rust,ignore
let mut prijzen = HashMap::new();

prijzen.insert("Potion", 25);
prijzen.insert("Brood", 5);
prijzen.insert("Vlamzwaard", 150);
```

Een speler kan een item kopen:

```rust,ignore
let item = "Potion";

if let Some(prijs) = prijzen.get(item) {
    println!("{item} kost {prijs} goud.");
} else {
    println!("Dit item bestaat niet.");
}
```

We gebruiken hier opnieuw `Option`.

Dat is een belangrijk patroon:

```text
HashMap
   ↓
get()
   ↓
Option
   ↓
match / if let
```

---

## 26. Een eenvoudige winkel

We kunnen dit uitbreiden:

```rust,ignore
fn koop_item(
    goud: &mut i32,
    prijzen: &HashMap<&str, i32>,
    item: &str,
) -> bool {
    if let Some(prijs) = prijzen.get(item) {
        if *goud >= *prijs {
            *goud -= *prijs;
            return true;
        }
    }

    false
}
```

Gebruik:

```rust,ignore
let mut goud = 100;

let mut prijzen = HashMap::new();
prijzen.insert("Potion", 25);
prijzen.insert("Brood", 5);

if koop_item(&mut goud, &prijzen, "Potion") {
    println!("Aankoop gelukt!");
} else {
    println!("Je kunt dit item niet kopen.");
}

println!("Goud: {goud}");
```

Dit voorbeeld combineert inmiddels:

- functies
- references
- mutable references
- `HashMap`
- `Option`
- `if let`
- `bool`
- ownership.

---

## 27. Wanneer gebruik je een HashMap?

Een `HashMap` is vooral handig als je gegevens wilt opzoeken aan de hand van een sleutel.

Gebruik bijvoorbeeld een `Vec` wanneer je vooral een lijst hebt:

```text
vijand
vijand
vijand
vijand
```

Gebruik een `HashMap` wanneer je vooral denkt:

```text
naam → gegevens
```

Bijvoorbeeld:

```text
"Arin"  → spelergegevens
"Bram"  → spelergegevens
"Cora"  → spelergegevens
```

Of:

```text
"Potion"     → 25
"Brood"      → 5
"Vlamzwaard" → 150
```

Of:

```text
"Potion" → 4
"Brood"  → 7
"Pijl"   → 23
```

De keuze hangt dus af van hoe je de gegevens wilt gebruiken.

---

## 28. HashMap tegenover Vec

Een eenvoudig voorbeeld:

```rust,ignore
let vijanden = vec![
    "Goblin",
    "Ork",
    "Draak",
];
```

Als we willen weten welke vijand op positie `1` staat:

```rust,ignore
println!("{}", vijanden[1]);
```

Een `Vec` is gebaseerd op een positie.

Een `HashMap` is gebaseerd op een sleutel:

```rust,ignore
let mut vijanden = HashMap::new();

vijanden.insert("goblin", 20);
vijanden.insert("ork", 50);
vijanden.insert("draak", 200);
```

Nu kunnen we vragen:

```rust,ignore
if let Some(gezondheid) = vijanden.get("draak") {
    println!("De draak heeft {gezondheid} HP.");
}
```

De vraag is dus niet:

> Welke waarde staat op positie 2?

maar:

> Welke waarde hoort bij de sleutel `"draak"`?

---

## 29. Veelgemaakte fouten

### Fout 1 — `get` behandelt alsof de waarde direct terugkomt

Dit werkt niet zoals je misschien verwacht:

```rust,ignore
let aantal = inventaris.get("Potion");

println!("{aantal}");
```

Het resultaat van `get` is een `Option<&T>`.

Gebruik bijvoorbeeld:

```rust,ignore
if let Some(aantal) = inventaris.get("Potion") {
    println!("{aantal}");
}
```

---

### Fout 2 — vergeten dat `get_mut` een reference geeft

Dit:

```rust,ignore
if let Some(aantal) = inventaris.get_mut("Potion") {
    aantal += 1;
}
```

werkt niet zoals bedoeld.

Gebruik:

```rust,ignore
if let Some(aantal) = inventaris.get_mut("Potion") {
    *aantal += 1;
}
```

---

### Fout 3 — bestaande waarde per ongeluk vervangen

Dit:

```rust,ignore
inventaris.insert(String::from("Potion"), 1);
```

verhoogt het aantal niet.

Als `"Potion"` al bestaat, vervangt dit de oude waarde.

Gebruik voor een teller bijvoorbeeld:

```rust,ignore
let aantal = inventaris
    .entry(String::from("Potion"))
    .or_insert(0);

*aantal += 1;
```

---

### Fout 4 — ownership vergeten

Dit:

```rust,ignore
let naam = String::from("Arin");

spelers.insert(naam, 10);

println!("{naam}");
```

geeft een ownership-fout.

`naam` is verplaatst naar de `HashMap`.

---

## 30. HashMap combineren met het RPG-project

We kunnen nu een steeds realistischer model van onze RPG maken.

Bijvoorbeeld:

```rust,ignore
use std::collections::HashMap;

struct Speler {
    naam: String,
    gezondheid: i32,
    goud: i32,
    inventaris: HashMap<String, i32>,
}
```

Een speler heeft daarmee:

```text
naam
gezondheid
goud
inventaris
```

De inventaris bevat vervolgens:

```text
Potion → 3
Brood  → 5
Pijl   → 12
```

Een methode om een item toe te voegen kan bijvoorbeeld zijn:

```rust,ignore
impl Speler {
    fn voeg_item_toe(&mut self, item: &str) {
        let aantal = self
            .inventaris
            .entry(String::from(item))
            .or_insert(0);

        *aantal += 1;
    }
}
```

Gebruik:

```rust,ignore
let mut speler = Speler {
    naam: String::from("Arin"),
    gezondheid: 100,
    goud: 50,
    inventaris: HashMap::new(),
};

speler.voeg_item_toe("Potion");
speler.voeg_item_toe("Potion");
speler.voeg_item_toe("Brood");
```

De inventaris bevat nu:

```text
Potion → 2
Brood  → 1
```

Dit is een goed voorbeeld van hoe de afzonderlijke onderdelen van Rust steeds meer samenkomen.

---

## 31. Wat je nu moet kunnen

Na dit hoofdstuk moet je de volgende zaken begrijpen en kunnen gebruiken:

### HashMap

```rust,ignore
use std::collections::HashMap;

let mut map = HashMap::new();
```

### Waarde toevoegen of vervangen

```rust,ignore
map.insert("Potion", 3);
```

### Waarde opzoeken

```rust,ignore
map.get("Potion");
```

### Controleren of een sleutel bestaat

```rust,ignore
map.contains_key("Potion");
```

### Waarde aanpassen

```rust,ignore
if let Some(aantal) = map.get_mut("Potion") {
    *aantal += 1;
}
```

### Waarde verwijderen

```rust,ignore
map.remove("Potion");
```

### Doorlopen

```rust,ignore
for (sleutel, waarde) in &map {
    println!("{sleutel}: {waarde}");
}
```

### Alleen sleutels

```rust,ignore
for sleutel in map.keys() {
    println!("{sleutel}");
}
```

### Alleen waarden

```rust,ignore
for waarde in map.values() {
    println!("{waarde}");
}
```

### Nieuwe sleutel met standaardwaarde

```rust,ignore
let waarde = map.entry("Potion").or_insert(0);
```

### Een teller verhogen

```rust,ignore
*waarde += 1;
```

En vooral moet je het volgende patroon herkennen:

```rust,ignore
if let Some(waarde) = map.get("sleutel") {
    // waarde bestaat
}
```

Een `HashMap` en `Option` werken dus heel natuurlijk samen.

---

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 25](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_25/).

---

## Controlelijst

Je bent klaar met dit artikel als je zonder hulp:

- [ ] een `HashMap` kunt aanmaken en importeren
- [ ] waarden kunt toevoegen met `insert()`
- [ ] waarden kunt opzoeken en afhandelen via `Option` met `get()`
- [ ] de `entry().or_insert()` methode kunt gebruiken om tellers bij te houden
- [ ] elementen kunt verwijderen met `remove()`
- [ ] door een `HashMap` kunt itereren met een `for`-lus.
