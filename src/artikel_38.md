# 38. Eindproject

## Wat gaan we leren?

In dit afsluitende project brengen we alle geleerde concepten uit de leerlijn samen in een volwaardig, zelfontworpen command-line RPG.

We passen toe:

- datastructuren en enums voor spelers, vijanden, locaties en items
- modulaire projectopbouw met encapsulatie en publieke interfaces
- een robuuste spel-lus met gebruikersinvoer en foutafhandeling
- turn-based gevechten en een beloningssysteem
- unit tests voor de kernlogica.

---

## 1. Doel van het eindproject

Breng alle onderdelen uit de voorgaande artikelen samen in een complete, speelbare command-line RPG. 

Werk in kleine, overzichtelijke stappen. Begin met een werkende basis (zoals je in de eerdere artikelen hebt gedaan), test tussentijds en breid stap voor stap uit.

> [!NOTE]
> Een klein, werkend spel dat je volledig begrijpt en zelfstandig kunt aanpassen is een veel beter eindproject dan een overdreven groot project dat vastloopt in compilerfouten.

---

## 2. Minimale eisen

Het eindproject bevat minimaal de volgende onderdelen:

1. **Speler en inventaris**: een speler met naam, gezondheid, goud, actueel wapen en een inventaris.
2. **Spelwereld**: meerdere locaties waartussen de speler kan reizen.
3. **Ontmoetingen en gevechten**: minimaal één vijandtype waarmee de speler turn-based kan vechten.
4. **Spel-lus**: een interactief menu waarin de speler keuzes kan invoeren via de console.
5. **Beloningen**: ervaringspunten (XP), buit of goud na het winnen van een gevecht.
6. **Tests**: automatische unit tests (`cargo test`) die de belangrijkste spelregels en berekeningen valideren.

---

## 3. Aanbevolen stappenplan

1. **Ontwerp**: schets eerst kort welke structs en enums je nodig hebt (`Speler`, `Vijand`, `Locatie`, `Spel`).
2. **Basisopzet**: maak een nieuw project met `cargo new eindproject_rpg` en definieer je modules (`speler.rs`, `vijand.rs`, `wereld.rs`, `gevecht.rs`).
3. **Kernlogica implementeren**: schrijf functies en methods voor bewegen, vechten en inventarisbeheer.
4. **Schrijf tests**: valideer schadeberekeningen en toestandsovergangen met tests.
5. **Bouw de gebruikersinterface**: implementeer de invoer-lus in `main.rs` of `spel.rs`.
6. **Refactoren**: ruim dubbele code op, geef duidelijke namen en controleer of `cargo clippy` en `cargo test` schoon zijn.

---

## Rustlings-oefeningen

Maak daarna de oefeningen uit de [Rustlings-map van Artikel 38](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_38/).

---

## Controlelijst

Je bent klaar met dit artikel en de leerlijn als je:

- [ ] een zelfstandig functionerend command-line RPG hebt gebouwd in Rust
- [ ] alle geschreven tests slagen met `cargo test`
- [ ] je code logisch hebt opgedeeld in modules met duidelijke verantwoordelijkheden
- [ ] trots bent op wat je allemaal hebt geleerd en gebouwd!
