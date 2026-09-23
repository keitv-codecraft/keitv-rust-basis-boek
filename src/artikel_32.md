# 32. De spel-lus en gebruikersinvoer

## Wat gaan we leren?

In dit artikel bouwen we een interactieve spel-lus waarin de speler keuzes kan maken via het toetsenbord.

We leren:

- gebruikersinvoer lezen met `std::io::stdin().read_line()`
- invoer omzetten naar getallen of commando's en robuust omgaan met foutieve invoer
- een hoofdmenu en actiemenu's weergeven en afhandelen met `match`
- de gameloop structureren als toestandmachine met `loop`, `break` en `continue`
- gebruikersinterface (I/O) netjes scheiden van de onderliggende spelregels.

---

## 1. Een programma dat wacht op invoer

We beginnen met een heel klein programma.

```rust
use std::io;

fn main() {
    println!("Hoe heet je?");

    let mut naam = String::new();

    io::stdin()
        .read_line(&mut naam)
        .expect("Kon invoer niet lezen.");

    println!("Welkom, {}!", naam.trim());
}
```

Als we dit uitvoeren, krijgen we bijvoorbeeld:

```text
Hoe heet je?
Arin
Welkom, Arin!
```

De computer wacht dus totdat de gebruiker iets invoert.

---

## 2. Wat doet `read_line`?

Deze code:

```rust,ignore
io::stdin().read_line(&mut naam)
```

leest een regel tekst van de standaardinvoer.

De ingevoerde tekst wordt in `naam` gezet.

Daarom moet `naam` mutable zijn:

```rust,ignore
let mut naam = String::new();
```

De functie verandert namelijk de `String`.

Het argument is daarom:

```rust,ignore
&mut naam
```

We geven een mutable reference door.

Dit sluit direct aan bij wat we eerder over borrowing hebben geleerd.

---

## 3. Waarom staat er `&mut`?

Stel dat we dit zouden schrijven:

```rust,ignore
let naam = String::new();

io::stdin().read_line(&mut naam);
```

Dan kan Rust dit niet accepteren.

`read_line` moet de `String` namelijk aanpassen.

Een niet-mutable variabele mag niet worden aangepast.

Daarom:

```rust,ignore
let mut naam = String::new();
```

en:

```rust,ignore
&mut naam
```

De betekenis is:

> Deze functie mag tijdelijk de inhoud van deze `String` aanpassen.

---

## 4. Waarom gebruiken we `expect`?

`read_line` kan mislukken.

De functie geeft daarom een `Result` terug.

In een eenvoudig programma kunnen we schrijven:

```rust,ignore
.expect("Kon invoer niet lezen.");
```

Als alles goed gaat, gaat het programma verder.

Als het lezen mislukt, stopt het programma met deze foutmelding.

Voor onze eerste invoerfunctie is dat acceptabel.

Later kunnen we invoerfouten op een gebruikersvriendelijkere manier afhandelen.

---

## 5. De newline

Als de gebruiker schrijft:

```text
Arin
```

komt de Enter-toets mee in de ingevoerde tekst.

De waarde is ongeveer:

```text
"Arin\n"
```

Daarom gebruiken we:

```rust,ignore
naam.trim()
```

`trim()` verwijdert witruimte aan het begin en einde.

We kunnen dan schrijven:

```rust,ignore
println!("Welkom, {}!", naam.trim());
```

---

## 6. Een eigen invoerfunctie

We willen niet overal dezelfde code schrijven.

Daarom maken we een functie:

```rust,ignore
use std::io;

fn lees_tekst() -> String {
    let mut invoer = String::new();

    io::stdin()
        .read_line(&mut invoer)
        .expect("Kon invoer niet lezen.");

    invoer.trim().to_string()
}
```

Nu kunnen we:

```rust,ignore
fn main() {
    println!("Hoe heet je?");

    let naam = lees_tekst();

    println!("Welkom, {}!", naam);
}
```

De functie geeft een `String` terug.

Dat is belangrijk.

We geven de invoer niet als reference terug, omdat de lokale variabele `invoer` na het uitvoeren van de functie verdwijnt.

We geven daarom ownership van de nieuwe `String` terug.

---

## 7. Een getal lezen

Voor onze menu's willen we meestal een getal.

Bijvoorbeeld:

```text
1. Verkennen
2. Reizen
3. Inventaris
4. Karakter
5. Stoppen
```

We kunnen de invoer eerst als tekst lezen:

```rust,ignore
let invoer = lees_tekst();
```

Daarna proberen we deze tekst om te zetten naar een getal:

```rust,ignore
let keuze = invoer.parse::<u32>();
```

`parse` geeft een `Result` terug.

Bijvoorbeeld:

```text
Ok(3)
```

of:

```text
Err(...)
```

---

## 8. Een keuze veilig lezen

We kunnen hiervan een functie maken:

```rust,ignore
fn lees_getal() -> u32 {
    loop {
        let invoer = lees_tekst();

        match invoer.parse::<u32>() {
            Ok(getal) => return getal,
            Err(_) => {
                println!("Voer een getal in.");
            }
        }
    }
}
```

Deze functie blijft vragen totdat de gebruiker een geldig getal invoert.

Bijvoorbeeld:

```text
appel
Voer een getal in.
abc
Voer een getal in.
3
```

Dan geeft de functie uiteindelijk:

```text
3
```

terug.

---

## 9. Waarom gebruiken we hier `loop`?

De bedoeling is:

> Blijf invoer vragen totdat de invoer geldig is.

Dat past precies bij:

```rust,ignore
loop {
    // blijf dit uitvoeren
}
```

De functie stopt wanneer deze regel wordt uitgevoerd:

```rust,ignore
return getal;
```

`return` beëindigt namelijk de hele functie.

We hadden ook `break` kunnen gebruiken, maar dan zouden we het getal eerst ergens moeten bewaren.

---

## 10. `return` en `break`

Deze twee lijken op elkaar, maar doen iets anders.

Bij:

```rust,ignore
loop {
    return 5;
}
```

wordt de **functie** beëindigd.

Bij:

```rust,ignore
loop {
    break;
}
```

wordt alleen de **lus** beëindigd.

Bijvoorbeeld:

```rust,ignore
loop {
    println!("Hallo");

    break;
}

println!("Het programma gaat verder.");
```

Na `break` gaat het programma verder na de lus.

---

## 11. Een menu maken

We kunnen nu een menu-functie maken:

```rust,ignore
fn toon_menu() {
    println!();
    println!("=== Avontuur ===");
    println!("1. Verkennen");
    println!("2. Reizen");
    println!("3. Inventaris");
    println!("4. Karakter");
    println!("5. Stoppen");
}
```

De functie hoeft niets terug te geven.

Dat betekent:

```rust,ignore
fn toon_menu() {
    // ...
}
```

De returnwaarde is automatisch `()`.

---

## 12. De keuze verwerken

We kunnen de keuze met `match` verwerken:

```rust,ignore
fn verwerk_keuze(keuze: u32) {
    match keuze {
        1 => println!("Je gaat de omgeving verkennen."),
        2 => println!("Je wilt reizen."),
        3 => println!("Je bekijkt je inventaris."),
        4 => println!("Je bekijkt je karakter."),
        5 => println!("Je wilt stoppen."),
        _ => println!("Ongeldige keuze."),
    }
}
```

Dit is overzichtelijker dan een lange reeks `if`-blokken.

---

## 13. De eerste spel-lus

We kunnen nu alles samenbrengen:

```rust,ignore
fn main() {
    loop {
        toon_menu();

        let keuze = lees_getal();

        match keuze {
            1 => println!("Verkennen"),
            2 => println!("Reizen"),
            3 => println!("Inventaris"),
            4 => println!("Karakter"),
            5 => break,
            _ => println!("Ongeldige keuze."),
        }
    }

    println!("Het spel is afgelopen.");
}
```

Dit is de kern van onze RPG.

De computer doet steeds:

```text
menu tonen
     ↓
keuze lezen
     ↓
keuze uitvoeren
     ↓
terug naar menu
     ↓
keuze lezen
     ↓
...
```

---

## 14. Waarom staat `break` bij keuze 5?

We willen het spel beëindigen wanneer de speler voor:

```text
5. Stoppen
```

kiest.

Daarom:

```rust,ignore
5 => break,
```

De `loop` stopt.

Daarna gaat het programma verder met:

```rust,ignore
println!("Het spel is afgelopen.");
```

---

## 15. Ongeldige keuzes

Stel dat de speler invoert:

```text
99
```

Dan komt die waarde terecht bij:

```rust,ignore
_ => println!("Ongeldige keuze."),
```

Het spel stopt niet.

De lus begint opnieuw.

Dit geeft:

```text
=== Avontuur ===
1. Verkennen
2. Reizen
3. Inventaris
4. Karakter
5. Stoppen

> 99

Ongeldige keuze.

=== Avontuur ===
1. Verkennen
2. Reizen
3. Inventaris
4. Karakter
5. Stoppen
```

Dat is precies wat we willen.

---

## 16. Ongeldige invoer is iets anders

Er zijn eigenlijk twee soorten fouten.

De speler kan een geldig getal invoeren dat geen geldige keuze is:

```text
99
```

Maar de speler kan ook helemaal geen getal invoeren:

```text
appel
```

Onze `lees_getal()` behandelt het tweede probleem.

De `match` in de spel-lus behandelt het eerste.

Dat zijn dus twee verschillende controles.

```text
"appel"
   ↓
parse()
   ↓
Err
   ↓
lees_getal vraagt opnieuw

"99"
   ↓
parse()
   ↓
Ok(99)
   ↓
match
   ↓
ongeldige keuze
```

Het is goed om deze twee situaties uit elkaar te houden.

---

## 17. De speler toevoegen

Nu willen we niet alleen tekst printen.

We willen onze bestaande `Speler` gebruiken.

Stel dat we hebben:

```rust,ignore
pub struct Speler {
    naam: String,
    gezondheid: i32,
    goud: i32,
    // ...
}
```

Dan kunnen we in `main` een speler maken:

```rust,ignore
let mut speler = Speler::nieuw("Arin");
```

We gebruiken `mut`, omdat de toestand van de speler tijdens het spel verandert.

Bijvoorbeeld:

```text
gezondheid
goud
inventaris
wapen
locatie
```

---

## 18. De speltoestand

Een probleem ontstaat als onze `main` steeds meer variabelen krijgt:

```rust,ignore
let mut speler = Speler::nieuw("Arin");
let mut wereld = Wereld::nieuw();
let mut winkel = Winkel::nieuw();
let mut spel_over = false;
```

Dat is nog niet verschrikkelijk.

Maar naarmate het spel groeit, komen er steeds meer onderdelen bij.

Daarom kunnen we de toestand van het spel bundelen.

```rust,ignore
pub struct Spel {
    speler: Speler,
    wereld: Wereld,
}
```

Dit object bevat de toestand van onze RPG.

---

## 19. Een constructor voor `Spel`

We kunnen een constructor maken:

```rust,ignore
impl Spel {
    pub fn nieuw(speler: Speler, wereld: Wereld) -> Self {
        Self {
            speler,
            wereld,
        }
    }
}
```

In `main`:

```rust,ignore
let speler = Speler::nieuw("Arin");
let wereld = Wereld::nieuw();

let mut spel = Spel::nieuw(speler, wereld);
```

We hebben nu één variabele die de belangrijkste toestand bevat:

```rust,ignore
spel
```

---

## 20. De spel-lus als methode

We kunnen de spel-lus vervolgens in `Spel` stoppen:

```rust,ignore
impl Spel {
    pub fn start(&mut self) {
        loop {
            self.toon_menu();

            let keuze = lees_getal();

            match keuze {
                1 => self.verken(),
                2 => self.reis(),
                3 => self.toon_inventaris(),
                4 => self.toon_speler(),
                5 => break,
                _ => println!("Ongeldige keuze."),
            }
        }
    }
}
```

Dit is een belangrijke stap.

`main` hoeft nu alleen nog het spel te maken en te starten:

```rust,ignore
fn main() {
    let speler = Speler::nieuw("Arin");
    let wereld = Wereld::nieuw();

    let mut spel = Spel::nieuw(speler, wereld);

    spel.start();
}
```

De details van de spel-lus staan niet meer in `main`.

---

## 21. Waarom `&mut self`?

Onze methode is:

```rust,ignore
pub fn start(&mut self)
```

Waarom mutable?

Omdat het spel verandert.

Tijdens het spelen kan:

- de gezondheid van de speler veranderen
- goud veranderen
- de inventaris veranderen
- het wapen veranderen
- de locatie veranderen
- een vijand verdwijnen.

De methode moet dus de toestand van `Spel` kunnen aanpassen.

Daarom:

```rust,ignore
&mut self
```

---

## 22. De methode `verken`

We kunnen een methode maken:

```rust,ignore
fn verken(&mut self) {
    println!("Je kijkt om je heen.");
}
```

Later kunnen we hier echte logica in zetten.

Bijvoorbeeld:

```rust,ignore
fn verken(&mut self) {
    let locatie = self.wereld.huidige_locatie(
        &self.speler.huidige_locatie()
    );

    match locatie {
        Some(locatie) => locatie.toon(),
        None => println!("Je bevindt je op een onbekende locatie."),
    }
}
```

Hier zien we opnieuw `Option`.

De locatie kan namelijk bestaan of ontbreken.

---

## 23. Reizen

De methode `reis` kan de beschikbare verbindingen tonen.

Bijvoorbeeld:

```rust,ignore
fn reis(&mut self) {
    println!("Waar wil je naartoe?");

    // bestemmingen tonen

    // keuze lezen

    // bestemming bepalen

    // speler verplaatsen
}
```

We kunnen hiervoor eerst een aparte functie maken:

```rust,ignore
fn toon_bestemmingen(&self) {
    // ...
}
```

En daarna:

```rust,ignore
fn reis(&mut self) {
    self.toon_bestemmingen();

    let keuze = lees_getal();

    // ...
}
```

Door kleine functies te gebruiken houden we de code overzichtelijk.

---

## 24. Menu's hoeven niet allemaal hetzelfde te zijn

Het hoofdmenu is:

```text
1. Verkennen
2. Reizen
3. Inventaris
4. Karakter
5. Stoppen
```

Maar het reismenu kan bijvoorbeeld zijn:

```text
Waar wil je naartoe?

1. Dommeldoor
2. Oude Toren
3. Rivierhaven
4. Terug
```

En het gevechtsmenu:

```text
Wat doe je?

1. Aanvallen
2. Potion gebruiken
3. Vluchten
```

Elke situatie heeft dus zijn eigen menu.

---

## 25. Een menu kan een `Option` teruggeven

Een menu kan bijvoorbeeld een keuze teruggeven:

```rust,ignore
enum Actie {
    Verkennen,
    Reizen,
    Inventaris,
    Karakter,
    Stoppen,
}
```

Daarna kunnen we een functie maken:

```rust,ignore
fn kies_hoofdactie(keuze: u32) -> Option<Actie> {
    match keuze {
        1 => Some(Actie::Verkennen),
        2 => Some(Actie::Reizen),
        3 => Some(Actie::Inventaris),
        4 => Some(Actie::Karakter),
        5 => Some(Actie::Stoppen),
        _ => None,
    }
}
```

Nu kunnen we schrijven:

```rust,ignore
match kies_hoofdactie(keuze) {
    Some(Actie::Verkennen) => println!("Verkennen"),
    Some(Actie::Reizen) => println!("Reizen"),
    Some(Actie::Inventaris) => println!("Inventaris"),
    Some(Actie::Karakter) => println!("Karakter"),
    Some(Actie::Stoppen) => break,
    None => println!("Ongeldige keuze."),
}
```

Hier combineren we:

- `enum`
- `Option`
- `match`
- functies.

---

## 26. Waarom een enum gebruiken?

We zouden ook overal getallen kunnen gebruiken:

```text
1 = verkennen
2 = reizen
3 = inventaris
4 = karakter
5 = stoppen
```

Maar in de rest van het programma is:

```rust,ignore
Actie::Verkennen
```

veel duidelijker dan:

```rust,ignore
1
```

Het getal hoort bij het gebruikersinterface.

`Actie::Verkennen` hoort bij de spelregels.

Dat zijn verschillende dingen.

---

## 27. Gebruikersinterface en spelregels scheiden

Dit is een belangrijk ontwerpprincipe.

De gebruiker ziet:

```text
1. Verkennen
2. Reizen
3. Inventaris
4. Karakter
5. Stoppen
```

Maar het programma kan intern werken met:

```rust,ignore
Actie::Verkennen
```

De omzetting gebeurt op één plek:

```rust,ignore
fn kies_hoofdactie(keuze: u32) -> Option<Actie>
```

Daardoor hoeven de rest van de spelregels niet te weten dat verkennen toevallig keuze `1` is.

Later kunnen we het menu veranderen zonder het hele spel aan te passen.

---

## 28. Een volledige hoofdloop

We kunnen nu iets maken als:

```rust,ignore
pub fn start(&mut self) {
    loop {
        self.toon_menu();

        let keuze = lees_getal();

        let actie = kies_hoofdactie(keuze);

        match actie {
            Some(Actie::Verkennen) => self.verken(),
            Some(Actie::Reizen) => self.reis(),
            Some(Actie::Inventaris) => self.toon_inventaris(),
            Some(Actie::Karakter) => self.toon_speler(),
            Some(Actie::Stoppen) => break,
            None => println!("Ongeldige keuze."),
        }
    }
}
```

Dit begint al behoorlijk veel op een echte spel-engine te lijken.

---

## 29. De hoofdloop hoeft niet alles te weten

Een veelgemaakte fout is om alles in `start()` te zetten.

Bijvoorbeeld:

```rust,ignore
pub fn start(&mut self) {
    loop {
        // honderden regels code
    }
}
```

Dat willen we voorkomen.

De hoofdloop moet vooral bepalen:

> Welke actie wil de speler uitvoeren?

De details horen in andere functies:

```text
start()
 ├── verken()
 ├── reis()
 ├── toon_inventaris()
 └── toon_speler()
```

En `reis()` kan weer eigen functies gebruiken.

Zo ontstaat een boomstructuur van verantwoordelijkheden.

---

## 30. Een spelactie kan meerdere stappen bevatten

Bijvoorbeeld:

```rust,ignore
fn verken(&mut self) {
    self.toon_huidige_locatie();

    if self.heeft_levende_vijand() {
        self.start_gevecht();
        return;
    }

    self.verwerk_ontmoeting();
}
```

De functie `verken()` hoeft dan niet zelf alle details van een gevecht te kennen.

Het zegt alleen:

> Als er een vijand is, start een gevecht.

Dit maakt grotere programma's veel makkelijker te begrijpen.

---

## 31. `return` in een spelactie

Stel:

```rust,ignore
fn verken(&mut self) {
    if self.heeft_levende_vijand() {
        self.start_gevecht();
        return;
    }

    println!("Er is hier geen vijand.");
}
```

Als er een vijand is, voeren we het gevecht uit.

Daarna:

```rust,ignore
return;
```

De functie stopt.

Zonder `return` zou ook de laatste regel kunnen worden uitgevoerd:

```text
Je hebt gevochten.
Er is hier geen vijand.
```

Dat willen we niet.

---

## 32. Een gevecht is zelf een spel-lus

Dit is een belangrijk inzicht.

De hoofdloop is bijvoorbeeld:

```text
wereldmenu
```

Maar een gevecht heeft zijn eigen loop:

```rust,ignore
loop {
    toon_gevechtsmenu();

    let keuze = lees_getal();

    match keuze {
        1 => aanval(),
        2 => gebruik_potion(),
        3 => vlucht(),
        _ => println!("Ongeldige keuze."),
    }

    if speler_is_dood() || vijand_is_dood() {
        break;
    }
}
```

Een programma kan dus meerdere lussen bevatten.

Een lus kan zelfs een andere functie aanroepen waarin weer een lus zit.

---

## 33. De toestand bepaalt wanneer een lus eindigt

De gevechtslus moet stoppen wanneer bijvoorbeeld:

```text
speler dood
```

of:

```text
vijand dood
```

We kunnen dat controleren:

```rust,ignore
if !speler.is_levend() || !vijand.is_levend() {
    break;
}
```

De `!` betekent:

> niet.

Dus:

```rust,ignore
!speler.is_levend()
```

betekent:

> de speler leeft niet.

---

## 34. Een gevecht heeft rondes

Een gevecht kan daardoor ongeveer zo werken:

```text
Ronde 1
  speler kiest aanval
  vijand valt aan

Ronde 2
  speler kiest aanval
  vijand valt aan

Ronde 3
  speler gebruikt potion
  vijand valt aan

...

één van beiden is dood
```

De lus zorgt ervoor dat dit blijft doorgaan.

---

## 35. De spel-lus is dus een toestandmachine

Je kunt onze RPG grofweg zien als verschillende toestanden:

```text
          ┌───────────────┐
          │ Hoofdmenu     │
          └───────┬───────┘
                  │
       ┌──────────┼──────────┐
       │          │          │
       ▼          ▼          ▼
   Verkennen    Reizen    Inventaris
       │
       ▼
   Ontmoeting
       │
       ▼
    Gevecht
       │
       ▼
   Hoofdmenu
```

We gebruiken hiervoor nog geen speciaal Rust-concept.

We modelleren het voorlopig met functies, `enum`, `match` en lussen.

Dat is voldoende.

---

## 36. Zelfstandige oefening — Een menu

Maak een programma met:

```text
=== Mijn programma ===

1. Zeg hallo
2. Geef een getal
3. Zeg gedag
4. Stoppen
```

Het programma moet blijven draaien totdat de gebruiker `4` kiest.

Gebruik:

- `loop`
- `break`
- `match`
- `lees_getal()`.

---

## 37. Zelfstandige oefening — RPG-menu

Maak een menu:

```text
=== Avontuur ===

1. Verkennen
2. Reizen
3. Inventaris
4. Karakter
5. Stoppen
```

Maak hiervoor:

```rust,ignore
enum Actie {
    Verkennen,
    Reizen,
    Inventaris,
    Karakter,
    Stoppen,
}
```

Maak vervolgens:

```rust,ignore
fn kies_hoofdactie(keuze: u32) -> Option<Actie>
```

Test iedere mogelijke invoer.

---

## 38. Zelfstandige oefening — Gevechtsmenu

Maak twee structs:

```rust,ignore
struct Speler {
    gezondheid: i32,
}

struct Vijand {
    gezondheid: i32,
}
```

Laat de speler kiezen:

```text
1. Aanvallen
2. Vluchten
```

Bij aanvallen verliest de vijand 10 gezondheid.

Het gevecht stopt zodra:

- de vijand dood is
- de speler dood is
- de speler vlucht.

---

## 39. Eindopdracht — Speelbare command-line RPG

Breid het RPG-project uit tot een eerste echte speelbare versie.

De speler moet:

1. een naam krijgen
2. in een wereld starten
3. een hoofdmenu krijgen
4. de huidige locatie kunnen bekijken
5. naar verbonden locaties kunnen reizen
6. de inventaris kunnen bekijken
7. het karakter kunnen bekijken
8. vijanden kunnen tegenkomen
9. gevechten kunnen voeren
10. het spel kunnen verlaten.

Gebruik minimaal:

```text
loop
break
match
enum
Option
Result
Vec
struct
methods
modules
borrowing
```

De hoofdstructuur moet ongeveer zijn:

```text
main
 │
 └── Spel::start()
       │
       ├── verken()
       ├── reis()
       ├── toon_inventaris()
       ├── toon_speler()
       └── gevecht()
```

De hoofdloop moet niet alle details van het spel bevatten.

Gebruik functies en methodes om verantwoordelijkheden te verdelen.

---

## 40. Tests

Niet alles aan een interactieve spel-lus is eenvoudig automatisch te testen.

We kunnen echter de onderdelen eromheen wel testen.

Bijvoorbeeld:

```rust,ignore
#[test]
fn geldige_keuze_wordt_vertaald() {
    let actie = kies_hoofdactie(1);

    assert!(matches!(actie, Some(Actie::Verkennen)));
}
```

Of:

```rust,ignore
#[test]
fn ongeldige_keuze_geeft_none() {
    let actie = kies_hoofdactie(99);

    assert!(actie.is_none());
}
```

De daadwerkelijke gebruikersinvoer testen we voorlopig handmatig.

Dat is normaal.

We testen dus vooral de **spelregels**, niet het feit dat `stdin` daadwerkelijk op een toetsenbord wacht.

---

## 41. Wat hebben we nu gebouwd?

We hebben een belangrijk omslagpunt bereikt.

Onze RPG bestaat niet meer alleen uit losse struct- en functietests.

Er is nu een programma dat voortdurend een toestand heeft en daarop reageert.

De basis is:

```text
                 ┌──────────────┐
                 │    Speler    │
                 └──────┬───────┘
                        │
                 ┌──────▼───────┐
                 │     Spel     │
                 └──────┬───────┘
                        │
                 ┌──────▼───────┐
                 │  Spel-lus    │
                 └──────┬───────┘
                        │
             ┌──────────┼──────────┐
             ▼          ▼          ▼
          Verken      Reizen    Inventaris
             │
             ▼
        Ontmoeting
             │
             ▼
          Gevecht
             │
             └──────────┐
                        ▼
                    Spel-lus
```

De belangrijkste nieuwe gedachte is daarom niet een stukje syntax, maar deze:

> Een programma kan een toestand hebben, een keuze ontvangen, die keuze verwerken en daarna met een nieuwe toestand verdergaan.

Dat patroon vormt de basis van vrijwel het hele RPG-project dat we vanaf nu gaan bouwen.

In het volgende artikel kunnen we daarom de **gevechten volledig uitwerken**: beurtvolgorde, aanvallen, verdedigen, schade, kritieke treffers, vluchten, winnen/verliezen en de koppeling met het bestaande `Aanvaller`-trait.

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 32](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_32/).

---

## Controlelijst

Je bent klaar met dit artikel als je zonder hulp:

- [ ] invoer van de gebruiker kunt inlezen en de newline kunt verwijderen
- [ ] een menufunctie kunt schrijven die herhaalt totdat geldige invoer is gegeven
- [ ] een interactieve spel-lus kunt bouwen met `loop` en `break`
- [ ] menu-keuzes kunt koppelen aan spelacties zonder het spel te laten crashen op ongeldige invoer.
