# 17. Automatische tests

## Wat gaan we leren?

In dit artikel leren we hoe we automatische tests schrijven om te controleren of onze code correct werkt.

We leren:

- wat automatische tests zijn en waarom ze een belangrijk vangnet vormen
- het `#[test]` attribuut gebruiken
- controleren met `assert_eq!`, `assert_ne!` en `assert!`
- tests uitvoeren met `cargo test`
- tests organiseren in een `tests`-module met `#[cfg(test)]` en `use super::*;`
- hoe falende tests ons helpen bugs snel en gericht op te lossen.

---

## 1. Onze eerste test

Stel dat we deze functie hebben:

```rust,ignore
fn verdubbel(getal: i32) -> i32 {
    getal * 2
}
```

We verwachten bijvoorbeeld:

```text
verdubbel(5) → 10
```

We kunnen dat automatisch testen:

```rust,ignore
#[test]
fn verdubbel_werkt() {
    assert_eq!(verdubbel(5), 10);
}
```

Het belangrijke onderdeel is:

```rust,ignore
#[test]
```

Daarmee vertellen we Rust:

> Dit is een test die automatisch uitgevoerd moet kunnen worden.

---

## 2. Tests uitvoeren

Tests worden uitgevoerd met:

```text id="5f6h2x"
cargo test
```

Cargo zoekt vervolgens naar functies met:

```rust,ignore
#[test]
```

en voert die uit.

Een geslaagde test ziet er ongeveer zo uit:

```text id="k5yq3a"
running 1 test
test tests::verdubbel_werkt ... ok

test result: ok. 1 passed; 0 failed
```

De precieze uitvoer kan per Rust-versie verschillen.

Het belangrijkste is:

```text id="l2b6a0"
1 passed
0 failed
```

---

## 3. Een test is een controle

Deze test:

```rust,ignore
#[test]
fn verdubbel_werkt() {
    assert_eq!(verdubbel(5), 10);
}
```

zegt eigenlijk:

> Ik verwacht dat `verdubbel(5)` gelijk is aan `10`.

Als dat waar is, slaagt de test.

Als het niet waar is, faalt de test.

---

## 4. Een test mag falen

Dit is belangrijk.

Een falende test betekent niet automatisch dat er iets verschrikkelijks is gebeurd.

Bijvoorbeeld:

```rust,ignore
#[test]
fn verdubbel_werkt() {
    assert_eq!(verdubbel(5), 11);
}
```

Onze functie geeft nog steeds `10` terug.

De test verwacht `11`.

De test zal dus falen.

Rust vertelt ons ongeveer:

```text
expected: 11
actual:   10
```

Dat is nuttige informatie.

We hebben ontdekt:

> Onze verwachting en het gedrag van het programma komen niet overeen.

Dat kan betekenen dat:

- de test verkeerd is
- de code verkeerd is
- we de bedoeling van het programma moeten aanpassen.

---

## 5. `assert_eq!`

De meest gebruikte test die we voorlopig nodig hebben is:

```rust,ignore
assert_eq!(links, rechts);
```

Dit betekent:

> Verwacht dat links en rechts gelijk zijn.

Bijvoorbeeld:

```rust,ignore
assert_eq!(2 + 3, 5);
```

Of:

```rust,ignore
assert_eq!(verdubbel(10), 20);
```

---

## 6. `assert_ne!`

We kunnen ook testen dat twee waarden **niet** gelijk zijn.

```rust,ignore
assert_ne!(verdubbel(5), 11);
```

Dit betekent:

> Verwacht dat `verdubbel(5)` niet gelijk is aan `11`.

---

## 7. `assert!`

Met `assert!` testen we of iets `true` is.

Bijvoorbeeld:

```rust,ignore
fn is_levend(gezondheid: i32) -> bool {
    gezondheid > 0
}
```

We kunnen schrijven:

```rust,ignore
#[test]
fn speler_met_gezondheid_leeft() {
    assert!(is_levend(100));
}
```

En:

```rust,ignore
#[test]
fn speler_met_nul_gezondheid_is_dood() {
    assert!(!is_levend(0));
}
```

Hier betekent:

```rust,ignore
assert!(...)
```

dat de waarde `true` moet zijn.

En:

```rust,ignore
assert!(!...)
```

betekent dat de waarde `false` moet zijn.

---

## 8. Meerdere tests

Een programma kan veel tests hebben.

```rust,ignore
fn verdubbel(getal: i32) -> i32 {
    getal * 2
}

#[test]
fn verdubbel_vijf() {
    assert_eq!(verdubbel(5), 10);
}

#[test]
fn verdubbel_tien() {
    assert_eq!(verdubbel(10), 20);
}

#[test]
fn verdubbel_nul() {
    assert_eq!(verdubbel(0), 0);
}
```

`cargo test` voert alle drie uit.

---

## 9. Tests met verschillende situaties

Een goede test kijkt niet alleen naar één normaal voorbeeld.

Stel:

```rust,ignore
fn bereken_schade(kracht: i32, wapenschade: i32) -> i32 {
    kracht + wapenschade
}
```

We kunnen testen:

```rust,ignore
#[test]
fn normale_schade() {
    assert_eq!(bereken_schade(10, 5), 15);
}

#[test]
fn geen_wapenschade() {
    assert_eq!(bereken_schade(10, 0), 10);
}

#[test]
fn geen_kracht() {
    assert_eq!(bereken_schade(0, 10), 10);
}
```

Iedere test controleert een andere situatie.

---

## 10. Tests voor onze RPG

We kunnen nu onze eigen gamecode testen.

Bijvoorbeeld:

```rust,ignore
fn genees(gezondheid: i32, hoeveelheid: i32) -> i32 {
    gezondheid + hoeveelheid
}
```

Daarbij horen tests:

```rust,ignore
#[test]
fn genezing_werkt() {
    assert_eq!(genees(50, 20), 70);
}
```

En:

```rust,ignore
#[test]
fn genezing_met_nul() {
    assert_eq!(genees(50, 0), 50);
}
```

---

## 11. Een test voor schade

```rust,ignore
fn neem_schade(gezondheid: i32, schade: i32) -> i32 {
    gezondheid - schade
}
```

Tests:

```rust,ignore
#[test]
fn schade_werkt() {
    assert_eq!(neem_schade(100, 25), 75);
}

#[test]
fn zware_schade() {
    assert_eq!(neem_schade(100, 100), 0);
}
```

---

## 12. Tests voor een struct

We kunnen ook onze `Vijand` testen.

```rust,ignore
struct Vijand {
    naam: String,
    gezondheid: i32,
}
```

We maken bijvoorbeeld een functie:

```rust,ignore
fn neem_schade(vijand: &mut Vijand, schade: i32) {
    vijand.gezondheid -= schade;
}
```

Daarbij kunnen we schrijven:

```rust,ignore
#[test]
fn vijand_krijgt_schade() {
    let mut vijand = Vijand {
        naam: String::from("Goblin"),
        gezondheid: 50,
    };

    neem_schade(&mut vijand, 20);

    assert_eq!(vijand.gezondheid, 30);
}
```

De test doet eigenlijk precies wat een speler zou doen:

1. maak een Goblin
2. geef hem schade
3. kijk hoeveel gezondheid hij nog heeft.

---

## 13. Tests voor methods

Onze eerdere `impl`-kennis kunnen we hier ook gebruiken.

Bijvoorbeeld:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
}

impl Speler {
    fn neem_schade(&mut self, schade: i32) {
        self.gezondheid -= schade;
    }

    fn is_levend(&self) -> bool {
        self.gezondheid > 0
    }
}
```

We kunnen beide methods testen:

```rust,ignore
#[test]
fn speler_krijgt_schade() {
    let mut speler = Speler {
        naam: String::from("Arin"),
        gezondheid: 100,
    };

    speler.neem_schade(30);

    assert_eq!(speler.gezondheid, 70);
}
```

En:

```rust,ignore
#[test]
fn speler_is_levend() {
    let speler = Speler {
        naam: String::from("Arin"),
        gezondheid: 100,
    };

    assert!(speler.is_levend());
}
```

---

## 14. De test voor een dode speler

We kunnen ook de andere situatie testen:

```rust,ignore
#[test]
fn speler_is_dood() {
    let speler = Speler {
        naam: String::from("Arin"),
        gezondheid: 0,
    };

    assert!(!speler.is_levend());
}
```

Het is vaak verstandig om beide kanten van een regel te testen.

Bij:

```rust,ignore
gezondheid > 0
```

testen we dus zowel:

```text
gezondheid = 100
```

als:

```text
gezondheid = 0
```

---

## 15. Een belangrijke gewoonte

Stel dat we deze regel hebben:

```rust,ignore
if speler.gezondheid > 0 {
    // leeft
}
```

Dan is het verstandig om niet alleen een test voor `100` te maken.

Test ook de grens:

```text id="s4e7kb"
1
0
```

Want juist rond grenzen ontstaan vaak fouten.

Bijvoorbeeld:

```rust,ignore
#[test]
fn een_gezondheid_is_levend() {
    let speler = Speler {
        naam: String::from("Arin"),
        gezondheid: 1,
    };

    assert!(speler.is_levend());
}
```

En:

```rust,ignore
#[test]
fn nul_gezondheid_is_dood() {
    let speler = Speler {
        naam: String::from("Arin"),
        gezondheid: 0,
    };

    assert!(!speler.is_levend());
}
```

---

## 16. Tests voor Vecs

Nu kunnen we ons recente werk met `Vec` testen.

Stel:

```rust,ignore
fn aoe_schade(vijanden: &mut Vec<Vijand>, schade: i32) {
    for vijand in vijanden {
        vijand.gezondheid -= schade;
    }
}
```

We kunnen dit testen.

```rust,ignore
#[test]
fn aoe_schade_raakt_alle_vijanden() {
    let mut vijanden = vec![
        Vijand {
            naam: String::from("Goblin"),
            gezondheid: 50,
        },
        Vijand {
            naam: String::from("Ork"),
            gezondheid: 70,
        },
        Vijand {
            naam: String::from("Trol"),
            gezondheid: 100,
        },
    ];

    aoe_schade(&mut vijanden, 20);

    assert_eq!(vijanden[0].gezondheid, 30);
    assert_eq!(vijanden[1].gezondheid, 50);
    assert_eq!(vijanden[2].gezondheid, 80);
}
```

Dit is al een behoorlijk serieuze test.

We controleren namelijk dat **ieder doel** geraakt wordt.

---

## 17. Een test mag ook falen

Stel dat we per ongeluk dit schrijven:

```rust,ignore
fn aoe_schade(vijanden: &mut Vec<Vijand>, schade: i32) {
    if let Some(vijand) = vijanden.get_mut(0) {
        vijand.gezondheid -= schade;
    }
}
```

Deze code raakt alleen de eerste vijand.

Onze test:

```rust,ignore
assert_eq!(vijanden[1].gezondheid, 50);
```

zal dan falen.

Dat is precies wat we willen.

De test heeft een bug gevonden.

---

## 18. Tests zijn een vangnet

Stel dat ons spel steeds groter wordt.

We veranderen later iets aan `aoe_schade`.

Zonder test moeten we onthouden:

> Ik moet controleren of alle vijanden nog steeds schade krijgen.

Met de test kunnen we gewoon:

```text id="m6z7aa"
cargo test
```

uitvoeren.

Als alle tests slagen, weten we dat de situaties die onze tests controleren nog steeds goed werken.

Dat betekent niet dat het hele programma gegarandeerd foutloos is.

Het betekent:

> De code voldoet nog steeds aan de verwachtingen die we in onze tests hebben vastgelegd.

---

## 19. Waar zet je tests?

Rust ondersteunt verschillende manieren om tests te organiseren.

Voor deze cursus gebruiken we eerst de eenvoudige vorm:

```rust,ignore
#[cfg(test)]
mod tests {
    #[test]
    fn mijn_test() {
        // ...
    }
}
```

Bijvoorbeeld:

```rust,ignore
fn verdubbel(getal: i32) -> i32 {
    getal * 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn verdubbel_werkt() {
        assert_eq!(verdubbel(5), 10);
    }
}
```

---

## 20. Wat doet `#[cfg(test)]`?

Dit vertelt Rust dat dit gedeelte bedoeld is voor tests.

```rust,ignore
#[cfg(test)]
mod tests {
    // tests
}
```

De tests staan dus bij elkaar in een aparte `tests`-module.

Voorlopig hoef je alleen te onthouden:

> Zet gewone unit tests in een `tests`-module met `#[cfg(test)]`.

We hoeven de precieze werking van `cfg` nog niet uitgebreid te behandelen.

---

## 21. Wat doet `use super::*`?

Binnen de `tests`-module willen we functies en structs gebruiken die buiten de module staan.

Daarom zien we vaak:

```rust,ignore
use super::*;
```

Voorlopig kun je dit lezen als:

> Maak de dingen uit de bovenliggende module beschikbaar in deze testmodule.

Ook hier hoeven we de precieze regels van modules en visibility nog niet volledig te behandelen.

---

## 22. Een compleet voorbeeld

Hier staat alles bij elkaar:

```rust,ignore
struct Speler {
    naam: String,
    gezondheid: i32,
}

impl Speler {
    fn neem_schade(&mut self, schade: i32) {
        self.gezondheid -= schade;
    }

    fn is_levend(&self) -> bool {
        self.gezondheid > 0
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn speler_krijgt_schade() {
        let mut speler = Speler {
            naam: String::from("Arin"),
            gezondheid: 100,
        };

        speler.neem_schade(25);

        assert_eq!(speler.gezondheid, 75);
    }

    #[test]
    fn speler_leeft() {
        let speler = Speler {
            naam: String::from("Arin"),
            gezondheid: 100,
        };

        assert!(speler.is_levend());
    }

    #[test]
    fn speler_is_dood() {
        let speler = Speler {
            naam: String::from("Arin"),
            gezondheid: 0,
        };

        assert!(!speler.is_levend());
    }
}
```

Voer uit:

```text id="c8q2zr"
cargo test
```

Alle drie de tests zouden moeten slagen.

---

## 23. Eindopdracht — Test je RPG

Maak een klein RPG-programma met:

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

Gebruik methods voor bijvoorbeeld:

```text id="f7n2kc"
neem_schade
genees
is_levend
```

Maak vervolgens tests voor:

1. normale schade
2. zware schade
3. genezing
4. een speler die leeft
5. een speler die dood is
6. één vijand
7. meerdere vijanden
8. AoE-schade.

Probeer minimaal één test bewust te laten falen.

Bekijk de foutmelding.

Herstel daarna de test of de code.

---

## 24. Een belangrijke nieuwe manier van werken

Tot nu toe was onze werkwijze vaak:

```text
Code schrijven
      ↓
Programma uitvoeren
      ↓
Kijken wat er gebeurt
```

Met tests kunnen we werken als:

```text
Verwachting bedenken
      ↓
Test schrijven
      ↓
Code schrijven
      ↓
cargo test
      ↓
Geslaagd?
   ↙       ↘
 ja         nee
 ↓           ↓
verder     fout zoeken
```

Een test is daarmee niet alleen iets dat je **achteraf** toevoegt.

Een test kan je helpen bepalen wat je programma eigenlijk moet doen.

---

## 25. Wat moet je onthouden?

Een test begint met:

```rust,ignore
#[test]
```

Een gelijkheidstest:

```rust,ignore
assert_eq!(werkelijk, verwacht);
```

Een ongelijkheidstest:

```rust,ignore
assert_ne!(werkelijk, verwacht);
```

Een booleaanse test:

```rust,ignore
assert!(voorwaarde);
```

Een negatieve booleaanse test:

```rust,ignore
assert!(!voorwaarde);
```

Tests voer je uit met:

```text id="w3f7pn"
cargo test
```

En misschien wel het belangrijkste:

> **Een falende test is nuttige informatie.**

De compiler vertelt ons of onze code technisch klopt.

Een test vertelt ons of onze code doet wat wij verwachten.

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 17](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_17/).

---

## Controlelijst

Je bent klaar met dit artikel als je zonder hulp:

- [ ] weet wat een testfunctie is en hoe je die markeert met `#[test]`
- [ ] beweringen kunt doen met `assert_eq!`, `assert_ne!` en `assert!`
- [ ] tests kunt uitvoeren met `cargo test` in de terminal
- [ ] begrijpt hoe je tests structureert met `#[cfg(test)]` en `use super::*;`
- [ ] een falende test kunt analyseren en de bijbehorende fout kunt oplossen.
