# caiitech — Review-Prozess und Maschinen-Identitäten

**Stand:** 2026-08-30 · gilt org-weit für alle caiitech-Repos.
Empfohlener Ablageort: Repo `caiitech/.github` (GitHub wendet dortige Community-Health-Dateien org-weit an); alternativ pro Repo verlinken.

## 1. Identitäten und ihre Grenzen

| Identität | Art | Darf | Darf nicht |
|---|---|---|---|
| Menschen (Julian, Team) | GitHub-Nutzer | Review, Approve, Merge, Admin | — |
| **Claude Code** | eigener GitHub-Nutzer (Maschinen-Account, eigener SSH-Schlüssel, `gh`-Login) | Branches pushen, PRs eröffnen, auf Review-Kommentare antworten | Approven, Mergen ohne Review, Admin-Rechte, Bypass |
| GitHub App „mirror-sync" | App | in `caiitech-mirrors` schreiben | irgendwo sonst schreiben |
| GitHub App / PAT „docs" | App/Token | Produkt-Repos lesen (nur aggregierte), `caiitech/docs` dispatchen | schreiben außer Dispatch |
| CI | `GITHUB_TOKEN` / WIF | bauen, prüfen, deployen | Branch-Protection umgehen |

**Governance des Claude-Code-Accounts (verbindlich):**
- Collaborator mit **Write** nur auf den Produkt-Repos, in denen er arbeitet — nie Owner/Admin, nie Maintain, nie in einer Bypass-Liste, nie in CODEOWNERS.
- **Kein Zugriff auf die `caiitech-mirrors`-Organisation** (dort schreibt ausschließlich die GitHub App — Mirror-Sync Regel 2).
- Der SSH-Schlüssel gilt account-weit; die Zugriffsbegrenzung erfolgt daher ausschließlich über die Repo-Mitgliedschaften des Accounts — bei jedem neuen Repo bewusst entscheiden.
- Empfohlen: **SSH-Commit-Signierung** für den Account aktivieren (`git config gpg.format ssh` + vorhandener Schlüssel als Signing Key, Schlüssel bei GitHub als Signing Key hinterlegen) — Commits der Maschine tragen dann ein „Verified"-Badge und sind kryptografisch als Maschinen-Commits unterscheidbar.

## 2. Standard-PR-Flow (alle Repos)

1. **Claude Code** arbeitet auf einem Feature-Branch (ein Arbeitspaket = eine Session = ein PR) und eröffnet den PR selbst (`gh pr create`). Die PR-Beschreibung enthält: Referenz auf Arbeitspaket/SPEC-Paragraphen, Zusammenfassung, **Belege** (Testausgaben, ausgeführte Kommandos, Abnahme-Nachweise).
2. **CI-Gates** laufen (Pfad-gefilterte Builds, Tests, Lints — inkl. Import-Lint, Drift-Check, `.repos`-Check je nach Repo). Rot = kein Merge, ohne Ausnahme.
3. **Menschliches Review ist Pflicht:** Branch-Protection verlangt **1 Approval**. Da Autor (Maschine) und Reviewer (Mensch) verschiedene Accounts sind, ist Selbst-Approval ausgeschlossen. Review-Fokus: Diff gegen SPEC, Belege plausibel, Konventions-Commits getrennt (`apps.yaml`, `/proto`, Katalog).
4. Offene Review-Konversationen müssen aufgelöst sein (Branch-Protection).
5. **Merge: Rebase and merge** (lineare Historie, Einzelcommits bleiben erhalten). Merge durch den Reviewer oder nach Approval durch den Autor-Account — Approval bleibt in jedem Fall menschlich.
6. Weicht die Umsetzung von der SPEC ab: erst SPEC-Änderungs-Commit (im selben PR, klar benannt), dann Code — nie stillschweigend.

## 3. Verschärfungen für den Sicherheitspfad

Für `/services/clearance`, `/vehicle/lease-monitor`, `/safety/lease-token` (Fleet-Repo) zusätzlich:
- **Eigener PR**, nie gemischt mit anderem Scope; explizite Begründung im PR-Text.
- **CODEOWNERS-Review** durch benannte Menschen (Branch-Protection: „Require review from Code Owners" aktiv).
- Reviewer liest **jede Zeile**; Sessions dort nie unbeaufsichtigt, Plan Mode immer.
- CI-Import-Lint ist required Check; wird nie übersprungen oder als „flaky" neu gestartet, bis er grün „zufällig" durchläuft.

## 4. Branch-Protection-Sollzustand (mit Maschinen-Account)

- Require PR before merging, **required approvals: 1**
- Require status checks (alle repo-spezifischen Checks als required, „up to date" aktiv)
- Require conversation resolution · Require linear history (Rebase-Merge)
- Do not allow bypassing (gilt auch für Admins)
- Force-Push und Löschen: aus
- Repos mit Sicherheitspfad: zusätzlich Code-Owners-Review
- Empfehlung: als **Org-Ruleset** pflegen statt pro Repo

## 5. Einmalige Verifikations-Checkliste (nach Einrichtung des Accounts)

Auf der Maschine (kann Claude Code selbst als erste Session ausführen und die Ausgaben als Beleg liefern):

1. `ssh -T git@github.com` → begrüßt den **Maschinen-Account** (nicht einen persönlichen).
2. `gh auth status` → eingeloggt als Maschinen-Account, erwartete Scopes.
3. `git config user.name` / `user.email` → Maschinen-Identität (keine Commits unter menschlichem Namen).
4. Clone eines Produkt-Repos ✓; **Negativtest:** direkter Push auf `main` **muss scheitern** (Branch-Protection-Beleg).
5. Voller Zyklus im Testrepo oder per Trivial-PR: Branch → PR → CI grün → **Merge-Versuch ohne Approval muss scheitern** → menschliches Approval → Rebase-Merge ✓.
6. **Negativtest Mirrors:** Push auf ein `caiitech-mirrors`-Repo **muss scheitern** (Lesen darf klappen).
7. Falls Signierung aktiviert: Test-Commit zeigt „Verified" auf GitHub.

Erst wenn alle Punkte — inklusive der Negativtests — belegt sind, gilt das Setup als abgenommen. Ein Negativtest, der „aus Versehen" durchgeht, ist ein Governance-Fund, kein Erfolg.
