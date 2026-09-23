# Rust 18 — Strings in de praktijk

Tot nu toe hebben we twee soorten tekst gezien:

```rust,ignore
let naam: String = String::from("Arin");
```

en:

```rust,ignore
let naam: &str = "Arin";
```

Beide bevatten tekst, maar ze gedragen zich op een belangrijk punt verschillend.

In dit artikel gaan we daar wat dieper op in. We leren vooral **wanneer je `String` en wanneer je `&str` gebruikt**.

Dit is belangrijk omdat je in echte Rust-programma's voortdurend met tekst werkt.

---

## 1. `String`

Een `String` is tekst die tijdens het programma kan worden opgebouwd en aangepast.

```rust,ignore
let mut naam = String::from("Arin");

naam.push_str(" de dappere");

println!("{naam}");
```

De uitvoer is:

```text
Arin de dappere
```

Omdat `naam` `mut` is, kunnen we de tekst aanpassen.

Bijvoorbeeld:

```rust,ignore
let mut naam = String::from("Arin");

naam.push_str(" de dappere");
naam.push('!');

println!("{naam}");
```

Uitvoer:

```text
Arin de dappere!
```

Een `String` is daarom handig voor tekst die tijdens het programma verandert.

---

## 2. Een `&str`

Een `&str` gebruiken we vaak voor tekst die al ergens bestaat en die we alleen willen bekijken.

Bijvoorbeeld:

```rust,ignore
let naam = "Arin";
```

Hier is `"Arin"` een `&str`.

Je kunt ook expliciet schrijven:

```rust,ignore
let naam: &str = "Arin";
```

Een string literal zoals `"Arin"` is tekst die onderdeel is van het programma zelf.

Je kunt deze tekst niet zomaar aanpassen:

```rust,ignore
let naam = "Arin";

// Dit kan niet:
// naam.push_str(" de dappere");
```

Een `&str` is dus vooral geschikt wanneer je tekst wilt **lezen**.

---

## 3. Waarom `&str` in functies?

Dit is een belangrijk patroon in Rust.

Stel dat we een functie maken die een naam afdrukt:

```rust,ignore
fn begroet(naam: &str) {
    println!("Welkom, {naam}!");
}
```

We kunnen deze functie aanroepen met een string literal:

```rust,ignore
begroet("Arin");
```

Maar ook met een `String`:

```rust,ignore
let naam = String::from("Arin");

begroet(&naam);
```

De functie hoeft namelijk alleen de tekst te lezen.

Daarom heeft de functie geen eigendom van de tekst nodig.

```text
String
   │
   │ lenen
   ▼
 &str
```

Je kunt dit zien als:

> `String` bezit de tekst; `&str` kijkt naar tekst zonder die te bezitten.

Dit sluit direct aan op wat we eerder over borrowing hebben geleerd.

---

## 4. `String` doorgeven

We kunnen ook een `String` rechtstreeks aan een functie doorgeven:

```rust,ignore
fn begroet(naam: String) {
    println!("Welkom, {naam}!");
}
```

Dan wordt de `String` eigendom van de functie.

Bijvoorbeeld:

```rust,ignore
let naam = String::from("Arin");

begroet(naam);
```

Na deze aanroep kunnen we `naam` niet meer gebruiken.

```rust,ignore
let naam = String::from("Arin");

begroet(naam);

println!("{naam}"); // fout
```

De functie heeft de `String` namelijk overgenomen.

Als de functie alleen maar hoeft te lezen, is `&str` meestal een handiger parameter:

```rust,ignore
fn begroet(naam: &str) {
    println!("Welkom, {naam}!");
}
```

Dan blijft de oorspronkelijke `String` gewoon bestaan.

```rust,ignore
let naam = String::from("Arin");

begroet(&naam);

println!("{naam}");
```

---

## 5. Een `String` maken

Er zijn verschillende manieren om een `String` te maken.

De meest bekende:

```rust,ignore
let naam = String::from("Arin");
```

Je kunt ook beginnen met een lege `String`:

```rust,ignore
let mut tekst = String::new();

tekst.push_str("Welkom ");
tekst.push_str("in het spel!");

println!("{tekst}");
```

Uitvoer:

```text
Welkom in het spel!
```

Dit is handig wanneer je tekst stap voor stap wilt opbouwen.

---

## 6. Tekst toevoegen

Met `push_str` voegen we meerdere tekens toe:

```rust,ignore
let mut bericht = String::from("Hallo");

bericht.push_str(" Arin");

println!("{bericht}");
```

Met `push` voegen we één `char` toe:

```rust,ignore
let mut bericht = String::from("Hallo");

bericht.push('!');

println!("{bericht}");
```

Let op het verschil:

```rust,ignore
push_str("!")
```

tegenover:

```rust,ignore
push('!')
```

Dubbele aanhalingstekens geven een string slice:

```rust,ignore
"!"
```

Enkele aanhalingstekens geven één `char`:

```rust,ignore
'!'
```

---

## 7. Een `String` en een `&str` combineren

Een veelvoorkomende situatie is dat een programma vaste tekst combineert met tekst die tijdens het programma is verzameld.

Bijvoorbeeld:

```rust,ignore
let naam = "Arin";

let mut bericht = String::from("Welkom, ");
bericht.push_str(naam);
bericht.push('!');

println!("{bericht}");
```

Hier is:

```rust,ignore
naam
```

een `&str`.

En:

```rust,ignore
bericht
```

is een `String`.

---

## 8. Een naam van een speler

Laten we dit toepassen op ons RPG-project.

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
}
```

We kunnen een speler maken:

```rust,ignore
let speler = Speler {
    naam: String::from("Arin"),
    gezondheid: 100,
};
```

De naam is een `String`, omdat de spelernaam onderdeel is van de gegevens van de speler.

We kunnen vervolgens een functie maken die de spelernaam alleen leest:

```rust,ignore
fn toon_naam(speler: &Speler) {
    println!("Speler: {}", speler.naam);
}
```

Of een functie die alleen een stukje tekst nodig heeft:

```rust,ignore
fn begroet(naam: &str) {
    println!("Welkom, {naam}!");
}
```

We kunnen die functie gebruiken met de naam van onze speler:

```rust,ignore
begroet(&speler.naam);
```

---

## 9. Waarom niet overal `String`?

Je zou kunnen denken:

> Waarom gebruiken we niet gewoon overal `String`?

Dat kan soms, maar dan geef je functies vaak meer eigendom dan nodig is.

Stel:

```rust,ignore
fn toon_naam(naam: String) {
    println!("{naam}");
}
```

Dan neemt de functie de `String` over.

Dit betekent dat je bijvoorbeeld dit niet meer kunt doen:

```rust,ignore
let naam = String::from("Arin");

toon_naam(naam);

println!("{naam}");
```

Als de functie alleen hoeft te kijken naar de tekst, is dit logischer:

```rust,ignore
fn toon_naam(naam: &str) {
    println!("{naam}");
}
```

En:

```rust,ignore
let naam = String::from("Arin");

toon_naam(&naam);

println!("{naam}");
```

De functie leent de tekst alleen.

---

## 10. `&String` versus `&str`

Bij borrowing hebben we eerder een functie gezien zoals:

```rust,ignore
fn toon_naam(naam: &String) {
    println!("{naam}");
}
```

Dat werkt.

Maar vaak schrijven we liever:

```rust,ignore
fn toon_naam(naam: &str) {
    println!("{naam}");
}
```

Waarom?

Omdat `&str` zegt:

> Ik heb alleen toegang tot tekst nodig.

De functie hoeft niet te weten of die tekst afkomstig is van een `String` of bijvoorbeeld rechtstreeks van een string literal.

Dus dit werkt:

```rust,ignore
fn toon_naam(naam: &str) {
    println!("{naam}");
}

let naam = String::from("Arin");

toon_naam(&naam);
toon_naam("Borin");
```

Beide aanroepen zijn mogelijk.

---

## 11. Een functie die een `String` teruggeeft

Soms moet een functie juist nieuwe tekst maken.

Dan is `String` geschikt als returntype.

```rust,ignore
fn maak_groet(naam: &str) -> String {
    let mut bericht = String::from("Welkom, ");
    bericht.push_str(naam);
    bericht.push('!');

    bericht
}
```

We kunnen dit gebruiken:

```rust,ignore
let groet = maak_groet("Arin");

println!("{groet}");
```

De functie maakt een nieuwe `String` en geeft die terug.

Dit past mooi bij wat we eerder over ownership hebben geleerd.

De functie maakt de `String` en geeft het eigendom ervan terug aan de aanroeper.

---

## 12. Ownership en strings

Dit is een goed moment om de belangrijkste regels nog eens naast elkaar te zetten.

### `String` als parameter

```rust,ignore
fn gebruik(tekst: String) {
    println!("{tekst}");
}
```

De functie krijgt eigendom.

---

### `&String` als parameter

```rust,ignore
fn gebruik(tekst: &String) {
    println!("{tekst}");
}
```

De functie leent een `String`.

---

### `&str` als parameter

```rust,ignore
fn gebruik(tekst: &str) {
    println!("{tekst}");
}
```

De functie leent tekst.

Voor functies die alleen tekst hoeven te lezen, is `&str` vaak de meest algemene keuze.

---

### `String` als returnwaarde

```rust,ignore
fn maak_tekst() -> String {
    String::from("Hallo!")
}
```

De functie geeft eigendom van de nieuwe `String` terug.

---

## 13. Strings in ons RPG-project

We kunnen dit combineren met onze bestaande kennis.

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
}

impl Speler {
    fn begroet(&self) {
        println!("Welkom, {}!", self.naam);
    }

    fn naam(&self) -> &str {
        &self.naam
    }
}
```

De methode `naam` geeft geen nieuwe `String` terug.

Hij geeft een referentie naar de bestaande tekst:

```rust,ignore
&str
```

We kunnen dan schrijven:

```rust,ignore
let speler = Speler {
    naam: String::from("Arin"),
    gezondheid: 100,
};

println!("De speler heet {}.", speler.naam());
```

Er wordt hierbij geen nieuwe naam gekopieerd.

---

## 14. Strings vergelijken

Tekst kan worden vergeleken met `==`:

```rust,ignore
let klasse = "Tovenaar";

if klasse == "Tovenaar" {
    println!("Je kunt magie gebruiken!");
}
```

Ook met `String` werkt dit:

```rust,ignore
let klasse = String::from("Tovenaar");

if klasse == "Tovenaar" {
    println!("Je kunt magie gebruiken!");
}
```

Dit kan bijvoorbeeld gebruikt worden bij keuzes in een RPG.

---

## 15. Een naam controleren

We kunnen een functie maken:

```rust,ignore
fn is_geldige_naam(naam: &str) -> bool {
    !naam.is_empty()
}
```

Daarmee kunnen we controleren of een speler een naam heeft ingevuld:

```rust,ignore
let naam = "Arin";

if is_geldige_naam(naam) {
    println!("Naam geaccepteerd.");
} else {
    println!("Je moet een naam invullen.");
}
```

Hier zien we opnieuw waarom `&str` handig is: de functie hoeft de naam alleen te bekijken.

---

## 16. Veelgebruikte String-functies

Er zijn veel functies voor tekst. We hoeven ze nog lang niet allemaal te kennen.

Een paar nuttige voorbeelden:

```rust,ignore
let naam = String::from("Arin");
```

Lengte:

```rust,ignore
println!("{}", naam.len());
```

Controleren of tekst leeg is:

```rust,ignore
naam.is_empty()
```

Controleren of tekst bepaalde tekst bevat:

```rust,ignore
naam.contains("Ar")
```

Tekst toevoegen:

```rust,ignore
tekst.push_str("Hallo");
```

Eén teken toevoegen:

```rust,ignore
tekst.push('!');
```

Tekst verwijderen of aanpassen kan ook, maar dat behandelen we voorlopig niet uitgebreid.

Het belangrijkste is dat je weet dat Rust hiervoor functies en methoden heeft. Je hoeft ze niet allemaal uit je hoofd te kennen.

---

## 17. Een belangrijk onderscheid

Onthoud vooral deze drie situaties:

```text
String
│
├── bezit tekst
├── kan groeien en veranderen
└── kan worden doorgegeven of geleend

&String
│
└── leent specifiek een String

&str
│
└── leent tekst
```

Voor een functie die alleen tekst leest, zie je daarom vaak:

```rust,ignore
fn toon_naam(naam: &str)
```

Voor een functie die nieuwe tekst moet maken:

```rust,ignore
fn maak_naam() -> String
```

En voor gegevens die een speler daadwerkelijk bezit:

```rust,ignore
struct Speler {
    naam: String,
}
```

---

## 18. Rustlings

Maak voor dit onderwerp een nieuwe map:

```text
exercises/strings/
```

De oefeningen moeten de leerlingen niet alleen `String`-methoden laten opzoeken, maar vooral laten begrijpen **waarom** `String` en `&str` verschillend worden gebruikt.

## 01_string.rs

Maak een `String` met de naam van een speler en druk deze af.

---

## 02_mut_string.rs

Maak een veranderbare `String` en voeg met `push_str` een achternaam toe.

---

## 03_push_char.rs

Voeg met `push` een uitroepteken toe aan een `String`.

---

## 04_string_new.rs

Begin met een lege `String` en bouw hiermee de tekst:

```text
Welkom in het spel!
```

---

## 05_string_function.rs

Maak:

```rust,ignore
fn toon_naam(naam: String)
```

en roep deze aan met een `String`.

---

## 06_debug_move.rs

De volgende code compileert niet:

```rust,ignore
fn toon_naam(naam: String) {
    println!("{naam}");
}

fn main() {
    let naam = String::from("Arin");

    toon_naam(naam);

    println!("{naam}");
}
```

Onderzoek de compilerfout.

Los het probleem op door de functie aan te passen zodat de functie de tekst alleen leent.

---

## 07_str_parameter.rs

Maak:

```rust,ignore
fn begroet(naam: &str)
```

De functie moet:

```text
Welkom, Arin!
```

afdrukken.

Test de functie met een string literal.

---

## 08_string_as_str.rs

Gebruik dezelfde functie uit de vorige oefening, maar geef nu een `String` door.

Hint:

```rust,ignore
let naam = String::from("Arin");
```

De functie verwacht:

```rust,ignore
&str
```

---

## 09_compare.rs

Maak een functie:

```rust,ignore
fn is_tovenaar(klasse: &str) -> bool
```

De functie moet `true` teruggeven als de klasse `"Tovenaar"` is.

---

## 10_valid_name.rs

Maak:

```rust,ignore
fn is_geldige_naam(naam: &str) -> bool
```

Een lege naam is ongeldig.

---

## 11_make_greeting.rs

Maak:

```rust,ignore
fn maak_groet(naam: &str) -> String
```

De functie moet bijvoorbeeld:

```text
Welkom, Arin!
```

teruggeven.

---

## 12_player_name.rs

Gebruik:

```rust,ignore
struct Speler {
    naam: String,
}
```

Maak een speler en schrijf een functie:

```rust,ignore
fn toon_speler(speler: &Speler)
```

die de naam afdrukt.

---

## 13_player_name_str.rs

Voeg aan `Speler` een methode toe:

```rust,ignore
fn naam(&self) -> &str
```

De methode moet de naam van de speler teruggeven zonder een nieuwe `String` te maken.

---

## 14_string_debug.rs

De volgende code geeft een compilerfout:

```rust,ignore
fn toon_naam(naam: String) {
    println!("{naam}");
}

fn main() {
    let naam = String::from("Arin");

    toon_naam(naam);

    println!("Speler: {naam}");
}
```

Pas de code aan zodat beide `println!`-regels werken.

Gebruik hiervoor borrowing.

---

## 15_game_greeting.rs

Maak:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
}
```

Maak vervolgens een methode:

```rust,ignore
fn begroet(&self)
```

die bijvoorbeeld afdrukt:

```text
Welkom, Arin!
Je hebt 100 levenspunten.
```

---

## 16_string_or_str.rs

Maak twee functies:

```rust,ignore
fn toon_tekst(tekst: &str)
fn maak_tekst() -> String
```

De eerste functie moet bestaande tekst afdrukken.

De tweede functie moet nieuwe tekst maken en teruggeven.

Test beide functies.

---

## 17_debug_wrong_type.rs

De volgende code is expres fout:

```rust,ignore
fn begroet(naam: &str) {
    println!("Welkom, {naam}!");
}

fn main() {
    let naam = String::from("Arin");

    begroet(naam);
}
```

Gebruik de compilerfout om uit te zoeken waarom dit niet werkt.

Los het op zonder de `String` te verplaatsen.

---

## 18_final_game.rs

Maak een klein RPG-programma met:

```rust,ignore
struct Speler {
    naam: String,
    klasse: String,
    gezondheid: i32,
}
```

Maak daarnaast:

```rust,ignore
fn begroet(speler: &Speler)
fn is_geldige_naam(naam: &str) -> bool
```

Het programma moet:

1. een speler maken
2. controleren of de naam geldig is
3. de speler begroeten
4. de klasse en gezondheid afdrukken.

Gebruik daarbij bewust zowel `String` als `&str`.

---

## 19. Wat moet je na dit artikel kunnen?

Na dit artikel hoef je niet alle String-methoden te kennen.

Je moet vooral begrijpen:

- wat een `String` is
- wat een `&str` is
- waarom een `String` kan veranderen
- waarom een `&str` vaak wordt gebruikt om tekst te lenen
- waarom functies die alleen tekst lezen vaak `&str` gebruiken
- hoe een `String` door ownership van eigenaar kan veranderen
- hoe je een `String` kunt lenen
- hoe je een nieuwe `String` vanuit een functie kunt teruggeven.

Het belangrijkste patroon om te onthouden is:

```rust,ignore
fn toon(tekst: &str) {
    println!("{tekst}");
}
```

en:

```rust,ignore
fn maak_tekst() -> String {
    String::from("Hallo!")
}
```

Je hoeft nog niet precies te weten hoe Rust strings intern in het geheugen opslaat. Het gaat er voorlopig om dat je weet **wie de tekst bezit en of een functie de tekst bezit of alleen leent**.

Dat sluit direct aan op ownership en borrowing en vormt een goede basis voor de volgende stap: **modules en zichtbaarheid**.

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 18](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_18/).

