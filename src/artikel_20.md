# 20. Option

## Wat gaan we leren?

In dit artikel leren we hoe Rust omgaat met waarden die er misschien wel zijn, maar misschien ook niet, met behulp van `Option<T>`.

We leren:

- wat `Option<T>` is en wat `Some` en `None` betekenen
- waarom Rust geen `null` heeft en hoe `Option` crashes voorkomt
- `Option` uitpakken met `match` en `if let`
- handige methods zoals `is_some()`, `is_none()`, `unwrap()` en `unwrap_or()`
- hoe je optionele velden in structs gebruikt (zoals een optioneel wapen)
- functies schrijven die een `Option` teruggeven.

---

## 1. Wat is `Option<T>`?

`Option<T>` betekent:

> Er is een waarde van type `T`, of er is geen waarde.

`T` is hier een plaatsaanduiding voor een datatype.

Een `Option<i32>` kan bijvoorbeeld zijn:

```rust,ignore
Some(100)
```

of:

```rust,ignore
None
```

Een `Option<String>` kan zijn:

```rust,ignore
Some(String::from("Zwaard"))
```

of:

```rust,ignore
None
```

De twee mogelijkheden zijn dus:

```text
Option<T>
├── Some(waarde)
└── None
```

---

## 2. `Some`

Als er wel een waarde is, gebruiken we `Some`.

```rust,ignore
let gezondheid: Option<i32> = Some(100);
```

De waarde `100` zit nu binnen een `Option`.

Bij een string:

```rust,ignore
let wapen: Option<String> =
    Some(String::from("Zwaard"));
```

---

## 3. `None`

Als er geen waarde is:

```rust,ignore
let wapen: Option<String> = None;
```

We weten dus dat `wapen` van het type `Option<String>` is, maar er zit op dit moment geen wapen in.

Bij sommige gevallen kan Rust het type zelf afleiden:

```rust,ignore
let wapen: Option<String> = None;
```

Bij `None` is het type vaak nodig omdat `None` zelf niet vertelt **waarvan** er geen waarde is.

---

## 4. Waarom niet gewoon `String`?

Stel dat een speler geen wapen heeft.

We zouden bijvoorbeeld een lege string kunnen gebruiken:

```rust,ignore
let wapen = String::from("");
```

Maar dan betekent een lege string ineens:

> Er is geen wapen.

Dat is verwarrend.

Een `Option<String>` maakt het verschil expliciet:

```rust,ignore
let wapen: Option<String> = None;
```

Er is daadwerkelijk geen wapen.

En:

```rust,ignore
let wapen = Some(String::from("Zwaard"));
```

Er is wel een wapen.

---

## 5. `match`

De meest directe manier om met een `Option` om te gaan is `match`.

```rust,ignore
let wapen: Option<String> =
    Some(String::from("Zwaard"));

match wapen {
    Some(naam) => {
        println!("Wapen: {naam}");
    }
    None => {
        println!("Geen wapen.");
    }
}
```

De `match` behandelt beide mogelijkheden.

Als er een waarde is:

```rust,ignore
Some(naam)
```

dan krijgen we die waarde beschikbaar als `naam`.

Als er geen waarde is:

```rust,ignore
None
```

dan voeren we de andere tak uit.

---

## 6. `match` met een getal

Hetzelfde werkt met andere datatypes.

```rust,ignore
let score: Option<i32> = Some(100);

match score {
    Some(waarde) => println!("Score: {waarde}"),
    None => println!("Geen score."),
}
```

---

## 7. `None`

Bijvoorbeeld:

```rust,ignore
let score: Option<i32> = None;

match score {
    Some(waarde) => println!("Score: {waarde}"),
    None => println!("Er is geen score."),
}
```

Hier wordt de tweede tak uitgevoerd.

---

## 8. `Option` als returnwaarde

Een van de belangrijkste toepassingen is een functie die misschien niets kan vinden.

Stel dat we een speler zoeken:

```rust,ignore
fn vind_speler(naam: &str) -> Option<String> {
    if naam == "Arin" {
        Some(String::from("Arin"))
    } else {
        None
    }
}
```

We kunnen dit gebruiken:

```rust,ignore
let resultaat = vind_speler("Arin");

match resultaat {
    Some(naam) => println!("Gevonden: {naam}"),
    None => println!("Speler niet gevonden."),
}
```

De functie kan dus niet alleen een speler teruggeven.

Hij kan ook aangeven:

> Ik heb niets gevonden.

---

## 9. `Option` in een struct

Dit is bijzonder nuttig voor ons RPG.

Een speler kan bijvoorbeeld een wapen hebben, maar dat hoeft niet.

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
    wapen: Option<String>,
}
```

Een speler zonder wapen:

```rust,ignore
let speler = Speler {
    naam: String::from("Arin"),
    gezondheid: 100,
    wapen: None,
};
```

Een speler met een zwaard:

```rust,ignore
let speler = Speler {
    naam: String::from("Arin"),
    gezondheid: 100,
    wapen: Some(String::from("Zwaard")),
};
```

Nu is de toestand van de speler duidelijk.

---

## 10. Een wapen controleren

We kunnen `match` gebruiken:

```rust,ignore
match &speler.wapen {
    Some(naam) => println!("Wapen: {naam}"),
    None => println!("De speler heeft geen wapen."),
}
```

Let goed op de `&`.

We willen het wapen alleen bekijken.

We willen het niet uit de speler halen.

Dit sluit aan op borrowing.

---

## 11. `Option` en ownership

Stel:

```rust,ignore
let wapen = Some(String::from("Zwaard"));
```

De `String` zit binnen de `Option`.

Als we dit doen:

```rust,ignore
match wapen {
    Some(naam) => println!("{naam}"),
    None => println!("Geen wapen"),
}
```

wordt de waarde uit `wapen` verplaatst naar `naam`.

Daarom kunnen we `wapen` daarna niet zomaar opnieuw gebruiken.

Als we alleen willen kijken, kunnen we lenen:

```rust,ignore
match &wapen {
    Some(naam) => println!("{naam}"),
    None => println!("Geen wapen"),
}
```

Dan blijft `wapen` bestaan.

Dit is een belangrijk voorbeeld van hoe `Option` samenwerkt met ownership en borrowing.

---

## 12. `Option<&T>`

Een `Option` kan ook een referentie bevatten.

Bijvoorbeeld:

```rust,ignore
fn vind_wapen<'a>(wapens: &'a [Wapen], naam: &str) -> Option<&'a Wapen> {
    // ...
}
```

Deze code bevat een lifetime.

**Die gaan we hier nog niet behandelen.**

Voorlopig is het voldoende om het patroon te herkennen:

```text
Option<T>
```

kan ook gebruikt worden voor een referentie.

Bijvoorbeeld conceptueel:

```rust,ignore
Option<&Wapen>
```

betekent:

> Misschien is er een referentie naar een `Wapen`.

De precieze lifetime-syntax bewaren we voor later, als we die überhaupt in deze cursus nodig hebben.

---

## 13. `if let`

Wanneer we maar één van de twee mogelijkheden interessant vinden, kan `if let` handiger zijn dan `match`.

Bijvoorbeeld:

```rust,ignore
let wapen = Some(String::from("Zwaard"));

if let Some(naam) = wapen {
    println!("Je hebt een {naam}.");
}
```

We zeggen hiermee eigenlijk:

> Als dit `Some` is, doe dan dit.

Als het `None` is, gebeurt er niets.

---

## 14. `if let` met `None`

We kunnen ook specifiek controleren op `None`:

```rust,ignore
let wapen: Option<String> = None;

if let None = wapen {
    println!("Je hebt geen wapen.");
}
```

Maar meestal is `match` duidelijker wanneer we beide mogelijkheden willen behandelen.

---

## 15. `is_some()` en `is_none()`

Soms willen we alleen weten **of** er een waarde is.

Dan kunnen we gebruiken:

```rust,ignore
let wapen = Some("Zwaard");

if wapen.is_some() {
    println!("De speler heeft een wapen.");
}
```

Of:

```rust,ignore
if wapen.is_none() {
    println!("De speler heeft geen wapen.");
}
```

Dit geeft alleen informatie over het bestaan van een waarde.

Het haalt de waarde niet automatisch voor ons tevoorschijn.

---

## 16. `unwrap()`

Er bestaat ook:

```rust,ignore
let waarde = Some(100);

let getal = waarde.unwrap();

println!("{getal}");
```

`unwrap()` haalt de waarde uit `Some`.

Maar er is een probleem.

Als we dit doen:

```rust,ignore
let waarde: Option<i32> = None;

let getal = waarde.unwrap();
```

dan stopt het programma met een panic.

Voor productiecode willen we daarom meestal niet zomaar `unwrap()` gebruiken.

Voor eenvoudige oefeningen is het soms handig om het gedrag te leren kennen.

---

## 17. `unwrap_or()`

Een veiliger alternatief voor sommige situaties is:

```rust,ignore
let gezondheid: Option<i32> = None;

let waarde = gezondheid.unwrap_or(100);

println!("{waarde}");
```

Als er een waarde is, wordt die gebruikt.

Als de waarde `None` is, wordt `100` gebruikt.

Bijvoorbeeld:

```rust,ignore
let gezondheid = Some(75);

let waarde = gezondheid.unwrap_or(100);

println!("{waarde}");
```

Hier wordt `75` gebruikt.

---

## 18. Een RPG-voorbeeld

We kunnen dit allemaal combineren.

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
    wapen: Option<String>,
}
```

Een speler:

```rust,ignore
let speler = Speler {
    naam: String::from("Arin"),
    gezondheid: 100,
    wapen: Some(String::from("Zwaard")),
};
```

We kunnen het wapen tonen:

```rust,ignore
match &speler.wapen {
    Some(wapen) => println!("Arin gebruikt een {wapen}."),
    None => println!("Arin heeft geen wapen."),
}
```

Een andere speler:

```rust,ignore
let speler = Speler {
    naam: String::from("Borin"),
    gezondheid: 100,
    wapen: None,
};
```

Geeft:

```text
Borin heeft geen wapen.
```

---

## 19. Een item zoeken

Stel dat we een eenvoudige inventory hebben:

```rust,ignore
struct Item {
    naam: String,
}
```

We kunnen een functie maken die een item zoekt:

```rust,ignore
fn vind_item(items: &[Item], gezocht: &str) -> Option<&Item> {
    for item in items {
        if item.naam == gezocht {
            return Some(item);
        }
    }

    None
}
```

Hier gebeurt iets belangrijks:

- als we het item vinden, geven we `Some(item)` terug
- als we het einde bereiken zonder resultaat, geven we `None` terug.

We gebruiken hier een slice:

```rust,ignore
&[Item]
```

Dat is nieuw.

Je hoeft dit nog niet uitgebreid te bestuderen. Voorlopig is het voldoende om te begrijpen dat we een lijst kunnen lenen zonder dat de functie eigenaar wordt.

We kunnen de functie bijvoorbeeld aanroepen met een `Vec<Item>`:

```rust,ignore
let items = vec![
    Item {
        naam: String::from("Potion"),
    },
    Item {
        naam: String::from("Zwaard"),
    },
];

match vind_item(&items, "Zwaard") {
    Some(item) => println!("Gevonden: {}", item.naam),
    None => println!("Item niet gevonden."),
}
```

---

## 20. Waarom `Option` zo belangrijk is

`Option` voorkomt dat we een speciale waarde moeten verzinnen voor:

> Er is niets.

Zonder `Option` zouden we bijvoorbeeld kunnen afspreken:

```text
gezondheid = -1
```

betekent:

> Geen speler gevonden.

Maar dan is `-1` ineens twee dingen:

- een getal
- een speciale betekenis.

Met:

```rust,ignore
Option<i32>
```

maken we dat verschil expliciet:

```text
Some(100)
```

betekent:

> Er is een waarde: 100.

En:

```text
None
```

betekent:

> Er is geen waarde.

Dat maakt programma's duidelijker.

---

## 21. Belangrijkste patronen

Je zult deze patronen vaak tegenkomen.

### Een waarde maken

```rust,ignore
let waarde = Some(100);
```

### Geen waarde

```rust,ignore
let waarde: Option<i32> = None;
```

### Beide gevallen behandelen

```rust,ignore
match waarde {
    Some(x) => println!("{x}"),
    None => println!("Geen waarde"),
}
```

### Alleen `Some` behandelen

```rust,ignore
if let Some(x) = waarde {
    println!("{x}");
}
```

### Alleen controleren

```rust,ignore
waarde.is_some()
```

of:

```rust,ignore
waarde.is_none()
```

### Een standaardwaarde gebruiken

```rust,ignore
waarde.unwrap_or(100)
```

---

## 22. Zelf oefenen met Option

### Oefening 1 — Optioneel wapen

Voeg aan je bestaande `Speler` een optioneel wapen toe.

Zorg ervoor dat een speler zonder wapen correct wordt weergegeven.

---

### Oefening 2 — Item zoeken

Maak een inventory met minstens vijf items.

Maak een functie die een item op naam zoekt.

Test zowel:

- een item dat bestaat
- een item dat niet bestaat.

---

### Oefening 3 — Optionele quest

Maak:

```rust,ignore
quest: Option<String>
```

Een speler kan daardoor wel of geen actieve quest hebben.

Toon een passende melding voor beide situaties.

---

### Oefening 4 — Optionele beloning

Maak een functie die soms een beloning geeft:

```rust,ignore
fn beloning(gewonnen: bool) -> Option<String>
```

Bij winst:

```text
Some("Gouden zwaard")
```

Bij verlies:

```text
None
```

---

### Oefening 5 — Tests

Schrijf tests voor:

- een speler met een wapen
- een speler zonder wapen
- een gevonden item
- een niet-gevonden item.

---

## 23. Samenvatting en vooruitblik

Na dit artikel moet je begrijpen:

- wat `Option<T>` betekent
- wat `Some(...)` betekent
- wat `None` betekent
- waarom `Option` nuttiger is dan speciale waarden zoals `-1`
- hoe je `Option` met `match` behandelt
- hoe `if let` werkt
- hoe `is_some()` en `is_none()` werken
- wat `unwrap()` en `unwrap_or()` doen
- hoe `Option` samenwerkt met ownership en borrowing
- hoe je een functie schrijft die mogelijk geen resultaat heeft.

Het belangrijkste mentale model is:

```text
Option<T>

    ┌───────────────┐
    │               │
    ▼               ▼
 Some(waarde)      None
    │               │
    │               │
 er is een waarde   er is geen waarde
```

Rust dwingt je hiermee om expliciet na te denken over de situatie waarin een waarde ontbreekt.

Dat is precies wat we nodig hebben voor het volgende onderwerp.

Want niet alleen waarden kunnen ontbreken.

Een actie kan ook **lukken of mislukken**.

Daarvoor gebruiken we `Result<T, E>`.

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 20](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_20/).

---

## Controlelijst

Je bent klaar met dit artikel als je zonder hulp:

- [ ] weet wat `Option<T>` is en wat `Some` en `None` voorstellen
- [ ] een `Option` veilig kunt afhandelen met `match` of `if let`
- [ ] weet waarom `unwrap()` gevaarlijk kan zijn en wanneer je `unwrap_or()` gebruikt
- [ ] een optioneel veld kunt toevoegen aan een struct
- [ ] een zoekfunctie kunt schrijven die `Option` teruggeeft.
