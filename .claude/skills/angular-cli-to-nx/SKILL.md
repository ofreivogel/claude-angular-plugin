---
name: angular-cli-to-nx
description: Konvertiert für angular bereitgestellte skill basierend auf Angular CLI für nx workspaces.
allowed-tools: Read, Write, Edit, Grep, Glob, WebFetch, Bash
---

## Verification sources nx

| Key | URL |
| :-- | :-- |
| `/introduction` | https://nx.dev/docs/technologies/angular/introduction |
| `/generators` | https://nx.dev/docs/technologies/angular/generators |
| `/executors` | https://nx.dev/docs/technologies/angular/executors |
| `/create-workspace` | https://nx.dev/docs/reference/create-nx-workspace |
| `generators.json` | https://raw.githubusercontent.com/nrwl/nx/master/packages/angular/generators.json |
| `executors.json` | https://raw.githubusercontent.com/nrwl/nx/master/packages/angular/executors.json |
| `/kb-tailwind` | https://nx.dev/docs/kb/using-tailwind-css-with-angular-projects |

## Abgrenzung

Translate Angular CLI specific only. Generic Nx knowledge — project graph, `affected`,
caching, library architecture, module boundaries, generator discovery — is **out of scope**

skip content concerning:
- set up new application and libaries, 
- e2e-testing
- migrations


## Procedure

1. Wenn nicht angegeben, Frage welcher skill transformiert werden soll
2. Kopiere den Inhalt des orginal skill nach nx
3. suche nach allen Angular CLI betroffenen Abschnitte im Skill file und den dazugehörigen Dateien wie Referenzen etc
4. Arbeite alle Abschnitte durch: Ersetzte angular CLI spezifische Befehle und Anweisungen mit passenden nx/angular befehlen. Entferne Inhalte die abgegrenzt sind. Verifiziere jede Änderung gegen die aktuelle API und Dokumentation des nx angular Plugin. (siehe [#Verification sources nx])
5. Prüfe deine Änderungen gegenüber der vorangehenden Version. Was nicht  angular CLI spezifisch ist sollte erhalten bleiben, wenn es "upstream" vorhanden ist.
6. verwende /code-review und behebe die Findings.
7. Stoppe ohne commit

Bsp für anpassungen 

| angular cli (source)                         | nx workspace (target)                                            |
|----------------------------------------------|------------------------------------------------------------------|
| `ng generate component /path/to/MyComponent` | `nx g @nx/angular:component apps/myApp/src/path/to/MyComponent`  |
| `ng generate service /path/to/MyService`     | `nx g @schematics/angular:service path/to/MyService`             |
