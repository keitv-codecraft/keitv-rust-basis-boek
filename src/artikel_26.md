# Een groter project structureren

Zet niet alle code in `main.rs`. Geef ieder onderdeel een eigen
verantwoordelijkheid.

```text
src/
├── main.rs
├── speler.rs
├── vijand.rs
├── inventaris.rs
├── wereld.rs
├── gevecht.rs
└── spel.rs
```

`main` start het programma. `spel` bestuurt de grote stappen. De andere
modules bevatten de regels van hun eigen onderdeel. Kleine functies en
duidelijke namen maken testen eenvoudiger.