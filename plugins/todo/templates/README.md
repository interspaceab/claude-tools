# Todo – <ORGANISATION>

Planering och todos för arbete inom <ORGANISATION>: *vad* som ska göras,
*varför*, och vad som blev gjort. Koden ligger i respektive kodrepo, aldrig här.

Repot är privat och delas bara med de som jobbar inom <ORGANISATION>. Lägg
aldrig in lösenord, tokens eller nycklar.

## Innehåll

```
projects/<projekt>/
  active/     pågående och planerade uppgifter, en fil per uppgift
  done/<år>/  klara och strukna uppgifter, sorterade på stängningsår
  backlog.md  lösa idéer, en rad var
templates/item.md
CONVENTIONS.md
```

Format, statusar och commit-regler står i [CONVENTIONS.md](CONVENTIONS.md).

## Kom igång

### 1. Klona till en gemensam rot

Alla todo-repon klonas bredvid varandra under samma rot, så att de inte krockar
med varandra eller med kodrepon:

```bash
# Linux/macOS
git clone <REMOTE-URL> ~/todo/<KORTNAMN>

# Windows (PowerShell)
git clone <REMOTE-URL> "$env:USERPROFILE\todo\<KORTNAMN>"
```

### 2a. Jobba manuellt

1. `git pull --rebase`
2. Kopiera `templates/item.md` till
   `projects/<projekt>/active/ÅÅÅÅ-MM-DD-<slug>.md` och fyll i den.
3. Commit + push: `todo(<projekt>): add <slug>`
4. När uppgiften är klar: fyll i `## Resultat`, sätt `status: done` och
   `closed:`, och kör `git mv active/<fil> done/<år>/<fil>`.

### 2b. Jobba med Claude Code

Installera pluginen:

```
/plugin marketplace add interspaceab/claude-tools
/plugin install todo@interspace
```

Skapa sedan `~/todo/map.yaml` som talar om vilket todo-repo och projekt varje
kodrepo hör till. Minimal variant:

```yaml
version: 1
root: ~/todo
orgs:
  <KORTNAMN>:
    path: <KORTNAMN>
    remote: <REMOTE-URL>
rules:
  - match: { remote: "<GIT-HOST>/<ORG>/*" }
    org: <KORTNAMN>
```

Fler exempel finns i
[map.example.yaml](https://github.com/interspaceab/claude-tools/blob/main/plugins/todo/templates/map.example.yaml).
Om ett kodrepo inte matchar någon regel frågar Claude var uppgiften ska ligga.
Den gissar aldrig.

## Regler i korthet

- En fil per uppgift. Deluppgifter är checkboxar i filen.
- Pull före ändring, pusha direkt efter. Force-pusha aldrig.
- Inga hemligheter, och inget innehåll kopierat från andra organisationers
  todo-repon.
- Skriv så att en kollega förstår uppgiften utan att ha varit med.
