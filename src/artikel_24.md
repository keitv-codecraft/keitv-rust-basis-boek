# 24. Generics

## Wat gaan we leren?

In dit artikel leren we hoe we herbruikbare code schrijven die met verschillende datatypes werkt met behulp van generics en trait bounds.

We leren:

- wat generics zijn en wat de type-parameter `T` betekent
- generieke functies en generieke structs schrijven
- eisen stellen aan types met trait bounds (`T: Trait` en `where`-clausules)
- het verschil tussen generieke code (`impl Trait`) en dynamische trait objects (`dyn Trait` / `Box<dyn Trait>`)
- statische dispatch (monomorphization) versus dynamische dispatch.

---

## 1. Waarom hebben we generics nodig?

Stel dat we een functie hebben die twee `i32`-waarden vergelijkt:

```rust,ignore
fn groter_getal(a: i32, b: i32) -> i32 {
    if a > b {
        a
    } else {
        b
    }
}
```

Dat werkt voor `i32`.

Maar wat als we hetzelfde ook voor andere typen willen doen?

Dan zouden we steeds een nieuwe functie kunnen schrijven:

```text id="e4m5fw"
groter_i32()
groter_i64()
groter_f32()
...
```

Dat is onhandig.

We willen eigenlijk zeggen:

> Deze functie werkt met één willekeurig type, zolang dat type aan bepaalde voorwaarden voldoet.

Daar zijn generics voor.

---

## 2. Een eenvoudige generieke functie

Een generiek type schrijven we bijvoorbeeld als:

```rust,ignore
T
```

De letter `T` is geen speciaal type.

Het is een naam die we zelf kiezen.

Bijvoorbeeld:

```rust,ignore
fn eerste<T>(waarde: T) -> T {
    waarde
}
```

Hier staat:

```text id="v6m8e2"
<T>
```

Dat betekent:

> Deze functie heeft een generiek type `T`.

De functie ontvangt een `T` en geeft dezelfde `T` terug.

We kunnen hem bijvoorbeeld gebruiken met een getal:

```rust,ignore
let getal = eerste(42);
```

Dan is:

```text id="ln5h23"
T = i32
```

Bij een string:

```rust,ignore
let naam = eerste(String::from("Aria"));
```

is:

```text id="s7l9yn"
T = String
```

De functie zelf hoefde niet twee keer geschreven te worden.

---

## 3. Wat betekent `T`?

Het is belangrijk om niet te denken dat `T` zelf een concreet type is.

Bij:

```rust,ignore
fn eerste<T>(waarde: T) -> T {
    waarde
}
```

zeggen we eigenlijk:

> Kies voor iedere aanroep een type `T`. De parameter en de returnwaarde gebruiken hetzelfde type.

Bij:

```rust,ignore
eerste(42)
```

is `T` bijvoorbeeld `i32`.

Bij:

```rust,ignore
eerste(true)
```

is `T` `bool`.

Bij:

```rust,ignore
eerste(String::from("Hallo"))
```

is `T` `String`.

---

## 4. Type inference werkt ook bij generics

We hoeven meestal niet zelf te schrijven welk type `T` is.

Rust kan het vaak afleiden.

Dus:

```rust,ignore
let getal = eerste(42);
```

is voldoende.

Je zou theoretisch ook expliciet kunnen schrijven:

```rust,ignore
let getal = eerste::<i32>(42);
```

Maar dat is meestal niet nodig.

De syntax:

```text id="v2k8dy"
::<i32>
```

noemen we een **turbofish**.

Je hoeft deze syntax voorlopig alleen te herkennen.

---

## 5. Generics bij structs

Generics kunnen ook bij structs worden gebruikt.

Bijvoorbeeld:

```rust,ignore
struct Zak<T> {
    inhoud: T,
}
```

Nu kan een `Zak` verschillende typen bevatten.

Een zak met een getal:

```rust,ignore
let goud = Zak {
    inhoud: 100,
};
```

Hier is:

```text id="7v4k2n"
T = i32
```

Een zak met een string:

```rust,ignore
let naam = Zak {
    inhoud: String::from("Schat"),
};
```

Hier is:

```text id="p1j8qa"
T = String
```

De struct is dus hetzelfde, maar het type van `inhoud` kan verschillen.

---

## 6. Een generieke RPG-struct

We kunnen dit bijvoorbeeld gebruiken voor een beloning:

```rust,ignore
struct Beloning<T> {
    waarde: T,
}
```

Een beloning met goud:

```rust,ignore
let goud = Beloning {
    waarde: 100,
};
```

Een beloning met een item:

```rust,ignore
let item = Beloning {
    waarde: String::from("Magisch zwaard"),
};
```

We hebben dus één struct die verschillende soorten waarden kan bevatten.

---

## 7. Meerdere generieke typen

We kunnen meerdere generieke typen gebruiken.

Bijvoorbeeld:

```rust,ignore
struct Paar<T, U> {
    eerste: T,
    tweede: U,
}
```

Dan kunnen we bijvoorbeeld schrijven:

```rust,ignore
let speler = Paar {
    eerste: String::from("Aria"),
    tweede: 100,
};
```

Hier zijn:

```text id="g8q2wl"
T = String
U = i32
```

De twee velden hoeven dus niet hetzelfde type te hebben.

---

## 8. Generieke functies met twee typen

Hetzelfde kan bij functies:

```rust,ignore
fn maak_paar<T, U>(eerste: T, tweede: U) -> Paar<T, U> {
    Paar {
        eerste,
        tweede,
    }
}
```

We kunnen dan:

```rust,ignore
let resultaat = maak_paar(
    String::from("Aria"),
    100,
);
```

gebruiken.

Rust leidt af:

```text id="q8k1sv"
T = String
U = i32
```

---

## 9. Een generieke functie kan nog niets met `T` doen

Hier komen we bij een belangrijk punt.

Kijk naar:

```rust,ignore
fn toon<T>(waarde: T) {
    println!("{waarde}");
}
```

Dit compileert niet zomaar.

Waarom?

Rust weet niet dat ieder mogelijk type `T` met `{waarde}` geprint kan worden.

We moeten aangeven welke eigenschappen `T` moet hebben.

Daarvoor gebruiken we **traits**.

---

## 10. Generics en traits

We hebben eerder traits geleerd.

Een trait beschrijft gedrag.

Bijvoorbeeld:

```rust,ignore
trait Aanvaller {
    fn aanvalskracht(&self) -> i32;
}
```

We kunnen een generieke functie schrijven die alleen typen accepteert die `Aanvaller` implementeren:

```rust,ignore
fn toon_aanval<T: Aanvaller>(aanvaller: &T) {
    println!(
        "Aanvalskracht: {}",
        aanvaller.aanvalskracht()
    );
}
```

Hier betekent:

```text id="0y4d6r"
<T: Aanvaller>
```

ongeveer:

> `T` mag ieder type zijn, zolang `T` de trait `Aanvaller` implementeert.

---

## 11. Trait bound

De voorwaarde:

```rust,ignore
T: Aanvaller
```

noemen we een **trait bound**.

Een trait bound beperkt welke typen als `T` gebruikt mogen worden.

Bijvoorbeeld:

```rust,ignore
struct Speler {
    naam: String,
    kracht: i32,
}
```

en:

```rust,ignore
struct Vijand {
    naam: String,
    kracht: i32,
}
```

Beide kunnen:

```rust,ignore
impl Aanvaller for Speler {
    fn aanvalskracht(&self) -> i32 {
        self.kracht
    }
}
```

en:

```rust,ignore
impl Aanvaller for Vijand {
    fn aanvalskracht(&self) -> i32 {
        self.kracht
    }
}
```

hebben.

Dan kunnen beide aan:

```rust,ignore
toon_aanval(&speler);
toon_aanval(&vijand);
```

worden doorgegeven.

---

## 12. Waarom is dit handig?

Zonder generics zouden we bijvoorbeeld kunnen schrijven:

```rust,ignore
fn toon_speler_aanval(speler: &Speler) {
    println!("{}", speler.aanvalskracht());
}

fn toon_vijand_aanval(vijand: &Vijand) {
    println!("{}", vijand.aanvalskracht());
}
```

Dat werkt, maar er staat veel dubbele code.

Met een generic:

```rust,ignore
fn toon_aanval<T: Aanvaller>(aanvaller: &T) {
    println!("{}", aanvaller.aanvalskracht());
}
```

hebben we één functie.

Het belangrijkste idee is:

> Niet het concrete type is belangrijk, maar het gedrag dat het type aanbiedt.

---

## 13. Meerdere trait bounds

Een type kan aan meerdere voorwaarden moeten voldoen.

Bijvoorbeeld:

```rust,ignore
fn toon<T: Aanvaller + Levend>(karakter: &T) {
    println!(
        "Aanval: {}, levend: {}",
        karakter.aanvalskracht(),
        karakter.is_levend()
    );
}
```

Hier moet `T` beide traits implementeren:

```text id="5v8q1j"
Aanvaller
    +
Levend
```

Je kunt dit lezen als:

> Deze functie werkt met ieder type dat zowel `Aanvaller` als `Levend` is.

---

## 14. Trait bounds met `where`

Bij meerdere voorwaarden kan de normale syntax lastig leesbaar worden.

Dan kunnen we `where` gebruiken:

```rust,ignore
fn toon<T>(karakter: &T)
where
    T: Aanvaller + Levend,
{
    println!(
        "Aanval: {}",
        karakter.aanvalskracht()
    );
}
```

Dit betekent hetzelfde als:

```rust,ignore
fn toon<T: Aanvaller + Levend>(karakter: &T) {
    // ...
}
```

Voor korte functies is de eerste vorm prima.

Bij ingewikkeldere functies is `where` vaak duidelijker.

---

## 15. Generics met gewone typen

Een trait bound hoeft niet altijd een zelfgemaakte trait te zijn.

Rust heeft veel standaardtraits.

Bijvoorbeeld:

```rust,ignore
fn toon<T: std::fmt::Debug>(waarde: T) {
    println!("{waarde:?}");
}
```

Hier zeggen we:

> `T` moet `Debug` implementeren.

Dat hebben we eerder al gezien bij `#[derive(Debug)]`.

Bijvoorbeeld:

```rust,ignore
#[derive(Debug)]
struct Speler {
    naam: String,
}
```

Dan kunnen we:

```rust,ignore
toon(speler);
```

gebruiken.

---

## 16. `Clone` als trait bound

Hetzelfde principe geldt voor `Clone`.

```rust,ignore
fn kopieer<T: Clone>(waarde: &T) -> T {
    waarde.clone()
}
```

Deze functie kan met ieder type werken dat `Clone` implementeert.

Bijvoorbeeld:

```rust,ignore
let getal = kopieer(&42);
```

of:

```rust,ignore
let naam = kopieer(&String::from("Aria"));
```

De voorwaarde is steeds:

```text id="h3k7yb"
T: Clone
```

---

## 17. Generics en `Vec`

We hebben al vaak functies geschreven die specifiek met `Vec<i32>` werken.

Bijvoorbeeld:

```rust,ignore
fn aantal(getallen: &Vec<i32>) -> usize {
    getallen.len()
}
```

We kunnen dit generieker maken:

```rust,ignore
fn aantal<T>(waarden: &Vec<T>) -> usize {
    waarden.len()
}
```

Nu werkt de functie met:

```text id="v5q8ka"
Vec<i32>
Vec<String>
Vec<Speler>
Vec<Vijand>
```

en nog veel meer.

De functie hoeft niet te weten wat `T` precies is.

Hij hoeft alleen te weten dat er een `Vec<T>` is.

---

## 18. `&[T]` is vaak handiger

We hebben eerder al slices gezien:

```rust,ignore
&[Vijand]
```

Een generieke functie kan bijvoorbeeld:

```rust,ignore
fn aantal<T>(waarden: &[T]) -> usize {
    waarden.len()
}
```

Hiermee kunnen we met allerlei typen werken.

Bijvoorbeeld:

```rust,ignore
let getallen = vec![1, 2, 3];
let namen = vec![
    String::from("Aria"),
    String::from("Borin"),
];

println!("{}", aantal(&getallen));
println!("{}", aantal(&namen));
```

---

## 19. Generics en iterators

Nu kunnen we beter begrijpen waarom iterator-methodes zo algemeen zijn.

Een iterator kan bijvoorbeeld waarden van type `T` leveren.

Conceptueel:

```text id="n5y1kr"
Iterator
   ↓
T
   ↓
map
   ↓
U
```

`map()` kan een type `T` veranderen in een ander type `U`.

Bijvoorbeeld:

```rust,ignore
let getallen = vec![1, 2, 3];

let teksten: Vec<String> = getallen
    .iter()
    .map(|getal| getal.to_string())
    .collect();
```

We gaan de precieze definitie van `Iterator::map()` nog niet ontleden.

Het belangrijke idee is:

> Generics maken het mogelijk om één stuk code met veel verschillende typen te laten werken.

---

## 20. `impl Trait`

Rust heeft een tweede syntax die we veel zullen tegenkomen:

```rust,ignore
impl Trait
```

Bijvoorbeeld:

```rust,ignore
fn toon_aanvaller(aanvaller: &impl Aanvaller) {
    println!("{}", aanvaller.aanvalskracht());
}
```

Dit lijkt sterk op:

```rust,ignore
fn toon_aanvaller<T: Aanvaller>(aanvaller: &T) {
    println!("{}", aanvaller.aanvalskracht());
}
```

Voor deze eenvoudige functie betekenen ze praktisch hetzelfde.

Je kunt `impl Trait` hier lezen als:

> Geef mij een type dat deze trait heeft.

---

## 21. Waarom bestaat `impl Trait`?

De generieke syntax:

```rust,ignore
fn toon_aanvaller<T: Aanvaller>(aanvaller: &T)
```

geeft het type een naam:

```text id="a5f7qn"
T
```

Bij:

```rust,ignore
fn toon_aanvaller(aanvaller: &impl Aanvaller)
```

zeggen we alleen:

> Er komt hier een type dat `Aanvaller` implementeert.

Voor eenvoudige functies kan dat prettiger lezen.

---

## 22. `impl Trait` als returntype

`impl Trait` wordt nog interessanter als returntype.

Bijvoorbeeld:

```rust,ignore
fn maak_getallen() -> impl Iterator<Item = i32> {
    1..=5
}
```

We krijgen hier een iterator terug.

De precieze betekenis van:

```text id="j4w7xm"
Item = i32
```

hoeven we nu nog niet volledig te begrijpen.

Lees het voorlopig als:

> Deze iterator levert `i32`-waarden.

Dit soort syntax zullen we veel tegenkomen wanneer we met iterators werken.

---

## 23. Waarom geen `Vec<i32>` teruggeven?

We zouden ook kunnen schrijven:

```rust,ignore
fn maak_getallen() -> Vec<i32> {
    vec![1, 2, 3, 4, 5]
}
```

Dat geeft een `Vec`.

Maar:

```rust,ignore
fn maak_getallen() -> impl Iterator<Item = i32> {
    1..=5
}
```

geeft een iterator.

Dat betekent dat de aanroeper zelf kan bepalen wat ermee gebeurt:

```rust,ignore
let getallen = maak_getallen()
    .filter(|getal| getal % 2 == 0)
    .collect::<Vec<i32>>();
```

De functie levert dus niet meteen een complete verzameling.

Ze levert een manier om de waarden één voor één te krijgen.

---

## 24. Trait objects

Tot nu toe gebruikten we:

```rust,ignore
T: Aanvaller
```

of:

```rust,ignore
impl Aanvaller
```

Er is nog een andere manier:

```rust,ignore
dyn Aanvaller
```

Dit noemen we een **trait object**.

Een trait object zegt:

> Ik weet niet precies welk concreet type hier zit, maar ik weet dat het deze trait heeft.

Bijvoorbeeld:

```rust,ignore
fn toon_aanval(aanvaller: &dyn Aanvaller) {
    println!("{}", aanvaller.aanvalskracht());
}
```

Deze functie accepteert iedere waarde die `Aanvaller` implementeert.

---

## 25. Generic tegenover trait object

Dit verschil is belangrijk.

Een generieke functie:

```rust,ignore
fn toon<T: Aanvaller>(aanvaller: &T) {
    println!("{}", aanvaller.aanvalskracht());
}
```

zegt:

> Er is een concreet type `T` dat `Aanvaller` implementeert.

Een trait object:

```rust,ignore
fn toon(aanvaller: &dyn Aanvaller) {
    println!("{}", aanvaller.aanvalskracht());
}
```

zegt:

> Ik hoef het concrete type niet te kennen. Ik gebruik alleen het gedrag van `Aanvaller`.

Je kunt het mentale verschil zien als:

```text id="w3k9vq"
Generics

T = Speler
of
T = Vijand
of
T = Robot

Trait object

dyn Aanvaller
       ↓
Speler óf Vijand óf Robot
```

---

## 26. Een functie die een trait object ontvangt

Stel:

```rust,ignore
trait Karakter {
    fn naam(&self) -> &str;
}
```

En:

```rust,ignore
struct Speler {
    naam: String,
}

struct Vijand {
    naam: String,
}
```

Beide implementeren de trait:

```rust,ignore
impl Karakter for Speler {
    fn naam(&self) -> &str {
        &self.naam
    }
}

impl Karakter for Vijand {
    fn naam(&self) -> &str {
        &self.naam
    }
}
```

Nu kunnen we:

```rust,ignore
fn toon_karakter(karakter: &dyn Karakter) {
    println!("Karakter: {}", karakter.naam());
}
```

aanroepen met:

```rust,ignore
toon_karakter(&speler);
toon_karakter(&vijand);
```

De functie hoeft niet te weten of hij een `Speler` of `Vijand` heeft gekregen.

---

## 27. Waarom zou je een trait object gebruiken?

Een belangrijk gebruik is wanneer je verschillende concrete typen in één verzameling wilt bewaren.

Bijvoorbeeld:

```rust,ignore
let karakters: Vec<Box<dyn Karakter>> = vec![
    Box::new(speler),
    Box::new(vijand),
];
```

Hier zitten verschillende concrete typen in dezelfde `Vec`.

Dat kan niet rechtstreeks met:

```rust,ignore
Vec<T>
```

als iedere waarde een ander concreet type heeft.

Met:

```rust,ignore
Vec<Box<dyn Karakter>>
```

zeggen we:

> Deze lijst bevat objecten die allemaal `Karakter` zijn, maar het concrete type mag verschillen.

---

## 28. Waarom `Box`?

Dit ziet er ingewikkeld uit:

```rust,ignore
Box<dyn Karakter>
```

We hebben `Box` nog niet uitgebreid behandeld.

Voorlopig is het voldoende om te weten:

> `Box` bewaart een waarde op de heap en geeft ons een eigenaar van die waarde.

Een trait object zoals:

```rust,ignore
dyn Karakter
```

heeft geen vaste grootte die de compiler vooraf kent.

Daarom gebruiken we vaak:

```rust,ignore
Box<dyn Karakter>
```

als we het object daadwerkelijk ergens willen opslaan.

We zullen `Box` later uitgebreider behandelen.

---

## 29. Een RPG met verschillende karakters

Stel:

```rust,ignore
trait Karakter {
    fn naam(&self) -> &str;
    fn gezondheid(&self) -> i32;
}
```

We hebben:

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

Beide implementeren `Karakter`.

Dan kunnen we:

```rust,ignore
let karakters: Vec<Box<dyn Karakter>> = vec![
    Box::new(speler),
    Box::new(vijand),
];
```

en:

```rust,ignore
for karakter in &karakters {
    println!(
        "{} heeft {} HP",
        karakter.naam(),
        karakter.gezondheid()
    );
}
```

De lus werkt met beide typen.

---

## 30. Een trait object teruggeven

Een functie kan ook een trait object teruggeven.

Bijvoorbeeld:

```rust,ignore
fn maak_karakter(soort: &str) -> Box<dyn Karakter> {
    if soort == "speler" {
        Box::new(Speler {
            naam: String::from("Aria"),
            gezondheid: 100,
        })
    } else {
        Box::new(Vijand {
            naam: String::from("Goblin"),
            gezondheid: 30,
        })
    }
}
```

De functie kan dus afhankelijk van de invoer verschillende concrete typen maken.

Toch heeft de functie één returntype:

```text id="z2v7nd"
Box<dyn Karakter>
```

De aanroeper hoeft niet te weten welk concreet type terugkomt.

---

## 31. Dit is anders dan een generic returntype

Dit:

```rust,ignore
fn maak<T: Karakter>() -> T {
    // ...
}
```

betekent iets anders.

Hier zegt de aanroeper eigenlijk:

> Ik wil een specifiek type `T`.

De functie moet dan precies dat type kunnen teruggeven.

Bij:

```rust,ignore
fn maak() -> Box<dyn Karakter> {
    // ...
}
```

kan de functie zelf kiezen welk concreet type ze teruggeeft.

Dat is een belangrijk verschil.

---

## 32. Generics en trait objects naast elkaar

Een handige vergelijking:

| Vorm | Betekenis |
| --- | --- |
| `T` | een concreet, generiek type |
| `T: Trait` | `T` moet `Trait` implementeren |
| `impl Trait` | een onbekend concreet type dat `Trait` implementeert |
| `dyn Trait` | een trait object; het concrete type is bewust verborgen |
| `Box<dyn Trait>` | een trait object dat eigendom van een waarde bevat |

Je hoeft deze tabel nog niet volledig uit je hoofd te kennen.

Het belangrijkste onderscheid is:

```text id="f7n3mc"
T
↓
het concrete type blijft onderdeel van de functie

dyn Trait
↓
alleen het gedrag van de trait is belangrijk
```

---

## 33. Statische en dynamische dispatch

Hier komen we een technisch begrip tegen.

Bij generics kan Rust de concrete typen tijdens het compileren kennen.

Bijvoorbeeld:

```rust,ignore
fn aanval<T: Aanvaller>(karakter: &T) {
    karakter.aanvalskracht();
}
```

Als we dit gebruiken met een `Speler`, weet Rust dat het om een `Speler` gaat.

Dit noemen we **statische dispatch**.

Bij:

```rust,ignore
fn aanval(karakter: &dyn Aanvaller) {
    karakter.aanvalskracht();
}
```

is het concrete type op dat punt bewust verborgen.

Rust bepaalt tijdens het uitvoeren welk passend gedrag gebruikt moet worden.

Dit noemen we **dynamische dispatch**.

Voor deze cursus is vooral het concept belangrijk:

```text id="2p6y8c"
generic
→ concreet type bekend

dyn Trait
→ concreet type verborgen
```

De keuze tussen beide is een technisch ontwerpbesluit. Het is niet zo dat één vorm altijd beter is.

---

## 34. Een trait object heeft alleen trait-gedrag

Stel:

```rust,ignore
trait Karakter {
    fn naam(&self) -> &str;
}
```

En `Speler` heeft daarnaast:

```rust,ignore
fn speciaal_aanval(&self) {
    println!("Speciale aanval!");
}
```

Als we een:

```rust,ignore
```

hebben, kunnen we alleen de methodes gebruiken die in `Karakter` staan.

Dus:

```rust,ignore
karakter.naam();
```

kan wel.

Maar:

```rust,ignore
karakter.speciaal_aanval();
```

kan niet.

Het trait object weet immers alleen van het gedrag dat door de trait wordt beschreven.

---

## 35. Een generic functie kan ook meerdere concrete typen verwerken

We kunnen bijvoorbeeld:

```rust,ignore
fn print_karakter<T: Karakter>(karakter: &T) {
    println!("{}", karakter.naam());
}
```

gebruiken voor:

```rust,ignore
print_karakter(&speler);
print_karakter(&vijand);
```

Maar we kunnen ook:

```rust,ignore
fn print_karakter(karakter: &dyn Karakter) {
    println!("{}", karakter.naam());
}
```

gebruiken.

Voor eenvoudige functies kunnen beide oplossingen er bijna hetzelfde uitzien.

Het verschil wordt vooral interessant wanneer we waarden willen **opslaan of combineren**.

---

## 36. Verschillende typen in één `Vec`

Dit kan niet:

```rust,ignore
let karakters = vec![
    speler,
    vijand,
];
```

als `speler` en `vijand` verschillende concrete typen zijn.

Een `Vec<T>` heeft namelijk één elementtype.

Dit kan wel:

```rust,ignore
let karakters: Vec<Box<dyn Karakter>> = vec![
    Box::new(speler),
    Box::new(vijand),
];
```

Nu hebben alle elementen hetzelfde zichtbare type:

```text id="d1v6rp"
Box<dyn Karakter>
```

Hoewel de concrete typen daarbinnen verschillen.

---

## 37. Een eenvoudige game-engine

Dit geeft ons een interessant RPG-patroon.

We kunnen bijvoorbeeld verschillende soorten objecten maken:

```rust,ignore
trait Aanvaller {
    fn aanvalskracht(&self) -> i32;
}
```

Daarna:

```rust,ignore
struct Speler {
    kracht: i32,
}

struct Monster {
    kracht: i32,
}

struct Valk {
    kracht: i32,
}
```

Alle drie implementeren `Aanvaller`.

We kunnen dan:

```rust,ignore
let aanvallers: Vec<Box<dyn Aanvaller>> = vec![
    Box::new(Speler { kracht: 20 }),
    Box::new(Monster { kracht: 15 }),
    Box::new(Valk { kracht: 8 }),
];
```

en:

```rust,ignore
for aanvaller in &aanvallers {
    println!("{}", aanvaller.aanvalskracht());
}
```

Dit patroon is erg bruikbaar wanneer een programma verschillende soorten objecten heeft die allemaal hetzelfde gedrag moeten aanbieden.

---

## 38. Generics versus trait objects: een praktische keuze

Gebruik generics bijvoorbeeld wanneer je zegt:

> Deze functie werkt voor ieder type dat aan deze voorwaarden voldoet.

Bijvoorbeeld:

```rust,ignore
fn toon<T: Aanvaller>(aanvaller: &T) {
    println!("{}", aanvaller.aanvalskracht());
}
```

Gebruik een trait object bijvoorbeeld wanneer je zegt:

> Ik wil verschillende concrete typen als één soort ding behandelen.

Bijvoorbeeld:

```rust,ignore
Vec<Box<dyn Aanvaller>>
```

Een eenvoudige vuistregel:

```text id="q3m8sv"
Generic
→ verschillende typen kunnen dezelfde functie gebruiken

Trait object
→ verschillende typen kunnen samen als één soort object worden behandeld
```

Dit is geen absolute regel, maar voor beginners is het een nuttig startpunt.

---

## 39. Een belangrijk verschil bij returnwaarden

Vergelijk:

```rust,ignore
fn maak_speler() -> impl Karakter {
    Speler {
        naam: String::from("Aria"),
        gezondheid: 100,
    }
}
```

met:

```rust,ignore
fn maak_karakter(speler: bool) -> Box<dyn Karakter> {
    if speler {
        Box::new(Speler {
            naam: String::from("Aria"),
            gezondheid: 100,
        })
    } else {
        Box::new(Vijand {
            naam: String::from("Goblin"),
            gezondheid: 30,
        })
    }
}
```

In het eerste geval is er één concreet type achter `impl Karakter`.

In het tweede geval kan de functie verschillende concrete typen teruggeven.

Dat is een belangrijk verschil:

```text id="s6f1kr"
impl Trait
→ één verborgen concreet type

dyn Trait
→ verschillende concrete typen mogelijk
```

---

## 40. `impl Trait` en iterator-functies

Dit is precies waarom we `impl Trait` bij iterators vaak tegenkomen.

Bijvoorbeeld:

```rust,ignore
fn levende_vijanden<'a>(
    vijanden: &'a [Vijand]
) -> impl Iterator<Item = &'a Vijand> {
    vijanden
        .iter()
        .filter(|vijand| vijand.gezondheid > 0)
}
```

De functie geeft een iterator terug zonder dat we het ingewikkelde concrete type van de iterator hoeven te schrijven.

Dat type kan namelijk behoorlijk lang en ingewikkeld worden.

Voorlopig is dit voldoende:

```text id="d2k7vx"
impl Iterator<Item = ...>
```

betekent:

> Deze functie geeft een iterator terug die waarden van een bepaald type levert.

De lifetime in dit voorbeeld bespreken we nu niet verder. Die komt later.

---

## 41. Zelfstandige opdrachten

### Opdracht 1 — Generieke schatkist

Maak:

```rust,ignore
struct Schatkist<T> {
    inhoud: T,
}
```

Een schatkist kan bijvoorbeeld bevatten:

- goud
- een `String`
- een `Item`.

Schrijf een generieke functie om de inhoud terug te geven.

---

### Opdracht 2 — Aanvallers

Maak:

```rust,ignore
trait Aanvaller {
    fn aanvalskracht(&self) -> i32;
}
```

Maak drie typen:

```text
Speler
Monster
Valk
```

Laat alle drie `Aanvaller` implementeren.

Schrijf zowel:

```rust,ignore
fn toon<T: Aanvaller>(...)
```

als:

```rust,ignore
fn toon_dyn(...)
```

en gebruik beide.

---

### Opdracht 3 — Gemengde groep

Maak:

```rust,ignore
Vec<Box<dyn Aanvaller>>
```

met meerdere verschillende typen.

Bereken de totale aanvalskracht.

---

### Opdracht 4 — Karakterfabriek

Maak:

```rust,ignore
fn maak_karakter(soort: &str) -> Box<dyn Karakter>
```

Ondersteun verschillende karaktertypen.

Breid de functie uit zodat een onbekend type netjes wordt afgehandeld.

Een `Result<Box<dyn Karakter>, String>` is hier een goede oplossing.

---

### Opdracht 5 — Generic RPG-functie

Maak een generieke functie die met ieder type werkt dat:

```text
Aanvaller
+
Levend
```

implementeert.

De functie moet alleen aanvallers tonen die nog leven.

---

## 42. Veelgemaakte fouten

### Fout 1 — denken dat `T` een concreet type is

Bij:

```rust,ignore
fn toon<T>(waarde: T)
```

is `T` geen concreet type zoals `i32`.

Het is een placeholder voor een type.

---

### Fout 2 — een methode gebruiken zonder trait bound

Dit:

```rust,ignore
fn toon<T>(waarde: T) {
    println!("{waarde:?}");
}
```

werkt niet automatisch.

Rust weet niet dat ieder type `Debug` implementeert.

Je hebt bijvoorbeeld nodig:

```rust,ignore
fn toon<T: std::fmt::Debug>(waarde: T) {
    println!("{waarde:?}");
}
```

---

### Fout 3 — `dyn Trait` verwarren met een generic

Deze:

```rust,ignore
T: Karakter
```

en:

```rust,ignore
dyn Karakter
```

zijn niet hetzelfde.

Bij generics blijft het concrete type onderdeel van de functie.

Bij een trait object verbergen we het concrete type bewust.

---

### Fout 4 — verschillende typen rechtstreeks in één `Vec`

Dit werkt niet als `Speler` en `Vijand` verschillende concrete typen zijn:

```rust,ignore
vec![speler, vijand]
```

Gebruik bijvoorbeeld:

```rust,ignore
Vec<Box<dyn Karakter>>
```

als je verschillende typen met hetzelfde gedrag samen wilt bewaren.

---

### Fout 5 — `impl Trait` zien als `dyn Trait`

Deze:

```rust,ignore
-> impl Karakter
```

en:

```rust,ignore
-> Box<dyn Karakter>
```

hebben een belangrijk verschil.

`impl Trait` verbergt één concreet type.

`dyn Trait` kan verschillende concrete typen achter hetzelfde trait object verbergen.

---

## 43. Het belangrijkste mentale model

Probeer generics niet te zien als een ingewikkelde speciale syntax.

Het basisidee is:

```text id="f8m3kc"
"Deze code moet met meerdere typen kunnen werken."
```

Dan kun je vragen:

### Heb ik alleen een willekeurig type nodig?

Gebruik:

```rust,ignore
<T>
```

### Heeft het type bepaald gedrag nodig?

Gebruik:

```rust,ignore
<T: Trait>
```

### Wil ik een functie schrijven zonder het concrete type te benoemen?

Gebruik bijvoorbeeld:

```rust,ignore
impl Trait
```

### Wil ik het concrete type bewust verbergen?

Gebruik:

```rust,ignore
dyn Trait
```

### Wil ik verschillende concrete typen samen bewaren?

Vaak:

```rust,ignore
Box<dyn Trait>
```

---

## 44. Samenvatting

We hebben in dit hoofdstuk geleerd dat generics code herbruikbaar maken voor verschillende typen.

Een eenvoudige generic:

```rust,ignore
fn geef_terug<T>(waarde: T) -> T
```

Een generic met een trait bound:

```rust,ignore
fn aanval<T: Aanvaller>(aanvaller: &T)
```

Een `where`-blok:

```rust,ignore
fn aanval<T>(aanvaller: &T)
where
    T: Aanvaller,
{
    // ...
}
```

`impl Trait`:

```rust,ignore
fn aanval(aanvaller: &impl Aanvaller)
```

Een trait object als parameter:

```rust,ignore
fn aanval(aanvaller: &dyn Aanvaller)
```

Een trait object als returnwaarde:

```rust,ignore
fn maak_aanvaller() -> Box<dyn Aanvaller>
```

En verschillende concrete typen samen:

```rust,ignore
let aanvallers: Vec<Box<dyn Aanvaller>> = vec![
    Box::new(speler),
    Box::new(monster),
    Box::new(valk),
];
```

Het belangrijkste onderscheid om te onthouden:

```text id="k2w7np"
GENERIC
T: Trait
    ↓
"Ik werk met ieder concreet type
 dat aan deze voorwaarde voldoet."

TRAIT OBJECT
dyn Trait
    ↓
"Ik hoef het concrete type niet
 te kennen; ik gebruik alleen
 het gedrag van deze trait."
```

Generics zijn daarmee geen losstaand onderwerp. Ze vormen een belangrijke verbinding tussen de concepten die we al hebben geleerd:

```text id="y5q8mc"
Structs
   ↓
Traits
   ↓
Generics
   ↓
Closures
   ↓
Iterators
   ↓
Iterator adaptors
```

En juist daardoor begint steeds duidelijker te worden waarom Rust veel van zijn code zo algemeen kan maken zonder de controle en typeveiligheid van de compiler kwijt te raken.

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 24](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_24/).

---

## Controlelijst

Je bent klaar met dit artikel als je zonder hulp:

- [ ] weet wat een generic type-parameter `T` is
- [ ] een generieke functie en een generieke struct kunt schrijven
- [ ] trait bounds kunt toepassen (`T: Debug + Clone`)
- [ ] `where`-clausules kunt gebruiken voor overzichtelijke trait bounds
- [ ] het verschil kent tussen statische dispatch (generics) en dynamische dispatch (`Box<dyn Trait>`).
