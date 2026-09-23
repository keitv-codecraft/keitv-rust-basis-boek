# TODO & Status

## Afgerond

- [x] Titels van alle 38 artikelen gestandaardiseerd naar `# N. TITEL` (synchroon met `SUMMARY.md`).
- [x] Consistente lesopbouw ingevoerd in alle 38 artikelen:
  - Bovenin: `## Wat gaan we leren?` met kernpunten en scheidingslijn `---`.
  - Onderin: directe verwijzing naar de bijbehorende map in `keitv-rustlings` (`https://github.com/keitv-codecraft/keitv-rust-basis-rustlings/tree/master/exercises/artikel_N/`).
  - Afsluitende scheidingslijn `---` en `## Controlelijst` met checklists voor leerdoelen.
- [x] Alle uitgeschreven ChatGPT-Rustlings opgaven verwijderd uit de artikelen; niet-Rustlings oefeningen (zelfstandige opdrachten, eindopdrachten, denkvragen) behouden met herstelde H2-nummering en logische H3-subkoppen.
- [x] Alle codevoorbeelden gecontroleerd op playground-compatibiliteit en testbaarheid:
  - Bewust foutieve voorbeelden voorzien van `ignore`.
  - Interactieve invoer/loops en bestands-I/O voorzien van `no_run` of `ignore`.
  - `mdbook test` slaagt 100% (alle 38 artikelen compileren en slagen).
- [x] Rustlings-repo gecontroleerd:
  - Alle 132 oplossingen geformatteerd met `rustfmt --edition 2024`.
  - `rustlings dev check` slaagt 100% (6/6 verificaties en 132/132 oplossingen).
- [x] Diagrammen en ASCII-schema's opgeschoond met Unicode boxed art.
- [x] Losse puntkomma's buiten codeblokken verwijderd.

## Toekomstige wensen & onderhoud

- [ ] Controleer na de eerste online publicatie de GitHub Pages URL en pas eventueel verdere stijlelementen aan.
- [ ] Voeg eventueel een logo, huisstijl en contactgegevens van KeiTV toe.
- [ ] Eventueel verdere interactieve Playground-verfijningen met verborgen `#`-regels waar gewenst voor specifieke losse expressies.
- [ ] Besluit nemen over publieke status van de `solutions/`-map in `keitv-rustlings` (zie `keitv-rustlings/TODO.md`).
