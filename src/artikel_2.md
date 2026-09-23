# 2. Cargo: Rust-projecten maken en dependencies gebruiken

## Wat gaan we leren?

In het vorige artikel hebben we een klein Rust-programma gemaakt.

Nu gaan we leren hoe we van zo'n programma een echt Rust-project maken.

Daarvoor gebruiken we **Cargo**.

> [!TIP]
> Werk vanuit de map van je project. Cargo zoekt daar naar `Cargo.toml`.

Na dit artikel kun je:

- uitleggen wat Cargo is
- een nieuw Rust-project maken
- de belangrijkste bestanden en mappen van een project herkennen
- een programma uitvoeren met `cargo run`
- een project bouwen met `cargo build`
- begrijpen wat `Cargo.toml` is
- een dependency toevoegen
- een dependency in een programma gebruiken
- begrijpen waarom dependencies handig zijn.

---

## 1. Wat is Cargo?

Een Rust-programma bestaat al snel uit meerdere bestanden.

We willen bijvoorbeeld een game maken met:

- spelers
- vijanden
- levels
- geluid
- afbeeldingen
- instellingen.

Dan wordt het onhandig om alles in één bestand te zetten.

We hebben daarom een goede manier nodig om een Rust-project te organiseren.

Daar helpt **Cargo** ons bij.

Cargo is het standaard hulpmiddel voor het maken en beheren van Rust-projecten.

Cargo kan onder andere:

- nieuwe projecten maken
- onze code compileren
- programma's uitvoeren
- dependencies beheren
- testen uitvoeren
- informatie over een project bijhouden.

Je kunt Cargo zien als de **projectmanager van een Rust-programma**.

---

## 2. Een nieuw project maken

Open een terminal.

Ga naar de map waarin je je Rust-projecten wilt bewaren.

Maak daarna een nieuw project:

```text
cargo new mijn_game
```

Cargo maakt nu een nieuwe map met de naam:

```text
mijn_game
```

Ga naar die map:

```text
cd mijn_game
```

Je kunt nu de inhoud bekijken.

Het project ziet er ongeveer zo uit:

```text
mijn_game/
├── Cargo.toml
└── src/
    └── main.rs
```

Cargo heeft dus al een eenvoudige projectstructuur voor ons gemaakt.

Dat is handig.

We hoeven niet iedere keer zelf alle bestanden en mappen te maken.

---

## 3. `main.rs`

Open:

```text
src/main.rs
```

Daar staat waarschijnlijk:

```rust
fn main() {
    println!("Hello, world!");
}
```

Dit is het programma dat we in het vorige artikel hebben gezien.

Verander het bijvoorbeeld in:

```rust
fn main() {
    println!("Welkom bij mijn game!");
}
```

---

## 4. Het programma uitvoeren

We kunnen ons programma uitvoeren met:

```text
cargo run
```

Cargo doet dan een aantal dingen voor ons.

In eenvoudige vorm gebeurt dit:

```text
┌──────────────────┐
│    Rust-code     │
└────────┬─────────┘
         ▼
┌──────────────────┐
│      Cargo       │
└────────┬─────────┘
         ▼
┌──────────────────┐
│  Rust compiler   │
└────────┬─────────┘
         ▼
┌──────────────────┐
│    Programma     │
│   (uitvoeren)    │
└──────────────────┘
```

Als alles goed gaat, zien we bijvoorbeeld:

```text
Welkom bij mijn game!
```

Cargo onthoudt ook wat er al gebouwd is.

Daardoor hoeft niet altijd alles opnieuw vanaf nul gebouwd te worden.

---

## 5. `cargo build`

Met:

```text
cargo build
```

vertellen we Cargo dat het project gebouwd moet worden.

Cargo controleert en compileert onze code.

Het programma wordt daarna opgeslagen in een map zoals:

```text
target/debug/
```

Je hoeft normaal gesproken niet zelf in deze map te werken.

Voor dagelijks programmeren kun je meestal gewoon:

```text
cargo run
```

gebruiken.

---

## 6. `Cargo.toml`

Naast `src` heeft ons project een bestand met een bijzondere naam:

```text
Cargo.toml
```

Dit bestand bevat informatie **over ons project**.

Open het eens.

Je ziet bijvoorbeeld iets zoals:

```toml
[package]
name = "mijn_game"
version = "0.1.0"
edition = "2024"

[dependencies]
```

Hier staan verschillende instellingen.

We bekijken eerst de belangrijkste.

---

### `[package]`

Dit gedeelte beschrijft ons project.

Bijvoorbeeld:

```toml
[package]
name = "mijn_game"
version = "0.1.0"
edition = "2024"
```

De naam van het project is:

```text
mijn_game
```

De versie is:

```text
0.1.0
```

En `edition` geeft aan welke Rust Edition het project gebruikt.

Voor nu hoef je de betekenis van versienummers en editions nog niet helemaal te kennen.

Het belangrijkste is:

> `Cargo.toml` bevat instellingen en informatie over ons Rust-project.

---

## 7. Wat is een dependency?

Stel dat we een game maken.

We willen daarin willekeurige getallen gebruiken.

Bijvoorbeeld:

```text
De vijand doet 7 schade.
```

De volgende keer misschien:

```text
De vijand doet 12 schade.
```

En daarna:

```text
De vijand doet 4 schade.
```

Willekeurige getallen zijn heel handig in games.

Rust heeft veel mogelijkheden ingebouwd, maar voor sommige taken bestaan er ook externe bibliotheken.

Zo'n externe bibliotheek noemen we in Rust meestal een **crate**.

Een crate kan bijvoorbeeld code bevatten voor:

- willekeurige getallen
- afbeeldingen
- geluid
- netwerkverbindingen
- gameontwikkeling
- databases.

Wanneer ons project zo'n crate gebruikt, noemen we die crate een **dependency**.

Een dependency is dus iets waar ons programma afhankelijk van is.

---

## 8. De crate `rand`

Voor willekeurige getallen kunnen we bijvoorbeeld de crate `rand` gebruiken.

We kunnen Cargo vragen om deze dependency toe te voegen:

```text
cargo add rand
```

Cargo past dan automatisch `Cargo.toml` aan.

Bijvoorbeeld:

```toml
[dependencies]
rand = "..."
```

Het exacte versienummer kan verschillen.

Dat is normaal.

Cargo zorgt er bovendien voor dat de benodigde code wordt opgehaald en beschikbaar wordt gemaakt voor ons project.

---

## 9. Onze eerste dobbelsteen

We kunnen `rand` gebruiken om een willekeurig getal te maken.

Maak in `src/main.rs` bijvoorbeeld:

```rust,ignore
fn main() {
    let worp = rand::random_range(1..=6);

    println!("Je gooide {worp}!");
}
```

Voer het programma uit:

```text
cargo run
```

Je zou bijvoorbeeld kunnen krijgen:

```text
Je gooide 4!
```

Voer het nog een keer uit.

Misschien krijg je:

```text
Je gooide 1!
```

En nog een keer:

```text
Je gooide 6!
```

De computer kiest iedere keer een willekeurig getal tussen 1 en 6.

We hebben nu een heel eenvoudige dobbelsteen gemaakt.

---

## 10. Wat gebeurt hier?

Bekijk deze regel:

```rust,ignore
let worp = rand::random_range(1..=6);
```

Hier gebeurt eigenlijk al behoorlijk veel.

We maken een variabele:

```rust,ignore
worp
```

Daarin bewaren we het resultaat van:

```rust,ignore
rand::random_range(1..=6)
```

De `rand`-crate levert de functie die we gebruiken om het willekeurige getal te maken.

Het gedeelte:

```text
1..=6
```

betekent dat de mogelijke getallen van 1 tot en met 6 lopen.

We behandelen bereiken (`..` en `..=`) later uitgebreider.

Voor nu is het voldoende om te onthouden:

> `1..=6` betekent hier: van 1 tot en met 6.

---

## 11. Waarom gebruiken we een dependency?

Je zou je kunnen afvragen:

> Waarom schrijven we dit niet gewoon zelf?

Dat kan soms.

Maar programmeurs hoeven niet alles zelf te bouwen.

Stel dat je een game maakt en je hebt een goed onderhouden bibliotheek nodig voor een bepaalde taak.

Dan kun je die bibliotheek gebruiken in plaats van zelf honderden of duizenden regels code te schrijven.

Dependencies kunnen daardoor veel werk besparen.

Een ander voordeel is dat veel andere programmeurs dezelfde code kunnen gebruiken.

Maar dependencies hebben ook nadelen.

Ons programma wordt afhankelijk van externe code.

Daarom moeten we nadenken over:

- welke dependencies we gebruiken
- welke versies we gebruiken
- of een dependency betrouwbaar is
- of een dependency nog onderhouden wordt.

Voor kleine oefeningen hoef je je daar nog niet druk over te maken.

---

## 12. `Cargo.lock`

Als je `cargo run` uitvoert nadat je een dependency hebt toegevoegd, verschijnt er meestal ook een bestand:

```text
Cargo.lock
```

Dit bestand bevat informatie over de exacte versies van dependencies die Cargo voor dit project heeft gekozen.

Dat is belangrijk.

Stel dat een project vandaag werkt met een bepaalde versie van een dependency.

We willen niet dat morgen automatisch een heel andere versie wordt gebruikt die zich anders gedraagt.

`Cargo.lock` helpt Cargo om dezelfde dependency-versies te blijven gebruiken.

Voor een programma dat je zelf bouwt, is het meestal verstandig om `Cargo.lock` te bewaren.

Je hoeft dit bestand voorlopig niet zelf te bewerken.

> Laat Cargo het bestand beheren.

---

## 13. `src` en `target`

Ons project ziet er inmiddels ongeveer zo uit:

```text
mijn_game/
├── Cargo.lock
├── Cargo.toml
├── src/
│   └── main.rs
└── target/
```

De belangrijkste onderdelen zijn:

### `Cargo.toml`

Hier staan projectinformatie en dependencies.

### `Cargo.lock`

Hier houdt Cargo onder andere bij welke exacte dependency-versies voor het project zijn vastgelegd.

### `src/`

Hier staat onze broncode.

### `src/main.rs`

Hier begint ons programma.

### `target/`

Hier zet Cargo onder andere de bestanden neer die tijdens het bouwen ontstaan.

Je hoeft normaal gesproken niet zelf bestanden in `target` te maken of te wijzigen.

---

## 14. Een eenvoudige game met een dependency

Laten we onze dobbelsteen gebruiken voor een klein spelletje.

```rust,ignore
fn main() {
    let speler_worp = rand::random_range(1..=6);
    let vijand_worp = rand::random_range(1..=6);

    println!("Jij gooide {speler_worp}.");
    println!("De vijand gooide {vijand_worp}.");
}
```

Voer het programma meerdere keren uit.

We hebben nu de eerste bouwsteen van een game:

```text
speler → gooit dobbelsteen
vijand  → gooit dobbelsteen
```

Later leren we hoe we kunnen bepalen wie er gewonnen heeft.

---

## 15. Zelf experimenteren

Probeer de grenzen te veranderen.

Van:

```rust,ignore
rand::random_range(1..=6)
```

naar:

```rust,ignore
rand::random_range(1..=20)
```

Nu hebben we een twintigzijdige dobbelsteen.

Of:

```rust,ignore
rand::random_range(10..=20)
```

Nu zijn de mogelijke resultaten 10 tot en met 20.

Experimenteer ook met meerdere worpen.

```rust,ignore
fn main() {
    let worp1 = rand::random_range(1..=6);
    let worp2 = rand::random_range(1..=6);
    let worp3 = rand::random_range(1..=6);

    println!("Worp 1: {worp1}");
    println!("Worp 2: {worp2}");
    println!("Worp 3: {worp3}");
}
```

We zullen later leren hoe we dit veel netter kunnen schrijven.

---

## 16. Wat gebeurt er als Cargo een fout vindt?

Maak expres een fout.

Bijvoorbeeld:

```rust,ignore
fn main() {
    let worp = rand::random_range(1..=6

    println!("Je gooide {worp}!");
}
```

Voer uit:

```text
cargo run
```

Rust geeft een foutmelding.

Dat is niet erg.

Kijk naar de melding en probeer te vinden wat er ontbreekt.

In dit geval ontbreekt bijvoorbeeld:

```text
);
```

De juiste versie is:

```rust,ignore
fn main() {
    let worp = rand::random_range(1..=6);

    println!("Je gooide {worp}!");
}
```

Cargo verandert dus niets aan het feit dat programmeren uit proberen, fouten vinden en verbeteren bestaat.

Cargo maakt dat proces alleen veel gemakkelijker.

---

## 17. Veelgebruikte Cargo-opdrachten

Voorlopig zijn deze opdrachten het belangrijkst:

| Opdracht | Wat doet het? |
| --- | --- |
| `cargo new naam` | Maakt een nieuw project |
| `cargo run` | Bouwt en voert het programma uit |
| `cargo build` | Bouwt het programma |
| `cargo check` | Controleert de code zonder een uitvoerbaar programma te bouwen |
| `cargo add naam` | Voegt een dependency toe |
| `cargo test` | Voert automatische tests uit |

De laatste twee onderwerpen behandelen we later uitgebreider.

Een bijzonder handige opdracht is:

```text
cargo check
```

Daarmee kunnen we snel controleren of onze code compileert.

Er wordt geen volledig uitvoerbaar programma gemaakt.

Voor grote projecten kan dat sneller zijn dan `cargo build`.

---

## 18. Wat heb je geleerd?

Je weet nu:

- wat Cargo is
- hoe je een Rust-project maakt
- waar `main.rs` staat
- waar `Cargo.toml` voor dient
- wat een dependency is
- wat een crate is
- hoe je een dependency toevoegt
- hoe je een programma uitvoert met `cargo run`
- hoe je een project bouwt met `cargo build`
- hoe je de code controleert met `cargo check`
- wat `Cargo.lock` ongeveer doet
- waarom dependencies handig kunnen zijn.

Je hebt bovendien een klein stukje gamefunctionaliteit gemaakt: een dobbelsteen.

---

## Oefeningen

Doe de volgende oefeningen voordat je met de Rustlings oefeningen aan de slag gaat.
De oefeningen bouwen langzaam op. Probeer eerst iedere oefening zelf te maken voordat je naar de volgende gaat.

### 1. Maak een project

Maak met Cargo een nieuw project met de naam:

```text
rustings_game
```

Voer het programma uit.

Zorg ervoor dat het programma:

```text
Rustings begint!
```

afdrukt.

---

### 2. Verander de boodschap

Verander het programma zodat het drie regels afdrukt:

```text
Rustings begint!
Ik ben klaar om te leren.
Mijn eerste game komt eraan!
```

---

### 3. Bekijk je project

Open je project in je editor.

Zoek:

- `Cargo.toml`
- `src/main.rs`.

Controleer wat er in beide bestanden staat.

Voeg daarna een korte regel toe aan `main.rs`.

---

### 4. Voeg `rand` toe

Voeg de dependency `rand` toe aan je project met:

```text
cargo add rand
```

Controleer daarna `Cargo.toml`.

Kun je de dependency terugvinden?

---

### 5. De dobbelsteen

Maak een programma dat één dobbelsteen gooit.

Gebruik een getal van 1 tot en met 6.

Bijvoorbeeld:

```text
Je gooide 4!
```

Het getal moet iedere keer opnieuw willekeurig worden gekozen.

---

### 6. Nog een keer

Pas je programma aan zodat er twee dobbelstenen worden gegooid.

Bijvoorbeeld:

```text
Je gooide 4.
De vijand gooide 2.
```

De twee worpen moeten onafhankelijk van elkaar zijn.

---

### 7. Een andere dobbelsteen

Maak een programma dat een twintigzijdige dobbelsteen gooit.

De mogelijke uitkomsten zijn:

```text
1 t/m 20
```

---

### 8. Schade

Gebruik een willekeurig getal om de schade van een vijand te bepalen.

De schade moet tussen 5 en 15 liggen.

Bijvoorbeeld:

```text
De draak doet 12 schade!
```

---

### 9. Schatkist

Een schatkist bevat een willekeurige hoeveelheid goud.

Laat het programma een hoeveelheid tussen 10 en 100 kiezen.

Bijvoorbeeld:

```text
Je opent de schatkist.
Je vindt 73 goudstukken!
```

---

### 10. Twee spelers

Maak een programma waarin twee spelers ieder een dobbelsteen gooien.

Bijvoorbeeld:

```text
Speler 1 gooit 5.
Speler 2 gooit 3.
```

Je hoeft nog niet te bepalen wie gewonnen heeft.

Dat leren we later met `if`.

---

### 11. Game-start

Maak een klein programma dat bij het starten van een game:

1. een titel laat zien
2. een spelernaam laat zien
3. een willekeurig startaantal goudstukken geeft.

Bijvoorbeeld:

```text
========================
       RUST QUEST
========================

Welkom, Ralph!

Je begint met 47 goudstukken.
```

---

### 12. Willekeurige vijand

Maak een programma waarin een vijand willekeurige eigenschappen krijgt.

Bijvoorbeeld:

```text
Er verschijnt een vijand!

Gezondheid: 83
Schade: 14
```

Gebruik voor de gezondheid een willekeurig getal tussen 50 en 100.

Gebruik voor de schade een willekeurig getal tussen 5 en 20.

---

### Losse oefening

Maak eerst zelf een werkend programma met `rand` als dependency.

Verwijder daarna tijdelijk de dependency uit `Cargo.toml`.

Probeer het programma opnieuw uit te voeren.

Wat gebeurt er?

Voeg de dependency daarna opnieuw toe.

---

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 2](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_2/).

---

## Controlelijst

Je bent klaar met dit artikel als je zonder hulp:

- [ ] met `cargo new` een project kunt maken
- [ ] `Cargo.toml` kunt vinden
- [ ] `src/main.rs` kunt vinden
- [ ] `cargo run` kunt gebruiken
- [ ] `cargo check` kunt gebruiken
- [ ] weet wat een dependency is
- [ ] een dependency met `cargo add` kunt toevoegen
- [ ] `rand` kunt gebruiken voor een willekeurig getal
- [ ] een eenvoudige game-oefening met Cargo kunt maken

Als je iets nog niet begrijpt, probeer dan vooral opnieuw een kleine oefening te maken. Het doel is niet om alle Cargo-commando's uit je hoofd te leren.

Het belangrijkste is dat je weet:

> [!TIP]
> **Cargo helpt ons om van losse Rust-code een echt Rust-project te maken.**

