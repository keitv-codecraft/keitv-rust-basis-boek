# Rust 33 — Gevechten: van eenvoudige aanval tot volledig gevecht

In de vorige artikelen hebben we een wereld en een spel-lus gemaakt. De speler kan rondlopen, locaties bezoeken en vijanden tegenkomen.

Nu gaan we een belangrijk onderdeel van een RPG bouwen:

**het gevecht.**

We beginnen expres heel eenvoudig.

Een speler heeft bijvoorbeeld:

```text
100 gezondheid
10 aanvalskracht
```

Een vijand heeft:

```text
30 gezondheid
5 aanvalskracht
```

De speler valt aan:

```text
vijand: 30 → 20
```

Daarna valt de vijand aan:

```text
speler: 100 → 95
```

En dat herhalen we totdat één van beiden verslagen is.

Daarna gaan we stap voor stap uitbreiden:

- wapens
- verschillende aanvallers
- schadeberekening
- verdedigen
- kritieke treffers
- vluchten
- potions
- gevechtsrondes
- verschillende acties
- tests
- en uiteindelijk de koppeling met de bestaande RPG.

Het belangrijkste doel van dit artikel is echter niet het gevechtssysteem zelf.

We willen leren hoe je een groter probleem **stap voor stap opbouwt**.

---

## 1. Begin met een klein probleem

Een veelgemaakte fout bij programmeren is meteen proberen het hele systeem te schrijven.

We gaan dat niet doen.

We beginnen met één vraag:

> Hoe kan een speler schade toebrengen aan een vijand?

We hebben daarvoor twee dingen nodig:

```text
speler
vijand
```

We maken eerst eenvoudige structs:

```rust,ignore
struct Speler {
    gezondheid: i32,
    aanvalskracht: i32,
}

struct Vijand {
    gezondheid: i32,
    aanvalskracht: i32,
}
```

Een speler maken:

```rust,ignore
let speler = Speler {
    gezondheid: 100,
    aanvalskracht: 10,
};
```

Een vijand:

```rust,ignore
let vijand = Vijand {
    gezondheid: 30,
    aanvalskracht: 5,
};
```

Dit is nog geen RPG.

Maar dat hoeft ook niet.

We bouwen eerst één klein stukje dat werkt.

---

## 2. Gezondheid verminderen

We kunnen een functie schrijven:

```rust,ignore
fn doe_schade(gezondheid: &mut i32, schade: i32) {
    *gezondheid -= schade;
}
```

We kunnen deze functie gebruiken:

```rust,ignore
let mut gezondheid = 30;

doe_schade(&mut gezondheid, 10);

println!("Gezondheid: {}", gezondheid);
```

Resultaat:

```text
Gezondheid: 20
```

Hier zien we borrowing opnieuw.

We geven:

```rust,ignore
&mut gezondheid
```

door omdat de functie de waarde moet veranderen.

---

## 3. Waarom `*gezondheid`?

Het argument:

```rust,ignore
gezondheid: &mut i32
```

is geen gewone `i32`.

Het is een mutable reference naar een `i32`.

Daarom gebruiken we:

```rust,ignore
*gezondheid
```

om bij de waarde te komen.

Dus:

```rust,ignore
*gezondheid -= schade;
```

betekent:

> Verminder de waarde waar deze reference naar verwijst.

Dit is dezelfde dereference-operator die we eerder hebben gezien.

---

## 4. De speler valt aan

We kunnen nu een functie maken die specifieker is:

```rust,ignore
fn val_aan(speler: &Speler, vijand: &mut Vijand) {
    vijand.gezondheid -= speler.aanvalskracht;
}
```

Gebruik:

```rust,ignore
let speler = Speler {
    gezondheid: 100,
    aanvalskracht: 10,
};

let mut vijand = Vijand {
    gezondheid: 30,
    aanvalskracht: 5,
};

val_aan(&speler, &mut vijand);

println!("Vijand: {}", vijand.gezondheid);
```

De vijand heeft nu:

```text
20
```

gezondheid.

---

## 5. Maak van gedrag een methode

We hebben inmiddels geleerd dat gedrag dat bij een struct hoort vaak in `impl` kan staan.

Dus in plaats van:

```rust,ignore
fn val_aan(speler: &Speler, vijand: &mut Vijand)
```

kunnen we schrijven:

```rust,ignore
impl Speler {
    fn val_aan(&self, vijand: &mut Vijand) {
        vijand.gezondheid -= self.aanvalskracht;
    }
}
```

We gebruiken dan:

```rust,ignore
speler.val_aan(&mut vijand);
```

Dit leest bijna als een zin:

> De speler valt de vijand aan.

---

## 6. `&self` en `&mut`

Let goed op de twee references.

De speler wordt zo gebruikt:

```rust,ignore
&self
```

De speler zelf verandert niet.

De vijand:

```rust,ignore
&mut Vijand
```

wordt wel veranderd.

We hebben dus:

```text
Speler
  │
  │ alleen lezen
  ▼
&self

Vijand
  │
  │ veranderen
  ▼
&mut Vijand
```

Dit is een patroon dat we in veel programma's zullen tegenkomen.

---

## 7. Een vijand kan doodgaan

We hebben een manier nodig om te bepalen of iemand nog leeft.

Bijvoorbeeld:

```rust,ignore
impl Vijand {
    fn is_levend(&self) -> bool {
        self.gezondheid > 0
    }
}
```

Voor de speler:

```rust,ignore
impl Speler {
    fn is_levend(&self) -> bool {
        self.gezondheid > 0
    }
}
```

Nu kunnen we:

```rust,ignore
if vijand.is_levend() {
    println!("De vijand leeft nog.");
} else {
    println!("De vijand is verslagen!");
}
```

---

## 8. De eerste gevechtsronde

We kunnen nu één ronde maken:

```rust,ignore
speler.val_aan(&mut vijand);

if vijand.is_levend() {
    println!("De vijand leeft nog.");
} else {
    println!("De vijand is verslagen!");
}
```

Maar een echt gevecht bestaat uit meerdere rondes.

Daarvoor hebben we een lus nodig.

---

## 9. Het gevecht als `loop`

```rust,ignore
loop {
    speler.val_aan(&mut vijand);

    if !vijand.is_levend() {
        break;
    }

    vijand.val_aan(&mut speler);

    if !speler.is_levend() {
        break;
    }
}
```

Hier komen we echter een probleem tegen.

Onze methode voor de vijand moet de speler kunnen veranderen.

Dus:

```rust,ignore
impl Vijand {
    fn val_aan(&self, speler: &mut Speler) {
        speler.gezondheid -= self.aanvalskracht;
    }
}
```

Nu kunnen we:

```rust,ignore
vijand.val_aan(&mut speler);
```

---

## 10. Waarom moet `speler` mutable zijn?

Omdat zijn gezondheid verandert.

We hebben:

```rust,ignore
&mut speler
```

nodig.

De vijand zelf verandert tijdens de aanval niet.

Daarom gebruiken we:

```rust,ignore
&self
```

voor de vijand.

Dit is precies de soort beslissing die Rust ons leert maken:

> Wie wordt gelezen en wie wordt veranderd?

---

## 11. Een volledig eenvoudig gevecht

We kunnen nu een functie maken:

```rust,ignore
fn vecht(speler: &mut Speler, vijand: &mut Vijand) {
    loop {
        println!("Speler: {}", speler.gezondheid);
        println!("Vijand: {}", vijand.gezondheid);

        speler.val_aan(vijand);

        if !vijand.is_levend() {
            println!("De vijand is verslagen!");
            break;
        }

        vijand.val_aan(speler);

        if !speler.is_levend() {
            println!("De speler is verslagen!");
            break;
        }
    }
}
```

Dit is al een volledig gevecht.

Het is nog simpel, maar het werkt.

---

## 12. Wat gebeurt er precies?

Stel:

```text
Speler
gezondheid: 100
aanvalskracht: 10

Vijand
gezondheid: 30
aanvalskracht: 5
```

Ronde 1:

```text
Speler valt aan.

Vijand:
30 → 20
```

Vijand valt aan:

```text
Speler:
100 → 95
```

Ronde 2:

```text
Vijand:
20 → 10

Speler:
95 → 90
```

Ronde 3:

```text
Vijand:
10 → 0
```

De vijand is dood.

De gevechtslus stopt.

---

## 13. Een probleem: schade kan gezondheid onder nul brengen

Stel:

```text
gezondheid = 5
schade = 10
```

Dan krijgen we:

```text
-5
```

Dat is misschien niet wat we willen.

In een RPG willen we meestal:

```text
0
```

als minimale gezondheid.

We kunnen daarom een functie maken:

```rust,ignore
fn doe_schade(gezondheid: &mut i32, schade: i32) {
    *gezondheid -= schade;

    if *gezondheid < 0 {
        *gezondheid = 0;
    }
}
```

Nu kan gezondheid nooit negatief worden.

---

## 14. Gebruik de functie

Onze aanval wordt dan:

```rust,ignore
impl Speler {
    fn val_aan(&self, vijand: &mut Vijand) {
        doe_schade(&mut vijand.gezondheid, self.aanvalskracht);
    }
}
```

En:

```rust,ignore
impl Vijand {
    fn val_aan(&self, speler: &mut Speler) {
        doe_schade(&mut speler.gezondheid, self.aanvalskracht);
    }
}
```

Nu hebben we één centrale plek waar de regel staat:

> Gezondheid kan niet onder nul komen.

Dat is beter dan dezelfde controle op meerdere plekken te kopiëren.

---

## 15. Tests voor schade

Dit is een mooie functie om te testen.

```rust,ignore
#[test]
fn schade_vermindert_gezondheid() {
    let mut gezondheid = 100;

    doe_schade(&mut gezondheid, 20);

    assert_eq!(gezondheid, 80);
}
```

Nog een test:

```rust,ignore
#[test]
fn schade_kan_gezondheid_niet_negatief_maken() {
    let mut gezondheid = 5;

    doe_schade(&mut gezondheid, 20);

    assert_eq!(gezondheid, 0);
}
```

En:

```rust,ignore
#[test]
fn nul_schade_verandert_gezondheid_niet() {
    let mut gezondheid = 50;

    doe_schade(&mut gezondheid, 0);

    assert_eq!(gezondheid, 50);
}
```

Dit is een goed voorbeeld van waarom tests nuttig zijn.

We controleren niet alleen de normale situatie.

We controleren ook de randgevallen.

---

## 16. Het `Aanvaller`-trait

We hebben eerder een trait gemaakt:

```rust,ignore
trait Aanvaller {
    fn aanvalskracht(&self) -> i32;
}
```

We kunnen dit nu gebruiken.

Voor de speler:

```rust,ignore
impl Aanvaller for Speler {
    fn aanvalskracht(&self) -> i32 {
        self.aanvalskracht
    }
}
```

Voor de vijand:

```rust,ignore
impl Aanvaller for Vijand {
    fn aanvalskracht(&self) -> i32 {
        self.aanvalskracht
    }
}
```

Nu hebben beide types hetzelfde gedrag.

---

## 17. Een algemene schadefunctie

We kunnen nu een functie maken die de aanvalskracht van een aanvaller gebruikt:

```rust,ignore
fn bereken_schade<A: Aanvaller>(aanvaller: &A) -> i32 {
    aanvaller.aanvalskracht()
}
```

De speler:

```rust,ignore
let schade = bereken_schade(&speler);
```

De vijand:

```rust,ignore
let schade = bereken_schade(&vijand);
```

Beide werken.

De functie hoeft niet te weten of de aanvaller een `Speler` of `Vijand` is.

Hij weet alleen:

> Dit object heeft `Aanvaller` geïmplementeerd.

Dit is een praktische toepassing van generics en trait bounds die we eerder hebben geleerd.

---

## 18. Een algemene aanval

We kunnen nu verder abstraheren:

```rust,ignore
fn aanval<A: Aanvaller>(
    aanvaller: &A,
    doelwit: &mut i32,
) {
    let schade = aanvaller.aanvalskracht();

    doe_schade(doelwit, schade);
}
```

De speler kan aanvallen:

```rust,ignore
aanval(&speler, &mut vijand.gezondheid);
```

En de vijand:

```rust,ignore
aanval(&vijand, &mut speler.gezondheid);
```

We hebben nu één functie voor beide situaties.

---

## 19. Waarom niet alles abstraheren?

Je zou nu kunnen denken:

> Kunnen we niet alles in één generieke functie stoppen?

Technisch gezien kunnen we veel dingen abstraheren.

Maar dat betekent niet dat we dat altijd moeten doen.

Deze code:

```rust,ignore
speler.val_aan(&mut vijand);
```

is heel duidelijk.

Een ingewikkelde generieke functie kan soms moeilijker te begrijpen zijn.

Een goede programmeur probeert dus niet zo weinig mogelijk code te schrijven.

Het doel is:

> Code schrijven die correct, begrijpelijk en onderhoudbaar is.

---

## 20. Wapens toevoegen

Onze RPG heeft al wapens.

Een wapen kan bijvoorbeeld zijn:

```rust,ignore
struct Wapen {
    naam: String,
    schade: i32,
}
```

Een speler kan een wapen hebben:

```rust,ignore
struct Speler {
    gezondheid: i32,
    aanvalskracht: i32,
    wapen: Option<Wapen>,
}
```

De aanvalskracht wordt dan:

```rust,ignore
impl Aanvaller for Speler {
    fn aanvalskracht(&self) -> i32 {
        match &self.wapen {
            Some(wapen) => self.aanvalskracht + wapen.schade,
            None => self.aanvalskracht,
        }
    }
}
```

Een speler zonder wapen doet bijvoorbeeld:

```text
10 schade
```

Een speler met een zwaard van 7 schade:

```text
17 schade
```

---

## 21. Waarom `match &self.wapen`?

`self.wapen` is:

```rust,ignore
Option<Wapen>
```

We willen alleen bekijken welk wapen de speler heeft.

We willen het wapen niet uit de speler halen.

Daarom:

```rust,ignore
match &self.wapen
```

De twee mogelijkheden zijn:

```text
Some(&Wapen)
None
```

We gebruiken dus borrowing om het wapen alleen te bekijken.

---

## 22. Een wapen veranderen

De speler kan tijdens het spel een ander wapen krijgen.

Bijvoorbeeld:

```rust,ignore
fn geef_wapen(&mut self, wapen: Wapen) {
    self.wapen = Some(wapen);
}
```

Dan:

```rust,ignore
speler.geef_wapen(zwaard);
```

De speler heeft vanaf dat moment een wapen.

De `Aanvaller`-implementatie gebruikt automatisch de nieuwe schade.

Dat is een mooi voorbeeld van hoe onderdelen samenwerken.

---

## 23. Verschillende vijanden

We kunnen verschillende vijanden maken:

```rust,ignore
let wolf = Vijand {
    gezondheid: 30,
    aanvalskracht: 5,
};

let goblin = Vijand {
    gezondheid: 50,
    aanvalskracht: 8,
};

let draak = Vijand {
    gezondheid: 200,
    aanvalskracht: 25,
};
```

De gevechtsfunctie hoeft niet te veranderen.

```rust,ignore
vecht(&mut speler, &mut wolf);
```

of:

```rust,ignore
vecht(&mut speler, &mut draak);
```

Het systeem werkt voor beide.

Dit is een belangrijk principe:

> Als we een nieuw object toevoegen, hoeven we bestaande code niet altijd te veranderen.

---

## 24. Een gevecht is niet altijd automatisch

Tot nu toe valt de speler automatisch aan.

Dat willen we veranderen.

De speler moet kunnen kiezen:

```text
Wat doe je?

1. Aanvallen
2. Potion gebruiken
3. Vluchten
```

We maken hiervoor een enum:

```rust,ignore
enum Gevechtsactie {
    Aanvallen,
    Potion,
    Vluchten,
}
```

Net als bij het hoofdmenu.

---

## 25. Een gevechtsactie kiezen

We kunnen:

```rust,ignore
fn kies_gevechtsactie(keuze: u32) -> Option<Gevechtsactie> {
    match keuze {
        1 => Some(Gevechtsactie::Aanvallen),
        2 => Some(Gevechtsactie::Potion),
        3 => Some(Gevechtsactie::Vluchten),
        _ => None,
    }
}
```

Nu is de interface losgekoppeld van de spelregels.

`1` betekent alleen tijdens het invoeren:

```text
Aanvallen
```

Intern werken we met:

```rust,ignore
Gevechtsactie::Aanvallen
```

---

## 26. De gevechtslus met keuzes

Onze gevechtslus wordt:

```rust,ignore
loop {
    println!("Wat doe je?");
    println!("1. Aanvallen");
    println!("2. Potion gebruiken");
    println!("3. Vluchten");

    let keuze = lees_getal();

    match kies_gevechtsactie(keuze) {
        Some(Gevechtsactie::Aanvallen) => {
            speler.val_aan(vijand);
        }

        Some(Gevechtsactie::Potion) => {
            println!("Je gebruikt een potion.");
        }

        Some(Gevechtsactie::Vluchten) => {
            println!("Je vlucht!");
            break;
        }

        None => {
            println!("Ongeldige keuze.");
            continue;
        }
    }

    if !vijand.is_levend() {
        println!("Je hebt gewonnen!");
        break;
    }

    vijand.val_aan(speler);

    if !speler.is_levend() {
        println!("Je bent verslagen.");
        break;
    }
}
```

Hier zien we een nieuw stukje:

```rust,ignore
continue;
```

---

## 27. `continue`

`continue` betekent:

> Stop deze huidige ronde en begin de volgende ronde van de lus.

Bijvoorbeeld:

```rust,ignore
loop {
    let keuze = lees_getal();

    if keuze == 0 {
        println!("Ongeldige keuze.");
        continue;
    }

    println!("Geldige keuze.");
}
```

Bij invoer `0` gaat het programma meteen terug naar het begin van de lus.

Het verschil:

```text
break
→ stop de hele lus

continue
→ stop deze ronde en begin opnieuw
```

---

## 28. Een potion gebruiken

Onze speler heeft al een inventaris.

We kunnen bijvoorbeeld:

```rust,ignore
fn gebruik_potion(&mut self) -> bool {
    self.inventaris.gebruik_item("Potion")
}
```

De methode geeft `true` terug als het gebruik lukt.

Dan kunnen we in het gevecht:

```rust,ignore
if speler.gebruik_potion() {
    println!("Je gebruikt een potion.");
} else {
    println!("Je hebt geen potion.");
}
```

Maar er ontbreekt nog iets.

Een potion moet ook gezondheid herstellen.

---

## 29. Gezondheid herstellen

We maken:

```rust,ignore
fn genees(&mut self, hoeveelheid: i32) {
    self.gezondheid += hoeveelheid;
}
```

Misschien willen we gezondheid niet boven de maximale gezondheid laten komen.

Daarom voegen we toe:

```rust,ignore
struct Speler {
    gezondheid: i32,
    maximale_gezondheid: i32,
    // ...
}
```

Dan:

```rust,ignore
fn genees(&mut self, hoeveelheid: i32) {
    self.gezondheid += hoeveelheid;

    if self.gezondheid > self.maximale_gezondheid {
        self.gezondheid = self.maximale_gezondheid;
    }
}
```

Nu geldt:

```text
maximale gezondheid = 100

gezondheid = 80
genezing = 30

80 + 30 = 110

maar maximaal:
100
```

---

## 30. Ook genezing testen

We schrijven tests:

```rust,ignore
#[test]
fn genezing_verhoogt_gezondheid() {
    let mut speler = Speler::nieuw("Arin");

    speler.neem_schade(20);
    speler.genees(10);

    assert_eq!(speler.gezondheid(), 90);
}
```

En:

```rust,ignore
#[test]
fn genezing_gaat_niet_boven_maximum() {
    let mut speler = Speler::nieuw("Arin");

    speler.genees(1000);

    assert_eq!(speler.gezondheid(), 100);
}
```

Dit soort tests maken regels expliciet.

---

## 31. Verdedigen

We kunnen nog een actie toevoegen:

```rust,ignore
enum Gevechtsactie {
    Aanvallen,
    Verdedigen,
    Potion,
    Vluchten,
}
```

Verdedigen kan bijvoorbeeld de inkomende schade halveren.

We hebben daarvoor toestand nodig.

```rust,ignore
struct Gevechtstoestand {
    verdedigt: bool,
}
```

De speler kan verdedigen:

```rust,ignore
gevecht.verdedigt = true;
```

Wanneer de vijand daarna aanvalt:

```rust,ignore
let mut schade = vijand.aanvalskracht();

if gevecht.verdedigt {
    schade /= 2;
}
```

Daarna resetten we:

```rust,ignore
gevecht.verdedigt = false;
```

---

## 32. Waarom een aparte toestand?

Je zou kunnen proberen een veld aan `Speler` toe te voegen:

```rust,ignore
verdedigt: bool
```

Maar verdedigen is eigenlijk geen permanente eigenschap van de speler.

Het is een toestand van de **huidige gevechtsronde**.

Daarom is:

```rust,ignore
struct Gevechtstoestand
```

een logische plek.

Dit is een voorbeeld van een ontwerpbeslissing:

> Welke informatie hoort bij welk onderdeel?

Dat wordt steeds belangrijker naarmate een programma groter wordt.

---

## 33. Kritieke treffers

We kunnen het gevecht uitbreiden met kritieke treffers.

Bijvoorbeeld:

```text
normale aanval:
10 schade

kritieke aanval:
20 schade
```

We kunnen een functie maken:

```rust,ignore
fn kritieke_schade(schade: i32) -> i32 {
    schade * 2
}
```

Voorlopig bepalen we nog niet wanneer een kritieke treffer optreedt.

Dat kunnen we later met `rand` doen.

Zo bouwen we eerst de **regel**:

```text
kritieke schade = normale schade × 2
```

en pas daarna de willekeurige kans.

---

## 34. Een kritieke kans

Met `rand` kunnen we bijvoorbeeld:

```rust,ignore
let worp = rand::random_range(1..=100);
```

Daarna:

```rust,ignore
if worp <= 10 {
    println!("Kritieke treffer!");
}
```

We hebben daarmee een kans van 10%.

We kunnen de schade bepalen:

```rust,ignore
let schade = if worp <= 10 {
    kritieke_schade(basisschade)
} else {
    basisschade
};
```

Hier gebruiken we `if` als expressie.

---

## 35. `if` als expressie

We hebben eerder gezien:

```rust,ignore
let schade = if kritisch {
    basisschade * 2
} else {
    basisschade
};
```

`if` levert hier een waarde op.

Die waarde wordt opgeslagen in:

```rust,ignore
schade
```

Beide takken moeten daarom hetzelfde type opleveren.

Dat maakt deze constructie handig voor eenvoudige beslissingen.

---

## 36. Een aanval kan informatie teruggeven

Tot nu toe verandert `val_aan()` alleen de gezondheid.

Maar misschien willen we weten:

- hoeveel schade er is gedaan
- of het een kritieke treffer was
- of het doelwit dood is.

We kunnen daarom een resultaat teruggeven.

Bijvoorbeeld:

```rust,ignore
struct Aanvalsresultaat {
    schade: i32,
    kritisch: bool,
    doelwit_dood: bool,
}
```

De aanval kan dan:

```rust,ignore
fn val_aan(&self, vijand: &mut Vijand) -> Aanvalsresultaat {
    // schade berekenen

    Aanvalsresultaat {
        schade,
        kritisch,
        doelwit_dood: !vijand.is_levend(),
    }
}
```

Nu kan de spel-lus het resultaat gebruiken.

---

## 37. Waarom een struct als resultaat?

Zonder struct zouden we bijvoorbeeld meerdere waarden kunnen teruggeven als tuple:

```rust,ignore
(schade, kritisch, doelwit_dood)
```

Dat kan.

Maar:

```rust,ignore
Aanvalsresultaat {
    schade,
    kritisch,
    doelwit_dood,
}
```

is duidelijker.

We kunnen schrijven:

```rust,ignore
let resultaat = speler.val_aan(&mut vijand);

println!("Schade: {}", resultaat.schade);

if resultaat.kritisch {
    println!("Kritieke treffer!");
}
```

De namen vertellen wat de waarden betekenen.

---

## 38. Een volledigere aanval

Onze aanval kan nu ongeveer dit worden:

```rust,ignore
fn val_aan(&self, vijand: &mut Vijand) -> Aanvalsresultaat {
    let basisschade = self.aanvalskracht();

    let worp = rand::random_range(1..=100);

    let kritisch = worp <= 10;

    let schade = if kritisch {
        basisschade * 2
    } else {
        basisschade
    };

    doe_schade(&mut vijand.gezondheid, schade);

    Aanvalsresultaat {
        schade,
        kritisch,
        doelwit_dood: !vijand.is_levend(),
    }
}
```

De code bevat inmiddels veel onderdelen die we eerder hebben geleerd:

- methodes
- traits
- `rand`
- `if` als expressie
- mutable borrowing
- structs
- booleans
- functies
- tests.

Dat is precies de bedoeling.

---

## 39. Het resultaat gebruiken

In het gevecht:

```rust,ignore
let resultaat = speler.val_aan(vijand);

println!("Je doet {} schade!", resultaat.schade);

if resultaat.kritisch {
    println!("Kritieke treffer!");
}

if resultaat.doelwit_dood {
    println!("De vijand is verslagen!");
    break;
}
```

De aanval zelf hoeft dus niet te bepalen hoe het resultaat wordt weergegeven.

De aanval **berekent het resultaat**.

Het gevecht bepaalt vervolgens **wat ermee gebeurt**.

Dat is een belangrijke scheiding.

---

## 40. Vluchten

Vluchten is weer een andere situatie.

We kunnen bijvoorbeeld:

```rust,ignore
enum Gevechtsresultaat {
    Gewonnen,
    Verloren,
    Gevlucht,
}
```

De gevechtsfunctie kan dan eindigen met één van deze waarden.

Bijvoorbeeld:

```rust,ignore
fn vecht(
    speler: &mut Speler,
    vijand: &mut Vijand,
) -> Gevechtsresultaat {
    loop {
        // ...
    }
}
```

Als de speler vlucht:

```rust,ignore
return Gevechtsresultaat::Gevlucht;
```

Als de vijand dood is:

```rust,ignore
return Gevechtsresultaat::Gewonnen;
```

Als de speler dood is:

```rust,ignore
return Gevechtsresultaat::Verloren;
```

---

## 41. Het resultaat verwerken

In de spelwereld:

```rust,ignore
match vecht(&mut speler, &mut vijand) {
    Gevechtsresultaat::Gewonnen => {
        println!("Je hebt gewonnen!");
    }

    Gevechtsresultaat::Verloren => {
        println!("Je bent verslagen.");
    }

    Gevechtsresultaat::Gevlucht => {
        println!("Je bent gevlucht.");
    }
}
```

Het gevecht vertelt dus wat er gebeurd is.

De wereld bepaalt wat er daarna gebeurt.

---

## 42. De vijand verwijderen na winst

Dit sluit aan op ons vorige artikel.

Een locatie had:

```rust,ignore
vijand: Option<Vijand>
```

Als het gevecht gewonnen is, kunnen we de vijand verwijderen.

Bijvoorbeeld:

```rust,ignore
if let Some(vijand) = locatie.vijand.as_mut() {
    let resultaat = vecht(&mut speler, vijand);

    match resultaat {
        Gevechtsresultaat::Gewonnen => {
            locatie.vijand = None;
        }

        _ => {}
    }
}
```

Hier zien we opnieuw `Option`.

---

## 43. Een belangrijke borrowing-uitdaging

Let goed op dit soort code.

We lenen:

```rust,ignore
locatie.vijand
```

mutably met:

```rust,ignore
as_mut()
```

Daarna gebruiken we die mutable reference in:

```rust,ignore
vecht(&mut speler, vijand)
```

Zolang die borrow actief is, kunnen we niet zomaar tegelijkertijd dezelfde `Option` opnieuw veranderen.

Een beginnende programmeur kan hierdoor bijvoorbeeld een compilerfout krijgen wanneer hij probeert:

```rust,ignore
locatie.vijand = None;
```

terwijl de mutable borrow nog actief is.

Dit is geen willekeurige beperking.

Rust voorkomt hiermee dat we dezelfde data tegelijk op conflicterende manieren gebruiken.

---

## 44. Een eenvoudige oplossing

We kunnen het gevecht eerst uitvoeren en daarna de vijand verwijderen.

Bijvoorbeeld met een blok:

```rust,ignore
let resultaat = {
    let vijand = locatie.vijand.as_mut().unwrap();

    vecht(&mut speler, vijand)
};

if matches!(resultaat, Gevechtsresultaat::Gewonnen) {
    locatie.vijand = None;
}
```

Het interne blok eindigt.

Daarmee eindigt ook de borrow van `vijand`.

Daarna kunnen we:

```rust,ignore
locatie.vijand = None;
```

uitvoeren.

Dit is een mooie praktische toepassing van scopes en borrowing.

---

## 45. `matches!`

Hier zagen we:

```rust,ignore
matches!(resultaat, Gevechtsresultaat::Gewonnen)
```

Dat geeft een `bool` terug.

Dus:

```rust,ignore
if matches!(resultaat, Gevechtsresultaat::Gewonnen) {
    // ...
}
```

betekent:

> Is `resultaat` de variant `Gewonnen`?

We hadden ook `match` kunnen gebruiken.

Voor één eenvoudige controle is `matches!` echter handig.

---

## 46. Het complete model

Ons gevecht bestaat nu uit verschillende onderdelen:

```text id="s6m2px"
Speler
  │
  ├── gezondheid
  ├── aanvalskracht
  ├── wapen
  └── inventaris
       │
       ▼
   Aanvallen
       │
       ▼
Aanvalsresultaat
       │
       ├── schade
       ├── kritisch
       └── doelwit dood
       │
       ▼
    Vijand
       │
       ▼
Gevechtsresultaat
       │
       ├── Gewonnen
       ├── Verloren
       └── Gevlucht
```

Dit lijkt inmiddels behoorlijk uitgebreid.

Maar bijna ieder onderdeel is opgebouwd uit concepten die we al afzonderlijk hebben geleerd.

Dat is precies de vaardigheid die we willen ontwikkelen.

---

## 47. Rustlings — Gevechten

Maak:

```text id="e9x3qa"
exercises/gevecht/
```

## Basis

1. `01_schade.rs`  
   Maak `doe_schade`.

2. `02_schade_zero.rs`  
   Zorg dat gezondheid niet onder nul komt.

3. `03_schade_test.rs`  
   Schrijf een test voor schade.

4. `04_is_levend.rs`  
   Maak `is_levend()`.

5. `05_speler_aanval.rs`  
   Laat de speler aanvallen.

6. `06_vijand_aanval.rs`  
   Laat de vijand aanvallen.

## Gevechtslus

7. `07_eenvoudig_gevecht.rs`  
   Maak een automatisch gevecht.

8. `08_gevecht_einde.rs`  
   Stop wanneer iemand dood is.

9. `09_gevechtsronde.rs`  
   Toon de gezondheid na iedere ronde.

10. `10_gevecht_test.rs`  
    Test een eenvoudig gevecht.

## Traits

11. `11_aanvaller.rs`  
    Implementeer `Aanvaller` voor `Speler`.

12. `12_vijand_aanvaller.rs`  
    Implementeer `Aanvaller` voor `Vijand`.

13. `13_bereken_schade.rs`  
    Maak een generieke functie met `T: Aanvaller`.

14. `14_algemene_aanval.rs`  
    Gebruik één algemene aanvalfunctie.

## Wapens

15. `15_wapen_schade.rs`  
    Laat een wapen extra schade geven.

16. `16_geen_wapen.rs`  
    Handel `None` correct af.

17. `17_wapen_veranderen.rs`  
    Laat de speler van wapen veranderen.

18. `18_wapen_test.rs`  
    Test de invloed van een wapen.

## Gevechtsacties

19. `19_gevechtsactie.rs`  
    Maak `Gevechtsactie`.

20. `20_kies_actie.rs`  
    Zet een getal om naar `Option<Gevechtsactie>`.

21. `21_gevechtsmenu.rs`  
    Maak een gevechtsmenu.

22. `22_continue.rs`  
    Gebruik `continue` bij ongeldige invoer.

23. `23_vluchten.rs`  
    Voeg vluchten toe.

## Genezing

24. `24_genezen.rs`  
    Maak `genees()`.

25. `25_max_gezondheid.rs`  
    Zorg dat gezondheid niet boven het maximum komt.

26. `26_potion.rs`  
    Gebruik een potion tijdens een gevecht.

27. `27_potion_test.rs`  
    Test het gebruik van een potion.

## Geavanceerder gevecht

28. `28_kritieke_treffer.rs`  
    Voeg kritieke treffers toe.

29. `29_aanvalsresultaat.rs`  
    Maak `Aanvalsresultaat`.

30. `30_gevechtsresultaat.rs`  
    Maak `Gevechtsresultaat`.

31. `31_resultaat_verwerken.rs`  
    Verwerk winst, verlies en vluchten.

32. `32_vijand_verwijderen.rs`  
    Verwijder een verslagen vijand uit een locatie.

## Debug-oefeningen

33. `33_debug_mut.rs`  
    Los een ontbrekende `mut` op.

34. `34_debug_borrow.rs`  
    Los een borrow-probleem op.

35. `35_debug_option.rs`  
    Los een fout met `Option` op.

36. `36_debug_match.rs`  
    Maak een onvolledige `match` correct.

37. `37_debug_lifetime_of_borrow.rs`  
    Onderzoek hoe een block de levensduur van een borrow beperkt.

## Integratie

38. `38_gevecht_met_wapen.rs`  
    Combineer speler, wapen en vijand.

39. `39_gevecht_met_potion.rs`  
    Combineer gevecht en inventaris.

40. `40_final_gevecht.rs`  
    Bouw een volledig gevecht.

---

## 48. Zelfstandige oefening — Een trainingsgevecht

Maak een programma met:

```text
Speler:
gezondheid = 50
aanvalskracht = 10

Slijmmonster:
gezondheid = 30
aanvalskracht = 5
```

Laat beide automatisch om de beurt aanvallen.

Print na iedere aanval:

```text
Speler doet 10 schade.
Slijmmonster heeft nog 20 gezondheid.
```

Het programma eindigt wanneer één van beide dood is.

---

## 49. Zelfstandige oefening — Een wapen

Voeg een wapen toe:

```text
IJzeren zwaard
+5 schade
```

Zonder wapen doet de speler:

```text
10 schade
```

Met het zwaard:

```text
15 schade
```

De aanvalscode zelf mag niet bepalen welk wapen de speler gebruikt.

De `Speler` moet dat zelf weten.

---

## 50. Zelfstandige oefening — Gevechtsmenu

Voeg deze keuzes toe:

```text
Wat doe je?

1. Aanvallen
2. Potion
3. Vluchten
```

Regels:

- Aanvallen veroorzaakt schade.
- Potion geneest de speler.
- Vluchten beëindigt het gevecht.
- Na aanvallen en potion mag de vijand terug aanvallen.
- Bij vluchten valt de vijand niet meer aan.

---

## 51. Zelfstandige oefening — Kritieke treffers

Voeg een kans van 10% op een kritieke treffer toe.

Normale schade:

```text
10
```

Kritieke schade:

```text
20
```

Print bijvoorbeeld:

```text
Je valt aan.
Kritieke treffer!
Je doet 20 schade.
```

Test daarnaast de functie die kritieke schade berekent.

Test niet of een willekeurige aanval precies 10% van de tijd kritisch is. Willekeurige tests zijn daarvoor ongeschikt.

---

## 52. Eindopdracht — Het volledige gevechtssysteem

Integreer het gevechtssysteem in de RPG.

Een gevecht moet minimaal bevatten:

### Speler

- gezondheid
- maximale gezondheid
- wapen
- inventaris
- aanvalskracht.

### Vijand

- naam
- gezondheid
- aanvalskracht.

### Acties

```text
1. Aanvallen
2. Verdedigen
3. Potion
4. Vluchten
```

### Aanvallen

Een aanval:

- gebruikt de aanvalskracht
- houdt rekening met het wapen
- kan een kritieke treffer zijn
- veroorzaakt nooit negatieve gezondheid
- geeft een `Aanvalsresultaat`.

### Verdedigen

Verdedigen:

- vermindert de volgende inkomende schade
- geldt slechts voor de huidige ronde.

### Potion

Een potion:

- moet uit de inventaris komen
- geneest de speler
- kan de maximale gezondheid niet overschrijden
- kan niet worden gebruikt als er geen potion is.

### Vluchten

Vluchten:

- beëindigt het gevecht
- laat de speler in leven
- verwijdert de vijand niet automatisch.

### Resultaat

Het gevecht geeft terug:

```rust,ignore
enum Gevechtsresultaat {
    Gewonnen,
    Verloren,
    Gevlucht,
}
```

---

## 53. Tests voor het gevechtssysteem

Schrijf minimaal tests voor:

```text
doe_schade()
is_levend()
genees()
kritieke_schade()
aanvalskracht()
kies_gevechtsactie()
```

En daarnaast integratietests voor:

```text
speler kan vijand verslaan
speler kan doodgaan
speler kan vluchten
wapen verhoogt schade
potion verhoogt gezondheid
gezondheid wordt nooit negatief
gezondheid wordt nooit hoger dan maximum
```

---

## 54. Een belangrijke programmeerles

Kijk nog eens naar hoe we begonnen.

We begonnen met:

```rust,ignore
vijand.gezondheid -= 10;
```

Daarna maakten we:

```rust,ignore
speler.val_aan(&mut vijand);
```

Daarna:

```rust,ignore
Aanvaller
```

Daarna:

```rust,ignore
Aanvalsresultaat
```

Daarna:

```rust,ignore
Gevechtsresultaat
```

En uiteindelijk hebben we:

```text
Speler
   ↓
Wapen
   ↓
Aanval
   ↓
Aanvalsresultaat
   ↓
Vijand
   ↓
Gevechtsresultaat
   ↓
Wereld
```

We hebben dus niet vooraf een enorm ontwerp hoeven bedenken.

We konden telkens een klein probleem oplossen en daarna het systeem uitbreiden.

Dat is een belangrijke manier van programmeren.

---

## 55. Niet alles hoeft meteen perfect te zijn

De eerste versie van ons gevecht was:

```rust,ignore
loop {
    speler.val_aan(vijand);

    if !vijand.is_levend() {
        break;
    }

    vijand.val_aan(speler);

    if !speler.is_levend() {
        break;
    }
}
```

Dat was niet onze uiteindelijke oplossing.

Maar het was wel een **werkende eerste versie**.

Daarna konden we vragen:

- Hoe voegen we keuzes toe?
- Hoe voegen we wapens toe?
- Hoe voegen we potions toe?
- Hoe bepalen we schade?
- Hoe testen we dit?
- Hoe weten we waarom een gevecht eindigde?

Dat is precies hoe grotere programma's vaak ontstaan.

Je hoeft niet meteen de perfecte oplossing te bedenken.

Je kunt beginnen met:

> Wat is de kleinste versie die werkt?

Daarna verbeter je die stap voor stap.

---

## 56. Wat hebben we geleerd?

In dit artikel hebben we veel bestaande Rust-concepten gecombineerd.

We hebben onder andere gebruikt:

```text
struct
impl
methods
&self
&mut
borrowing
dereferencing
enum
match
Option
Vec
traits
generics
trait bounds
if als expressie
loop
break
continue
return
tests
rand
```

Maar het belangrijkste nieuwe inzicht is niet één van deze syntax-elementen.

Het belangrijkste is:

> Een groter programma ontstaat door kleine, begrijpelijke onderdelen met elkaar te verbinden.

Een gevecht is bijvoorbeeld niet één gigantisch probleem.

Het bestaat uit kleinere problemen:

```text
Hoeveel schade doet een aanval?
        ↓
Hoe verandert gezondheid?
        ↓
Wanneer is iemand dood?
        ↓
Hoe valt een speler aan?
        ↓
Hoe valt een vijand aan?
        ↓
Welke keuzes heeft de speler?
        ↓
Wanneer eindigt een ronde?
        ↓
Wanneer eindigt het gevecht?
        ↓
Wat gebeurt er daarna in de wereld?
```

Als je die vragen één voor één kunt beantwoorden, wordt een groot programma veel minder intimiderend.

---

## 57. Vooruitblik

We hebben nu een speelbaar gevechtssysteem.

De volgende logische stap is om het gevechtssysteem verder te verbinden met de rest van de RPG.

We kunnen bijvoorbeeld gaan werken aan:

- verschillende soorten vijanden
- vijanden met verschillende aanvalstypen
- ervaringspunten
- levels
- sterker worden na een level-up
- beloningen na een gevecht
- goud en loot
- meerdere vijanden
- groepen vijanden
- speciale aanvallen.

Daarbij krijgen de leerlingen opnieuw vooral de kans om bestaande Rust-kennis toe te passen.

Het doel blijft hetzelfde:

**steeds minder code voorschrijven en steeds meer code door de leerling zelf laten ontwerpen.**

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 33](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_33/).

