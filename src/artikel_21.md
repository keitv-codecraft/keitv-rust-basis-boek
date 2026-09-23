# Rust 21 — `Result<T, E>` en foutafhandeling

Tot nu toe hebben we geleerd dat een functie met `Option<T>` kan aangeven:

> Er is een waarde, of er is geen waarde.

Bijvoorbeeld:

```rust,ignore
fn vind_wapen(naam: &str) -> Option<String> {
    if naam == "Zwaard" {
        Some(String::from("Zwaard"))
    } else {
        None
    }
}
```

Maar soms is `None` niet genoeg.

Stel dat we een savegame proberen te laden.

Er zijn verschillende redenen waarom dat kan mislukken:

- het bestand bestaat niet
- het bestand kan niet worden gelezen
- de inhoud is ongeldig
- een getal kan niet worden omgezet
- de savegame is beschadigd.

Dan willen we niet alleen weten:

> Het is niet gelukt.

We willen ook weten **waarom**.

Daarvoor gebruikt Rust `Result<T, E>`.

---

## 1. Wat is `Result<T, E>`?

Een `Result` heeft twee mogelijkheden:

```text id="g3m6rj"
Result<T, E>
├── Ok(waarde)
└── Err(fout)
```

`T` staat voor het type van een succesvolle waarde.

`E` staat voor het type van de fout.

Bijvoorbeeld:

```rust,ignore
let resultaat: Result<i32, String> = Ok(100);
```

Dit betekent:

> De operatie is gelukt en leverde `100` op.

Of:

```rust,ignore
let resultaat: Result<i32, String> =
    Err(String::from("Er ging iets mis."));
```

Dit betekent:

> De operatie is mislukt en de fout is een `String`.

---

## 2. `Ok`

Wanneer een operatie succesvol is:

```rust,ignore
let resultaat = Ok(100);
```

Bij een expliciete typeaanduiding:

```rust,ignore
let resultaat: Result<i32, String> = Ok(100);
```

We kunnen het resultaat met `match` bekijken:

```rust,ignore
match resultaat {
    Ok(waarde) => println!("Gelukt: {waarde}"),
    Err(fout) => println!("Mislukt: {fout}"),
}
```

---

## 3. `Err`

Wanneer iets mislukt:

```rust,ignore
let resultaat: Result<i32, String> =
    Err(String::from("Geen savegame gevonden."));
```

Met `match`:

```rust,ignore
match resultaat {
    Ok(waarde) => println!("Geladen: {waarde}"),
    Err(fout) => println!("Fout: {fout}"),
}
```

---

## 4. `Result` als returnwaarde

Een functie kan een `Result` teruggeven.

Bijvoorbeeld:

```rust,ignore
fn deel(a: i32, b: i32) -> Result<i32, String> {
    if b == 0 {
        Err(String::from("Delen door nul is niet toegestaan."))
    } else {
        Ok(a / b)
    }
}
```

Bij een geldige berekening:

```rust,ignore
let resultaat = deel(10, 2);
```

krijgen we:

```text id="m25g4f"
Ok(5)
```

Bij:

```rust,ignore
let resultaat = deel(10, 0);
```

krijgen we:

```text id="f5z4jx"
Err("Delen door nul is niet toegestaan.")
```

---

## 5. `match`

De meest expliciete manier om een `Result` te behandelen is `match`.

```rust,ignore
match deel(10, 2) {
    Ok(waarde) => println!("Uitkomst: {waarde}"),
    Err(fout) => println!("Fout: {fout}"),
}
```

Dit lijkt sterk op wat we bij `Option` hebben gedaan.

Bij `Option`:

```text id="8qz6hk"
Some
None
```

Bij `Result`:

```text id="7f6k2s"
Ok
Err
```

Het verschil is belangrijk:

```text id="v0w6q4"
Option<T>
    waarde
    of niets

Result<T, E>
    succes
    of fout
```

---

## 6. Een savegame laden

Dit is een goed voorbeeld voor ons RPG-project.

Stel dat we een savegame uit een bestand willen lezen.

```rust,ignore
use std::fs;

fn laad_savegame() -> Result<String, String> {
    match fs::read_to_string("savegame.txt") {
        Ok(inhoud) => Ok(inhoud),
        Err(_) => Err(String::from("Savegame kon niet worden geladen.")),
    }
}
```

De functie geeft dus:

```text id="2s2j8v"
Ok(inhoud)
```

als het lezen lukt.

Of:

```text id="z6x3cs"
Err(...)
```

als het niet lukt.

---

## 7. De fout niet weggooien

In het vorige voorbeeld deden we:

```rust,ignore
Err(_)
```

De `_` betekent dat we de concrete fout niet gebruiken.

Maar vaak willen we juist weten wat er misging.

We kunnen de fout bijvoorbeeld doorgeven:

```rust,ignore
fn laad_savegame() -> Result<String, String> {
    match std::fs::read_to_string("savegame.txt") {
        Ok(inhoud) => Ok(inhoud),
        Err(fout) => Err(fout.to_string()),
    }
}
```

Hiermee wordt de oorspronkelijke fout omgezet naar een `String`.

---

## 8. Een `match` binnen een `match`

We kunnen de functie gebruiken:

```rust,ignore
match laad_savegame() {
    Ok(inhoud) => {
        println!("Savegame geladen:");
        println!("{inhoud}");
    }
    Err(fout) => {
        println!("Laden mislukt: {fout}");
    }
}
```

De fout wordt hier netjes afgehandeld.

Het programma hoeft niet zomaar te stoppen.

---

## 9. `unwrap()`

Net als bij `Option` bestaat er bij `Result` een `unwrap()`.

```rust,ignore
let resultaat: Result<i32, String> = Ok(100);

let waarde = resultaat.unwrap();

println!("{waarde}");
```

Bij `Ok(100)` krijgen we:

```text id="x7d1wz"
100
```

Maar:

```rust,ignore
let resultaat: Result<i32, String> =
    Err(String::from("Fout!"));

let waarde = resultaat.unwrap();
```

veroorzaakt een panic.

Gebruik `unwrap()` daarom niet als algemene manier om fouten af te handelen.

Het is soms handig in eenvoudige voorbeelden of wanneer je **zeker weet** dat iets niet kan mislukken, maar voor normale foutafhandeling is `match` vaak duidelijker.

---

## 10. `unwrap_or()`

Net als bij `Option` kunnen we een standaardwaarde gebruiken:

```rust,ignore
let resultaat: Result<i32, String> =
    Err(String::from("Fout!"));

let waarde = resultaat.unwrap_or(100);

println!("{waarde}");
```

Dan krijgen we:

```text id="bq3g9c"
100
```

Bij:

```rust,ignore
let resultaat: Result<i32, String> = Ok(75);
```

wordt:

```rust,ignore
resultaat.unwrap_or(100)
```

gewoon `75`.

---

## 11. `is_ok()` en `is_err()`

Soms willen we alleen controleren of iets gelukt is.

```rust,ignore
if resultaat.is_ok() {
    println!("Gelukt!");
}
```

Of:

```rust,ignore
if resultaat.is_err() {
    println!("Er is een fout opgetreden.");
}
```

Net als bij `Option` krijgen we hiermee alleen informatie over de toestand.

We krijgen de daadwerkelijke waarde of fout niet automatisch.

---

## 12. Het verschil tussen `Option` en `Result`

Dit onderscheid moet goed duidelijk zijn.

Stel dat we zoeken naar een item:

```rust,ignore
fn vind_item(naam: &str) -> Option<String>
```

Het item bestaat of bestaat niet.

```text id="p2u3ki"
Some(item)
None
```

Maar bij een savegame:

```rust,ignore
fn laad_savegame() -> Result<String, String>
```

kan het zijn:

```text id="0qf3ka"
Ok(savegame)
Err(foutmelding)
```

Een handige vuistregel:

> Gebruik `Option` wanneer "geen waarde" een normale mogelijkheid is.
> Gebruik `Result` wanneer er iets kan mislukken en je informatie over de fout wilt bewaren.

---

## 13. `Result` met een `bool`

Je kunt technisch gezien ook:

```rust,ignore
Result<bool, String>
```

gebruiken.

Maar denk goed na over wat dat betekent.

Je krijgt dan vier mogelijke situaties:

```text id="w9y5dm"
Ok(true)
Ok(false)
Err(...)
```

Dat kan soms nuttig zijn, maar vaak betekent het dat je meerdere soorten informatie door elkaar gebruikt.

Voor beginners is het meestal beter om eerst duidelijke returntypes te kiezen.

---

## 14. De `?`-operator

Nu komt een belangrijk onderdeel van Rust.

Stel dat we dit hebben:

```rust,ignore
fn laad_savegame() -> Result<String, String> {
    match std::fs::read_to_string("savegame.txt") {
        Ok(inhoud) => Ok(inhoud),
        Err(fout) => Err(fout.to_string()),
    }
}
```

De functie geeft de fout alleen maar door.

Rust heeft daarvoor een kortere manier:

```rust,ignore
fn laad_savegame() -> Result<String, String> {
    let inhoud = std::fs::read_to_string("savegame.txt")
        .map_err(|fout| fout.to_string())?;

    Ok(inhoud)
}
```

De `?` betekent hier ongeveer:

> Als dit lukt, ga verder met de waarde. Als het mislukt, geef de fout terug aan de aanroeper.

De precieze werking is belangrijk, maar je hoeft de `?`-operator nog niet te zien als magie.

Conceptueel:

```text id="2pj1f7"
Ok(waarde)
    ↓
ga verder

Err(fout)
    ↓
geef de fout door
```

---

## 15. Een eenvoudiger voorbeeld

We kunnen de `?` eerst bekijken zonder bestanden.

```rust,ignore
fn positieve_waarde(getal: i32) -> Result<i32, String> {
    if getal < 0 {
        return Err(String::from("Getal is negatief."));
    }

    Ok(getal)
}
```

Daarna:

```rust,ignore
fn verdubbel_positief(getal: i32) -> Result<i32, String> {
    let waarde = positieve_waarde(getal)?;

    Ok(waarde * 2)
}
```

Als `positieve_waarde` een fout geeft, wordt die fout automatisch doorgegeven.

---

## 16. Een RPG-voorbeeld

Stel dat een speler een level omhoog wil gaan.

```rust,ignore
fn level_up(xp: i32) -> Result<i32, String> {
    if xp < 100 {
        Err(String::from("Niet genoeg XP."))
    } else {
        Ok(xp + 100)
    }
}
```

Gebruik:

```rust,ignore
match level_up(150) {
    Ok(xp) => println!("Nieuwe XP: {xp}"),
    Err(fout) => println!("Level omhoog mislukt: {fout}"),
}
```

---

## 17. Meerdere mogelijke fouten

Een functie kan verschillende fouten hebben.

Bijvoorbeeld:

```rust,ignore
fn koop_item(goud: i32, prijs: i32) -> Result<i32, String> {
    if prijs <= 0 {
        return Err(String::from("Ongeldige prijs."));
    }

    if goud < prijs {
        return Err(String::from("Niet genoeg goud."));
    }

    Ok(goud - prijs)
}
```

Er zijn hier drie mogelijkheden:

```text id="z4f8x7"
Ok(nieuw_goud)

Err("Ongeldige prijs.")

Err("Niet genoeg goud.")
```

Dit is veel informatiever dan alleen `false`.

---

## 18. `Result` en automatische tests

`Result` werkt goed samen met onze tests.

```rust,ignore
#[test]
fn kopen_met_genoeg_goud_werkt() {
    let resultaat = koop_item(100, 50);

    assert_eq!(resultaat, Ok(50));
}
```

En:

```rust,ignore
#[test]
fn kopen_zonder_genoeg_goud_mislukt() {
    let resultaat = koop_item(20, 50);

    assert!(resultaat.is_err());
}
```

We kunnen zelfs controleren welke fout we krijgen.

---

## 19. `Result` met een eigen fouttype

Voorlopig gebruiken we vaak:

```rust,ignore
Result<T, String>
```

Dat is eenvoudig.

In grotere Rust-programma's worden fouten vaak voorgesteld door een eigen `enum`.

Bijvoorbeeld:

```rust,ignore
enum KoopFout {
    OngeldigePrijs,
    NietGenoegGoud,
}
```

Daarna:

```rust,ignore
fn koop_item(goud: i32, prijs: i32) -> Result<i32, KoopFout> {
    if prijs <= 0 {
        return Err(KoopFout::OngeldigePrijs);
    }

    if goud < prijs {
        return Err(KoopFout::NietGenoegGoud);
    }

    Ok(goud - prijs)
}
```

Dit is krachtiger, maar we hoeven hier nog niet diep op in te gaan.

We gebruiken dit vooral om te laten zien dat het fouttype niet per se `String` hoeft te zijn.

---

## 20. `Result` en bestanden

Nu kunnen we terug naar savegames.

```rust,ignore
use std::fs;

fn sla_op(inhoud: &str) -> Result<(), String> {
    match fs::write("savegame.txt", inhoud) {
        Ok(()) => Ok(()),
        Err(fout) => Err(fout.to_string()),
    }
}
```

Hier is het succesresultaat:

```rust,ignore
Ok(())
```

Wat betekent dat?

De operatie is gelukt, maar er is geen interessante waarde om terug te geven.

`()` heet de **unit value**.

Voorlopig kun je het zien als:

> Gelukt, maar ik hoef niets terug te geven.

---

## 21. Savegame schrijven

We kunnen dit gebruiken:

```rust,ignore
match sla_op("Arin\n100\n50") {
    Ok(()) => println!("Spel opgeslagen."),
    Err(fout) => println!("Opslaan mislukt: {fout}"),
}
```

Daarmee hebben we een echte toepassing van `Result`.

---

## 22. Een complete savegame-functie

Bijvoorbeeld:

```rust,ignore
use std::fs;

fn sla_speler_op(
    naam: &str,
    gezondheid: i32,
    goud: i32,
) -> Result<(), String> {
    let inhoud = format!(
        "{naam}\n{gezondheid}\n{goud}"
    );

    fs::write("savegame.txt", inhoud)
        .map_err(|fout| fout.to_string())?;

    Ok(())
}
```

Hier gebruiken we een paar dingen die we al kennen:

- `&str`
- parameters
- `String`-achtige tekst
- `Result`
- `?`.

We introduceren hier ook `format!`.

`format!` maakt een nieuwe `String` zonder deze direct af te drukken.

---

## 23. Een fout is geen crash

Dit is een belangrijk verschil.

Een `Result` betekent niet:

> Het programma is kapot.

Het betekent:

> De operatie kan mislukken en de code moet bepalen wat daarmee gebeurt.

Bijvoorbeeld:

```text id="k13f0a"
Savegame laden
       │
       ▼
    gelukt?
    /    \
  ja      nee
  │        │
  ▼        ▼
Ok(...)  Err(...)
  │        │
  ▼        ▼
verder   fout afhandelen
```

Dat is een normaal onderdeel van programma's.

---

## 24. Compilerfouten, tests en runtime-fouten

We hebben inmiddels drie verschillende soorten feedback gezien.

### Compilerfout

Rust kan de code niet compileren.

Bijvoorbeeld:

```text id="2om8fy"
verwacht i32
maar kreeg String
```

### Testfout

De code compileert, maar een test faalt.

```text id="5xk2q9"
expected: 100
actual: 80
```

### Runtime-fout

Het programma draait, maar tijdens het uitvoeren gebeurt iets onverwachts.

Bijvoorbeeld een `unwrap()` op `Err`.

`Result` helpt om veel situaties die anders tot runtime-problemen kunnen leiden **expliciet te behandelen**.

---

## 25. Belangrijkste patronen

Een eenvoudig `Result`:

```rust,ignore
Result<i32, String>
```

Succes:

```rust,ignore
Ok(100)
```

Fout:

```rust,ignore
Err(String::from("Fout!"))
```

Behandelen:

```rust,ignore
match resultaat {
    Ok(waarde) => println!("{waarde}"),
    Err(fout) => println!("{fout}"),
}
```

Alleen controleren:

```rust,ignore
resultaat.is_ok()
```

of:

```rust,ignore
resultaat.is_err()
```

Een standaardwaarde:

```rust,ignore
resultaat.unwrap_or(100)
```

Een fout doorgeven:

```rust,ignore
let waarde = een_functie()?; 
```

---

## 26. Rustlings

Maak:

```text id="b9x4k7"
exercises/result/
```

## 01_ok.rs

Maak een `Result<i32, String>` met `Ok(100)`.

Gebruik `match` om de waarde af te drukken.

---

## 02_err.rs

Maak een `Result<i32, String>` met een foutmelding.

Gebruik `match` om de fout af te drukken.

---

## 03_match_result.rs

Maak:

```rust,ignore
fn deel(a: i32, b: i32) -> Result<i32, String>
```

Geef een fout terug wanneer `b` nul is.

---

## 04_result_success.rs

Test `deel(10, 2)`.

De uitkomst moet `Ok(5)` zijn.

---

## 05_result_error.rs

Test `deel(10, 0)`.

De uitkomst moet een fout zijn.

---

## 06_is_ok.rs

Gebruik `is_ok()` om te controleren of een berekening gelukt is.

---

## 07_is_err.rs

Gebruik `is_err()` om te controleren of een operatie is mislukt.

---

## 08_unwrap.rs

Maak een `Ok(100)` en gebruik `unwrap()`.

Test daarna bewust een `Err`.

Beschrijf in een commentaar wat er gebeurt.

---

## 09_unwrap_or.rs

Gebruik `unwrap_or()` om een standaardwaarde te gebruiken wanneer een berekening mislukt.

---

## 10_shop.rs

Maak:

```rust,ignore
fn koop_item(goud: i32, prijs: i32) -> Result<i32, String>
```

Geef een fout als er onvoldoende goud is.

---

## 11_shop_invalid_price.rs

Breid `koop_item` uit zodat een prijs van nul of lager ook een fout geeft.

---

## 12_level_up.rs

Maak:

```rust,ignore
fn level_up(xp: i32) -> Result<i32, String>
```

De speler heeft minimaal 100 XP nodig.

---

## 13_savegame.rs

Maak:

```rust,ignore
fn maak_savegame(
    naam: &str,
    gezondheid: i32,
) -> Result<String, String>
```

Geef een fout wanneer de gezondheid negatief is.

---

## 14_savegame_file.rs

Gebruik:

```rust,ignore
std::fs::write
```

om een savegame op te slaan.

Behandel de mogelijke fout met `Result`.

---

## 15_load_savegame.rs

Gebruik:

```rust,ignore
std::fs::read_to_string
```

om een savegame te lezen.

Geef de inhoud terug als `Ok`.

Geef de fout terug als `Err`.

---

## 16_question_mark.rs

Maak twee functies:

```rust,ignore
fn controleer(getal: i32) -> Result<i32, String>
```

en:

```rust,ignore
fn verdubbel(getal: i32) -> Result<i32, String>
```

Gebruik in `verdubbel` de `?`-operator.

---

## 17_question_mark_debug.rs

De volgende functie compileert niet:

```rust,ignore
fn controleer(getal: i32) -> Result<i32, String> {
    if getal < 0 {
        Err(String::from("Negatief"))
    }

    Ok(getal)
}
```

Onderzoek waarom.

Los de functie op.

---

## 18_result_move.rs

Onderzoek wat er gebeurt wanneer een `String` binnen een `Result` wordt verplaatst.

Gebruik:

```rust,ignore
Result<String, String>
```

en los het ownership-probleem op met borrowing.

---

## 19_rpg_save.rs

Maak een `Speler` en een functie:

```rust,ignore
fn sla_speler_op(speler: &Speler) -> Result<(), String>
```

Schrijf de spelergegevens naar een bestand.

---

## 20_rpg_load.rs

Maak een functie:

```rust,ignore
fn laad_speler() -> Result<String, String>
```

Lees de savegame en behandel een ontbrekend bestand netjes.

---

## 21_shop_tests.rs

Schrijf automatische tests voor:

- aankoop met genoeg goud
- aankoop met te weinig goud
- ongeldige prijs.

---

## 22_save_tests.rs

Test de savegamefuncties.

Test minimaal:

- geldige gegevens
- ongeldige gegevens
- succesvolle opslag.

---

## 23_debug_result.rs

De volgende code bevat een fout:

```rust,ignore
fn controleer(gezondheid: i32) -> Result<i32, String> {
    if gezondheid > 0 {
        Ok(gezondheid)
    } else {
        Err(String::from("Dood"))
    }
}

fn main() {
    let gezondheid = controleer(-10);

    println!("{}", gezondheid + 10);
}
```

Gebruik de compiler om te ontdekken waarom dit niet werkt.

---

## 24_final_result.rs

Maak een klein RPG-programma met:

- een speler
- een inventory
- een shop
- een savegame
- minimaal drie functies die `Result` teruggeven
- minimaal één functie die `?` gebruikt
- automatische tests voor de belangrijkste foutgevallen.

---

## 27. Oefeningen

## Oefening 1 — Shop

Maak een eenvoudige winkel.

Een speler heeft:

```text id="7m8f3q"
100 goud
```

Een zwaard kost:

```text id="3y1v8s"
75 goud
```

Laat de aankoop slagen wanneer er genoeg goud is en anders een fout opleveren.

---

## Oefening 2 — Savegame

Maak een savegamebestand voor:

- naam
- gezondheid
- goud.

Gebruik `Result` voor zowel opslaan als laden.

---

## Oefening 3 — Ongeldige gegevens

Laat een savegame mislukken wanneer:

- gezondheid kleiner dan 0 is
- goud kleiner dan 0 is
- de naam leeg is.

---

## Oefening 4 — Fouten doorgeven

Maak drie functies die elkaar aanroepen.

Gebruik de `?`-operator om een fout helemaal terug naar `main` te laten gaan.

---

## Oefening 5 — Fouten testen

Schrijf tests voor alle foutgevallen.

Probeer daarbij zowel `Ok(...)` als `Err(...)` te testen.

---

## 28. Wat moet je na dit artikel kunnen?

Je moet nu begrijpen:

- wat `Result<T, E>` is
- wat `Ok(...)` betekent
- wat `Err(...)` betekent
- wanneer je `Result` gebruikt
- hoe je `Result` met `match` behandelt
- wat `is_ok()` en `is_err()` doen
- wat `unwrap()` en `unwrap_or()` doen
- waarom `unwrap()` niet hetzelfde is als foutafhandeling
- wat de `?`-operator doet
- hoe `Result` gebruikt kan worden voor bestanden
- hoe je foutgevallen met tests controleert.

Het belangrijkste onderscheid met `Option`:

```text id="w4x8c2"
Option<T>
    Some(waarde)
    None

    → "Is er een waarde?"

Result<T, E>
    Ok(waarde)
    Err(fout)

    → "Is de operatie gelukt?"
```

En een belangrijk programmeerpatroon:

```text id="5v2n8a"
doe iets
   │
   ▼
gelukt?
 /    \
ja     nee
│       │
Ok      Err
│       │
▼       ▼
verder  fout afhandelen
```

Met `Result` leren we dus niet alleen een nieuw datatype. We leren ook een belangrijke manier van denken:

> Een fout is een mogelijke uitkomst van een operatie en moet onderdeel zijn van het ontwerp van je programma.

In het volgende artikel gaan we verder met **iterators**. Daar verbinden we `Vec`, `for`, closures en de nieuwe kennis uit deze artikelen met elkaar.

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 21](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_21/).

