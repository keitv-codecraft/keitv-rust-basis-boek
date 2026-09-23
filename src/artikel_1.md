# 1. Kennismaken met programmeren en Rust

## Wat gaan we leren?

In dit artikel maken we voor het eerst kennis met programmeren en met de programmeertaal Rust.

Na dit artikel kun je:

- uitleggen wat een programma is;
- een eenvoudig Rust-programma herkennen;
- een Rust-programma uitvoeren;
- tekst op het scherm laten zien;
- eenvoudige veranderingen in een programma maken;
- begrijpen wat de compiler doet;
- een eenvoudige compilerfout herkennen en proberen op te lossen.

Je hoeft nog geen ervaring met programmeren te hebben.

---

## 1. Wat is programmeren?

Een computer is heel goed in het uitvoeren van opdrachten. Hij kan bijvoorbeeld:

- getallen optellen;
- tekst laten zien;
- bestanden opslaan;
- afbeeldingen tekenen;
- geluid afspelen;
- een spel uitvoeren.

Maar een computer bedenkt meestal niet zelf welke opdrachten hij moet uitvoeren.

Wij moeten hem vertellen wat hij moet doen.

Een **programma** is een verzameling opdrachten voor een computer.

Bijvoorbeeld:

```rust
println!("Hallo!");
```

Deze opdracht betekent ongeveer:

> Laat de tekst `Hallo!` op het scherm zien.

Een programmeertaal is een taal waarin we zulke opdrachten kunnen opschrijven.

Rust is één van die programmeertalen.

---

## 2. Waarom gebruiken we Rust?

Rust is een programmeertaal waarmee we allerlei soorten programma's kunnen maken.

Bijvoorbeeld:

- computersoftware;
- servers;
- programma's voor kleine apparaten;
- gereedschappen voor programmeurs;
- games;
- onderdelen van grotere programma's.

Rust probeert programma's tegelijk **snel** en **veilig** te maken.

Dat betekent onder andere dat Rust veel fouten al controleert voordat het programma wordt uitgevoerd.

Dat kan in het begin soms lastig lijken.

Je krijgt namelijk regelmatig een foutmelding van Rust wanneer je iets verkeerd hebt geschreven.

Dat is normaal.

Een compilerfout betekent niet:

> Ik kan niet programmeren.

Het betekent meestal:

> Rust heeft iets gevonden dat we eerst moeten oplossen.

Fouten maken hoort bij programmeren.

---

## 3. Ons eerste programma

Een nieuw Rust-programma bevat meestal een functie met de naam `main`.

Een functie is een stukje programma dat een bepaalde taak uitvoert. We leren later veel meer over functies.

Voor nu is het genoeg om te weten dat `main` het startpunt van ons programma is.

Ons eerste programma kan er zo uitzien:

```rust
fn main() {
    println!("Hallo wereld!");
}
```

Laten we dit stap voor stap bekijken.

### `fn`

`fn` vertelt Rust dat we een functie gaan maken.

### `main`

`main` is de naam van onze functie.

Een Rust-programma begint met het uitvoeren van deze functie.

### `{` en `}`

De accolades geven aan waar de functie begint en eindigt.

Alles tussen deze twee tekens hoort bij de functie.

### `println!`

`println!` zorgt ervoor dat Rust iets op het scherm laat zien.

### `"Hallo wereld!"`

Dit is de tekst die we willen laten zien.

### `;`

De puntkomma geeft hier het einde van de opdracht aan.

We komen later nog situaties tegen waarin een puntkomma anders werkt. Voor nu kun je onthouden:

> Veel opdrachten in Rust eindigen met een puntkomma.

---

## 4. Ons programma uitvoeren

Een Rust-programma moet eerst worden **gecompileerd**.

Compileren betekent dat Rust onze broncode controleert en omzet naar een programma dat de computer kan uitvoeren.

Wanneer we met Cargo werken, kunnen we ons programma uitvoeren met:

```text
cargo run
```

Cargo zorgt er onder andere voor dat Rust onze code compileert en daarna het programma uitvoert.

Als ons programma dit bevat:

```rust
fn main() {
    println!("Hallo wereld!");
}
```

dan krijgen we:

```text
Hallo wereld!
```

te zien.

---

## 5. Zelf iets veranderen

Programmeren leer je vooral door dingen uit te proberen.

Verander:

```rust
println!("Hallo wereld!");
```

in:

```rust
println!("Hallo Rust!");
```

Voer het programma opnieuw uit.

Je zou nu moeten zien:

```text
Hallo Rust!
```

Verander daarna de tekst bijvoorbeeld in:

```rust
println!("Ik ga games maken!");
```

Je hebt zojuist je eerste programma aangepast.

Dat lijkt misschien heel eenvoudig.

Dat is het ook.

En dat is precies de bedoeling.

Bij programmeren bouwen we vaak ingewikkelde programma's op uit heel veel kleine stappen.

---

## 6. Meerdere opdrachten

We kunnen meerdere opdrachten onder elkaar zetten.

Bijvoorbeeld:

```rust,ignore
fn main() {
    println!("Welkom bij mijn game!");
    println!("Je hebt 3 levens.");
    println!("Veel succes!");
}
```

Het programma voert de opdrachten van boven naar beneden uit.

Het resultaat is:

```text
Welkom bij mijn game!
Je hebt 3 levens.
Veel succes!
```

Dit is een belangrijk idee:

> Een programma voert opdrachten uit volgens de volgorde die wij hebben aangegeven.

Later leren we hoe we die volgorde kunnen veranderen.

Een programma kan bijvoorbeeld zeggen:

> Als de speler dood is, laat dan "Game over!" zien.

Of:

> Blijf vijanden maken zolang het level nog niet voorbij is.

Dat noemen we **control flow**. Dat behandelen we later.

---

## 7. Een eerste game

We kunnen `println!` gebruiken om alvast een heel eenvoudige game te maken.

```rust
fn main() {
    println!("======================");
    println!("     RUST ADVENTURE");
    println!("======================");

    println!("Je staat voor een donkere grot.");
    println!("Je hebt een zwaard bij je.");
    println!("Wat ga je doen?");
}
```

Dit is natuurlijk nog geen echte game.

De computer doet alleen precies wat wij hem hebben opgedragen.

Toch hebben we hiermee al iets belangrijks geleerd:

> Een game is uiteindelijk ook gewoon een programma.

Een echte game bestaat uit heel veel meer onderdelen, zoals:

- invoer van de speler;
- afbeeldingen;
- geluid;
- beweging;
- game-logica;
- vijanden;
- levels;
- scores.

Maar ook een grote game bestaat uiteindelijk uit heel veel kleine opdrachten die de computer uitvoert.

---

## 8. Wat doet de compiler?

De code die wij schrijven heet **broncode**.

De computer kan onze Rust-broncode niet zomaar rechtstreeks uitvoeren.

Daarom gebruiken we een **compiler**.

De compiler leest onze Rust-code en controleert bijvoorbeeld:

- staat de code correct geschreven?
- gebruiken we de juiste Rust-regels?
- zijn namen correct gespeld?
- kloppen de soorten gegevens die we gebruiken?

Als alles goed is, kan Rust de code omzetten naar een uitvoerbaar programma.

Je kunt de compiler dus zien als een soort controleur.

Wij schrijven:

```text
Rust-broncode
      ↓
    compiler
      ↓
programma dat de computer kan uitvoeren
```

De compiler is daarbij niet onze vijand.

Sterker nog: een compiler die een fout ontdekt, helpt ons om betere programma's te maken.

---

## 9. Een fout maken

Laten we expres een fout maken.

Schrijf:

```rust,ignore
fn main() {
    println!("Hallo!)
}
```

Voer het programma uit.

Rust geeft een foutmelding.

Dat is logisch: we zijn het afsluitende aanhalingsteken vergeten.

We moeten schrijven:

```rust
fn main() {
    println!("Hallo!");
}
```

Probeer daarna opnieuw.

Dit is een belangrijk onderdeel van programmeren:

1. We schrijven code.
2. We proberen de code uit te voeren.
3. Rust vindt misschien een fout.
4. We bekijken de foutmelding.
5. We passen de code aan.
6. We proberen het opnieuw.

Dit proces herhalen we heel vaak.

Zelfs ervaren programmeurs maken voortdurend fouten.

---

## 10. De compiler vertelt waar het probleem zit

Compilerfouten kunnen in het begin nogal indrukwekkend lijken.

Bijvoorbeeld:

```text
error: expected `,`, found `}`
```

Je hoeft zo'n foutmelding niet meteen volledig te begrijpen.

Kijk eerst naar de plek waar Rust het probleem heeft gevonden.

Rust geeft meestal aan:

- in welk bestand het probleem zit;
- op welke regel;
- waar ongeveer het probleem zit;
- soms wat er volgens Rust mis is.

Leer daarom eerst rustig de foutmelding te lezen.

Je hoeft niet alle Engelse woorden te kennen om ermee te kunnen werken.

Een goede eerste vraag is:

> Op welke regel denkt Rust dat er iets fout gaat?

Daarna kun je de code rond die regel bekijken.

---

## 11. Experimenteren

Een goede programmeur is niet iemand die alles uit zijn hoofd weet.

Een goede programmeur kan iets proberen en vervolgens onderzoeken wat er gebeurt.

Probeer bijvoorbeeld eens:

```rust
fn main() {
    println!("Hallo!");
    println!("Dit is mijn eerste programma.");
    println!("Rust is interessant.");
}
```

Verander daarna:

- de teksten;
- de volgorde;
- het aantal regels;
- hoofdletters;
- leestekens.

Kijk steeds wat er gebeurt.

Je kunt zelfs expres fouten maken.

Bijvoorbeeld:

```rust,ignore
fn main() {
    println!("Hallo!");
    println!("Dit is een fout.)
    println!("Of toch niet?");
}
```

Kijk vervolgens naar de foutmelding en probeer het probleem zelf te vinden.

---

## 12. Wat heb je geleerd?

Je weet nu:

- Een **programma** is een verzameling opdrachten voor een computer.
- **Rust** is een programmeertaal.
- `main` is het startpunt van een Rust-programma.
- `println!` kan tekst op het scherm laten zien.
- `cargo run` kan een Rust-programma bouwen en uitvoeren.
- De **compiler** controleert onze Rust-code.
- Compilerfouten zijn normaal bij het programmeren.
- Programmeren bestaat voor een groot deel uit schrijven, uitvoeren, fouten vinden en verbeteren.

Je hoeft nog niet alles te begrijpen.

In de volgende artikelen gaan we stap voor stap nieuwe mogelijkheden toevoegen.

Uiteindelijk kunnen we met deze kleine bouwstenen echte game-logica maken.

---

## Oefeningen

Aan het einde van ieder artikel vind je oefeningen. Probeer deze allemaal te doen om het programmeren in de vingers te krijgen.

Kom je er niet uit? Kijk dan terug of vraag om hulp.

## Opzet

### Rustlings

Voor de oefeningen gebruiken we [Rustlings](https://rustlings.rust-lang.org/). Als je dit nog niet gedaan hebt kun je het installeren met

```bash
cargo install rustlings
```

### KeiTV Rustlings oefeningen

Haal om te beginnen alle Rustlings opgaven op van alle artikelen. Dit hoef je dus ook maar één keer te doen.
Navigeer eerst naar je projectmap en gebruik daarna `git` om de oefeningen naar een nieuwe map te downloaden.

```bash
cd C:\\projects
git clone git@github.com:keitv-codecraft/keitv-rust-basis-rustlings.git
```

De repository is ook te vinden op [github.com/keitv-codecraft/keitv-rust-basis-rustlings](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings).

## Aan de slag met de oefeningen

Navigeer naar de KeiTV Rustlings map en start vanuit daar VsCodium en `rustlings`.

```bash
cd C:\\projects\\keitv-rustlings
codium .
rustlings
```

Je kunt nu werken aan de oefeningen bij dit artikel, zoals je gewend bent van de standaard Rustlings oefeningen.

Wanneer je aangekomen bent bij de eerste oefening van het volgende artikel kom je hier terug en lees je het volgende artikel. Zo ga je door tot alle artikelen zijn gedaan.

---

## Controlelijst

Je bent klaar met dit artikel als je zonder hulp:

- [ ] een Rust-project kunt uitvoeren
- [ ] `fn main()` herkent
- [ ] weet wat `println!` doet
- [ ] meerdere regels tekst kunt afdrukken
- [ ] begrijpt wat een compiler doet
- [ ] een eenvoudige compilerfout kunt zoeken
- [ ] zelf kleine veranderingen in een programma durft te maken.

Als iets nog niet lukt, is dat geen probleem. Herhaal vooral de oefeningen waarbij je nog hulp nodig hebt.
