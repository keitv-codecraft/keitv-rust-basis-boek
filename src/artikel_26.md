# Rust 26 — Bestanden lezen en schrijven

Tot nu toe leeft alle informatie van ons programma in het geheugen.

Als we bijvoorbeeld dit programma starten:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
    goud: i32,
}
```

en een speler maken:

```rust,ignore
let speler = Speler {
    naam: String::from("Arin"),
    gezondheid: 100,
    goud: 50,
};
```

dan bestaat deze informatie alleen zolang het programma draait.

Zodra het programma stopt, zijn de gegevens weg.

Voor een RPG is dat natuurlijk onhandig.

We willen bijvoorbeeld:

```text
Speler speelt
    ↓
spel opslaan
    ↓
programma afsluiten
    ↓
programma later opnieuw starten
    ↓
spel laden
```

Daarvoor moeten we leren werken met **bestanden**.

---

## 1. Bestanden op schijf

Rust kan bestanden lezen en schrijven via de standaardbibliotheek.

Daarvoor gebruiken we:

```rust,ignore
use std::fs;
```

`fs` staat voor *file system*.

We hoeven hiervoor dus geen extra crate aan Cargo toe te voegen.

---

## 2. Een bestand schrijven

De eenvoudigste manier om een bestand te schrijven is:

```rust,ignore
fs::write("save.txt", "Hallo wereld!");
```

Hiermee vragen we Rust om:

> Schrijf de tekst `"Hallo wereld!"` naar `save.txt`.

Een volledig programma:

```rust
use std::fs;

fn main() {
    fs::write("save.txt", "Hallo wereld!");
}
```

Maar hier ontbreekt nog iets belangrijks.

`fs::write` kan namelijk mislukken.

Bijvoorbeeld omdat:

- de map niet bestaat
- het bestand niet geschreven mag worden
- de schijf niet beschikbaar is
- er onvoldoende ruimte is.

Daarom geeft `fs::write` een `Result` terug.

---

## 3. `Result` bij bestanden

De functie:

```rust,ignore
fs::write("save.txt", "Hallo wereld!")
```

geeft conceptueel terug:

```rust,ignore
Result<(), std::io::Error>
```

We hebben `Result` al eerder geleerd.

Er zijn twee mogelijkheden:

```text
Ok(())
Err(fout)
```

We kunnen daarom `match` gebruiken:

```rust
use std::fs;

fn main() {
    match fs::write("save.txt", "Hallo wereld!") {
        Ok(()) => println!("Bestand opgeslagen."),
        Err(fout) => println!("Opslaan mislukt: {fout}"),
    }
}
```

Dit is een realistisch voorbeeld van foutafhandeling.

---

## 4. Een bestand lezen

We kunnen een tekstbestand lezen met:

```rust,ignore
fs::read_to_string
```

Bijvoorbeeld:

```rust
use std::fs;

fn main() {
    match fs::read_to_string("save.txt") {
        Ok(tekst) => println!("{tekst}"),
        Err(fout) => println!("Lezen mislukt: {fout}"),
    }
}
```

Ook hier krijgen we een `Result`.

Bij succes krijgen we:

```text
Ok(String)
```

Bij een fout:

```text
Err(...)
```

Dit is logisch:

```text
bestand bestaat
        ↓
   tekst ontvangen

bestand bestaat niet
        ↓
      fout
```

---

## 5. `?` gebruiken

Omdat bestanden vaak fouten kunnen veroorzaken, zouden we overal `match` kunnen schrijven.

Dat kan snel veel code opleveren.

We kunnen daarom de `?`-operator gebruiken.

Bijvoorbeeld:

```rust,ignore
use std::fs;

fn laad_spel() -> Result<String, std::io::Error> {
    let tekst = fs::read_to_string("save.txt")?;

    Ok(tekst)
}
```

De `?` betekent hier ongeveer:

> Als dit lukt, ga verder. Als het mislukt, geef de fout door aan de aanroeper.

Dit is een belangrijk patroon bij functies die zelf een `Result` teruggeven.

---

## 6. Een tekstbestand opslaan

We kunnen een eenvoudige savegame maken.

```rust,ignore
use std::fs;

fn sla_op(naam: &str, gezondheid: i32, goud: i32) -> Result<(), std::io::Error> {
    let tekst = format!(
        "naam={}\ngezondheid={}\ngoud={}",
        naam, gezondheid, goud
    );

    fs::write("save.txt", tekst)?;

    Ok(())
}
```

We kunnen hem gebruiken:

```rust,ignore
fn main() -> Result<(), std::io::Error> {
    sla_op("Arin", 80, 125)?;

    println!("Spel opgeslagen.");

    Ok(())
}
```

Het bestand ziet er dan bijvoorbeeld zo uit:

```text
naam=Arin
gezondheid=80
goud=125
```

We hebben hier verschillende dingen gecombineerd:

- `String`
- `format!`
- functies
- `Result`
- `?`
- bestanden.

---

## 7. Waarom gebruiken we `format!`?

We willen meerdere waarden in één tekst zetten.

Bijvoorbeeld:

```rust,ignore
let naam = "Arin";
let gezondheid = 80;
let goud = 125;
```

Met:

```rust,ignore
let tekst = format!(
    "naam={}\ngezondheid={}\ngoud={}",
    naam,
    gezondheid,
    goud
);
```

krijgen we:

```text
naam=Arin
gezondheid=80
goud=125
```

`\n` betekent een nieuwe regel.

`format!` maakt een nieuwe `String`.

Het verschil met `println!` is dat `println!` de tekst afdrukt, terwijl `format!` de tekst maakt zodat we hem bijvoorbeeld kunnen opslaan.

---

## 8. Een savegame laden

We kunnen het opgeslagen bestand weer lezen:

```rust,ignore
use std::fs;

fn laad_spel() -> Result<String, std::io::Error> {
    let tekst = fs::read_to_string("save.txt")?;

    Ok(tekst)
}
```

In `main`:

```rust,ignore
fn main() -> Result<(), std::io::Error> {
    let savegame = laad_spel()?;

    println!("Savegame:");
    println!("{savegame}");

    Ok(())
}
```

Dit leest de volledige inhoud van het bestand in één `String`.

---

## 9. Een bestand dat niet bestaat

Stel dat `save.txt` nog niet bestaat.

Dan:

```rust,ignore
fs::read_to_string("save.txt")
```

geen tekst teruggeven.

We krijgen een fout.

Bijvoorbeeld:

```text
No such file or directory
```

Daarom is het belangrijk dat we niet zomaar aannemen dat een savegame bestaat.

We kunnen bijvoorbeeld:

```rust,ignore
match fs::read_to_string("save.txt") {
    Ok(tekst) => println!("Savegame geladen:\n{tekst}"),
    Err(_) => println!("Er is nog geen savegame."),
}
```

Hier negeren we de precieze fout met `_`.

Voor een echte applicatie willen we soms wel weten wat de fout precies was.

---

## 10. Een betere foutmelding

We kunnen de fout gewoon afdrukken:

```rust,ignore
match fs::read_to_string("save.txt") {
    Ok(tekst) => println!("Savegame geladen:\n{tekst}"),
    Err(fout) => println!("Savegame kon niet worden geladen: {fout}"),
}
```

Dan blijft de informatie uit de fout beschikbaar.

---

## 11. De inhoud van een bestand is gewoon tekst

Een belangrijk inzicht:

> Een bestand is niet automatisch een `Speler`.

Als we dit schrijven:

```rust,ignore
fs::write("save.txt", "Arin")?;
```

staat er tekst in het bestand.

Rust weet niet automatisch dat `"Arin"` de naam van een `Speler` is.

Als we een speler willen opslaan, moeten we bepalen hoe we de gegevens in tekst omzetten.

Bijvoorbeeld:

```text
naam=Arin
gezondheid=80
goud=125
```

En bij het laden moeten we die tekst weer interpreteren.

---

## 12. Een savegame regel voor regel

We kunnen de tekst opsplitsen met:

```rust,ignore
lines()
```

Bijvoorbeeld:

```rust,ignore
let tekst = "naam=Arin\ngoud=125";

for regel in tekst.lines() {
    println!("{regel}");
}
```

De uitvoer is:

```text
naam=Arin
goud=125
```

Dit is handig omdat onze savegame uit meerdere regels bestaat.

---

## 13. Een regel splitsen

We kunnen een regel splitsen met `split_once`.

Bijvoorbeeld:

```rust,ignore
let regel = "naam=Arin";

if let Some((sleutel, waarde)) = regel.split_once('=') {
    println!("Sleutel: {sleutel}");
    println!("Waarde: {waarde}");
}
```

De uitvoer:

```text
Sleutel: naam
Waarde: Arin
```

Dit geeft opnieuw een `Option`.

Als `=` niet voorkomt, krijgen we:

```text
None
```

---

## 14. Een getal uit tekst halen

In een savegame staat:

```text
gezondheid=80
```

Maar `80` is daar onderdeel van een `String`.

We moeten het dus omzetten naar een `i32`.

Daarvoor hebben we:

```rust,ignore
parse()
```

Bijvoorbeeld:

```rust,ignore
let tekst = "80";

let gezondheid: i32 = tekst.parse().unwrap();
```

`parse()` kan mislukken.

Daarom geeft `parse()` een `Result`.

Een veiligere variant is:

```rust,ignore
let gezondheid: i32 = match tekst.parse() {
    Ok(waarde) => waarde,
    Err(_) => 0,
};
```

Later kunnen we deze foutafhandeling verder verbeteren.

---

## 15. Een savegame inladen

We kunnen nu een eenvoudige loader schrijven:

```rust,ignore
fn laad_speler(tekst: &str) -> Result<(String, i32, i32), String> {
    let mut naam = String::new();
    let mut gezondheid = 0;
    let mut goud = 0;

    for regel in tekst.lines() {
        let Some((sleutel, waarde)) = regel.split_once('=') else {
            return Err(String::from("Ongeldige regel."));
        };

        match sleutel {
            "naam" => naam = String::from(waarde),

            "gezondheid" => {
                gezondheid = waarde
                    .parse()
                    .map_err(|_| String::from("Ongeldige gezondheid."))?;
            }

            "goud" => {
                goud = waarde
                    .parse()
                    .map_err(|_| String::from("Ongeldig goud."))?;
            }

            _ => {}
        }
    }

    Ok((naam, gezondheid, goud))
}
```

Dit is al een behoorlijk realistisch stukje Rust.

We gebruiken hier:

- `String`
- `&str`
- `lines()`
- `Option`
- `Result`
- `match`
- `parse`
- `?`
- een tuple.

We hoeven nog niet alles tegelijk te beheersen. Het belangrijkste is dat je kunt herkennen welke onderdelen samenwerken.

---

## 16. Van tuple naar struct

Voor een echte RPG willen we liever een `Speler` teruggeven dan drie losse waarden.

We kunnen daarom:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
    goud: i32,
}
```

gebruiken.

De functie wordt dan:

```rust,ignore
fn laad_speler(tekst: &str) -> Result<Speler, String> {
    // ...
}
```

En uiteindelijk:

```rust,ignore
Ok(Speler {
    naam,
    gezondheid,
    goud,
})
```

Nu kunnen we het resultaat gebruiken als normale `Speler`.

---

## 17. Een volledige eenvoudige save/load

Een eerste complete versie:

```rust
use std::fs;

struct Speler {
    naam: String,
    gezondheid: i32,
    goud: i32,
}

fn sla_speler_op(speler: &Speler) -> Result<(), std::io::Error> {
    let tekst = format!(
        "naam={}\ngezondheid={}\ngoud={}",
        speler.naam,
        speler.gezondheid,
        speler.goud
    );

    fs::write("save.txt", tekst)?;

    Ok(())
}

fn main() -> Result<(), std::io::Error> {
    let speler = Speler {
        naam: String::from("Arin"),
        gezondheid: 80,
        goud: 125,
    };

    sla_speler_op(&speler)?;

    println!("Spel opgeslagen.");

    Ok(())
}
```

Dit is nog geen complete savegame, maar het belangrijkste principe werkt.

---

## 18. `std::io::Error`

Tot nu toe hebben we:

```rust,ignore
Result<(), std::io::Error>
```

gezien.

`std::io::Error` is een type dat gebruikt wordt voor fouten bij input en output.

Bijvoorbeeld:

- bestand kan niet worden geopend
- bestand kan niet worden geschreven
- bestand bestaat niet
- gegevens kunnen niet worden gelezen.

Je hoeft de interne structuur van `std::io::Error` voorlopig niet te kennen.

Je moet vooral herkennen:

```text
Result<T, std::io::Error>
```

als:

> Deze bewerking kan slagen, maar ook een bestand-/I/O-fout opleveren.

---

## 19. Een bestand openen

`fs::read_to_string` en `fs::write` zijn handige eenvoudige functies.

Soms willen we meer controle.

Dan kunnen we `File` gebruiken:

```rust,ignore
use std::fs::File;
```

Een bestand openen:

```rust,ignore
let bestand = File::open("save.txt")?;
```

Ook dit geeft een `Result`.

We kunnen bijvoorbeeld schrijven:

```rust,ignore
let bestand = File::open("save.txt");

match bestand {
    Ok(_) => println!("Bestand geopend."),
    Err(fout) => println!("Openen mislukt: {fout}"),
}
```

Voor onze eenvoudige savegame is `read_to_string` meestal overzichtelijker.

---

## 20. Wanneer gebruik je `File`?

`File` wordt vooral interessant wanneer je meer controle wilt over het lezen of schrijven.

Bijvoorbeeld:

- grote bestanden
- gegevens stukje voor stukje lezen
- meerdere schrijfoperaties
- specifieke bestandopties.

Voor onze eerste RPG-savegame gebruiken we voorlopig:

```rust,ignore
fs::read_to_string(...)
fs::write(...)
```

Dat houdt de aandacht bij het belangrijkste concept.

---

## 21. Bestanden en `HashMap`

Onze RPG heeft inmiddels een inventaris:

```rust,ignore
HashMap<String, i32>
```

We zouden die ook kunnen opslaan.

Bijvoorbeeld:

```text
naam=Arin
gezondheid=80
goud=125
item=Potion:3
item=Brood:5
item=Pijl:12
```

Tijdens het laden kunnen we:

1. het bestand lezen
2. iedere regel verwerken
3. gewone spelergegevens herkennen
4. inventarisregels herkennen
5. items toevoegen aan een `HashMap`.

Daarmee beginnen onze verschillende Rust-onderwerpen echt samen te werken.

---

## 22. Een eenvoudig inventarisformaat

Stel dat we dit bestand hebben:

```text
naam=Arin
gezondheid=80
goud=125
item=Potion:3
item=Brood:5
item=Pijl:12
```

Een inventarisregel:

```text
item=Potion:3
```

kunnen we eerst splitsen op `=`:

```text
item
Potion:3
```

Daarna kunnen we `Potion:3` splitsen op `:`:

```text
Potion
3
```

Dit soort tekstverwerking is een goede oefening in het combineren van:

- `&str`
- `String`
- `split_once`
- `parse`
- `Option`
- `Result`
- `HashMap`.

---

## 23. Fouten bij savegames

Een echte savegame kan beschadigd zijn.

Bijvoorbeeld:

```text
naam=Arin
gezondheid=appel
goud=125
```

`appel` kan niet naar `i32` worden omgezet.

Of:

```text
naam=Arin
gezondheid=80
dit klopt niet
goud=125
```

Ook dat is geen geldig formaat.

Daarom moet een loader niet blind vertrouwen op de inhoud van een bestand.

Een goede loader geeft een fout terug.

Bijvoorbeeld:

```rust,ignore
fn laad_speler(tekst: &str) -> Result<Speler, String>
```

Dat betekent:

```text
Ok(Speler)
```

of:

```text
Err(String)
```

Dit is precies waarvoor `Result` bedoeld is.

---

## 24. `unwrap` bij bestanden

Je zult soms code tegenkomen zoals:

```rust,ignore
let tekst = fs::read_to_string("save.txt").unwrap();
```

Dat is toegestaan.

Maar als het bestand niet bestaat, stopt het programma met een panic.

Voor kleine oefeningen kan `unwrap()` handig zijn.

Voor onze RPG willen we liever expliciet omgaan met fouten:

```rust,ignore
let tekst = fs::read_to_string("save.txt")?;
```

of:

```rust,ignore
match fs::read_to_string("save.txt") {
    Ok(tekst) => {
        // laden
    }

    Err(fout) => {
        // fout afhandelen
    }
}
```

Een savegame is immers een plek waar fouten daadwerkelijk kunnen voorkomen.

---

## 25. Een apart savebestand

Het is verstandig om de savegame niet zomaar naast willekeurige bestanden te plaatsen.

Voorlopig kunnen we eenvoudig gebruiken:

```text
save.txt
```

Later, wanneer we de projectstructuur verbeteren, kunnen we bijvoorbeeld een aparte map gebruiken:

```text
saves/
├── slot1.txt
├── slot2.txt
└── slot3.txt
```

Dan kunnen we meerdere spelstanden bewaren.

---

## 26. Save slots

Een functie kan bijvoorbeeld een slotnummer ontvangen:

```rust,ignore
fn save_pad(slot: u32) -> String {
    format!("saves/slot{slot}.txt")
}
```

Dan:

```rust,ignore
let pad = save_pad(1);
println!("{pad}");
```

geeft:

```text
saves/slot1.txt
```

Let op: als de map `saves` niet bestaat, kan `fs::write` niet zomaar het bestand maken.

We moeten dan eerst de map maken.

---

## 27. Een map maken

Daarvoor bestaat:

```rust,ignore
fs::create_dir_all("saves")?;
```

Bijvoorbeeld:

```rust
use std::fs;

fn main() -> Result<(), std::io::Error> {
    fs::create_dir_all("saves")?;

    fs::write("saves/slot1.txt", "naam=Arin")?;

    Ok(())
}
```

`create_dir_all` maakt de benodigde mappen aan.

Bestaat de map al, dan is dat geen probleem.

---

## 28. Een savegame met een slot

We kunnen nu:

```rust,ignore
use std::fs;

fn sla_op(slot: u32, tekst: &str) -> Result<(), std::io::Error> {
    fs::create_dir_all("saves")?;

    let pad = format!("saves/slot{slot}.txt");

    fs::write(pad, tekst)?;

    Ok(())
}
```

Gebruik:

```rust,ignore
fn main() -> Result<(), std::io::Error> {
    sla_op(1, "naam=Arin")?;

    println!("Slot 1 opgeslagen.");

    Ok(())
}
```

Dit is een mooie eerste stap naar een echte savegame.

---

## 29. Een savegame laden uit een slot

Daarbij kunnen we dezelfde naamgeving gebruiken:

```rust,ignore
fn laad_op(slot: u32) -> Result<String, std::io::Error> {
    let pad = format!("saves/slot{slot}.txt");

    let tekst = std::fs::read_to_string(pad)?;

    Ok(tekst)
}
```

Dan:

```rust,ignore
fn main() -> Result<(), std::io::Error> {
    let tekst = laad_op(1)?;

    println!("{tekst}");

    Ok(())
}
```

---

## 30. Wat hebben we geleerd?

We kunnen nu:

### Een bestand schrijven

```rust,ignore
fs::write("bestand.txt", "tekst")?;
```

### Een bestand lezen

```rust,ignore
let tekst = fs::read_to_string("bestand.txt")?;
```

### Een map maken

```rust,ignore
fs::create_dir_all("saves")?;
```

### Een tekst maken

```rust,ignore
let tekst = format!("naam={naam}");
```

### Regels lezen

```rust,ignore
for regel in tekst.lines() {
    // ...
}
```

### Een regel splitsen

```rust,ignore
let Some((sleutel, waarde)) = regel.split_once('=') else {
    // fout
};
```

### Tekst naar een getal omzetten

```rust,ignore
let getal: i32 = tekst.parse()?;
```

### Fouten doorgeven

```rust,ignore
fn laad() -> Result<String, std::io::Error> {
    let tekst = fs::read_to_string("save.txt")?;

    Ok(tekst)
}
```

---

## 31. Compilerfouten, tests en runtimefouten

We hebben inmiddels drie verschillende soorten problemen gezien.

## Compilerfout

De code is niet geldig Rust.

Bijvoorbeeld:

```rust,ignore
let aantal: i32 = "drie";
```

De compiler stopt voordat het programma wordt uitgevoerd.

---

## Testfout

De code compileert, maar een automatische test vindt dat het gedrag niet klopt.

Bijvoorbeeld:

```rust,ignore
assert_eq!(bereken_schade(10), 15);
```

als de functie `10` teruggeeft.

---

## Runtimefout

Het programma compileert en begint te draaien, maar tijdens het uitvoeren gebeurt iets onverwachts.

Bij bestanden kan bijvoorbeeld:

```text
save.txt bestaat niet
```

een runtimeprobleem opleveren.

Met `Result` kunnen we zulke verwachte fouten gecontroleerd afhandelen.

Dit onderscheid blijft belangrijk naarmate ons RPG-project groter wordt.

---

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 26](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_26/).

