# Todo-konventioner

Det här repot innehåller planering och todos för en organisation. Kod ligger i
respektive kodrepo, aldrig här. Samma format i alla org-repon så att människor
och Claude gör likadant.

## Struktur

```
CONVENTIONS.md
templates/item.md
projects/
  <projekt>/
    README.md                       # vad projektet är, länkar till kodrepon
    backlog.md                      # lösa idéer, en rad var
    active/2026-11-23-invoice-sync.md
    done/2026/2026-11-02-api-auth.md
```

- **En fil per arbetsuppgift.** En uppgift = något som ska byggas/ändras och
  kan bockas av som helhet. Små deluppgifter är checkboxar i filen, inte egna filer.
- **Filnamn:** `ÅÅÅÅ-MM-DD-<slug>.md`, datum = skapad, slug i kebab-case på
  engelska, max ~5 ord. Filnamnet ändras aldrig (git-historiken följer med).
- **Projektnamn:** kebab-case, normalt samma som kodrepots namn.
- **backlog.md:** `- [typ] kort beskrivning (ÅÅÅÅ-MM-DD)`. Lyfts till en egen
  fil i `active/` när den ska göras, och raden tas bort.

## Livscykel

| Status    | Plats                   | Betyder                                   |
|-----------|-------------------------|-------------------------------------------|
| `active`  | `active/`               | Planerad eller pågående                   |
| `blocked` | `active/`               | Väntar på något, ange `blocked_by`        |
| `done`    | `done/<stängningsår>/`  | Klar, `## Resultat` ifylld                |
| `dropped` | `done/<stängningsår>/`  | Struken, skriv varför under `## Resultat` |

Stängning = sätt `status` + `closed`, fyll i `## Resultat`, sedan
`git mv active/<fil> done/<år>/<fil>`. Ingen radering — strukna uppgifter är
också historik.

## Frontmatter

```yaml
title:      Kort, beskrivande rubrik
type:       feat | fix | refactor | infra | docs | chore
status:     active | blocked | done | dropped
priority:   low | normal | high
created:    2026-11-23
closed:                       # sätts vid done/dropped
owner:      <användare>
repos:      [<org>/<kodrepo>]            # berörda kodrepon
refs:       []                           # commits, PR:er, ärenden
blocked_by:                              # fritext eller länk
tags:       []
```

`type` följer conventional commits så att det matchar commit-meddelandena.
Filtrera med t.ex. `rg -l '^type: fix' projects/*/done/2026`.

## Git

- `git pull --rebase` innan du läser eller ändrar.
- En commit per logisk ändring, pusha direkt efter.
- Meddelanden: `todo(<projekt>): add|update|close|drop <slug>`
- Konflikt vid rebase: lös manuellt, skriv inte över andras ändringar.

## Innehåll

- **Inga hemligheter.** Lösenord, tokens, nycklar → hänvisa till valvet.
- **Inget innehåll mellan organisationer.** Referera till ett id/kodrepo om det
  behövs, kopiera aldrig text från ett annat org-repo hit.
- Skriv så att en kollega förstår uppgiften utan att ha varit med i sessionen.
