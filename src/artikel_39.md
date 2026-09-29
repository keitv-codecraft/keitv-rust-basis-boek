# Rust installeren

We gaan hier leren programmeren in Rust. Je hoeft hiervoor nog niets van programmeren te weten.
Wel heb je wat programma's op je computer nodig om aan de slag te kunnen. We gaan er vanuit dat je Windows gebruikt.

## Visual Studio Build Tools 2026

Microsoft maakt software beschikbaar op Windows om programma's te maken. We hebben dit ook nodig om Rust programma's te maken.

* Download en installeer de build tools vanaf [hier](https://aka.ms/vs/stable/vs_BuildTools.exe).
* Vink tijdens de installatie alleen de optie **Desktop development with C++** aan.

## Rustup

Omdat we de programmeertaal Rust gaan gebruiken hebben we ook software nodig om onze code te begrijpen. We gebruiken hiervoor de officiële tools. Rustup is een programma om verschillende versie van die tools te beheren. Op dit moment hebben we alleen de standaard tools nodig.

* Download en installeer Rustup vanaf [hier](https://win.rustup.rs/x86_64).
* Je hoeft de instellingen tijdens de installatie niet te veranderen.

## Rustlings

Aan het einde van ieder artikel vind je oefeningen. Om deze te maken gebruiken we een programma dat rustlings heet. We kunnen dit eenvoudig installeren met de tools die we nu hebben.

* Open Windows Powershell. Bijvoorbeeld via het start menu of door gelijktijdig op de **Windows toets** en **X** te drukken en voor Windows Powershell te kiezen.
* Typ nu het volgende commando om rustlings te installeren

```bash
cargo install rustlings
```

Je kunt nu de rustlings opgaven downloaden die bij deze artikelen horen.

* Download het ZIP-bestand vanaf [hier](https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/archive/refs/heads/master.zip)
* Pak het uit naar een map van je keuze. Bijvoorbeeld, `C:\projects\keitv-rust-basis-rustlings`
* Navigeer naar deze map met Powershell

```bash
cd C:\projects\keitv-rustlings
```

* Voer nu rustlings uit om met de oefeningen te beginnen.

```bash
rustlings
```

Iedere keer als je aan het einde van een artikel gekomen bent kun je rustlings vanuit deze map uitvoeren om verder te gaan waar je gebleven bent. Rustlings onthoud je voortgang.

## VsCodium

Om het makkelijker te maken om Rust code te lezen en te schrijven is het fijn om gebruik te maken van een slimme text editor. Als je hiervoor nog geen voorkeur hebt is VsCodium een veilige keuze. Je kunt deze vanaf [hier](https://github.com/VSCodium/vscodium/releases/download/1.135.06055/VSCodium-x64-1.135.06055.msi) downloaden en installeren.

VsCodium ondersteunt veel verschillende technologiën door gebruik te maken van extensies. Nadat je VsCodium hebt geïinstalleerd en geopend klik je aan de linkerkant op de vier bouwsteentjes om extensies te installeren. Voor Rust is het handig om de volgende extensies te installeren.

* rust-analyzer (geeft kleur aan de Rust code en laat hints zien waar dit handig is)
* CodeLLDB (analyseer je programma terwijl het draait)
* Dependi (geeft hints over afhankelijkheden naar Rust code van andere mensen)
* Even Better TOML (geeft kleur en hints aan configuratie bestanden)

## Klaar voor de start

Je bent nu helemaal klaar om zelf aan de slag te gaan met Rust op jouw computer.
Ga terug naar het vorige artikel om te beginnen met de oefeningen.
