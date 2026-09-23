# Functies

## Wat gaan we leren?

Een functie is een klein stukje code met een naam. We kunnen dat stukje code
opnieuw gebruiken. Daardoor blijft een programma overzichtelijk.

Na dit artikel kun je:

- een functie maken;
- een functie aanroepen;
- parameters meegeven;
- een waarde teruggeven;
- functies gebruiken in een game.

> [!NOTE]
> Begin klein. Een functie hoeft maar één duidelijke taak te hebben.

## 1. Een functie maken

Een functie begint met `fn`, daarna volgt de naam en een paar haakjes.

```rust
fn toon_welkom() {
    println!("Welkom in de game!");
}

fn main() {
    toon_welkom();
}
```

De regel `toon_welkom();` **roept** de functie aan. Een functie wordt pas
uitgevoerd wanneer we haar aanroepen.

## 2. Parameters

Een functie kan informatie ontvangen. Zo'n stukje informatie heet een
parameter. Achter de naam zetten we het type.

```rust
fn toon_levens(levens: i32) {
    println!("Je hebt {levens} levens.");
}

fn main() {
    toon_levens(3);
}
```

De waarde `3` noemen we bij de aanroep een argument. Een functie kan meerdere
parameters hebben:

```rust
fn toon_schade(kracht: i32, wapenschade: i32) {
    println!("Schade: {}", kracht + wapenschade);
}

fn main() {
    toon_schade(10, 7);
}
```

De volgorde en de types moeten kloppen.

## 3. Een waarde teruggeven

Soms wil een functie een antwoord teruggeven. Het type van dat antwoord staat
na `->`.

```rust
fn bereken_schade(kracht: i32, wapenschade: i32) -> i32 {
    kracht + wapenschade
}

fn main() {
    let schade = bereken_schade(10, 7);
    println!("Je doet {schade} schade!");
}
```

De laatste regel van de functie is de teruggegeven waarde. Er mag ook
`return` staan, maar voor korte functies is de laatste regel duidelijker:

```rust
fn verdubbel(getal: i32) -> i32 {
    getal * 2
}
```

## 4. Functies in een game

Een functie kan een regel uit de game beschrijven. Dat maakt de `main`-functie
korter en makkelijker te lezen.

```rust
fn neem_schade(gezondheid: i32, schade: i32) -> i32 {
    gezondheid - schade
}

fn main() {
    let gezondheid = neem_schade(100, 25);
    println!("De speler heeft nog {gezondheid} gezondheid.");
}
```

## Oefeningen

Maak eerst de oefeningen in de Rustlings-repository. De bestanden voor dit
artikel staan in `exercises/artikel_4/`.

1. [Een functie maken](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_4/1_functie.rs)
2. [Een functie aanroepen](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_4/2_aanroepen.rs)
3. [Parameters gebruiken](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_4/3_parameters.rs)
4. [Een resultaat teruggeven](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/blob/main/exercises/artikel_4/4_resultaat.rs)

> [!TIP]
> Pas de URL aan als jouw GitHub-gebruikersnaam of repositorynaam anders is.

Na de laatste oefening kun je teruggaan naar dit artikel.