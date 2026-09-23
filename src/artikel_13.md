# Rust 13 — Traits: gedrag delen

## 1. Wat gaan we leren?

We hebben inmiddels geleerd om:

- gegevens te bewaren in `structs`
- verschillende mogelijkheden te beschrijven met `enums`
- gedrag toe te voegen met `impl`
- methods te maken
- met `match` verschillende situaties te behandelen.

We kunnen bijvoorbeeld een speler maken:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
}
```

En een vijand:

```rust,ignore
struct Vijand {
    naam: String,
    gezondheid: i32,
}
```

Beide hebben bijvoorbeeld gezondheid.

Misschien willen we voor beide types een method maken:

```rust,ignore
speler.is_levend();
vijand.is_levend();
```

Maar hoe vertellen we Rust dat **zowel een speler als een vijand "levend" kunnen zijn**?

Daarvoor gebruiken we een `trait`.

Na dit artikel kun je:

- begrijpen wat een trait beschrijft
- een eenvoudige trait maken
- een trait implementeren voor een struct
- methods uit een trait gebruiken
- begrijpen wat het verschil is tussen `impl Speler` en `impl Trait for Speler`
- dezelfde trait gebruiken voor verschillende structs
- traits gebruiken in een game.

---

## 2. Eerst het probleem

Stel dat we deze twee structs hebben:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
}

struct Vijand {
    naam: String,
    gezondheid: i32,
}
```

We kunnen voor beide een `is_levend()`-method maken.

Voor de speler:

```rust,ignore
impl Speler {
    fn is_levend(&self) -> bool {
        self.gezondheid > 0
    }
}
```

En voor de vijand:

```rust,ignore
impl Vijand {
    fn is_levend(&self) -> bool {
        self.gezondheid > 0
    }
}
```

Dat werkt prima.

Maar de twee methods zijn eigenlijk hetzelfde.

Beide betekenen:

> Kijk of de gezondheid groter is dan nul.

We zouden graag kunnen zeggen:

> Alles wat `Levend` is, moet kunnen vertellen of het nog leeft.

Daarvoor maken we een trait.

---

## 3. Wat is een trait?

Een trait beschrijft **gedrag dat een type kan hebben**.

Bijvoorbeeld:

```rust,ignore
trait Levend {
    fn is_levend(&self) -> bool;
}
```

Dit betekent ongeveer:

> Een type dat `Levend` gebruikt, moet een method `is_levend()` hebben die een `bool` teruggeeft.

Een trait bevat dus een soort afspraak.

We kunnen vervolgens zeggen dat `Speler` deze trait uitvoert:

```rust,ignore
impl Levend for Speler {
    fn is_levend(&self) -> bool {
        self.gezondheid > 0
    }
}
```

En hetzelfde voor `Vijand`:

```rust,ignore
impl Levend for Vijand {
    fn is_levend(&self) -> bool {
        self.gezondheid > 0
    }
}
```

Nu hebben beide types de eigenschap `Levend`.

---

## 4. Een trait maken

De eenvoudigste vorm is:

```rust,ignore
trait Levend {
    fn is_levend(&self) -> bool;
}
```

Hier staat alleen de **beschrijving** van de method.

Er staat nog niet wat de method precies doet.

We hebben dus:

```text
Levend
└── is_levend()
```

Een struct kan deze afspraak vervolgens invullen.

Bijvoorbeeld:

```rust,ignore
struct Speler {
    gezondheid: i32,
}
```

En:

```rust,ignore
impl Levend for Speler {
    fn is_levend(&self) -> bool {
        self.gezondheid > 0
    }
}
```

De volledige code:

```rust,ignore
trait Levend {
    fn is_levend(&self) -> bool;
}

struct Speler {
    gezondheid: i32,
}

impl Levend for Speler {
    fn is_levend(&self) -> bool {
        self.gezondheid > 0
    }
}
```

---

## 5. Een trait gebruiken

We kunnen daarna gewoon schrijven:

```rust,ignore
let speler = Speler {
    gezondheid: 100,
};

if speler.is_levend() {
    println!("De speler leeft!");
}
```

De method komt uit de trait.

Dit werkt omdat `Speler` de trait `Levend` heeft geïmplementeerd.

---

## 6. Dezelfde trait voor meerdere structs

Dit is waar traits echt interessant worden.

We maken:

```rust,ignore
struct Speler {
    gezondheid: i32,
}

struct Vijand {
    gezondheid: i32,
}
```

En:

```rust,ignore
trait Levend {
    fn is_levend(&self) -> bool;
}
```

Nu kunnen beide de trait implementeren:

```rust,ignore
impl Levend for Speler {
    fn is_levend(&self) -> bool {
        self.gezondheid > 0
    }
}

impl Levend for Vijand {
    fn is_levend(&self) -> bool {
        self.gezondheid > 0
    }
}
```

Daarmee hebben beide:

```rust,ignore
speler.is_levend()
```

en:

```rust,ignore
vijand.is_levend()
```

De code in de twee methods is op dit moment hetzelfde.

Dat is niet erg.

Het belangrijke is dat beide types dezelfde **soort gedrag** aanbieden.

---

## 7. Traits kunnen verschillend worden uitgevoerd

De implementatie hoeft niet hetzelfde te zijn.

Stel dat we een `Dier` hebben:

```rust,ignore
struct Dier {
    naam: String,
    gezondheid: i32,
}
```

We kunnen dezelfde trait gebruiken:

```rust,ignore
impl Levend for Dier {
    fn is_levend(&self) -> bool {
        self.gezondheid > 0
    }
}
```

Maar een ander type zou het anders kunnen doen.

Bijvoorbeeld:

```rust,ignore
struct Robot {
    energie: i32,
}
```

Een robot heeft geen `gezondheid`, maar `energie`.

Toch kunnen we zeggen dat een robot "levend" is zolang hij energie heeft:

```rust,ignore
impl Levend for Robot {
    fn is_levend(&self) -> bool {
        self.energie > 0
    }
}
```

De afspraak is hetzelfde:

```text
is_levend() -> bool
```

Maar ieder type bepaalt zelf hoe dat werkt.

---

## 8. Trait versus `impl`

We hebben eerder dit geleerd:

```rust,ignore
impl Speler {
    fn neem_schade(&mut self, schade: i32) {
        self.gezondheid -= schade;
    }
}
```

Dit betekent:

> Voeg deze method toe aan `Speler`.

Een trait-implementatie ziet er anders uit:

```rust,ignore
impl Levend for Speler {
    fn is_levend(&self) -> bool {
        self.gezondheid > 0
    }
}
```

Dit betekent:

> `Speler` voldoet aan de afspraken van `Levend`.

Het verschil zit dus vooral in:

```text
impl Speler
```

tegenover:

```text
impl Levend for Speler
```

De eerste voegt eigen methods van `Speler` toe.

De tweede voert een trait uit voor `Speler`.

---

## 9. Een trait met meer dan één method

Een trait kan meerdere methods bevatten.

Bijvoorbeeld:

```rust,ignore
trait Karakter {
    fn naam(&self) -> &str;
    fn gezondheid(&self) -> i32;
}
```

Een `Speler` moet dan beide methods leveren:

```rust,ignore
struct Speler {
    naam: String,
    hp: i32,
}

impl Karakter for Speler {
    fn naam(&self) -> &str {
        &self.naam
    }

    fn gezondheid(&self) -> i32 {
        self.hp
    }
}
```

We kunnen ze gebruiken:

```rust,ignore
let speler = Speler {
    naam: String::from("Arin"),
    hp: 100,
};

println!("Naam: {}", speler.naam());
println!("HP: {}", speler.gezondheid());
```

Je ziet hier ook iets nieuws:

```rust,ignore
&str
```

We hebben `&str` eerder al gezien.

Hier geven we een verwijzing naar de tekst in `naam` terug:

```rust,ignore
&self.naam
```

De `String` blijft eigendom van de `Speler`.

---

## 10. Een trait kan ook standaardgedrag hebben

Een trait hoeft niet voor iedere method alleen een beschrijving te bevatten.

Je kunt ook een standaardimplementatie geven.

Bijvoorbeeld:

```rust,ignore
trait Levend {
    fn is_levend(&self) -> bool;

    fn status(&self) {
        if self.is_levend() {
            println!("Leeft!");
        } else {
            println!("Dood!");
        }
    }
}
```

Nu hoeft een struct alleen `is_levend()` te implementeren.

```rust,ignore
struct Speler {
    gezondheid: i32,
}

impl Levend for Speler {
    fn is_levend(&self) -> bool {
        self.gezondheid > 0
    }
}
```

Maar `Speler` heeft nu ook:

```rust,ignore
speler.status();
```

De `status()`-method komt uit de trait.

Dit is handig als verschillende types hetzelfde gedrag kunnen delen.

---

## 11. Een gamevoorbeeld: aanvallers

Stel dat we verschillende dingen hebben die kunnen aanvallen.

Een speler:

```rust,ignore
struct Speler {
    schade: i32,
}
```

Een vijand:

```rust,ignore
struct Vijand {
    schade: i32,
}
```

En misschien later een toren:

```rust,ignore
struct Toren {
    schade: i32,
}
```

We kunnen een trait maken:

```rust,ignore
trait Aanvaller {
    fn schade(&self) -> i32;
}
```

De speler:

```rust,ignore
impl Aanvaller for Speler {
    fn schade(&self) -> i32 {
        self.schade
    }
}
```

De vijand:

```rust,ignore
impl Aanvaller for Vijand {
    fn schade(&self) -> i32 {
        self.schade
    }
}
```

En de toren:

```rust,ignore
impl Aanvaller for Toren {
    fn schade(&self) -> i32 {
        self.schade
    }
}
```

Nu hebben deze drie types hetzelfde soort gedrag:

```text
Speler → kan schade doen
Vijand → kan schade doen
Toren  → kan schade doen
```

---

## 12. Een trait gebruiken in een functie

Hier wordt het interessant.

We kunnen een functie maken die iets nodig heeft dat `Aanvaller` is.

Daarvoor gebruiken we voorlopig deze syntax:

```rust,ignore
fn toon_schade<T: Aanvaller>(aanvaller: &T) {
    println!("Schade: {}", aanvaller.schade());
}
```

Dit ziet er ingewikkeld uit.

Daarom behandelen we `T: Aanvaller` voorlopig alleen als een vaste vorm die je moet herkennen.

Het betekent:

> Deze functie kan werken met een type dat de trait `Aanvaller` heeft.

We kunnen dan bijvoorbeeld doen:

```rust,ignore
let speler = Speler { schade: 20 };
let vijand = Vijand { schade: 10 };

toon_schade(&speler);
toon_schade(&vijand);
```

Dezelfde functie kan dus met beide werken.

**Belangrijk:** generics (`<T>`) zijn nog geen onderwerp van dit artikel. We zullen die later apart behandelen. Voor nu is het voldoende om te begrijpen dat de trait ervoor zorgt dat de functie weet dat het object `schade()` heeft.

---

## 13. Een eenvoudiger voorbeeld zonder generics

Omdat generics nog niet zijn uitgelegd, kunnen we hetzelfde idee voorlopig ook bekijken zonder die extra syntax.

We kunnen bijvoorbeeld twee aparte functies maken:

```rust,ignore
fn toon_speler_schade(speler: &Speler) {
    println!("Schade: {}", speler.schade());
}

fn toon_vijand_schade(vijand: &Vijand) {
    println!("Schade: {}", vijand.schade());
}
```

De trait zorgt er dan vooral voor dat beide types dezelfde soort method hebben.

Voor dit artikel is dat voldoende.

De generics komen later terug wanneer we precies willen leren hoe we één functie voor meerdere types kunnen gebruiken.

---

## 14. Een trait voor gamekarakters

We kunnen een wat groter voorbeeld maken.

```rust,ignore
trait Karakter {
    fn naam(&self) -> &str;
    fn gezondheid(&self) -> i32;
    fn is_levend(&self) -> bool {
        self.gezondheid() > 0
    }
}
```

Nu maken we een speler:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
}
```

En implementeren we de trait:

```rust,ignore
impl Karakter for Speler {
    fn naam(&self) -> &str {
        &self.naam
    }

    fn gezondheid(&self) -> i32 {
        self.gezondheid
    }
}
```

En een vijand:

```rust,ignore
struct Vijand {
    naam: String,
    gezondheid: i32,
}
```

Met:

```rust,ignore
impl Karakter for Vijand {
    fn naam(&self) -> &str {
        &self.naam
    }

    fn gezondheid(&self) -> i32 {
        self.gezondheid
    }
}
```

Omdat `is_levend()` al een standaardimplementatie heeft, hoeven we die niet opnieuw te schrijven.

We kunnen nu:

```rust,ignore
let speler = Speler {
    naam: String::from("Arin"),
    gezondheid: 100,
};

let vijand = Vijand {
    naam: String::from("Goblin"),
    gezondheid: 40,
};

println!("{}: {}", speler.naam(), speler.gezondheid());
println!("{}: {}", vijand.naam(), vijand.gezondheid());

println!("Speler leeft: {}", speler.is_levend());
println!("Goblin leeft: {}", vijand.is_levend());
```

Dit is een mooi voorbeeld van een trait die gedrag beschrijft dat verschillende gameobjecten gemeenschappelijk hebben.

---

## 15. Traits met mutable methods

Een trait kan natuurlijk ook methods bevatten die iets veranderen.

Bijvoorbeeld:

```rust,ignore
trait Beschadigbaar {
    fn neem_schade(&mut self, schade: i32);
}
```

Een speler:

```rust,ignore
struct Speler {
    gezondheid: i32,
}
```

Implementatie:

```rust,ignore
impl Beschadigbaar for Speler {
    fn neem_schade(&mut self, schade: i32) {
        self.gezondheid -= schade;
    }
}
```

En een vijand:

```rust,ignore
struct Vijand {
    gezondheid: i32,
}
```

Implementatie:

```rust,ignore
impl Beschadigbaar for Vijand {
    fn neem_schade(&mut self, schade: i32) {
        self.gezondheid -= schade;
    }
}
```

Nu hebben beide:

```rust,ignore
speler.neem_schade(20);
vijand.neem_schade(20);
```

Hier gebruiken we dus precies wat we eerder geleerd hebben over:

```text
&mut self
```

en borrowing.

---

## 16. Traits en enums

Traits kunnen ook bij enums worden gebruikt.

Bijvoorbeeld:

```rust,ignore
enum Wapen {
    Zwaard,
    Boog,
    Staf,
}
```

We kunnen een trait maken:

```rust,ignore
trait Schade {
    fn schade(&self) -> i32;
}
```

En vervolgens:

```rust,ignore
impl Schade for Wapen {
    fn schade(&self) -> i32 {
        match self {
            Wapen::Zwaard => 20,
            Wapen::Boog => 15,
            Wapen::Staf => 10,
        }
    }
}
```

Nu kunnen we schrijven:

```rust,ignore
let wapen = Wapen::Zwaard;

println!("Schade: {}", wapen.schade());
```

Dit combineert twee dingen die we net geleerd hebben:

- `enum`
- `trait`.

---

## 17. Een trait is een afspraak

Dit is misschien de belangrijkste manier om over traits na te denken.

Stel:

```rust,ignore
trait Aanvaller {
    fn schade(&self) -> i32;
}
```

Dan zegt deze trait eigenlijk:

> Als je zegt dat iets een `Aanvaller` is, moet het kunnen vertellen hoeveel schade het doet.

Daarna kunnen verschillende types die afspraak invullen:

```text
Speler
  ↓
Aanvaller

Vijand
  ↓
Aanvaller

Toren
  ↓
Aanvaller
```

De precieze manier waarop ze hun schade bepalen kan verschillen.

---

## 18. Veelgemaakte fouten

## Fout 1 — Een method vergeten

Als je trait zegt:

```rust,ignore
trait Levend {
    fn is_levend(&self) -> bool;
}
```

dan moet de implementatie die method bevatten:

```rust,ignore
impl Levend for Speler {
    fn is_levend(&self) -> bool {
        self.gezondheid > 0
    }
}
```

Als je hem vergeet, geeft Rust een foutmelding.

---

## Fout 2 — Verkeerde returnwaarde

Als de trait zegt:

```rust,ignore
fn is_levend(&self) -> bool;
```

dan moet de implementatie ook `bool` teruggeven.

Dit is dus fout:

```rust,ignore
fn is_levend(&self) -> i32 {
    self.gezondheid
}
```

---

## Fout 3 — Verkeerde parameters

Als de trait zegt:

```rust,ignore
fn neem_schade(&mut self, schade: i32);
```

dan moet de implementatie dezelfde vorm hebben:

```rust,ignore
fn neem_schade(&mut self, schade: i32) {
    self.gezondheid -= schade;
}
```

---

## Fout 4 — `impl` door elkaar halen

Dit:

```rust,ignore
impl Speler {
    ...
}
```

en:

```rust,ignore
impl Levend for Speler {
    ...
}
```

zijn niet hetzelfde.

Het eerste voegt gewone methods aan `Speler` toe.

Het tweede implementeert de trait `Levend` voor `Speler`.

---

## 19. Zelf proberen

## Opdracht 1 — Eerste trait

Maak:

```rust,ignore
trait Praat {
    fn zeg_iets(&self);
}
```

Maak daarna een `Speler` en implementeer deze trait.

---

## Opdracht 2 — Tweede type

Maak een `Vijand` die dezelfde `Praat`-trait gebruikt.

Laat de speler iets anders zeggen dan de vijand.

---

## Opdracht 3 — Levend

Maak:

```rust,ignore
trait Levend {
    fn is_levend(&self) -> bool;
}
```

Implementeer deze voor `Speler`.

---

## Opdracht 4 — Vijand

Implementeer `Levend` ook voor `Vijand`.

---

## Opdracht 5 — Beschadigbaar

Maak:

```rust,ignore
trait Beschadigbaar {
    fn neem_schade(&mut self, schade: i32);
}
```

Implementeer dit voor `Speler`.

---

## Opdracht 6 — Wapen

Maak:

```rust,ignore
enum Wapen {
    Zwaard,
    Boog,
    Staf,
}
```

Maak daarna:

```rust,ignore
trait Schade {
    fn schade(&self) -> i32;
}
```

Implementeer `Schade` voor `Wapen`.

---

## Opdracht 7 — Eigen gedrag

Bedenk een trait voor iets uit een game.

Bijvoorbeeld:

```text
Openbaar
Beschadigbaar
Geneesbaar
Beweegbaar
Aanvaller
```

Maak een trait en implementeer die voor één van je structs.

---

## 20. Rustlings-opgaven

Maak:

```text
exercises/
└── traits/
    ├── 01_first_trait.rs
    ├── 02_implement_trait.rs
    ├── 03_second_type.rs
    ├── 04_trait_method.rs
    ├── 05_alive.rs
    ├── 06_enemy_alive.rs
    ├── 07_damageable.rs
    ├── 08_mut_trait.rs
    ├── 09_default_method.rs
    ├── 10_multiple_methods.rs
    ├── 11_weapon_trait.rs
    ├── 12_enum_trait.rs
    ├── 13_game_character.rs
    ├── 14_debug_missing_method.rs
    ├── 15_debug_return.rs
    ├── 16_debug_parameters.rs
    ├── 17_debug_impl.rs
    └── 18_game_traits.rs
```

## ⭐ Niveau 1 — de basis

### 1. Eerste trait

Maak een trait:

```rust,ignore
trait Praat {
    fn praat(&self);
}
```

Maak een `Speler` en implementeer de trait.

---

### 2. Implementeren

Maak een tweede struct:

```rust,ignore
struct Vijand {
}
```

Laat ook deze `Praat` implementeren.

---

### 3. Eigen tekst

Laat de speler bijvoorbeeld zeggen:

```text
Ik ben de held!
```

en de vijand:

```text
Ik ben een goblin!
```

---

### 4. Trait-method gebruiken

Maak beide objecten en roep hun `praat()`-method aan.

---

### 5. `Levend`

Maak een `Levend` trait met:

```rust,ignore
fn is_levend(&self) -> bool;
```

Implementeer hem voor `Speler`.

---

### 6. Tweede implementatie

Implementeer `Levend` ook voor `Vijand`.

---

## ⭐⭐ Niveau 2 — traits combineren met eerdere kennis

### 7. Beschadigbaar

Maak:

```rust,ignore
trait Beschadigbaar {
    fn neem_schade(&mut self, schade: i32);
}
```

Gebruik `&mut self`.

---

### 8. Gezondheid veranderen

Maak een `Speler` met gezondheid en implementeer `Beschadigbaar`.

Test:

```rust,ignore
speler.neem_schade(25);
```

---

### 9. Standaardmethod

Maak een trait met:

```rust,ignore
fn is_levend(&self) -> bool;

fn status(&self) {
    println!("Status gecontroleerd.");
}
```

Implementeer alleen `is_levend()`.

Test daarna `status()`.

---

### 10. Meerdere methods

Maak:

```text
Karakter
├── naam()
├── gezondheid()
└── is_levend()
```

Implementeer deze voor `Speler`.

---

### 11. Wapen

Maak:

```rust,ignore
enum Wapen {
    Zwaard,
    Boog,
    Staf,
}
```

Maak een trait:

```rust,ignore
trait Schade {
    fn schade(&self) -> i32;
}
```

Geef ieder wapen een eigen hoeveelheid schade.

---

### 12. Enum + trait

Gebruik `match` in de implementatie van `Schade`.

Bijvoorbeeld:

```text
Zwaard → 20
Boog   → 15
Staf   → 10
```

---

## ⭐⭐⭐ Niveau 3 — combineren

### 13. Gamekarakter

Maak:

```rust,ignore
trait Karakter {
    fn naam(&self) -> &str;
    fn gezondheid(&self) -> i32;
}
```

Implementeer dit voor `Speler` en `Vijand`.

---

### 14. Debug — ontbrekende method

Je krijgt een trait met:

```rust,ignore
trait Levend {
    fn is_levend(&self) -> bool;
}
```

De implementatie bevat geen `is_levend()`.

Laat de compilerfout je vertellen wat ontbreekt.

---

### 15. Debug — verkeerde returnwaarde

De trait verwacht:

```rust,ignore
fn is_levend(&self) -> bool;
```

maar de implementatie geeft een `i32` terug.

Zoek en herstel de fout.

---

### 16. Debug — verkeerde parameters

De trait verwacht:

```rust,ignore
fn neem_schade(&mut self, schade: i32);
```

maar de implementatie gebruikt bijvoorbeeld:

```rust,ignore
fn neem_schade(&self);
```

Zoek uit waarom Rust dit niet accepteert.

---

### 17. Debug — verkeerde `impl`

Je krijgt:

```rust,ignore
impl Speler {
    fn is_levend(&self) -> bool {
        self.gezondheid > 0
    }
}
```

maar `is_levend()` hoort bij de trait `Levend`.

Pas de code aan zodat de trait correct wordt geïmplementeerd.

---

### 18. Eindopdracht — gamekarakters

Maak twee types:

```text
Speler
Vijand
```

Maak een trait:

```text
Karakter
```

met minstens:

```text
naam()
gezondheid()
is_levend()
```

Laat beide types de trait implementeren.

Maak vervolgens een klein programma waarin je beide karakters toont.

---

## 21. Kleine eindopdracht — wie kan aanvallen?

We gaan ons RPG-project verder uitbreiden.

Maak:

```rust,ignore
trait Aanvaller {
    fn schade(&self) -> i32;
}
```

Maak vervolgens:

```text
Speler
Vijand
```

Beide moeten `Aanvaller` implementeren.

De speler doet bijvoorbeeld 20 schade.

De vijand doet bijvoorbeeld 10 schade.

Maak vervolgens een klein gevecht waarin je de waarden gebruikt.

Breid het daarna uit met:

```rust,ignore
trait Beschadigbaar {
    fn neem_schade(&mut self, schade: i32);
}
```

Laat zowel `Speler` als `Vijand` deze trait implementeren.

Je hebt dan bijvoorbeeld:

```text
Speler
 ├── Aanvaller
 └── Beschadigbaar

Vijand
 ├── Aanvaller
 └── Beschadigbaar
```

Dat is precies het soort structuur waarvoor traits nuttig zijn.

---

## 22. Wat moet je nu kunnen?

Na dit artikel zou je moeten kunnen:

- [ ] uitleggen wat een trait is
- [ ] een trait maken
- [ ] een trait implementeren voor een struct
- [ ] dezelfde trait voor meerdere structs implementeren
- [ ] trait-methods gebruiken
- [ ] `impl Speler` herkennen
- [ ] `impl Trait for Speler` herkennen
- [ ] een trait met meerdere methods maken
- [ ] een standaardmethod in een trait maken
- [ ] `&self` en `&mut self` in traits gebruiken
- [ ] een trait gebruiken met een enum
- [ ] traits combineren met structs en methods.

Het belangrijkste idee:

> **Een trait beschrijft gedrag dat een type kan hebben.**

Bijvoorbeeld:

```rust,ignore
trait Aanvaller {
    fn schade(&self) -> i32;
}
```

Daarna kan een speler zeggen:

```rust,ignore
impl Aanvaller for Speler {
    fn schade(&self) -> i32 {
        self.schade
    }
}
```

en een vijand:

```rust,ignore
impl Aanvaller for Vijand {
    fn schade(&self) -> i32 {
        self.schade
    }
}
```

Beide kunnen dan zeggen:

```rust,ignore
speler.schade()
vijand.schade()
```

Hoewel `Speler` en `Vijand` verschillende types zijn, hebben ze hetzelfde soort gedrag.

Dat is de kern van traits.

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 13](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_13/).

