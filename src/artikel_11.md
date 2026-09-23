# Rust 11 — `impl`-blokken en methods

## 1. Wat gaan we leren?

In het vorige artikel leerden we **structs** gebruiken om gegevens bij elkaar te bewaren.

We hadden bijvoorbeeld:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
    goud: i32,
}
```

En functies die met een `Speler` werken:

```rust,ignore
fn neem_schade(speler: &mut Speler, schade: i32) {
    speler.gezondheid -= schade;
}
```

Dat werkt prima. Maar er is een manier om functies die bij een bepaald type horen overzichtelijker te organiseren.

Daarvoor gebruiken we:

```rust,ignore
impl
```

Na dit artikel kun je:

- een `impl`-blok maken
- een method toevoegen aan een struct
- `self`, `&self` en `&mut self` begrijpen
- methods aanroepen met `.` zoals `speler.neem_schade(20)`
- methods maken die waarden teruggeven
- meerdere methods bij één struct zetten
- begrijpen wanneer een gewone functie en wanneer een method handig is.

---

## 2. Waarom hebben we dit nodig?

Stel dat we een speler hebben:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
    goud: i32,
}
```

We kunnen allerlei functies maken:

```rust,ignore
fn neem_schade(speler: &mut Speler, schade: i32) {
    speler.gezondheid -= schade;
}

fn genees(speler: &mut Speler, hoeveelheid: i32) {
    speler.gezondheid += hoeveelheid;
}

fn toon_naam(speler: &Speler) {
    println!("Naam: {}", speler.naam);
}
```

Maar al deze functies horen eigenlijk bij het concept **Speler**.

Bij een groter programma kan het onoverzichtelijk worden als we tientallen functies hebben die allemaal een `Speler` als eerste parameter hebben.

Met `impl` kunnen we zeggen:

> Deze functies horen bij `Speler`.

Dan kunnen we schrijven:

```rust,ignore
impl Speler {
    fn neem_schade(&mut self, schade: i32) {
        self.gezondheid -= schade;
    }

    fn genees(&mut self, hoeveelheid: i32) {
        self.gezondheid += hoeveelheid;
    }

    fn toon_naam(&self) {
        println!("Naam: {}", self.naam);
    }
}
```

En vervolgens:

```rust,ignore
speler.neem_schade(20);
speler.genees(10);
speler.toon_naam();
```

Dat leest veel meer alsof we tegen de speler zeggen:

> Speler, neem 20 schade.

---

## 3. Het eenvoudigste voorbeeld

We beginnen met een heel eenvoudige struct:

```rust,ignore
struct Speler {
    naam: String,
}
```

Daar maken we een `impl`-blok voor:

```rust,ignore
impl Speler {
    fn zeg_hoi(&self) {
        println!("Hallo! Ik ben {}.", self.naam);
    }
}
```

Nu kunnen we een speler maken:

```rust,ignore
fn main() {
    let speler = Speler {
        naam: String::from("Arin"),
    };

    speler.zeg_hoi();
}
```

De volledige code:

```rust
struct Speler {
    naam: String,
}

impl Speler {
    fn zeg_hoi(&self) {
        println!("Hallo! Ik ben {}.", self.naam);
    }
}

fn main() {
    let speler = Speler {
        naam: String::from("Arin"),
    };

    speler.zeg_hoi();
}
```

De belangrijke regel is:

```rust,ignore
speler.zeg_hoi();
```

De punt `.` betekent hier:

> Voer de method `zeg_hoi` uit op deze `speler`.

---

## 4. Wat betekent `&self`?

Dit is waarschijnlijk het belangrijkste nieuwe onderdeel.

We hadden eerder:

```rust,ignore
fn toon_speler(speler: &Speler) {
    println!("{}", speler.naam);
}
```

In een `impl`-blok schrijven we:

```rust,ignore
fn toon_speler(&self) {
    println!("{}", self.naam);
}
```

`self` betekent:

> de specifieke instantie waarop de method wordt uitgevoerd.

Dus bij:

```rust,ignore
speler.toon_speler();
```

verwijst `self` naar `speler`.

Je kunt het ongeveer zien als:

```rust,ignore
fn toon_speler(speler: &Speler)
```

maar binnen een `impl` gebruiken we de speciale naam:

```rust,ignore
self
```

Omdat we de speler alleen willen bekijken en niet aanpassen, gebruiken we:

```rust,ignore
&self
```

---

## 5. `&self`, `&mut self` en `self`

Er zijn drie belangrijke varianten.

## `&self`

We willen de struct alleen bekijken.

```rust,ignore
fn toon_gezondheid(&self) {
    println!("HP: {}", self.gezondheid);
}
```

De eigenaar blijft gewoon eigenaar.

---

## `&mut self`

We willen de struct veranderen.

```rust,ignore
fn neem_schade(&mut self, schade: i32) {
    self.gezondheid -= schade;
}
```

Daarom moet de speler ook mutable zijn:

```rust,ignore
let mut speler = Speler {
    naam: String::from("Arin"),
    gezondheid: 100,
};

speler.neem_schade(20);
```

Zonder `mut` werkt dit niet:

```rust,ignore
let speler = Speler {
    naam: String::from("Arin"),
    gezondheid: 100,
};

speler.neem_schade(20); // fout
```

Waarom?

Omdat `neem_schade` de speler verandert.

---

## `self`

Je kunt ook schrijven:

```rust,ignore
fn vernietig(self) {
    println!("{} verdwijnt!", self.naam);
}
```

Hierbij neemt de method het eigenaarschap van de speler over.

Na:

```rust,ignore
speler.vernietig();
```

kun je `speler` niet meer gebruiken.

Dit lijkt op wat we bij functies met `String` hebben geleerd:

```rust,ignore
fn gebruik_naam(naam: String) {
    println!("{naam}");
}
```

De `self`-variant is dus een manier om het volledige eigenaarschap van de instantie over te nemen.

Voorlopig zul je vooral deze twee gebruiken:

```rust,ignore
&self
&mut self
```

---

## 6. Methods die iets teruggeven

Een method hoeft niet alleen iets te printen.

Hij kan ook een waarde teruggeven.

Bijvoorbeeld:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
}

impl Speler {
    fn is_levend(&self) -> bool {
        self.gezondheid > 0
    }
}
```

Gebruik:

```rust,ignore
let speler = Speler {
    naam: String::from("Arin"),
    gezondheid: 100,
};

if speler.is_levend() {
    println!("De speler leeft!");
}
```

Dit leest prettig:

```rust,ignore
speler.is_levend()
```

In plaats van:

```rust,ignore
is_levend(&speler)
```

Beide kunnen hetzelfde doen. De eerste vorm maakt duidelijk dat `is_levend` een eigenschap/gedrag van `Speler` is.

---

## 7. Methods met parameters

Een method kan natuurlijk ook extra informatie krijgen.

```rust,ignore
impl Speler {
    fn neem_schade(&mut self, schade: i32) {
        self.gezondheid -= schade;
    }
}
```

Hier zijn twee soorten gegevens:

```rust,ignore
&mut self
```

is de speler zelf.

En:

```rust,ignore
schade: i32
```

is extra informatie die we aan de method meegeven.

Bij:

```rust,ignore
speler.neem_schade(25);
```

wordt `25` de waarde van `schade`.

---

## 8. Een complete speler

We kunnen nu een iets uitgebreidere `Speler` maken:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
    goud: i32,
}
```

En daar verschillende methods aan toevoegen:

```rust,ignore
impl Speler {
    fn toon_status(&self) {
        println!("Naam: {}", self.naam);
        println!("Gezondheid: {}", self.gezondheid);
        println!("Goud: {}", self.goud);
    }

    fn neem_schade(&mut self, schade: i32) {
        self.gezondheid -= schade;
    }

    fn genees(&mut self, hoeveelheid: i32) {
        self.gezondheid += hoeveelheid;
    }

    fn verdien_goud(&mut self, hoeveelheid: i32) {
        self.goud += hoeveelheid;
    }

    fn is_levend(&self) -> bool {
        self.gezondheid > 0
    }
}
```

We kunnen ze allemaal gebruiken:

```rust,ignore
fn main() {
    let mut speler = Speler {
        naam: String::from("Arin"),
        gezondheid: 100,
        goud: 50,
    };

    speler.toon_status();

    speler.neem_schade(30);
    speler.verdien_goud(25);

    if speler.is_levend() {
        println!("De speler leeft nog!");
    }

    speler.genees(10);

    speler.toon_status();
}
```

Dit begint al behoorlijk op echte gamecode te lijken.

---

## 9. Waarom is dit beter dan losse functies?

Vergelijk:

```rust,ignore
neem_schade(&mut speler, 20);
genees(&mut speler, 10);
verdien_goud(&mut speler, 50);
```

met:

```rust,ignore
speler.neem_schade(20);
speler.genees(10);
speler.verdien_goud(50);
```

De tweede vorm maakt meteen duidelijk dat deze acties bij `Speler` horen.

Dit wordt vooral nuttig wanneer een programma groter wordt.

We kunnen bijvoorbeeld later hebben:

```rust,ignore
struct Speler { ... }
struct Vijand { ... }
struct Wapen { ... }
struct Kist { ... }
struct Winkel { ... }
```

Dan kunnen ieder van deze types hun eigen methods krijgen.

Bijvoorbeeld:

```rust,ignore
impl Vijand {
    fn neem_schade(&mut self, schade: i32) {
        self.gezondheid -= schade;
    }
}
```

En:

```rust,ignore
impl Wapen {
    fn toon_info(&self) {
        println!("{} doet {} schade.", self.naam, self.schade);
    }
}
```

---

## 10. Een gamevoorbeeld

We maken een eenvoudige RPG-strijd.

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
    schade: i32,
}

struct Vijand {
    naam: String,
    gezondheid: i32,
}
```

De speler krijgt methods:

```rust,ignore
impl Speler {
    fn aanval(&self, vijand: &mut Vijand) {
        vijand.gezondheid -= self.schade;

        println!(
            "{} valt {} aan voor {} schade!",
            self.naam,
            vijand.naam,
            self.schade
        );
    }

    fn is_levend(&self) -> bool {
        self.gezondheid > 0
    }
}
```

En de vijand:

```rust,ignore
impl Vijand {
    fn is_levend(&self) -> bool {
        self.gezondheid > 0
    }
}
```

Nu kunnen we schrijven:

```rust,ignore
fn main() {
    let speler = Speler {
        naam: String::from("Arin"),
        gezondheid: 100,
        schade: 25,
    };

    let mut goblin = Vijand {
        naam: String::from("Goblin"),
        gezondheid: 60,
    };

    speler.aanval(&mut goblin);
    speler.aanval(&mut goblin);
    speler.aanval(&mut goblin);

    if !goblin.is_levend() {
        println!("De goblin is verslagen!");
    }
}
```

Let vooral op:

```rust,ignore
speler.aanval(&mut goblin);
```

De speler wordt alleen gelezen, dus:

```rust,ignore
&self
```

is voldoende.

De goblin moet veranderen, dus geven we een mutable reference door:

```rust,ignore
&mut goblin
```

Dit is precies de borrowing die we in het vorige artikel hebben geleerd.

---

## 11. Methods kunnen ook andere structs gebruiken

Dit is een belangrijk inzicht.

Een method hoeft niet alleen zijn eigen struct te veranderen.

Bijvoorbeeld:

```rust,ignore
impl Speler {
    fn aanval(&self, vijand: &mut Vijand) {
        vijand.gezondheid -= self.schade;
    }
}
```

Hier gebruikt `Speler` informatie uit zichzelf:

```rust,ignore
self.schade
```

en verandert hij een andere struct:

```rust,ignore
vijand.gezondheid
```

Dat is heel normaal in gamecode.

Een speler kan bijvoorbeeld:

- een vijand aanvallen
- een kist openen
- een wapen oppakken
- een winkel bezoeken
- een deur openen.

Later zullen we deze systemen steeds verder uitbreiden.

---

## 12. Een method die een waarde berekent

Methods zijn ook handig voor berekeningen.

```rust,ignore
struct Wapen {
    naam: String,
    schade: i32,
}

impl Wapen {
    fn totale_schade(&self, kracht: i32) -> i32 {
        self.schade + kracht
    }
}
```

Gebruik:

```rust,ignore
let zwaard = Wapen {
    naam: String::from("IJzeren zwaard"),
    schade: 10,
};

let schade = zwaard.totale_schade(5);

println!("Schade: {schade}");
```

De method gebruikt gegevens van het wapen:

```rust,ignore
self.schade
```

en extra informatie:

```rust,ignore
kracht
```

en geeft vervolgens een waarde terug.

---

## 13. Methods en `mut`

Een veelgemaakte fout is vergeten dat een method de struct verandert.

Dit werkt:

```rust
struct Speler {
    gezondheid: i32,
}

impl Speler {
    fn neem_schade(&mut self, schade: i32) {
        self.gezondheid -= schade;
    }
}

fn main() {
    let mut speler = Speler {
        gezondheid: 100,
    };

    speler.neem_schade(20);
}
```

Maar dit niet:

```rust,ignore
let speler = Speler {
    gezondheid: 100,
};

speler.neem_schade(20);
```

De compiler zal aangeven dat je probeert een immutable variabele te veranderen.

Dit is geen willekeurige beperking.

Je hebt immers gezegd:

```rust,ignore
&mut self
```

Daarmee zeg je:

> Deze method wil de instantie veranderen.

Dan moet de instantie ook veranderbaar zijn.

---

## 14. Meerdere `impl`-blokken

Je kunt meerdere `impl`-blokken voor hetzelfde type hebben:

```rust,ignore
impl Speler {
    fn neem_schade(&mut self, schade: i32) {
        self.gezondheid -= schade;
    }
}

impl Speler {
    fn genees(&mut self, hoeveelheid: i32) {
        self.gezondheid += hoeveelheid;
    }
}
```

Dit mag.

In de meeste eenvoudige programma's is één `impl`-blok overzichtelijker:

```rust,ignore
impl Speler {
    fn neem_schade(&mut self, schade: i32) {
        self.gezondheid -= schade;
    }

    fn genees(&mut self, hoeveelheid: i32) {
        self.gezondheid += hoeveelheid;
    }
}
```

Meerdere blokken kunnen later nuttig worden wanneer een project groter wordt.

---

## 15. Gewone functie of method?

Een gewone functie:

```rust,ignore
fn bereken_schade(kracht: i32, wapen: i32) -> i32 {
    kracht + wapen
}
```

Een method:

```rust,ignore
impl Wapen {
    fn bereken_schade(&self, kracht: i32) -> i32 {
        self.schade + kracht
    }
}
```

Wanneer gebruik je welke?

Een simpele vuistregel:

**Gebruik een method als de actie duidelijk bij een bepaald type hoort.**

Bijvoorbeeld:

```text
Speler.neem_schade()
Speler.genees()
Speler.is_levend()

Wapen.totale_schade()
Wapen.toon_info()

Vijand.is_levend()
Vijand.neem_schade()
```

Een algemene berekening hoeft niet per se bij een type te horen:

```text
bereken_percentage()
bereken_gemiddelde()
```

We leren later nog uitgebreidere manieren om gedrag aan types te koppelen.

---

## 16. Veelgemaakte fouten

### Fout 1 — `self` vergeten

Dit:

```rust,ignore
impl Speler {
    fn toon_naam() {
        println!("{}", self.naam);
    }
}
```

werkt niet.

De method moet weten over welke `Speler` het gaat:

```rust,ignore
fn toon_naam(&self) {
    println!("{}", self.naam);
}
```

---

### Fout 2 — `&self` gebruiken terwijl je wilt veranderen

Dit werkt niet:

```rust,ignore
fn neem_schade(&self, schade: i32) {
    self.gezondheid -= schade;
}
```

Gebruik:

```rust,ignore
fn neem_schade(&mut self, schade: i32) {
    self.gezondheid -= schade;
}
```

---

### Fout 3 — `mut` vergeten

Als de method `&mut self` gebruikt:

```rust,ignore
fn neem_schade(&mut self, schade: i32)
```

moet de instantie mutable zijn:

```rust,ignore
let mut speler = Speler { ... };
```

---

### Fout 4 — method verkeerd aanroepen

Als je hebt:

```rust,ignore
fn genees(&mut self, hoeveelheid: i32)
```

roep je die aan met:

```rust,ignore
speler.genees(20);
```

Niet:

```rust,ignore
speler.genees();
```

De method verwacht immers nog een parameter.

---

## 17. Zelf proberen

Probeer de volgende opdrachten eerst zelf te maken.

## Opdracht 1 — Hallo

Maak:

```rust,ignore
struct Speler {
    naam: String,
}
```

Maak een method:

```text
zeg_hoi()
```

die bijvoorbeeld afdrukt:

```text
Hallo, ik ben Arin!
```

---

## Opdracht 2 — Gezondheid

Maak:

```rust,ignore
struct Speler {
    gezondheid: i32,
}
```

Maak een method:

```text
toon_gezondheid()
```

die de gezondheid afdrukt.

---

## Opdracht 3 — Schade

Voeg een method toe:

```text
neem_schade(schade)
```

De gezondheid moet worden verminderd.

---

## Opdracht 4 — Genezen

Voeg toe:

```text
genees(hoeveelheid)
```

Hiermee wordt de gezondheid verhoogd.

---

## Opdracht 5 — Levend?

Maak:

```text
is_levend()
```

Deze method geeft `true` terug als de gezondheid groter is dan nul.

Gebruik hem met:

```rust,ignore
if speler.is_levend() {
    println!("Je leeft!");
}
```

---

## Opdracht 6 — Goud

Voeg een veld toe:

```rust,ignore
goud: i32,
```

Maak een method:

```text
verdien_goud(hoeveelheid)
```

---

## Opdracht 7 — Vijand

Maak een struct:

```rust,ignore
struct Vijand {
    naam: String,
    gezondheid: i32,
}
```

Maak een method:

```text
is_levend()
```

---

## Opdracht 8 — Aanvallen

Laat `Speler` een method krijgen:

```text
aanval(vijand)
```

De aanval moet de gezondheid van de vijand verminderen.

---

## 18. Rustlings-opgaven

Deze oefeningen horen bij dit artikel:

```text
exercises/
└── impl/
    ├── 01_first_method.rs
    ├── 02_self.rs
    ├── 03_show_health.rs
    ├── 04_damage.rs
    ├── 05_heal.rs
    ├── 06_is_alive.rs
    ├── 07_add_gold.rs
    ├── 08_return_value.rs
    ├── 09_multiple_methods.rs
    ├── 10_enemy.rs
    ├── 11_attack.rs
    ├── 12_weapon.rs
    ├── 13_debug_self.rs
    ├── 14_debug_mut_self.rs
    ├── 15_debug_mut_variable.rs
    └── 16_game_character.rs
```

### ⭐ Niveau 1 — `self` leren kennen

### 1. Eerste method

Maak een `Speler` met een `zeg_hoi()` method.

### 2. `&self`

Maak een method die de naam van de speler afdrukt.

### 3. Gezondheid

Maak een `toon_gezondheid()` method.

### 4. Returnwaarde

Maak:

```text
is_levend()
```

die een `bool` teruggeeft.

### 5. Meerdere methods

Maak een speler met:

```text
toon_naam()
toon_gezondheid()
is_levend()
```

---

### ⭐⭐ Niveau 2 — veranderen met `&mut self`

### 6. Schade

Maak:

```text
neem_schade(10)
```

waardoor de gezondheid met 10 afneemt.

### 7. Genezen

Maak:

```text
genees(20)
```

waardoor de gezondheid met 20 toeneemt.

### 8. Goud

Maak:

```text
verdien_goud(50)
```

### 9. Wapen

Maak:

```rust,ignore
struct Wapen {
    naam: String,
    schade: i32,
}
```

Voeg een method toe die de informatie over het wapen afdrukt.

### 10. Schade berekenen

Maak:

```text
totale_schade(kracht)
```

die de schade van het wapen plus de kracht teruggeeft.

---

### ⭐⭐⭐ Niveau 3 — gamegedrag

### 11. Vijand

Maak een `Vijand` met:

```text
naam
gezondheid
schade
```

Voeg `is_levend()` toe.

### 12. Aanvallen

Laat `Speler` een `Vijand` aanvallen:

```text
speler.aanval(&mut vijand)
```

### 13. Vijand valt terug aan

Geef `Vijand` een method:

```text
aanval(&mut speler)
```

De vijand doet zijn eigen hoeveelheid schade.

### 14. Debuggen

Je krijgt een programma waarin:

```rust,ignore
fn neem_schade(&self, schade: i32)
```

staat, maar de functie probeert de gezondheid te veranderen.

Zoek de fout.

### 15. Debuggen

Een speler wordt zo gemaakt:

```rust,ignore
let speler = Speler {
    gezondheid: 100,
};
```

Daarna wordt:

```rust,ignore
speler.neem_schade(20);
```

aangeroepen.

Zoek uit waarom de compiler klaagt.

### 16. Eindopdracht — karakter

Maak een compleet karakter:

```text
Speler
├── naam
├── gezondheid
├── goud
└── schade
```

Maak minstens deze methods:

```text
toon_status()
neem_schade()
genees()
verdien_goud()
is_levend()
aanval()
```

Maak daarna een kleine strijd tegen een vijand.

---

## 19. Kleine eindopdracht — een gevecht

Maak een programma waarin:

1. de speler 100 HP heeft
2. de vijand 60 HP heeft
3. de speler 25 schade doet
4. de speler de vijand drie keer aanvalt
5. na iedere aanval de HP van de vijand wordt afgedrukt
6. wordt gecontroleerd of de vijand nog leeft
7. uiteindelijk wordt gemeld dat de vijand verslagen is.

Probeer zoveel mogelijk gedrag met methods te schrijven.

Bijvoorbeeld:

```rust,ignore
speler.aanval(&mut vijand);
```

in plaats van losse code die rechtstreeks de velden van `vijand` aanpast.

---

## 20. Wat moet je nu kunnen?

Na dit artikel zou je moeten begrijpen:

- [ ] wat een `impl`-blok is
- [ ] hoe je een method maakt
- [ ] wat `self` betekent
- [ ] wat `&self` betekent
- [ ] wat `&mut self` betekent
- [ ] wanneer een method een `&mut self` nodig heeft
- [ ] hoe je een method aanroept met `.`
- [ ] hoe je parameters aan een method geeft
- [ ] hoe je een waarde uit een method teruggeeft
- [ ] hoe methods samenwerken met borrowing
- [ ] waarom methods handig zijn voor gameobjecten.

Het belangrijkste nieuwe patroon is:

```rust,ignore
impl Speler {
    fn neem_schade(&mut self, schade: i32) {
        self.gezondheid -= schade;
    }
}
```

waardoor je vervolgens kunt schrijven:

```rust,ignore
speler.neem_schade(20);
```

Je hebt hiermee een belangrijke stap gezet richting object-georiënteerde programmeerideeën, zonder dat Rust zelf traditionele classes gebruikt.

In het volgende artikel kunnen we verdergaan met **enums**. Daarmee kunnen we bijvoorbeeld dingen modelleren als verschillende soorten wapens, speltoestanden of acties: `Zwaard`, `Boog`, `Magie`, enzovoort.

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 11](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_11/).

