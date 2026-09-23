# 4. Functies

## Wat gaan we leren?

In dit artikel leren we:

- wat een functie is
- waarom functies handig zijn
- hoe je een functie maakt
- hoe je een functie aanroept
- hoe je informatie aan een functie meegeeft
- hoe een functie een resultaat kan teruggeven
- hoe we functies kunnen gebruiken om een game overzichtelijker te maken.

Aan het einde kunnen we bijvoorbeeld zoiets schrijven:

```rust
fn bereken_schade(kracht: i32, wapenschade: i32) -> i32 {
    kracht + wapenschade
}

fn main() {
    let schade = bereken_schade(10, 7);

    println!("Je doet {schade} schade!");
}
```

We hebben hiermee eigenlijk een klein stukje van onze game in een aparte functie gestopt.

---

## 1. Waarom hebben we functies nodig?

Stel dat we een game maken waarin een speler drie keer schade doet.

Zonder functies zouden we bijvoorbeeld kunnen schrijven:

```rust
fn main() {
    let schade1 = 10 + 5;
    let schade2 = 12 + 5;
    let schade3 = 8 + 5;

    println!("Schade: {schade1}");
    println!("Schade: {schade2}");
    println!("Schade: {schade3}");
}
```

Dat werkt.

Maar wat gebeurt er als de manier waarop schade wordt berekend verandert?

Bijvoorbeeld:

> Wapens doen nu 10% extra schade.

Dan moeten we alle berekeningen aanpassen.

Met een functie kunnen we de berekening op één plek zetten:

```rust,ignore
fn bereken_schade(kracht: i32, wapenschade: i32) -> i32 {
    kracht + wapenschade
}
```

Daarna kunnen we de functie meerdere keren gebruiken:

```rust,ignore
fn main() {
    let schade1 = bereken_schade(10, 5);
    let schade2 = bereken_schade(12, 5);
    let schade3 = bereken_schade(8, 5);

    println!("Schade: {schade1}");
    println!("Schade: {schade2}");
    println!("Schade: {schade3}");
}
```

Een functie voorkomt dus dat we dezelfde code steeds opnieuw moeten schrijven.

---

## 2. Wat is een functie?

Een functie is een stukje code dat een bepaalde taak uitvoert.

Een functie heeft een naam.

Bijvoorbeeld:

```rust,ignore
fn spring() {
    println!("De speler springt!");
}
```

Hier heet de functie `spring`.

De functie wordt pas uitgevoerd wanneer we haar aanroepen:

```rust
fn main() {
    spring();
}

fn spring() {
    println!("De speler springt!");
}
```

De regel:

```rust,ignore
spring();
```

betekent:

> Voer de functie `spring` uit.

---

## 3. `main` is ook een functie

We hebben al heel veel functies gebruikt zonder dat we het misschien zo bekeken.

Bijvoorbeeld:

```rust
fn main() {
    println!("Hallo!");
}
```

`main` is namelijk ook gewoon een functie.

```rust,ignore
fn main()
```

betekent:

> Dit is een functie met de naam `main`.

Rust begint een normaal programma bij deze functie.

Daarom staat bijna ieder eenvoudig Rust-programma hiermee:

```rust
fn main() {
    // programma
}
```

---

## 4. Een functie maken

Een eenvoudige functie ziet er zo uit:

```rust,ignore
fn naam_van_functie() {
    // code
}
```

Bijvoorbeeld:

```rust
# fn main() {
fn start_game() {
    println!("Welkom bij het spel!");
}
# }
```

En:

```rust,ignore
fn main() {
    start_game();
}
```

We kunnen meerdere functies maken:

```rust
fn start_game() {
    println!("Welkom bij het spel!");
}

fn toon_instructies() {
    println!("Gebruik WASD om te bewegen.");
}

fn main() {
    start_game();
    toon_instructies();
}
```

De uitvoer:

```text
Welkom bij het spel!
Gebruik WASD om te bewegen.
```

Dit wordt vooral interessant wanneer programma's groter worden.

In plaats van één enorme `main`-functie kunnen we verschillende taken verdelen over verschillende functies.

---

## 5. Functies met parameters

Een functie kan informatie nodig hebben.

Stel dat we een functie willen maken die de score van de speler laat zien.

Dan willen we de score aan de functie kunnen meegeven.

```rust,ignore
fn toon_score(score: i32) {
    println!("Je score is {score}.");
}
```

Hier staat:

```rust,ignore
score: i32
```

Dat betekent:

> Deze functie verwacht een waarde met datatype `i32` en noemt die waarde binnen de functie `score`.

We kunnen de functie vervolgens gebruiken:

```rust,ignore
fn main() {
    toon_score(100);
}
```

De `100` noemen we het **argument**.

De functie krijgt dus:

```text
100
 ↓
toon_score
 ↓
score
```

De uitvoer is:

```text
Je score is 100.
```

---

## 6. Meerdere parameters

Een functie kan meerdere stukken informatie krijgen.

Bijvoorbeeld voor schade:

```rust,ignore
fn bereken_schade(kracht: i32, wapenschade: i32) {
    let schade = kracht + wapenschade;

    println!("Je doet {schade} schade!");
}
```

We kunnen deze functie aanroepen met twee argumenten:

```rust,ignore
fn main() {
    bereken_schade(10, 7);
}
```

De waarden worden dan:

```text
kracht      = 10
wapenschade = 7
```

en de functie berekent:

```text
10 + 7 = 17
```

Dus:

```text
Je doet 17 schade!
```

De volgorde is belangrijk.

Bij:

```rust,ignore
bereken_schade(10, 7);
```

is `10` de `kracht` en `7` de `wapenschade`.

---

## 7. Een functie kan een resultaat teruggeven

Tot nu toe lieten onze functies iets zien op het scherm.

Maar een functie kan ook een waarde **teruggeven**.

Bijvoorbeeld:

```rust,ignore
fn verdubbel(getal: i32) -> i32 {
    getal * 2
}
```

Hier gebeurt iets nieuws.

Achter de haakjes staat:

```rust,ignore
-> i32
```

Dit betekent:

> Deze functie geeft een `i32` terug.

We kunnen het resultaat bewaren:

```rust,ignore
fn main() {
    let resultaat = verdubbel(5);

    println!("{resultaat}");
}
```

De uitvoer:

```text
10
```

De functie krijgt `5` en geeft `10` terug.

---

## 8. Een resultaat gebruiken

Het mooie van een functie die een waarde teruggeeft, is dat we het resultaat verder kunnen gebruiken.

Bijvoorbeeld:

```rust
fn verdubbel(getal: i32) -> i32 {
    getal * 2
}

fn main() {
    let score = 100;
    let nieuwe_score = verdubbel(score);

    println!("Nieuwe score: {nieuwe_score}");
}
```

We kunnen het resultaat ook direct gebruiken:

```rust,ignore
println!("{}", verdubbel(10));
```

Of verder berekenen:

```rust,ignore
let resultaat = verdubbel(10) + 5;
```

Dan wordt:

```text
10
↓
verdubbel
↓
20
↓
+ 5
↓
25
```

---

## 9. Hoe weet Rust wat de functie teruggeeft?

Kijk naar deze functie:

```rust,ignore
fn verdubbel(getal: i32) -> i32 {
    getal * 2
}
```

Het belangrijkste stukje is:

```rust,ignore
getal * 2
```

Dit staat als laatste in de functie.

Omdat er **geen puntkomma** achter staat:

```rust,ignore
getal * 2
```

is dit de waarde die de functie teruggeeft.

Dit is een belangrijk onderdeel van de Rust-syntax.

Vergelijk:

```rust,ignore
fn verdubbel(getal: i32) -> i32 {
    getal * 2
}
```

met:

```rust,ignore
fn verdubbel(getal: i32) -> i32 {
    getal * 2;
}
```

De tweede versie werkt niet zoals we bedoelen.

Door de `;` is de berekening een statement geworden en levert deze niet meer de gewenste waarde op.

Een eenvoudige manier om dit te onthouden:

> **De laatste expressie zonder `;` is de teruggegeven waarde.**

---

## 10. Je kunt ook `return` gebruiken

Rust heeft ook het woord `return`.

Bijvoorbeeld:

```rust,ignore
fn verdubbel(getal: i32) -> i32 {
    return getal * 2;
}
```

Dit werkt.

Maar meestal schrijven Rust-programmeurs liever:

```rust,ignore
fn verdubbel(getal: i32) -> i32 {
    getal * 2
}
```

Voorlopig is het vooral belangrijk dat je beide vormen kunt herkennen.

---

## 11. Een gamefunctie

Nu maken we iets dat echt op een game begint te lijken.

We hebben:

- de kracht van een speler
- de schade van een wapen
- een functie die de totale schade berekent.

```rust
fn bereken_schade(kracht: i32, wapenschade: i32) -> i32 {
    kracht + wapenschade
}

fn main() {
    let kracht = 10;
    let wapenschade = 7;

    let schade = bereken_schade(kracht, wapenschade);

    println!("Je valt aan!");
    println!("Je doet {schade} schade.");
}
```

Uitvoer:

```text
Je valt aan!
Je doet 17 schade.
```

De functie kent de rest van de game niet.

Hij krijgt twee getallen:

```text
kracht
wapenschade
```

en geeft één getal terug:

```text
schade
```

Dat maakt de functie eenvoudig te begrijpen.

---

## 12. Genezen

We kunnen hetzelfde doen voor genezen.

```rust
fn genees(gezondheid: i32, genezing: i32) -> i32 {
    gezondheid + genezing
}

fn main() {
    let gezondheid = 40;

    let nieuwe_gezondheid = genees(gezondheid, 20);

    println!("Gezondheid: {nieuwe_gezondheid}");
}
```

De functie geeft:

```text
40 + 20 = 60
```

Dus:

```text
Gezondheid: 60
```

---

## 13. Functies kunnen ook `bool` teruggeven

Een functie hoeft niet altijd een getal terug te geven.

We kunnen bijvoorbeeld controleren of een speler nog leeft:

```rust,ignore
fn is_levend(gezondheid: i32) -> bool {
    gezondheid > 0
}
```

We kunnen de functie gebruiken:

```rust,ignore
fn main() {
    let gezondheid = 50;

    let levend = is_levend(gezondheid);

    println!("Levend: {levend}");
}
```

Het resultaat is:

```text
Levend: true
```

We kunnen hem ook gebruiken met een andere waarde:

```rust,ignore
let gezondheid = 0;
let levend = is_levend(gezondheid);

println!("Levend: {levend}");
```

Dan krijgen we:

```text
Levend: false
```

Dit is een eerste voorbeeld van een functie die iets **berekent** in plaats van iets **weergeeft**.

---

## 14. Functies combineren

De echte kracht van functies wordt duidelijk wanneer we ze combineren.

Bijvoorbeeld:

```rust
fn bereken_schade(kracht: i32, wapenschade: i32) -> i32 {
    kracht + wapenschade
}

fn genees(gezondheid: i32, genezing: i32) -> i32 {
    gezondheid + genezing
}

fn is_levend(gezondheid: i32) -> bool {
    gezondheid > 0
}

fn main() {
    let kracht = 10;
    let wapenschade = 7;
    let mut gezondheid = 50;

    let schade = bereken_schade(kracht, wapenschade);

    println!("Je doet {schade} schade.");

    gezondheid -= schade;

    let levend = is_levend(gezondheid);

    println!("Gezondheid: {gezondheid}");
    println!("Levend: {levend}");

    gezondheid = genees(gezondheid, 10);

    println!("Je geneest.");
    println!("Gezondheid: {gezondheid}");
}
```

We hebben nu verschillende kleine taken:

```text
bereken_schade()
        ↓
    schade berekenen

is_levend()
        ↓
    controleren

genees()
        ↓
    gezondheid verhogen
```

Dat is veel overzichtelijker dan één grote hoeveelheid code.

---

## 15. Functies moeten meestal één duidelijke taak hebben

Een goede functie doet bij voorkeur één duidelijke taak.

Bijvoorbeeld:

```rust,ignore
fn bereken_schade(kracht: i32, wapen: i32) -> i32
```

Deze functie berekent schade.

Dat is duidelijk.

Een functie zoals:

```rust,ignore
fn doe_alles()
```

die tegelijkertijd:

- schade berekent
- gezondheid aanpast
- score verhoogt
- geluid afspeelt
- vijanden verwijdert
- een nieuw level start

wordt al snel moeilijk te begrijpen.

Kleinere functies maken programma's meestal gemakkelijker te begrijpen, testen en veranderen.

---

## 16. Functies hebben hun eigen variabelen

Een functie kan eigen variabelen hebben.

Bijvoorbeeld:

```rust,ignore
fn bereken_schade(kracht: i32, wapen: i32) -> i32 {
    let bonus = 2;

    kracht + wapen + bonus
}
```

`bonus` bestaat binnen deze functie.

We kunnen niet zomaar vanuit `main` schrijven:

```rust,ignore
fn main() {
    println!("{bonus}");
}
```

Dat werkt niet.

De variabele `bonus` hoort bij de functie waarin hij gemaakt is.

Dit noemen we **scope**.

Voorlopig hoeven we alleen te onthouden:

> Een variabele die je binnen een functie maakt, is normaal gesproken alleen binnen die functie beschikbaar.

---

## 17. Veelgemaakte fouten

### Fout 1 — Vergeten de functie aan te roepen

Je hebt:

```rust,ignore
fn spring() {
    println!("De speler springt!");
}
```

maar:

```rust
fn main() {
}
```

Dan gebeurt er niets.

De functie bestaat wel, maar wordt niet gebruikt.

Je moet schrijven:

```rust,ignore
fn main() {
    spring();
}
```

---

### Fout 2 — Het verkeerde aantal argumenten

Deze functie:

```rust,ignore
fn tel_op(a: i32, b: i32) -> i32 {
    a + b
}
```

verwacht twee waarden.

Dit is dus goed:

```rust,ignore
tel_op(5, 3);
```

Maar dit niet:

```rust,ignore
tel_op(5);
```

En dit ook niet:

```rust,ignore
tel_op(5, 3, 2);
```

Rust zal hier een foutmelding geven.

---

### Fout 3 — Verkeerd datatype

Bij:

```rust,ignore
fn verdubbel(getal: i32) -> i32 {
    getal * 2
}
```

kun je niet zomaar een `bool` meegeven:

```rust,ignore
verdubbel(true);
```

De functie verwacht een `i32`.

---

### Fout 4 — Vergeten wat de functie teruggeeft

Deze functie:

```rust,ignore
fn verdubbel(getal: i32) -> i32 {
    getal * 2
}
```

geeft een getal terug.

We kunnen dat dus gebruiken:

```rust,ignore
let resultaat = verdubbel(5);
```

Het is daarom belangrijk om te weten:

> Geeft mijn functie een resultaat terug, of doet hij alleen iets?


---

## 18. Kleine eindopdracht — Het gevecht

Maak een klein programma waarin een speler tegen een vijand vecht.

Gebruik minstens deze functies:

```rust,ignore
fn bereken_schade(kracht: i32, wapen: i32) -> i32 {
    // ...
}

fn is_levend(gezondheid: i32) -> bool {
    // ...
}
```

Begin bijvoorbeeld met:

```text
Speler:
gezondheid = 100
kracht = 12
wapenschade = 8

Vijand:
gezondheid = 50
```

Laat de speler aanvallen.

Bereken de schade met een functie.

Verminder de gezondheid van de vijand.

Controleer met een functie of de vijand nog leeft.

Probeer daarna een tweede aanval toe te voegen.

Een mogelijke uitvoer:

```text
De speler valt aan!
De speler doet 20 schade.

De vijand heeft nog 30 gezondheid.

De speler valt opnieuw aan!
De speler doet 20 schade.

De vijand heeft nog 10 gezondheid.
```

Maak het programma vooral zelf. Het hoeft nog niet perfect te zijn.

---

## De belangrijkste gedachte

Een programma hoeft niet één groot blok code te zijn.

Je kunt een programma opdelen in kleine stukjes:

```text
programma
│
├── start_game()
│
├── bereken_schade()
│
├── genees()
│
├── is_levend()
│
└── toon_score()
```

Elke functie heeft een duidelijke taak.

Dat is een van de eerste belangrijke stappen van **gestructureerd programmeren**.

En vanaf dit punt kunnen onze games steeds groter worden zonder dat alle code in één grote `main` hoeft te staan.

---

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 4](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_4/).

---

## Controlelijst

Je bent klaar met dit artikel als je zonder hulp:

- [ ] wat een functie is
- [ ] waarom functies handig zijn
- [ ] hoe je een functie maakt met `fn`
- [ ] hoe je een functie aanroept
- [ ] wat een parameter en een argument zijn
- [ ] hoe je meerdere parameters gebruikt en het datatype van een parameter opgeeft
- [ ] hoe je een waarde teruggeeft (bijvoorbeeld `-> i32`)
- [ ] waarom de laatste expressie zonder `;` de returnwaarde oplevert
- [ ] dat een functie ook een `bool` kan teruggeven
- [ ] dat functies hun eigen lokale variabelen (scope) hebben
- [ ] hoe je functies kunt combineren in een klein programma
- [ ] hoe je compilerfouten bij functies kunt lezen en oplossen.
