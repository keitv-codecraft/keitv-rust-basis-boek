# 37. Refactoring

## Wat gaan we leren?

In dit artikel leren we hoe we code stapsgewijs kunnen verbeteren (refactoren) zonder het gedrag van het programma te veranderen.

We leren:

- wat refactoring is en wanneer (en wanneer vooral niet) je het toepast
- functies opsplitsen volgens het principe: één functie, één duidelijke taak
- heldere, betekenisvolle namen kiezen voor variabelen, functies en types
- code-duplicatie elimineren
- stapsgewijs werken met behulp van compilercontroles en tests als vangnet.

---

## 1. Wat is refactoring?

Stel dat we deze functie hebben:

```rust,ignore
fn speler_status(speler: &Speler) {
    println!("Naam: {}", speler.naam());
    println!("Gezondheid: {}", speler.gezondheid());
    println!("Goud: {}", speler.goud());

    if speler.gezondheid() > 0 {
        println!("De speler leeft.");
    } else {
        println!("De speler is dood.");
    }
}
```

De functie werkt prima.

Maar misschien willen we de status op verschillende plaatsen in het spel tonen. Dan kunnen we bijvoorbeeld een aparte functie maken voor het bepalen van de tekst:

```rust,ignore
fn levens_status(speler: &Speler) -> &str {
    if speler.gezondheid() > 0 {
        "De speler leeft."
    } else {
        "De speler is dood."
    }
}
```

Daarna:

```rust,ignore
fn speler_status(speler: &Speler) {
    println!("Naam: {}", speler.naam());
    println!("Gezondheid: {}", speler.gezondheid());
    println!("Goud: {}", speler.goud());
    println!("{}", levens_status(speler));
}
```

Het programma doet nog steeds hetzelfde.

We hebben alleen de structuur verbeterd.

Dat is refactoring.

---

## 2. Refactoring is geen nieuwe functionaliteit

Het is belangrijk om twee soorten veranderingen uit elkaar te houden.

### Functionaliteit toevoegen

Bijvoorbeeld:

> Voeg een ervaringspuntensysteem toe.

Dat verandert wat het programma kan.

### Refactoring

Bijvoorbeeld:

> Verplaats de berekening van schade naar `gevecht.rs`.

Dat verandert vooral waar en hoe de bestaande code georganiseerd is.

Tijdens refactoring proberen we het gedrag gelijk te houden.

Dat geeft ons een belangrijk doel:

> Na de refactoring moet het programma nog steeds hetzelfde doen.

---

## 3. Waarom zouden we refactoren?

Een programma kan prima werken terwijl de code steeds moeilijker te begrijpen wordt.

Bijvoorbeeld:

```rust,ignore
fn speel_spel(speler: &mut Speler) {
    println!("Welkom!");

    println!("Naam: {}", speler.naam());
    println!("Gezondheid: {}", speler.gezondheid());

    println!("1. Verkennen");
    println!("2. Inventaris");
    println!("3. Stoppen");

    let keuze = lees_getal();

    if keuze == 1 {
        println!("Je gaat op pad.");

        if speler.gezondheid() > 0 {
            println!("Je bent gezond genoeg om te reizen.");

            let mut vijand = Vijand::nieuw("Goblin", 30, 8);

            loop {
                if !speler.is_levend() {
                    println!("Je bent verslagen.");
                    break;
                }

                if !vijand.is_levend() {
                    println!("De vijand is verslagen.");
                    break;
                }

                speler.val_aan(&mut vijand);

                if vijand.is_levend() {
                    vijand.val_aan(speler);
                }
            }
        }
    } else if keuze == 2 {
        speler.toon_inventaris();
    } else if keuze == 3 {
        println!("Tot ziens!");
    }
}
```

Dit kan werken.

Maar er zitten verschillende verantwoordelijkheden in één functie:

- menu tonen
- invoer lezen
- reizen
- vijand maken
- gevecht uitvoeren
- inventaris tonen
- controleren of de speler leeft.

Dat maakt wijzigingen steeds lastiger.

---

## 4. Eén functie, één duidelijke taak

Een nuttige vuistregel is:

> Een functie moet een duidelijke verantwoordelijkheid hebben.

Dat betekent niet dat iedere functie precies één regel mag bevatten.

Het betekent dat je de functie gemakkelijk kunt omschrijven.

Bijvoorbeeld:

```rust,ignore
fn start_gevecht(speler: &mut Speler, vijand: &mut Vijand) {
    loop {
        if !speler.is_levend() {
            break;
        }

        if !vijand.is_levend() {
            break;
        }

        speler.val_aan(vijand);

        if vijand.is_levend() {
            vijand.val_aan(speler);
        }
    }
}
```

Deze functie heeft een duidelijke taak:

> Een gevecht uitvoeren.

De details van het gevecht hoeven niet meer in `speel_spel()` te staan.

---

## 5. Een grote functie opsplitsen

Dit is een van de meest voorkomende refactorings.

We beginnen bijvoorbeeld met:

```rust,ignore
fn speel_spel(speler: &mut Speler) {
    println!("Welkom!");

    println!("1. Verkennen");
    println!("2. Inventaris");
    println!("3. Stoppen");

    let keuze = lees_getal();

    if keuze == 1 {
        println!("Je gaat verkennen.");

        let mut vijand = Vijand::nieuw("Goblin", 30, 8);

        loop {
            if !speler.is_levend() {
                break;
            }

            if !vijand.is_levend() {
                break;
            }

            speler.val_aan(&mut vijand);

            if vijand.is_levend() {
                vijand.val_aan(speler);
            }
        }
    }

    if keuze == 2 {
        speler.toon_inventaris();
    }

    if keuze == 3 {
        println!("Tot ziens!");
    }
}
```

We kunnen eerst alleen het gevecht eruit halen:

```rust,ignore
fn start_gevecht(speler: &mut Speler, vijand: &mut Vijand) {
    loop {
        if !speler.is_levend() {
            break;
        }

        if !vijand.is_levend() {
            break;
        }

        speler.val_aan(vijand);

        if vijand.is_levend() {
            vijand.val_aan(speler);
        }
    }
}
```

Daarna wordt `speel_spel()`:

```rust,ignore
fn speel_spel(speler: &mut Speler) {
    println!("Welkom!");

    println!("1. Verkennen");
    println!("2. Inventaris");
    println!("3. Stoppen");

    let keuze = lees_getal();

    if keuze == 1 {
        println!("Je gaat verkennen.");

        let mut vijand = Vijand::nieuw("Goblin", 30, 8);

        start_gevecht(speler, &mut vijand);
    }

    if keuze == 2 {
        speler.toon_inventaris();
    }

    if keuze == 3 {
        println!("Tot ziens!");
    }
}
```

De functie is nu korter én duidelijker.

---

## 6. Waarom werkt dit beter?

We hebben nu twee niveaus.

Op het hogere niveau:

```rust,ignore
start_gevecht(speler, &mut vijand);
```

Op het lagere niveau:

```rust,ignore
fn start_gevecht(...) {
    ...
}
```

Dat betekent dat `speel_spel()` niet hoeft te weten hoe een gevecht precies werkt.

Dat is een belangrijk idee:

> Code op een hoger niveau moet kunnen zeggen wat er gebeurt zonder alle details zelf te bevatten.

Bijvoorbeeld:

```rust,ignore
start_gevecht(...)
```

zegt:

> Start een gevecht.

Je hoeft in `speel_spel()` niet te lezen:

```rust,ignore
if !speler.is_levend() {
    ...
}

if !vijand.is_levend() {
    ...
}
```

Dat hoort bij het gevecht.

---

## 7. Goede namen zijn onderdeel van de structuur

Bekijk deze code:

```rust,ignore
fn doe_ding(speler: &mut Speler, vijand: &mut Vijand) {
    // ...
}
```

De functie kan technisch correct zijn.

Maar wat doet `doe_ding`?

Dat weten we niet.

Een naam als:

```rust,ignore
fn start_gevecht(speler: &mut Speler, vijand: &mut Vijand)
```

vertelt direct wat de functie doet.

Andere voorbeelden:

```rust,ignore
fn verwerk(speler: &mut Speler)
```

kan bijvoorbeeld worden:

```rust,ignore
fn gebruik_potion(speler: &mut Speler)
```

En:

```rust,ignore
fn doe_schade(speler: &mut Speler, vijand: &mut Vijand)
```

kan misschien worden:

```rust,ignore
fn voer_aanval_uit(speler: &mut Speler, vijand: &mut Vijand)
```

Een goede naam kan daardoor een commentaar overbodig maken.

---

## 8. Een slechte naam is vaak een probleem voor later

Stel dat we dit schrijven:

```rust,ignore
fn verwerk(speler: &mut Speler) {
    speler.genees(20);
}
```

Later gebruiken we deze functie op vijf plaatsen.

Dan lijkt alles prima.

Maar later veranderen we de functie:

```rust,ignore
fn verwerk(speler: &mut Speler) {
    speler.genees(20);

    speler.geef_goud(10);
}
```

De naam `verwerk` zegt nog steeds niet wat er gebeurt.

Een betere naam maakt de bedoeling duidelijk:

```rust,ignore
fn beloon_speler(speler: &mut Speler) {
    speler.genees(20);
    speler.geef_goud(10);
}
```

Daarom is hernoemen zelf ook een vorm van refactoring.

---

## 9. Code duplicatie

Een ander belangrijk probleem is dubbele code.

Bijvoorbeeld:

```rust,ignore
if speler.is_levend() {
    println!("Je kunt verder.");
}

if vijand.is_levend() {
    println!("De vijand kan verder.");
}
```

Dit is nog niet noodzakelijk een probleem.

Maar stel dat we op verschillende plaatsen hetzelfde doen:

```rust,ignore
if speler.is_levend() {
    println!("Je kunt verder.");
}
```

en later:

```rust,ignore
if speler.is_levend() {
    println!("Je kunt verder.");
}
```

en nog ergens:

```rust,ignore
if speler.is_levend() {
    println!("Je kunt verder.");
}
```

Dan hebben we drie kopieën.

Als de tekst later moet veranderen, moeten we drie plaatsen aanpassen.

---

## 10. Duplicatie verwijderen

We kunnen bijvoorbeeld een functie maken:

```rust,ignore
fn toon_levens_status(speler: &Speler) {
    if speler.is_levend() {
        println!("Je kunt verder.");
    } else {
        println!("Je bent verslagen.");
    }
}
```

Nu gebruiken we:

```rust,ignore
toon_levens_status(speler);
```

op iedere plek waar we deze informatie willen tonen.

Als de tekst verandert, hoeven we maar één functie aan te passen.

---

## 11. Maar niet iedere herhaling moet verdwijnen

Dit is belangrijk.

Niet iedere twee vergelijkbare stukken code hoeven tot één functie te worden samengevoegd.

Bijvoorbeeld:

```rust,ignore
println!("De speler heeft gewonnen.");
```

en:

```rust,ignore
println!("De vijand is verslagen.");
```

lijken op elkaar, maar hebben een verschillende betekenis.

Te veel abstraheren kan de code juist moeilijker maken.

Gebruik daarom niet de regel:

> Alles wat twee keer voorkomt moet een functie worden.

Gebruik liever:

> Als dezelfde logica meerdere keren voorkomt en dezelfde verantwoordelijkheid heeft, onderzoek dan of die logica op één plaats hoort.

---

## 12. Parameters kunnen ook een probleem worden

Soms groeit een functie langzaam.

Eerst:

```rust,ignore
fn start_gevecht(speler: &mut Speler, vijand: &mut Vijand)
```

Later:

```rust,ignore
fn start_gevecht(
    speler: &mut Speler,
    vijand: &mut Vijand,
    moeilijkheid: i32,
    kritisch_percentage: i32,
    maximale_rondes: i32,
    genezing: i32,
)
```

Technisch kan dit werken.

Maar de functie begint veel informatie nodig te hebben.

Dat kan een signaal zijn dat verantwoordelijkheden verkeerd verdeeld zijn.

Bijvoorbeeld:

```rust,ignore
struct Gevechtsinstellingen {
    moeilijkheid: i32,
    kritisch_percentage: i32,
    maximale_rondes: i32,
    genezing: i32,
}
```

Dan kunnen we schrijven:

```rust,ignore
fn start_gevecht(
    speler: &mut Speler,
    vijand: &mut Vijand,
    instellingen: &Gevechtsinstellingen,
) {
    // ...
}
```

Dit is een voorbeeld van een refactoring die we later kunnen uitvoeren.

Het belangrijkste is niet dat je meteen de perfecte structuur bedenkt.

Het belangrijkste is dat je leert herkennen wanneer een functie moeilijk te begrijpen begint te worden.

---

## 13. Refactoren met tests

Hier wordt ons eerdere werk met automatische tests erg belangrijk.

Stel dat we deze functie hebben:

```rust,ignore
fn bereken_schade(aanval: i32, verdediging: i32) -> i32 {
    if aanval > verdediging {
        aanval - verdediging
    } else {
        1
    }
}
```

We kunnen tests schrijven:

```rust,ignore
#[test]
fn hoge_aanval_doet_schade() {
    assert_eq!(bereken_schade(10, 3), 7);
}

#[test]
fn lage_aanval_doet_minimaal_een_schade() {
    assert_eq!(bereken_schade(3, 10), 1);
}
```

Nu kunnen we de functie veranderen zonder telkens alles handmatig te controleren.

Bijvoorbeeld:

```rust,ignore
fn bereken_schade(aanval: i32, verdediging: i32) -> i32 {
    let schade = aanval - verdediging;

    if schade < 1 {
        1
    } else {
        schade
    }
}
```

Het gedrag is hetzelfde.

We voeren uit:

```text
cargo test
```

en controleren dat de tests nog steeds slagen.

---

## 14. De compiler is ook een hulpmiddel bij refactoring

Stel dat we:

```rust,ignore
fn start_gevecht(speler: &mut Speler, vijand: &mut Vijand)
```

verplaatsen naar `gevecht.rs`.

Daarna geeft Rust misschien fouten zoals:

```text
cannot find function `start_gevecht` in this scope
```

Dat betekent niet automatisch dat onze refactoring mislukt is.

We hebben de functie verplaatst, maar de code die haar gebruikt moet nog worden aangepast.

Bijvoorbeeld:

```rust,ignore
crate::gevecht::start_gevecht(speler, vijand);
```

of met:

```rust,ignore
use crate::gevecht::start_gevecht;
```

De compiler vertelt ons dus waar de oude structuur nog wordt gebruikt.

Dit is één van de redenen waarom kleine refactorings prettig zijn.

Je kunt de fouten één voor één oplossen.

---

## 15. Verplaatsen naar een module

Stel dat `main.rs` steeds meer gevechtscode bevat.

We hebben:

```text
src/
├── main.rs
└── gevecht.rs
```

In `gevecht.rs`:

```rust,ignore
pub fn start_gevecht(speler: &mut Speler, vijand: &mut Vijand) {
    // ...
}
```

En in `main.rs`:

```rust,ignore
mod gevecht;

use gevecht::start_gevecht;
```

Daarna:

```rust,ignore
start_gevecht(&mut speler, &mut vijand);
```

De functionaliteit is niet veranderd.

Alleen de plaats van de code is veranderd.

Dit is een echte refactoring:

> dezelfde verantwoordelijkheid, duidelijker georganiseerd.

---

## 16. Maak niet alles `pub`

Een veelgemaakte fout bij het reorganiseren van een project is:

```rust,ignore
pub struct Speler {
    pub naam: String,
    pub gezondheid: i32,
    pub goud: i32,
}
```

Daarmee kan iedere andere module alles rechtstreeks aanpassen.

Bijvoorbeeld:

```rust,ignore
speler.gezondheid = -500;
speler.goud = -10000;
```

Dat willen we meestal niet.

Beter:

```rust,ignore
pub struct Speler {
    naam: String,
    gezondheid: i32,
    goud: i32,
}
```

en gecontroleerde methoden:

```rust,ignore
impl Speler {
    pub fn gezondheid(&self) -> i32 {
        self.gezondheid
    }

    pub fn geef_goud(&mut self, hoeveelheid: i32) {
        self.goud += hoeveelheid;
    }
}
```

De buitenwereld kan de informatie gebruiken via de interface, maar niet zomaar de interne toestand veranderen.

---

## 17. Refactoring kan daardoor leiden tot betere interfaces

Stel dat we eerst overal dit hebben:

```rust,ignore
speler.gezondheid -= 20;
```

Na refactoring maken we:

```rust,ignore
pub fn neem_schade(&mut self, schade: i32) {
    self.gezondheid -= schade;

    if self.gezondheid < 0 {
        self.gezondheid = 0;
    }
}
```

Daarna wordt de rest van het programma:

```rust,ignore
speler.neem_schade(20);
```

Dit is niet alleen mooier.

Het betekent ook dat de regel:

> gezondheid mag nooit onder nul komen

op één plaats staat.

Als we die regel later veranderen, hoeven we niet door het hele project te zoeken.

---

## 18. Een praktische refactoringstrategie

Gebruik bij grotere veranderingen deze volgorde.

### Stap 1 — Zorg dat de code werkt

Begin niet met refactoren als je niet weet wat de huidige code doet.

Voer bijvoorbeeld uit:

```text
cargo check
cargo test
cargo run
```

Noteer eventueel wat het programma hoort te doen.

---

### Stap 2 — Zoek een klein probleem

Bijvoorbeeld:

- een functie is te groot
- een naam is onduidelijk
- dezelfde code staat meerdere keren
- een module heeft te veel verantwoordelijkheden
- velden zijn onnodig publiek
- een functie staat op de verkeerde plaats.

Kies één probleem.

---

### Stap 3 — Maak één verandering

Niet:

> Ik ga meteen het hele project opnieuw ontwerpen.

Wel:

> Ik haal alleen het gevecht uit `main.rs`.

---

### Stap 4 — Compileer

```text
cargo check
```

Los eventuele compilerfouten op.

---

### Stap 5 — Test

```text
cargo test
```

Als een test faalt, onderzoek waarom.

---

### Stap 6 — Controleer het programma

```text
cargo run
```

Controleer vooral het gedrag dat door de refactoring geraakt kan zijn.

---

### Stap 7 — Pas daarna verder

Nu pas ga je naar de volgende verbetering.

---

## 19. Een complete kleine refactoring

We nemen een vereenvoudigd voorbeeld.

### Voor

```rust,ignore
fn speel_spel(speler: &mut Speler) {
    println!("Welkom bij het avontuur!");

    println!("Naam: {}", speler.naam());
    println!("Gezondheid: {}", speler.gezondheid());

    println!("Je ontmoet een goblin.");

    let mut vijand = Vijand::nieuw("Goblin", 30, 8);

    loop {
        if !speler.is_levend() {
            println!("Je bent verslagen.");
            break;
        }

        if !vijand.is_levend() {
            println!("Je hebt gewonnen.");
            break;
        }

        speler.val_aan(&mut vijand);

        if vijand.is_levend() {
            vijand.val_aan(speler);
        }
    }
}
```

Dit werkt.

Maar `speel_spel()` doet drie dingen:

1. spelerstatus tonen
2. ontmoeting starten
3. gevecht uitvoeren.

---

### Stap 1: status eruit

```rust,ignore
fn toon_speler_status(speler: &Speler) {
    println!("Naam: {}", speler.naam());
    println!("Gezondheid: {}", speler.gezondheid());
}
```

Dan:

```rust,ignore
fn speel_spel(speler: &mut Speler) {
    println!("Welkom bij het avontuur!");

    toon_speler_status(speler);

    println!("Je ontmoet een goblin.");

    let mut vijand = Vijand::nieuw("Goblin", 30, 8);

    loop {
        if !speler.is_levend() {
            println!("Je bent verslagen.");
            break;
        }

        if !vijand.is_levend() {
            println!("Je hebt gewonnen.");
            break;
        }

        speler.val_aan(&mut vijand);

        if vijand.is_levend() {
            vijand.val_aan(speler);
        }
    }
}
```

Compileer en test.

---

### Stap 2: gevecht eruit

```rust,ignore
fn start_gevecht(speler: &mut Speler, vijand: &mut Vijand) {
    loop {
        if !speler.is_levend() {
            println!("Je bent verslagen.");
            break;
        }

        if !vijand.is_levend() {
            println!("Je hebt gewonnen.");
            break;
        }

        speler.val_aan(vijand);

        if vijand.is_levend() {
            vijand.val_aan(speler);
        }
    }
}
```

Nu:

```rust,ignore
fn speel_spel(speler: &mut Speler) {
    println!("Welkom bij het avontuur!");

    toon_speler_status(speler);

    println!("Je ontmoet een goblin.");

    let mut vijand = Vijand::nieuw("Goblin", 30, 8);

    start_gevecht(speler, &mut vijand);
}
```

Opnieuw:

```text
cargo check
cargo test
cargo run
```

Het gedrag moet hetzelfde zijn.

---

## 20. De belangrijke verandering

Kijk naar de uiteindelijke functie:

```rust,ignore
fn speel_spel(speler: &mut Speler) {
    println!("Welkom bij het avontuur!");

    toon_speler_status(speler);

    println!("Je ontmoet een goblin.");

    let mut vijand = Vijand::nieuw("Goblin", 30, 8);

    start_gevecht(speler, &mut vijand);
}
```

Je kunt deze code bijna als een verhaal lezen:

1. Welkom tonen.
2. Status tonen.
3. Vijand ontmoeten.
4. Gevecht starten.

Dat is een belangrijk doel van goede structuur.

De details zijn niet verdwenen.

Ze zijn alleen naar de juiste plaats verhuisd.

---

## 21. Refactoring verandert soms alleen de vorm

Dit:

```rust,ignore
if speler.gezondheid() > 0 {
    println!("Levend");
} else {
    println!("Dood");
}
```

kan bijvoorbeeld worden:

```rust,ignore
toon_levens_status(speler);
```

Dat lijkt misschien alsof we minder informatie hebben.

Maar eigenlijk hebben we informatie op verschillende niveaus.

Op hoog niveau:

```rust,ignore
toon_levens_status(speler);
```

Op laag niveau:

```rust,ignore
fn toon_levens_status(speler: &Speler) {
    if speler.gezondheid() > 0 {
        println!("Levend");
    } else {
        println!("Dood");
    }
}
```

Beide zijn nuttig.

Je hebt alleen niet altijd beide tegelijkertijd nodig.

---

## 22. Wanneer moet je stoppen met refactoren?

Ook dit is belangrijk.

Je kunt eindeloos blijven verbeteren.

Een project hoeft niet perfect georganiseerd te zijn.

Een goede refactoring heeft meestal een concreet doel.

Bijvoorbeeld:

> `main.rs` is te groot.

Dan refactor je totdat `main.rs` overzichtelijk is.

Niet:

> Ik ga alle code herschrijven totdat iedere functie exact vijf regels heeft.

Dat is geen nuttig doel.

Goede code is niet de code met de meeste functies of modules.

Goede code is code waarvan de structuur duidelijk helpt om het programma te begrijpen en te veranderen.

---

## 23. Zelfstandige opdrachten

Naast Rustlings zijn er een aantal grotere opdrachten.

### Opdracht 1 — Maak `main` kleiner

Neem jullie bestaande RPG-project.

Onderzoek `main.rs`.

Schrijf op welke verantwoordelijkheden daar nog allemaal in zitten.

Verplaats vervolgens verantwoordelijkheden naar geschikte functies of modules.

Het eindresultaat hoeft niet minimaal te zijn.

Het moet vooral duidelijk zijn.

---

### Opdracht 2 — Zoek duplicatie

Zoek minstens drie voorbeelden van dubbele of bijna dubbele logica.

Bespreek per voorbeeld:

- Is het werkelijk dezelfde verantwoordelijkheid?
- Kan de logica op één plaats staan?
- Zou een functie de code duidelijker maken?
- Wordt de code hierdoor daadwerkelijk makkelijker te onderhouden?

Verwijder daarna minstens één duplicatie.

---

### Opdracht 3 — Verbeter namen

Zoek minstens vijf namen die weinig vertellen.

Bijvoorbeeld:

```rust,ignore
doe_ding
verwerk
x
data
temp
```

Geef ze duidelijkere namen.

Let op:

> Een lange naam is niet automatisch een goede naam.

Bijvoorbeeld:

```rust,ignore
speler_die_op_dit_moment_aan_het_spelen_is
```

is niet noodzakelijk beter dan:

```rust,ignore
speler
```

De naam moet duidelijk zijn binnen de context.

---

### Opdracht 4 — Refactor zonder gedrag te veranderen

Kies één bestaand onderdeel van het RPG-project.

Maak de structuur duidelijker.

Voeg eventueel tests toe voordat je begint.

Daarna:

```text
cargo check
cargo test
cargo run
```

Controleer of het gedrag hetzelfde gebleven is.

Schrijf kort op:

1. Wat was er moeilijk aan de oorspronkelijke code?
2. Wat heb je veranderd?
3. Waarom is de nieuwe structuur duidelijker?
4. Hoe wist je dat het gedrag niet veranderd was?

---

## 24. De belangrijkste les

Tot nu toe hebben we vooral geleerd hoe we nieuwe dingen kunnen bouwen.

Nu leren we iets anders:

> Goede programmeurs schrijven niet alleen nieuwe code. Ze verbeteren ook bestaande code.

Daarbij hoef je niet meteen alles opnieuw te ontwerpen.

Integendeel.

Een veilige aanpak is:

```text
Werkende code
     ↓
Tests
     ↓
Kleine verandering
     ↓
cargo check
     ↓
cargo test
     ↓
Gedrag controleren
     ↓
Volgende kleine verandering
```

Dat proces geeft je controle.

Een compilerfout betekent niet dat je hebt gefaald.

Een falende test betekent niet dat je project mislukt is.

Tijdens refactoring zijn ze juist nuttige signalen:

- de compiler vertelt waar de structuur nog niet klopt
- tests vertellen of het gedrag veranderd is
- je eigen beoordeling vertelt of de nieuwe structuur duidelijker is.

---

## 25. Eindopdracht — Refactor het RPG-project

Neem jullie huidige RPG-project.

Voer de volgende stappen uit.

### Stap 1

Zorg dat het project werkt:

```text
cargo check
cargo test
cargo run
```

### Stap 2

Maak een lijst van de belangrijkste verantwoordelijkheden.

Bijvoorbeeld:

```text
Speler
Vijand
Gevecht
Inventaris
Wapen
Wereld
Winkel
Savegame
Menu
```

### Stap 3

Controleer of deze verantwoordelijkheden logisch verdeeld zijn over functies en modules.

### Stap 4

Maak minstens één te grote functie kleiner.

### Stap 5

Verbeter minstens vijf onduidelijke namen.

### Stap 6

Verwijder minstens twee stukken echte duplicatie.

### Stap 7

Controleer of interne gegevens niet onnodig `pub` zijn.

### Stap 8

Voer na iedere belangrijke verandering uit:

```text
cargo check
cargo test
```

### Stap 9

Voer uiteindelijk het hele programma uit:

```text
cargo run
```

### Stap 10

Maak een korte lijst:

```text
Voor de refactoring:
- ...

Na de refactoring:
- ...

Belangrijkste verbetering:
- ...
```

---

## 26. Samenvatting

In dit artikel hebben we weinig nieuwe Rust-syntax geleerd. Dat is bewust.

We hebben bestaande kennis gebruikt om betere code te maken.

We hebben geleerd:

- wat refactoring is
- dat refactoring gedrag niet bewust hoort te veranderen
- waarom grote functies moeilijk te onderhouden zijn
- hoe je functies opsplitst
- hoe goede namen helpen
- hoe je duplicatie herkent
- waarom niet iedere herhaling moet worden samengevoegd
- hoe modules bij refactoring kunnen helpen
- waarom niet alles `pub` moet zijn
- hoe tests refactoring veiliger maken
- hoe compilerfouten nuttige feedback geven
- hoe je stap voor stap kunt refactoren
- wanneer je moet stoppen met refactoren.

De belangrijkste vaardigheid is uiteindelijk niet:

> "Ik kan een programma schrijven."

Maar:

> **"Ik kan bestaande code begrijpen, voorzichtig veranderen en controleren of mijn verandering correct is."**

Dat is de stap van losse oefeningen naar daadwerkelijk software ontwikkelen.

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 37](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_37/).

---

## Controlelijst

Je bent klaar met dit artikel als je zonder hulp:

- [ ] een complexe functie kunt opsplitsen in kleinere, gerichte hulpfuncties
- [ ] onduidelijke namen kunt vervangen door expressieve identificatienamen
- [ ] code kunt herstructureren in kleine stappen en na elke stap kunt compileren en testen
- [ ] weet wanneer verdere refactoring overbodig wordt en wanneer code ''goed genoeg'' is.
