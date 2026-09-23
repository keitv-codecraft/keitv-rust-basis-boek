# Rust 16 — Closures: acties als waarden

We hebben inmiddels geleerd hoe we een groep vijanden kunnen opslaan:

```rust,ignore
let mut vijanden = vec![
    // vijanden...
];
```

En we kunnen met een `for`-lus iedere vijand bewerken:

```rust,ignore
for vijand in &mut vijanden {
    vijand.gezondheid -= 20;
}
```

Maar stel dat we verschillende aanvallen willen hebben.

Een gewone aanval doet 20 schade.

Een vuurstorm doet 30 schade.

Een ijsstorm doet 15 schade.

Een genezingsspreuk geeft 25 gezondheid.

We zouden voor iedere actie aparte code kunnen schrijven.

Maar er is een interessantere mogelijkheid:

> We kunnen het **gedrag van een actie** bewaren in een closure.

---

## 1. Onze eerste closure

Een closure ziet er bijvoorbeeld zo uit:

```rust,ignore
let dubbel = |x| x * 2;
```

We kunnen hem uitvoeren:

```rust,ignore
let resultaat = dubbel(5);

println!("{resultaat}");
```

De uitkomst is:

```text
10
```

Een closure is dus een klein stukje gedrag dat we in een variabele kunnen bewaren.

---

## 2. Closures met meerdere regels

We kunnen ook meerdere regels gebruiken:

```rust,ignore
let aanval = |schade| {
    println!("De aanval doet {schade} schade.");
};
```

We voeren hem uit met:

```rust,ignore
aanval(20);
```

---

## 3. Een closure kan een waarde teruggeven

Bijvoorbeeld:

```rust,ignore
let bereken_schade = |basis| basis + 10;

let schade = bereken_schade(20);

println!("{schade}");
```

De uitkomst is:

```text
30
```

---

## 4. Onze eerste RPG-aanval

Laten we nu onze `Vijand` gebruiken:

```rust,ignore
struct Vijand {
    naam: String,
    gezondheid: i32,
}
```

We maken een vijand:

```rust,ignore
let mut vijand = Vijand {
    naam: String::from("Goblin"),
    gezondheid: 50,
};
```

En een aanval:

```rust,ignore
let aanval = |vijand: &mut Vijand| {
    vijand.gezondheid -= 20;
};
```

We kunnen de aanval uitvoeren:

```rust,ignore
aanval(&mut vijand);
```

De Goblin heeft daarna nog 30 gezondheid.

---

## 5. Waarom `&mut Vijand`?

De closure moet de gezondheid van de vijand veranderen.

Daarom krijgt hij een mutable reference:

```rust,ignore
&mut Vijand
```

Dit sluit aan op wat we eerder leerden over borrowing.

De closure neemt niet de eigenaar van de vijand over.

Hij krijgt tijdelijk toestemming om de vijand te wijzigen.

---

## 6. De closure op meerdere vijanden gebruiken

Nu wordt het interessant.

We maken een groep:

```rust,ignore
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
```

We maken één aanval:

```rust,ignore
let vuurstorm = |vijand: &mut Vijand| {
    vijand.gezondheid -= 20;
};
```

En gebruiken hem op alle vijanden:

```rust,ignore
for vijand in &mut vijanden {
    vuurstorm(vijand);
}
```

De vuurstorm raakt dus iedereen.

Dat is onze eerste echte **AoE-aanval**.

---

## 7. Meerdere aanvallen

We kunnen meerdere closures maken:

```rust,ignore
let vuurstorm = |vijand: &mut Vijand| {
    vijand.gezondheid -= 30;
};

let ijsstorm = |vijand: &mut Vijand| {
    vijand.gezondheid -= 15;
};
```

We kunnen vervolgens kiezen welke aanval we gebruiken.

Bijvoorbeeld:

```rust,ignore
for vijand in &mut vijanden {
    vuurstorm(vijand);
}
```

Of:

```rust,ignore
for vijand in &mut vijanden {
    ijsstorm(vijand);
}
```

De `for`-lus blijft hetzelfde.

Alleen het gedrag dat we uitvoeren verandert.

---

## 8. Een gewone aanval

We kunnen ook een aanval maken die maar één vijand raakt:

```rust,ignore
let aanval = |vijand: &mut Vijand| {
    vijand.gezondheid -= 20;
};
```

Gebruik:

```rust,ignore
aanval(&mut vijanden[0]);
```

Alleen de eerste vijand krijgt schade.

---

## 9. Een AoE-aanval

Voor een aanval op iedereen gebruiken we:

```rust,ignore
for vijand in &mut vijanden {
    vuurstorm(vijand);
}
```

Het verschil is dus:

**Eén doel:**

```rust,ignore
aanval(&mut vijanden[0]);
```

**Alle doelen:**

```rust,ignore
for vijand in &mut vijanden {
    aanval(vijand);
}
```

Dit is een patroon dat we nog vaak zullen gebruiken.

---

## 10. Een genezingsspreuk

Een closure hoeft natuurlijk geen schade te doen.

```rust,ignore
let genees = |vijand: &mut Vijand| {
    vijand.gezondheid += 25;
};
```

We kunnen deze ook op iedere vijand uitvoeren:

```rust,ignore
for vijand in &mut vijanden {
    genees(vijand);
}
```

We hebben nu dus hetzelfde mechanisme voor totaal verschillende acties.

---

## 11. Acties zijn gedrag

We kunnen nu bijvoorbeeld drie acties maken:

```rust,ignore
let vuurstorm = |vijand: &mut Vijand| {
    vijand.gezondheid -= 30;
};

let ijsstorm = |vijand: &mut Vijand| {
    vijand.gezondheid -= 15;
};

let genees = |vijand: &mut Vijand| {
    vijand.gezondheid += 25;
};
```

Iedere closure heeft hetzelfde soort taak:

```text
Vijand → aanpassen
```

Maar ze doen iets anders.

---

## 12. De bonus uit de omgeving

Closures kunnen ook waarden uit hun omgeving gebruiken.

```rust,ignore
let vuur_bonus = 10;

let vuurstorm = |vijand: &mut Vijand| {
    vijand.gezondheid -= 30 + vuur_bonus;
};
```

De aanval doet nu:

```text
30 + 10 = 40 schade
```

De closure gebruikt dus `vuur_bonus` zonder dat die als parameter wordt meegegeven.

---

## 13. Een aanval met een instelling

We kunnen ook een spelinstelling gebruiken:

```rust,ignore
let moeilijke_modus = true;

let aanval = |vijand: &mut Vijand| {
    if moeilijke_modus {
        vijand.gezondheid -= 30;
    } else {
        vijand.gezondheid -= 20;
    }
};
```

Dezelfde closure gedraagt zich anders afhankelijk van de instelling.

---

## 14. Een closure kan een resultaat teruggeven

Closures hoeven niet altijd direct een vijand te veranderen.

We kunnen ook schade berekenen:

```rust,ignore
let bereken_schade = |basis| basis + 10;

let schade = bereken_schade(20);
```

Daarna gebruiken we die schade:

```rust,ignore
vijand.gezondheid -= schade;
```

We kunnen dus closures gebruiken voor:

- berekeningen
- acties
- spelregels
- effecten.

---

## 15. Closures en `Vec`

Dit is de combinatie die we voor ons spel willen leren.

```rust,ignore
let mut vijanden = vec![
    Vijand {
        naam: String::from("Goblin"),
        gezondheid: 40,
    },
    Vijand {
        naam: String::from("Ork"),
        gezondheid: 60,
    },
    Vijand {
        naam: String::from("Trol"),
        gezondheid: 100,
    },
];

let vuurstorm = |vijand: &mut Vijand| {
    vijand.gezondheid -= 25;
};

for vijand in &mut vijanden {
    vuurstorm(vijand);
}
```

De closure beschrijft **wat er moet gebeuren**.

De `for`-lus bepaalt **op welke vijanden het gebeurt**.

Dat onderscheid is erg handig.

---

## 16. Een volledige ronde

We kunnen nu een kleine gevechtsronde maken:

```rust
struct Vijand {
    naam: String,
    gezondheid: i32,
}

fn main() {
    let mut vijanden = vec![
        Vijand {
            naam: String::from("Goblin"),
            gezondheid: 40,
        },
        Vijand {
            naam: String::from("Ork"),
            gezondheid: 60,
        },
        Vijand {
            naam: String::from("Trol"),
            gezondheid: 100,
        },
    ];

    let vuurstorm = |vijand: &mut Vijand| {
        vijand.gezondheid -= 25;
    };

    println!("Vuurstorm!");

    for vijand in &mut vijanden {
        vuurstorm(vijand);
    }

    for vijand in &vijanden {
        println!(
            "{} heeft {} gezondheid.",
            vijand.naam,
            vijand.gezondheid
        );
    }
}
```

Mogelijke uitvoer:

```text
Vuurstorm!
Goblin heeft 15 gezondheid.
Ork heeft 35 gezondheid.
Trol heeft 75 gezondheid.
```

Dit is al een behoorlijk bruikbaar stukje van ons RPG-systeem.

---

## 17. Eén doel of alle doelen

We kunnen verschillende soorten spreuken maken.

### Eén doel

```rust,ignore
let vuurbal = |vijand: &mut Vijand| {
    vijand.gezondheid -= 40;
};

vuurbal(&mut vijanden[0]);
```

### Alle doelen

```rust,ignore
let vuurstorm = |vijand: &mut Vijand| {
    vijand.gezondheid -= 20;
};

for vijand in &mut vijanden {
    vuurstorm(vijand);
}
```

De closure zelf hoeft niet te weten of hij één of honderd vijanden gaat raken.

Dat bepaalt de code die de closure aanroept.

---

## 18. Oefeningen

## Oefening 1 — Eerste actie

Maak een closure:

```text
aanval
```

die een `Vijand` 10 schade geeft.

Voer hem uit op één vijand.

---

## Oefening 2 — AoE

Maak een `Vec<Vijand>` met drie vijanden.

Gebruik een closure om alle vijanden 15 schade te geven.

---

## Oefening 3 — Vuurstorm

Maak een closure die 30 schade geeft.

Gebruik hem op alle vijanden.

---

## Oefening 4 — IJsstorm

Maak een tweede closure die 20 schade geeft.

Gebruik beide aanvallen na elkaar.

---

## Oefening 5 — Genezing

Maak een closure:

```text
genees
```

die 25 gezondheid toevoegt.

Gebruik hem op alle vijanden.

---

## Oefening 6 — Bonus

Maak:

```rust,ignore
let vuur_bonus = 10;
```

Laat je vuurstorm deze bonus gebruiken.

---

## Oefening 7 — Kritieke aanval

Maak een closure die de gegeven basisschade verdubbelt.

Gebruik bijvoorbeeld:

```text
20 → 40
```

---

## Oefening 8 — Verschillende doelen

Maak drie vijanden.

Gebruik:

- een normale aanval op de eerste
- een zware aanval op de tweede
- een AoE-aanval op alle drie.

Print daarna hun gezondheid.

---

## 19. Rustlings

Maak:

```text
exercises/closures/
```

met:

```text
01_first_closure.rs
02_two_parameters.rs
03_no_parameters.rs
04_damage.rs
05_healing.rs
06_enemy.rs
07_single_target.rs
08_aoe.rs
09_firestorm.rs
10_ice_storm.rs
11_bonus.rs
12_critical.rs
13_game_mode.rs
14_game_actions.rs
15_debug_return.rs
16_debug_mut.rs
17_debug_borrow.rs
18_game_battle.rs
```

De opdrachten moeten geleidelijk moeilijker worden.

Bijvoorbeeld `15_debug_return.rs`:

```rust,ignore
fn main() {
    let bereken_schade = |basis| {
        basis + 10;
    };

    let schade = bereken_schade(20);

    println!("{schade}");
}
```

De compiler vertelt dat hier iets niet klopt met de waarde die de closure teruggeeft.

---

## 20. Eindopdracht — Het gevecht

Maak een volledig klein gevecht.

Gebruik:

```rust,ignore
struct Vijand {
    naam: String,
    gezondheid: i32,
}
```

Maak minimaal vijf vijanden.

Maak vervolgens minimaal drie closures:

```text
vuurbal
vuurstorm
genees
```

De regels:

```text
vuurbal       → 40 schade aan één vijand
vuurstorm     → 20 schade aan alle vijanden
genees        → 25 gezondheid aan één vijand
```

Voer vervolgens een aantal acties uit.

Bijvoorbeeld:

```text
Vuurbal op Goblin!
Vuurstorm!
Genees de Trol!
```

Print na iedere actie de gezondheid van alle vijanden.

---

## 21. Wat moet je onthouden?

Een closure is een klein stukje gedrag dat we kunnen bewaren in een variabele.

Bijvoorbeeld:

```rust,ignore
let aanval = |vijand: &mut Vijand| {
    vijand.gezondheid -= 20;
};
```

We kunnen hem uitvoeren:

```rust,ignore
aanval(&mut vijand);
```

En combineren met een `Vec`:

```rust,ignore
for vijand in &mut vijanden {
    aanval(vijand);
}
```

Daarmee kunnen we heel gemakkelijk acties maken die op een groep objecten werken.

Het belangrijkste idee:

> **Een closure bewaart gedrag. Een `Vec` bewaart een groep waarden. Een `for`-lus kan dat gedrag op ieder element toepassen.**

We hebben hiermee een krachtige combinatie gekregen:

```text
Vec + for + borrowing + struct + closure
```

Maar we hoeven nog niet te weten hoe Rust dit intern technisch implementeert.

Voorlopig is het genoeg om zelf closures te kunnen maken, aanroepen en gebruiken met onze spelobjecten.

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 16](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_16/).

