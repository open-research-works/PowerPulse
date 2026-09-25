# IT'S NOT YOUR BUSINESS — Bauanleitung für die Social Game Engine

**Stand:** 2026-09-25  
**Status:** VERBINDLICHE ENGINE-BAULEITUNG / NOCH NICHT LIVE FREIGEGEBEN  
**Projekt:** It's Not Your Business  
**Kanonische Codequelle:** GitHub `open-research-works/PowerPulse`, Branch `main`  
**Verbindlicher Scope:** Standard 3–4 Personen, harte Obergrenze 5; 1:1 nur als eigener optionaler Modus  
**Bezug:** `03_PLAN__Social_Roleplay__Matching_WhatsApp_Gruppen_Game_Engine__2026-09-25.md`

---

## 0. Zweck dieser Bauanleitung

Dieses Dokument beschreibt **nicht nur einzelne Spielideen**, sondern die technische und spielmechanische Architektur einer wiederverwendbaren **Social Game Engine**.

Die Engine soll:

- kleine Gruppen von 3–5 Personen bedienen,
- Spiele nicht rein zufällig, sondern passend zur Gruppe auswählen,
- Rollen, geheime Informationen, Runden, Timer, Abstimmungen, Hinweise und Twists steuern,
- WhatsApp zunächst nur als Kommunikations-/Spielfeld verwenden,
- private Rollen und öffentliche Zustände sauber trennen,
- von sehr einfachen Kennenlernspielen bis zu komplexeren Social-Deduction-/Narrative-Spielen reichen,
- neue Spieltemplates ohne Umbau der Engine aufnehmen,
- Wiederspielbarkeit durch Variablen, Rollenrotation, modulare Inhalte und Repeat-Vermeidung erzeugen,
- LIVE niemals mehr als 5 Personen in eine Session setzen.

Die Leitidee lautet:

```text
Website/App = Game Master + Regelmaschine + Matching + private Informationen
WhatsApp     = Gespräch / Improvisation / Gruppenchat
```

Die Engine wird **aus abstrakten Mechaniken erfolgreicher und etablierter Spiele abgeleitet**, aber Texte, Rollen, Karten, Szenarien und konkrete proprietäre Inhalte werden **nicht kopiert**.

---

# TEIL A — Welche erfolgreichen Spielmuster wir übernehmen

## 1. Referenzmatrix

### 1.1 Just One — extrem niedrige Einstiegshürde

**Signal:** Spiel des Jahres 2019. Die Jury hebt ausdrücklich die Einfachheit und niedrige Einstiegshürde hervor.  
**Spieler:** 3–7.  
**Mechanik:** Eine Person kennt das Ziel nicht; andere geben unabhängig Hinweise; doppelte Hinweise fallen weg; danach wird geraten.

**Für unsere Engine relevant:**

- Regeln müssen in unter ca. 60–90 Sekunden verständlich sein,
- erste sinnvolle Aktion sehr früh,
- jede Person produziert Input,
- ein einfacher Constraint erzeugt automatisch interessante Gruppendynamik,
- kooperative Ziele funktionieren sehr gut für neue Gruppen.

**Engine-Primitiven:**

- `PROMPT`
- `PRIVATE_INPUT`
- `DEDUPLICATE`
- `REVEAL`
- `GUESS`
- `SCORE`

Quelle: https://www.spiel-des-jahres.de/just-one-ist-das-spiel-des-jahres-2019/  
Quelle: https://www.rprod.com/de/games/just-one

---

### 1.2 Codenames — einfache Regel, starke Asymmetrie

**Signal:** Spiel des Jahres 2016.  
**Mechanik:** Ein Spieler besitzt mehr Information als sein Team. Er darf nur sehr begrenzt kommunizieren: ein Hinweiswort plus Zahl.

**Für unsere Engine relevant:**

- asymmetrische Information ist ein starker Gesprächsmotor,
- ein klarer Kommunikations-Constraint erzeugt Kreativität,
- private und öffentliche Informationen müssen technisch strikt getrennt sein,
- Spannung entsteht nicht durch komplizierte Regeln, sondern durch Informationsverteilung.

**Engine-Primitiven:**

- `PRIVATE_MAP`
- `CLUE`
- `COMMUNICATION_CONSTRAINT`
- `TEAM_GUESS`
- `DANGER_TARGET`
- `REVEAL`

Quelle: https://www.spiel-des-jahres.de/erjahr/2016/  
Regelreferenz: https://czechgames.com/files/rules/codenames-rules-en.pdf

---

### 1.3 Die Crew — Missionen + limitierte Kommunikation

**Signal:** Kennerspiel des Jahres 2020.  
**Spieler:** 3–5.  
**Mechanik:** Kooperative Missionen, bei denen nicht frei über private Informationen gesprochen werden darf; Kommunikation ist gezielt begrenzt.

**Für unsere Engine relevant:**

- kleine Missionen geben der Gruppe ein gemeinsames Ziel,
- Informationsknappheit macht Kooperation interessant,
- Missionen können im Schwierigkeitsgrad steigen,
- ein einziges begrenztes Kommunikationsfenster pro Runde ist ein starkes Designwerkzeug,
- 3–5 Personen funktionieren hier bereits als bewährte Kernspanne.

**Engine-Primitiven:**

- `MISSION`
- `PRIVATE_STATE`
- `LIMITED_SIGNAL`
- `TASK_ASSIGNMENT`
- `ROUND_SUCCESS`
- `RETRY`
- `DIFFICULTY_STEP`

Quelle: https://www.spiel-des-jahres.de/spiel-des-jahres-2020-kennerspiel-des-jahres/  
Regelreferenz: https://thamesandkosmos.com/manuals/full/691868_Crew_Manual.pdf

---

### 1.4 The Mind — minimale Regeln, emergente Gruppendynamik

**Signal:** Nominiert zum Spiel des Jahres 2018.  
**Spieler:** 2–4.  
**Mechanik:** Gemeinsame Aufgabe bei praktisch fehlender Kommunikation; Koordination entsteht aus Timing und gegenseitiger Einschätzung.

**Für unsere Engine relevant:**

- nicht jedes Spiel braucht viele Texte oder Rollen,
- kurze, fast wortlose Mechaniken eignen sich als Icebreaker,
- die Engine sollte auch **micro games** von 2–5 Minuten unterstützen,
- Timing kann selbst Spielmechanik sein.

**Engine-Primitiven:**

- `SILENT_PHASE`
- `PLAYER_READY`
- `TIMED_ACTION`
- `ORDER_CHECK`
- `SHARED_LIVES`
- `LEVEL_UP`

Quelle: https://www.spiel-des-jahres.de/preiskat/nominiert-zum-spiel-des-jahres/page/2/  
Quelle: https://pandasaurusgames.com/products/the-mind

---

### 1.5 Decrypto — Geheimnisse, Hinweise, Interpretation, Gegeninterpretation

**Signal:** Hersteller beschreibt Decrypto als mehrfach ausgezeichnet und in 48 Ländern veröffentlicht.  
**Mechanik:** Teams geben eigene Hinweise so, dass Teammitglieder sie verstehen, Gegner aber nicht; Hinweise sammeln über Runden Bedeutung.

**Für unsere Engine relevant:**

- Hinweise können über mehrere Runden eine gemeinsame Sprache bilden,
- dieselbe Information kann für unterschiedliche Spieler verschiedene Bedeutung haben,
- private Geheimnisse + öffentliche Hinweise + Historie erzeugen Tiefe,
- der Wechsel einer aktiven Rolle pro Runde verhindert, dass immer dieselbe Person dominiert.

**Engine-Primitiven:**

- `SECRET_CODE`
- `ROTATING_ROLE`
- `CLUE_HISTORY`
- `OWN_TEAM_DECODE`
- `OPPONENT_INTERCEPT`
- `ROUND_LOG`

Quelle: https://www.scorpionmasque.com/en/decrypto  
Quelle: https://shop.scorpionmasque.com/products/decrypto

---

### 1.6 Fiasco — Beziehungen statt statischer Charakterprofile

**Signal:** Offiziell als award-winning GM-less game beschrieben.  
**Spieler:** 3–5.  
**Mechanik:** Beziehungen und strukturierte Story-Elemente erzeugen gemeinsam eine Geschichte; moderne Edition arbeitet u. a. mit Relationship-, Need-, Object- und Location-Elementen.

**Für unsere Engine relevant:**

- eine Gruppe wird interessanter, wenn **Beziehungen zwischen Spielern** generiert werden,
- Rollen sollten nicht nur Eigenschaften haben, sondern ein Ziel gegenüber anderen Rollen,
- Story kann aus kombinierbaren Datenbausteinen entstehen,
- die Engine sollte Beziehungsgraphen unterstützen.

**Engine-Primitiven:**

- `RELATIONSHIP_EDGE`
- `NEED`
- `OBJECT`
- `LOCATION`
- `SCENE_SEED`
- `ESCALATION`
- `AFTERMATH`

Quelle: https://bullypulpitgames.com/products/fiasco  
Lizenz-/Strukturhinweis: https://bullypulpitgames.com/pages/fiasco-diy-license

**Wichtig:** Keine Fiasco-Playsets oder konkreten Karten kopieren. Nur die abstrakte Erkenntnis übernehmen: Beziehungen + Ziele + Orte + Objekte erzeugen kombinatorisch Geschichten.

---

### 1.7 Alice is Missing — Messenger kann das eigentliche Spielfeld sein

**Signal:** Etabliertes textbasiertes RPG; offizieller Anbieter beschreibt 3–5 Spieler, 90 Minuten Kernspiel und vollständige Kommunikation über Textnachrichten.  
**Mechanik:** Figuren, Beziehungen, private und gemeinsame Chats, zeitgesteuerte Entwicklung.

**Für unsere Engine relevant:**

- WhatsApp kann tatsächlich als Spielfeld dienen,
- Website muss nicht selbst Chat bauen,
- private Rollen/Informationen können auf der Website liegen, während Interaktion im Messenger stattfindet,
- zeitgesteuerte Ereignisse können einen Chat strukturiert vorantreiben,
- Charakterbeziehungen sollten vor Spielbeginn definiert sein.

**Engine-Primitiven:**

- `CHARACTER`
- `RELATIONSHIP`
- `PRIVATE_PROMPT`
- `PUBLIC_EVENT`
- `TIMELINE_TRIGGER`
- `TEXT_PHASE`
- `FINAL_REVEAL`

Quelle: https://www.huntersentertainment.com/alice-is-missing  
Quelle: https://www.huntersentertainment.com/product-page/alice-is-missing

---

### 1.8 For the Queen — Prompt-Karten als teach-as-you-play-System

**Signal:** Offizielle Beschreibung: 2–6+, no-prep, Regeln werden beim Spielen gelernt; Story entsteht über Fragen.  
**Mechanik:** Fragekarten bauen Figuren, Beziehungen und gemeinsame Geschichte schrittweise auf.

**Für unsere Engine relevant:**

- Regeln können während des Spiels eingeführt werden,
- Prompt-Folgen sind ideal für ROLEPLAY → REAL → DEEP,
- dieselbe Engine kann verschiedene Genres fahren,
- Spieler müssen keinen Regeltext vorab lesen.

**Engine-Primitiven:**

- `PROMPT_DECK`
- `PROMPT_LEVEL`
- `ROTATING_RESPONDER`
- `FOLLOW_UP`
- `FINAL_DECISION`

Quelle: https://darringtonpress.com/forthequeen/  
Quelle: https://darringtonpress.com/announcing-for-the-queen/

---

### 1.9 Wavelength / Fun Facts — soziale Kalibrierung statt richtig/falsch

**Mechanik:** Menschen ordnen Einschätzungen auf Skalen ein oder versuchen, andere einzuschätzen. Das erzeugt automatisch Gespräch und Überraschung.

**Für unsere Engine relevant:**

- nicht alle Spiele brauchen Sieg/Niederlage,
- Einschätzungen über Menschen sind ein guter Übergang zu REAL,
- Skalen erzeugen Diskussion, ohne sofort intim zu werden,
- gute Mechanik für 3–5 neue Personen.

**Engine-Primitiven:**

- `SPECTRUM`
- `SECRET_ESTIMATE`
- `ORDERING`
- `REVEAL`
- `COMPARE`
- `DISCUSSION`

Quelle: https://www.cmyk.games/products/wavelength  
Quelle: https://www.rprod.com/en/press/fun-facts

---

## 2. Designrahmen: MDA als Prüfwerkzeug

Die Engine verwendet MDA als Review-Schema:

```text
MECHANICS  → Was erlaubt/erzwingt das System?
DYNAMICS   → Was tun Menschen dadurch miteinander?
AESTHETICS → Wie soll es sich anfühlen?
```

Für jedes Template muss deshalb nicht nur die Regel beschrieben werden, sondern auch die erwartete Gruppendynamik.

Beispiel:

```text
Mechanik: Jeder bekommt eine andere geheime Information.
Dynamik: Spieler müssen reden, vergleichen, misstrauen, überzeugen.
Erlebnisziel: Neugier + leichter Verdacht + Lachen, nicht Stress.
```

Quelle: Hunicke, LeBlanc, Zubek, MDA Framework  
https://aaai.org/papers/ws04-04-001-mda-a-formal-approach-to-game-design-and-game-research/

---

# TEIL B — Produktregeln der Engine

## 3. Harte Designprinzipien

### P1 — Standard 3–4, Maximum 5

Keine Engine-Funktion darf stillschweigend größere Gruppen erzeugen.

```text
preferred_size = 3..4
hard_max       = 5
```

1:1 ist ein separater Modus.

### P2 — Erste echte Aktion schnell

Ziel:

- erste verständliche Entscheidung/Antwort in < 2 Minuten,
- keine langen Regelblöcke,
- Regeln möglichst kontextuell vor der jeweiligen Phase.

### P3 — Keine frühe Eliminierung

Niemand soll 20 Minuten zuschauen müssen.

Wenn jemand „ausscheidet“, wechselt die Rolle stattdessen z. B. zu:

- Beobachter mit Sonderinformation,
- Ghost Role,
- Moderator,
- Saboteur mit neuem Ziel,
- Jury.

### P4 — Jede Person bekommt wiederholt Spotlight

Die Engine trackt, wer zuletzt aktive Rolle hatte.

```text
active_role_fairness = required
```

### P5 — Private Information ist serverseitig privat

UI-Verstecken reicht nicht.

Player A darf private Daten von Player B auch durch direkte API-Aufrufe nicht lesen können.

### P6 — Randomness ist Gewürz, nicht Architektur

Zufall darf:

- Rollen innerhalb geeigneter Kandidaten verteilen,
- gleichwertige Templates tie-breaken,
- Variationen auswählen.

Zufall darf **nicht**:

- ungeeignete Gruppengrößen erzeugen,
- Intensitätsgrenzen überschreiten,
- Sicherheitsregeln umgehen,
- Wiederholungsvermeidung ignorieren.

### P7 — Eskalation kontrolliert statt chaotisch

Ein gutes Template hat eine Kurve:

```text
HOOK → EASY ACTION → INTERACTION → COMPLICATION → REVEAL → PAYOFF → OPTIONAL REAL/DEEP
```

### P8 — Spielinhalt und Engine strikt trennen

Neue Spiele sollen über Daten/Templates entstehen, nicht über neue App-Logik.

### P9 — Safety ist Engine-Funktion

Jede Session unterstützt:

- `SKIP`
- `LOWER_INTENSITY`
- `LEAVE_SESSION`
- Report/Block später
- keine Pflicht zu REAL/DEEP

### P10 — Resume statt Totalschaden

Bei Reload/Disconnect muss der Server wissen:

- aktuelle Phase,
- Runde,
- aktive Person,
- bereits eingereichte Aktionen,
- private Rolle,
- Timer-Start,
- nächste erlaubte Transition.

---

# TEIL C — Die eigentliche Engine

## 4. Engine-Primitiven

Die Engine braucht einen kleinen Satz wiederverwendbarer Bausteine.

### 4.1 Informationsbausteine

- `PUBLIC_PROMPT`
- `PRIVATE_PROMPT`
- `SECRET`
- `ROLE`
- `RELATIONSHIP_EDGE`
- `OBJECT`
- `LOCATION`
- `GOAL`
- `CONSTRAINT`

### 4.2 Aktionsbausteine

- `ANSWER_TEXT`
- `ANSWER_SCALE`
- `CHOICE`
- `VOTE`
- `GUESS`
- `ORDER`
- `READY`
- `REVEAL`
- `PASS`
- `SKIP`

### 4.3 Zeit-/Ablaufbausteine

- `TIMER`
- `ROUND`
- `ROTATE_ACTIVE_PLAYER`
- `WAIT_FOR_ALL`
- `WAIT_FOR_QUORUM`
- `AUTO_ADVANCE`
- `MANUAL_ADVANCE`

### 4.4 Bewertungsbausteine

- `COOP_SUCCESS`
- `SCORE_DELTA`
- `TEAM_SCORE`
- `MISSION_SUCCESS`
- `INTERCEPTION`
- `SIMILARITY`
- `CONSENSUS`

### 4.5 Story-Bausteine

- `SCENE_SEED`
- `TWIST`
- `ESCALATION`
- `RELATIONSHIP_CHANGE`
- `FINAL_DECISION`
- `AFTERMATH`
- `REFLECTION`

---

## 5. Verbindliche State Machine

Jedes Spieltemplate läuft auf derselben Oberstruktur.

```text
CREATED
→ LOBBY
→ READY_CHECK
→ SETUP
→ ROUND_INTRO
→ PRIVATE_PHASE      (optional)
→ PUBLIC_ACTION      (optional)
→ DISCUSSION         (optional)
→ DECISION           (optional)
→ RESOLUTION
→ REVEAL             (optional)
→ REFLECTION         (optional)
→ NEXT_ROUND | FINISHED
```

Zusätzliche Zustände:

```text
PAUSED
ABORTED
EXPIRED
RECOVERY_REQUIRED
```

### 5.1 Transition-Regel

Der Client darf niemals einfach `round = 4` setzen.

Transitions erfolgen serverseitig:

```text
advance(run_id, expected_state, action)
```

Server prüft:

1. Session existiert,
2. Spieler gehört zur Session,
3. Aktion ist in diesem Zustand erlaubt,
4. Idempotency-Key wurde noch nicht verarbeitet,
5. alle erforderlichen Inputs liegen vor,
6. Zielzustand ist im Template erlaubt.

---

## 6. Template-DSL

Ein Spiel ist Daten + kleine deklarative Regeln.

Beispielstruktur:

```yaml
id: city_secret_001
version: 1
status: draft
name: "Der verschwundene Umschlag"
player_count:
  min: 3
  preferred: 4
  max: 5
estimated_minutes: 22
complexity: 2
modes: [roleplay, mystery, real]
intensity: light
requires_whatsapp_group: true
requires_private_roles: true
elimination: false
repeat_after_days: 30

experience_goals:
  - curiosity
  - laughter
  - light_suspicion

setup:
  relationship_graph: true
  role_assignment: balanced_random

phases:
  - id: intro
    type: public_prompt
    content_ref: city_secret_intro_v1

  - id: private_roles
    type: private_prompt
    source: role_payload
    wait_for: all_acknowledged

  - id: round_1
    type: discussion
    timer_seconds: 240
    constraint_ref: no_direct_role_reveal

  - id: choice_1
    type: vote
    visibility: secret_until_all

  - id: reveal_1
    type: reveal
    reveal_ref: twist_a

  - id: real_bridge
    type: reflection
    optional: true
    intensity: real_1

end:
  type: outcome
  outcome_ref: city_secret_end_v1
```

### 6.1 Template muss validierbar sein

Build/Test schlägt fehl, wenn:

- `max_players > 5`,
- `min_players > max_players`,
- unbekannter Phase-Type,
- `private_prompt` ohne private Rollenreferenz,
- Endzustand fehlt,
- Transition auf nicht existente Phase zeigt,
- Pflichtphase keine Exit-Bedingung besitzt,
- Deep-Phase nicht `optional: true` ist.

---

# TEIL D — Spiel-Familien, die wir bauen

## 7. Familie A: Narrative Story Web

**Spieler:** 3–4 bevorzugt, max. 5  
**Inspiration:** Fiasco + For the Queen, aber komplett eigene Inhalte.

### Aufbau

```text
1. gemeinsamer Ort / Anlass
2. Beziehungsgraph zwischen Spielern
3. jede Rolle erhält 1 öffentliches Merkmal
4. jede Rolle erhält 1 privates Ziel
5. gemeinsames Problem
6. Runde 1: Positionen zeigen
7. Twist
8. Runde 2: Konflikt/Kooperation
9. Entscheidung
10. optional REAL-Frage
11. Abschluss
```

### Warum funktioniert es

Nicht „wer bist du?“ erzeugt Story, sondern:

```text
A will etwas von B
B verbirgt etwas vor C
C besitzt etwas, das A braucht
```

Die Engine erzeugt deshalb einen **gerichteten Beziehungsgraphen**.

### Mindestdaten

```yaml
role:
  public_trait
  private_goal
  secret
  leverage
  relationship_to_next_player
```

---

## 8. Familie B: Secret Signal / Deduction Light

**Spieler:** 3–5  
**Inspiration:** Codenames, Decrypto, Die Crew.

### Aufbau

```text
1. System verteilt geheime Informationen
2. aktive Person darf nur begrenzt Hinweise geben
3. Rest der Gruppe interpretiert
4. optional: eine Person hat leicht anderes Ziel
5. Reveal
6. Rollenrotation
7. nächste Runde etwas schwieriger
```

### Wichtige Designregel

Der Constraint muss **glasklar** sein.

Gute Constraints:

- nur 3 Wörter,
- kein Eigenname,
- nur Emoji,
- nur eine Frage,
- nur eine Nachricht innerhalb 30 Sekunden,
- bestimmte verbotene Wörter.

Schlechte Constraints:

- mehrseitige Ausnahmeregeln,
- subjektive Regelinterpretation,
- zehn Sonderfälle.

---

## 9. Familie C: Cooperative Mission

**Spieler:** 3–5  
**Inspiration:** Die Crew + The Mind.

### Aufbau

```text
MISSION BRIEF
→ private Ressourcen/Infos
→ Kommunikationslimit
→ gemeinsame Entscheidung
→ serverseitige Auflösung
→ Erfolg/Fehlschlag
→ nächste Mission
```

### Progression

Mission 1 lehrt eine Mechanik.
Mission 2 fügt eine zweite hinzu.
Mission 3 kombiniert beide.

Nie direkt alle Regeln auf einmal.

---

## 10. Familie D: Social Calibration

**Spieler:** 3–5  
**Inspiration:** Wavelength, Fun Facts.

### Aufbau

```text
Frage / Spektrum
→ jeder gibt geheim Einschätzung ab
→ Gruppe sortiert / schätzt
→ Reveal
→ kurze Diskussion
→ nächste Runde
```

Beispiele für **eigene** Achsen:

```text
"Planen" ←────────→ "Einfach los"
"Sag es direkt" ←──→ "Lass mich erst raten"
"Safe choice" ←────→ "Chaos choice"
```

Ideal als Icebreaker oder REAL-Übergang.

---

## 11. Familie E: Text Roleplay Timeline

**Spieler:** 3–5  
**Inspiration:** abstrakt aus Alice is Missing.

### Aufbau

```text
SETUP
→ Rollen + Beziehungen
→ gemeinsamer WhatsApp-Raum
→ T+00 öffentliches Ereignis
→ T+05 privater Hinweis an A
→ T+08 privater Hinweis an C
→ T+12 Twist
→ T+18 Entscheidung
→ T+22 Finale
```

### Engine-Anforderung

Timer dürfen nicht nur clientseitig laufen.

Server speichert:

```text
phase_started_at
scheduled_event_at
fired_at
```

Bei Reload wird Ereignis nicht doppelt ausgelöst.

---

## 12. Familie F: REAL / DEEP Ladder

**Spieler:** 2–5  
**Kein klassisches Punktespiel.**

### Progression

```text
LEVEL 0 PLAYFUL
LEVEL 1 REAL
LEVEL 2 PERSONAL
LEVEL 3 DEEP
```

Regeln:

- Level 2/3 niemals ohne vorheriges Opt-in,
- jede Karte hat `skip_allowed: true`,
- niemand muss begründen, warum er skippt,
- Antworten aus Deep-Level werden nicht für Matchingprofile gespeichert.

Diese Familie kann standalone oder als kurze Phase in anderen Spielen eingesetzt werden.

---

# TEIL E — Auswahl: Welches Spiel bekommt welche Gruppe?

## 13. Gruppenprofil

Nach Matching besitzt jede Gruppe ein internes Profil:

```yaml
player_count: 4
newcomer_ratio: 0.75
shared_modes: [roleplay, mystery]
max_intensity: light
preferred_duration: 25
shared_interests: [music, outdoors]
recent_template_ids: []
roleplay_experience: low
```

### 13.1 Hard Filter

Template wird ausgeschlossen, wenn:

- Spielerzahl nicht passt,
- Intensität zu hoch,
- Modus nicht kompatibel,
- erforderliche Funktion nicht verfügbar,
- Template kürzlich von zu vielen Gruppenmitgliedern gespielt,
- Safety-Flag blockiert,
- WhatsApp erforderlich, aber Room nicht bereit.

### 13.2 Soft Score

```text
TEMPLATE_SCORE =
  + player_count_fit
  + mode_fit
  + newcomer_fit
  + duration_fit
  + variety_bonus
  + group_interest_fit
  - repeat_penalty
  - complexity_penalty
```

### 13.3 Kein Black-Box-Zufall

Bei Auswahl speichern:

```json
{
  "template_id": "city_secret_001",
  "score": 8.4,
  "components": {
    "player_count_fit": 2.0,
    "mode_fit": 2.0,
    "newcomer_fit": 1.5,
    "duration_fit": 1.2,
    "variety_bonus": 1.7,
    "repeat_penalty": 0
  }
}
```

So können wir später nachvollziehen, warum ein Spiel gewählt wurde.

---

# TEIL F — Rollenverteilung

## 14. Rollen sind nicht gleich Matching

Matching wählt Menschen für eine Gruppe.
Rollenverteilung passiert **danach**.

### 14.1 Rollen-Constraints

Ein Template kann deklarieren:

```yaml
roles:
  - id: planner
    tags: [structured, verbal]
  - id: wildcard
    tags: [chaos, improv]
  - id: observer
    tags: [analysis, quiet_friendly]
  - id: connector
    tags: [social, bridge]
```

### 14.2 Fairness

Die Engine merkt sich Rollenhistorie:

- aktive Führungsrolle nicht immer an dieselbe Person,
- introvertiertere Rolle darf genauso relevant sein,
- keine Rolle darf nur „Statist“ sein,
- private Information muss echten Handlungswert haben.

---

# TEIL G — Content-System

## 15. Content-Pack-Struktur

```text
content/
  packs/
    starter/
      pack.yaml
      templates/
      roles/
      prompts/
      twists/
      reflections/
    mystery/
    chaos/
    adventure/
    real-talk/
```

### 15.1 Jedes Content-Element bekommt Metadaten

```yaml
id: prompt_real_014
language: de
intensity: real_1
age_min: 18
tags: [decision, spontaneity]
reusable: true
cooldown_days: 14
safety_tags: []
status: tested
```

### 15.2 Status-Lifecycle

```text
DRAFT
→ INTERNAL_TEST
→ PLAYTEST
→ APPROVED
→ LIVE
→ RETIRED
```

Codex darf DRAFT nicht automatisch auf LIVE setzen.

---

## 16. Autoren-Checkliste für jedes neue Spiel

Vor Aufnahme in Registry:

1. Was ist der Hook in einem Satz?
2. Was tut jede Person in den ersten 2 Minuten?
3. Welche Information ist privat?
4. Welche Information ist öffentlich?
5. Was zwingt die Gruppe miteinander zu interagieren?
6. Gibt es eine interessante Entscheidung?
7. Wo steigt die Spannung?
8. Wie wird aufgelöst?
9. Kann jemand lange untätig sein?
10. Kann jemand unfreiwillig bloßgestellt werden?
11. Was passiert bei 3, 4 und 5 Spielern?
12. Was macht einen zweiten Durchlauf anders?
13. Welche Teile sind Daten, welche Engine-Logik?
14. Gibt es proprietäre Vorlage, die zu nah kopiert wäre?
15. Welche Metrik zeigt nach dem Test, ob es funktioniert?

---

# TEIL H — Technische Architektur

## 17. Empfohlener Codeaufbau

```text
src/
  engine/
    types.ts
    state-machine.ts
    transitions.ts
    registry.ts
    template-validator.ts
    role-assignment.ts
    scoring.ts
    timers.ts
    recovery.ts

  matching/
    hard-filters.ts
    group-builder.ts
    group-score.ts
    fairness.ts
    repeat-penalty.ts

  content/
    packs/
    loader.ts
    schemas.ts

  server/
    auth.ts
    room-service.ts
    run-service.ts
    event-service.ts

  ui/
    pool/
    waiting/
    room/
    private-role/
    game-master/
    reflection/

tests/
  engine/
  matching/
  authorization/
  simulations/
  e2e/
```

Dateinamen können an vorhandenes Framework angepasst werden; die Trennung der Verantwortungen bleibt.

---

## 18. Datenmodell

### 18.1 `player_session`

```text
id
user_id / anonymous_session_id
age
city
region
language
mode_preferences
intensity
interests
vibe
status
created_at
expires_at
```

### 18.2 `pool_entry`

```text
id
player_id
joined_at
availability_until
preferred_group_size
status
matching_version
```

### 18.3 `game_room`

```text
id
city
capacity                # 3..5
status
whatsapp_invite_url     # protected server-side
current_run_id
created_at
cooldown_until
```

### 18.4 `match_assignment`

```text
id
player_id
room_id
match_batch_id
score_components jsonb
assigned_at
accepted_at
joined_confirmed_at
```

Unique Constraints:

```text
one active assignment per player
room active assignments <= capacity
capacity <= 5
```

### 18.5 `game_template`

```text
id
version
status
min_players
preferred_players
max_players
complexity
modes
intensity
schema jsonb
checksum
```

DB-Check:

```text
max_players <= 5
```

### 18.6 `game_run`

```text
id
room_id
template_id
template_version
state
phase_id
round_number
phase_started_at
version
started_at
ended_at
```

### 18.7 `game_player`

```text
run_id
player_id
seat
role_id
active_order
status
```

### 18.8 `private_payload`

```text
id
run_id
player_id
payload_type
payload jsonb
created_at
revealed_at nullable
```

**RLS:** nur `player_id == auth.uid/session identity` darf lesen.

### 18.9 `game_event`

Append-only Audit/Event Log:

```text
id
run_id
sequence
actor_player_id nullable
event_type
payload
created_at
idempotency_key
```

### 18.10 `template_history`

```text
player_id
template_id
played_at
completion_state
```

---

## 19. Supabase / Realtime

Supabase eignet sich für den MVP, weil Postgres + RLS + Realtime verfügbar sind.

### 19.1 RLS

Auf allen exponierten Tabellen RLS aktivieren. Supabase empfiehlt RLS für granularen Zugriff und weist darauf hin, dass exponierte Tabellen ohne passende Schutzregeln problematisch sind.

Quelle: https://supabase.com/docs/guides/database/postgres/row-level-security

### 19.2 Realtime

Verwendung:

- `Broadcast` für Spielereignisse und Zustandsänderungen,
- `Presence` sparsam für `online / ready`,
- Postgres bleibt authoritative source.

Supabase beschreibt Broadcast explizit auch für Game Events; für Datenänderungen wird Broadcast gegenüber Postgres Changes für Skalierbarkeit/Sicherheit empfohlen.

Quelle: https://supabase.com/docs/guides/realtime  
Quelle: https://supabase.com/docs/guides/realtime/subscribing-to-database-changes

### 19.3 Private Channels

Room-Kanäle:

```text
room:{room_id}
run:{run_id}
player:{player_id}:{run_id}
```

Private Player-Payload niemals über gemeinsamen Room-Channel schicken.

---

## 20. Server-Funktionen / API

Minimum:

```text
POST /api/pool/join
POST /api/pool/leave
POST /api/match/run
POST /api/match/{id}/accept
GET  /api/room/current
POST /api/room/join-confirm
POST /api/run/start
GET  /api/run/current
GET  /api/run/my-private-state
POST /api/run/action
POST /api/run/skip
POST /api/run/advance
POST /api/run/abort
```

### 20.1 Server ist authoritative

Client darf vorschlagen:

```json
{
  "action": "submit_vote",
  "choice": "A"
}
```

Client darf nicht setzen:

```json
{
  "state": "FINISHED",
  "winner": "player_4"
}
```

---

## 21. Concurrency / Idempotenz

### Matching

Gruppenbildung muss transaktional erfolgen.

Problem:

Zwei parallele Matcher dürfen denselben Spieler nicht gleichzeitig zuweisen.

Lösung:

- DB-Transaktion,
- `SELECT ... FOR UPDATE SKIP LOCKED` oder passende serverseitige DB-Funktion,
- Unique Constraint auf aktive Assignment,
- Assignment + Room Count atomar.

### Aktionen

Jede Mutation bekommt:

```text
idempotency_key
```

Doppelklick / Retry darf keine zweite Stimme, keine zweite Zuweisung und kein zweites Event erzeugen.

---

# TEIL I — WhatsApp-Schicht

## 22. V1: vorab angelegte Räume

Nicht pro Match neue Gruppe erstellen.

Pilot:

```text
4 WhatsApp Game Rooms
je Zielgröße 3–5
```

Backend speichert Invite-Link geschützt.

Flow:

```text
MATCHED
→ Room assigned
→ Consent-Hinweis
→ Invite-Link anzeigen
→ Spieler tritt WhatsApp-Gruppe bei
→ "Ich bin drin"
→ READY
→ Spielstart
```

### 22.1 WhatsApp-Link ist kein öffentliches Profilelement

Nur zugewiesene Spieler dürfen ihn abrufen.

### 22.2 Browserautomation durch Codex

Erst nach Approval Gate:

1. exakte Room-Liste ausgeben,
2. Nutzer bestätigt,
3. WhatsApp Web öffnen,
4. nur bestätigte Gruppen erzeugen,
5. Name + Settings setzen,
6. Invite-Link holen,
7. Backend-Konfiguration schreiben,
8. jeden Room per Readback prüfen,
9. bei Warnung/Rate-Limit stoppen.

---

# TEIL J — UX des Game Masters

## 23. Gemeinsame Ansicht

Zeigt nur gemeinsame Informationen:

- Spielname,
- Phase,
- Timer,
- gemeinsame Aufgabe,
- Fortschritt,
- Buttons `Ready`, `Nächste Runde`, falls erlaubt.

## 24. Private Ansicht

Pro Spieler:

- Rolle,
- geheimes Ziel,
- geheime Information,
- persönliche Aktion,
- niemals Daten anderer Spieler.

## 25. Wiederkehrende UX-Komponenten

```text
<PhaseHeader />
<Timer />
<PublicPrompt />
<PrivateCard />
<ChoiceGrid />
<VotePanel />
<RevealCard />
<ReadyCheck />
<SkipControl />
<IntensityControl />
<ReconnectBanner />
```

---

# TEIL K — Testing

## 26. Testpyramide

### Unit Tests

- Template Validator,
- Transition Validator,
- Score-Berechnung,
- Repeat Penalty,
- Role Assignment,
- Timer-Berechnung.

### Property / Invariant Tests

Immer wahr:

```text
room.capacity <= 5
active_assignments <= room.capacity
player has <= 1 active assignment
private_payload belongs to exactly one player
run phase exists in template
Deep phase is skippable
```

### Simulation

Mindestens:

- 20 simulierte Spieler für MVP-Abnahme,
- zusätzlich 100 simulierte Sessions vor breiter Beta.

Szenarien:

- ungerade Poolgröße,
- mehrere Städte,
- 3er/4er/5er-Gruppen,
- ein Spieler lehnt Match ab,
- zwei Spieler disconnecten,
- Room voll,
- Room-Link fehlt,
- alle haben dasselbe Template kürzlich gespielt,
- lange wartender Spieler,
- parallele Matchmaker.

### Authorization / Two-User Negative Tests

Verbindlich:

```text
A cannot GET B private payload
A cannot PATCH B action
A cannot fetch another room invite
A cannot move B to another room
A cannot mark B ready
A cannot reveal B secret
```

### E2E

Mindestens zwei reale Browser-Sessions:

```text
User A
User B
(+ optional C/D/E)
```

Test vollständiger Ablauf:

```text
Join Pool
→ Match
→ Accept
→ Room
→ Ready
→ private role
→ public phase
→ action
→ reveal
→ next round
→ finish
```

---

# TEIL L — Playtesting wie ein Produkt, nicht wie ein Bauchgefühl

## 27. Nach jeder Session erfassen

Keine sensiblen Inhalte speichern; nur Produktmetriken.

```text
join_to_match_seconds
match_accept_rate
ready_completion_rate
session_completion_rate
time_to_first_action
average_round_duration
skip_rate
lower_intensity_rate
reconnect_count
rules_help_opened
abandon_phase
would_play_again
would_play_with_new_people
```

### 27.1 Qualitative Fragen

Maximal 3 kurze Fragen:

1. Wo warst du kurz verwirrt?
2. Welche Runde war am besten?
3. Würdest du mit einer neuen Gruppe nochmal spielen?

### 27.2 Erfolgsdefinition für ein Template

Ein Template wird nicht wegen einer lustigen Einzelrunde LIVE.

Vorschlag:

```text
>= 10 Test-Sessions
keine kritischen Safety-Probleme
keine wiederkehrende Regelverwirrung
hohe Completion
mehrheitlich "würde nochmal spielen"
```

Keine starre statistische Aussage bei kleinen Samples; Daten dienen zunächst zum Tuning.

---

# TEIL M — Bau-Reihenfolge für Codex

## 28. Milestone 0 — Repo verstehen

Codex:

1. `AGENTS.md` lesen.
2. `.agents/skills/social-roleplay-mvp/SKILL.md` lesen.
3. GitHub Issue #1 lesen.
4. dieses Dokument lesen.
5. aktuellen Flow und Deploymentstruktur prüfen.
6. `/plan` erstellen.

**Kein Code ändern, bevor Plan + Risiken dokumentiert sind.**

Abnahme:

- Architekturplan,
- offene Fragen,
- Migrationsplan,
- Testplan.

---

## 29. Milestone 1 — Engine Types + Template Validator

Bauen:

- TypeScript-Typen,
- Template-Schema,
- Validator,
- State/Phase-Enums,
- Beispieltemplate als Testfixture.

Abnahme:

- `max_players > 5` wird abgelehnt,
- ungültige Transitions werden abgelehnt,
- Deep ohne Skip wird abgelehnt,
- alle Tests grün.

---

## 30. Milestone 2 — Persistenz + RLS

Bauen:

- Datenbankmigrationen,
- RLS,
- Testnutzer,
- private Payloads.

Abnahme:

- Two-user negative tests grün,
- keine private Cross-Read-Möglichkeit,
- keine Room-Link-Cross-Read-Möglichkeit.

---

## 31. Milestone 3 — Pool + Matching

Bauen:

- Join/Leave,
- Hard Filters,
- Group Builder,
- Soft Score,
- Repeat Penalty,
- Fairness,
- atomare Assignments.

Abnahme:

- 20 simulierte Spieler,
- keine Doppelzuweisung,
- nie >5,
- bevorzugt 3–4,
- lange Wartende werden nicht dauerhaft verdrängt.

---

## 32. Milestone 4 — Run Engine / State Machine

Bauen:

- `startRun`,
- `submitAction`,
- `advance`,
- `reveal`,
- `finish`,
- Audit Log,
- Recovery.

Abnahme:

- Reload mitten in Runde funktioniert,
- doppelte Requests idempotent,
- ungültiger State-Sprung blockiert.

---

## 33. Milestone 5 — Vier erste Spieltemplates

Nicht vier Varianten desselben Spiels.

### Template 1 — Story Web
3–4 Spieler

### Template 2 — Secret Signal
3–5 Spieler

### Template 3 — Cooperative Mission
3–5 Spieler

### Template 4 — Social Calibration + optional REAL
3–5 Spieler

Abnahme pro Template:

- eigene Inhalte,
- Schema valide,
- 3/4/5er Verhalten getestet,
- kein Spieler ohne Aufgabe,
- kompletter Run automatisiert testbar.

---

## 34. Milestone 6 — Game Master UI

Bauen:

- Shared View,
- Private View,
- Timer,
- Ready,
- Reveal,
- Skip,
- Reconnect.

Abnahme:

- Mobil gut bedienbar,
- private Informationen nicht im DOM/Netzwerkpayload anderer Spieler,
- klar erkennbar, was als Nächstes zu tun ist.

---

## 35. Milestone 7 — WhatsApp Room Routing ohne Browserautomation

Bauen:

- Room Config,
- geschützte Invite-Links,
- Consent-Screen,
- `I joined`,
- Ready-State.

Mit Dummy-/manuell hinterlegten Links testen.

**Noch keine WhatsApp-Gruppen automatisch erzeugen.**

---

## 36. Milestone 8 — Browser-Provisioning

Erst nach Nutzerfreigabe.

Codex erzeugt die vereinbarten 4 Rooms, nicht mehr.

Abnahme:

- jeder Room existiert,
- Name korrekt,
- Settings geprüft,
- Invite-Link korrekt im Backend,
- Readback dokumentiert,
- keine Kontakte massenhaft hinzugefügt.

---

## 37. Milestone 9 — echter Pilot

Ziel:

```text
10–20 Personen
1 Stadt
3–5 Personen je Session
4 Rooms
4 Templates
```

Erst danach entscheiden:

- mehr Content,
- bessere Matching-Gewichte,
- Monetarisierung,
- Gift-/Date-/Travel-Packs,
- zusätzliche Städte.

---

# TEIL N — Beispiel eines vollständigen ORIGINALEN Testtemplates

## 38. `lost_package_001` — nur interne Testfixture

**Nicht automatisch LIVE stellen.**

```yaml
id: lost_package_001
version: 1
status: internal_test
name: "Das falsche Paket"
player_count:
  min: 3
  preferred: 4
  max: 5
estimated_minutes: 18
complexity: 2
modes: [mystery, roleplay]
intensity: light
requires_whatsapp_group: true
requires_private_roles: true
elimination: false
repeat_after_days: 21

roles:
  - id: receiver
    public_trait: "Das Paket wurde dir zugestellt."
    private_goal: "Finde heraus, für wen es wirklich gedacht war."
  - id: organizer
    public_trait: "Du kennst den Treffpunkt."
    private_goal: "Verhindere, dass die Gruppe zu früh den Ort wechselt."
  - id: witness
    public_trait: "Du hast etwas gesehen."
    private_goal: "Gib deinen wichtigsten Hinweis erst nach der ersten Abstimmung."
  - id: wildcard
    optional_for_players: [4,5]
    public_trait: "Du bist zufällig hineingeraten."
    private_goal: "Bringe mindestens zwei andere dazu, ihre erste Vermutung zu ändern."
  - id: courier
    optional_for_players: [5]
    public_trait: "Du weißt, wie die Lieferung normalerweise läuft."
    private_goal: "Schütze eine Information bis zum Twist."

phases:
  - id: intro
    type: public_prompt
    text: "Ein Paket liegt am vereinbarten Ort. Name und Inhalt passen zu niemandem vollständig. Ihr habt 15 Minuten, bevor es abgeholt wird."

  - id: private
    type: private_prompt
    payload: role_payload
    wait_for: all_acknowledged

  - id: discuss_1
    type: discussion
    timer_seconds: 240
    constraint: "Niemand darf sein geheimes Ziel wörtlich nennen."

  - id: vote_1
    type: vote
    question: "Für wen war das Paket wahrscheinlich gedacht?"
    visibility: secret_until_all

  - id: twist
    type: reveal
    text_ref: twist_variant_pool

  - id: discuss_2
    type: discussion
    timer_seconds: 300

  - id: final_choice
    type: choice
    question: "Was macht ihr mit dem Paket?"
    options_ref: ending_choices

  - id: real_bridge
    type: reflection
    optional: true
    intensity: real_1
    text: "Wann hast du zuletzt deine Meinung geändert, weil jemand einen guten Punkt hatte?"

end:
  type: aftermath
  text_ref: aftermath_variant_pool
```

### Warum dieses Template als Fixture geeignet ist

Es testet gleichzeitig:

- 3/4/5 Spieler,
- optionale Rollen,
- private Ziele,
- Timer,
- Diskussion,
- geheime Abstimmung,
- Reveal,
- optionalen REAL-Übergang,
- unterschiedliche Enden,
- keine Eliminierung.

---

# TEIL O — Was Codex selbst herausfinden darf und was nicht

## 39. Darf Codex selbst entscheiden

- konkrete Framework-/Dateistruktur innerhalb des bestehenden Repos,
- geeignete Libraries,
- Migrationssyntax,
- Testframework-Details,
- Implementierungsdetails für State Machine,
- UI-Komponentenstruktur,
- Performanceoptimierung,
- interne Helper.

## 40. Darf Codex NICHT eigenmächtig ändern

- Maximal 5 Spieler,
- 18+,
- WhatsApp = Kommunikationsraum, App = Game Master,
- keine Telefonnummern im Backend,
- keine Massenzugabe/Scraping,
- Deep optional,
- keine privaten Cross-Reads,
- keine Live-Szene bevor Pool/Room-Zuweisung vorhanden,
- kein WhatsApp-Provisioning ohne Approval,
- keine proprietären Spieltexte kopieren,
- kein `DONE` ohne Tests + Readback.

---

# TEIL P — Definition of Done der Engine V1

## 41. V1 ist fertig, wenn

- Engine 3, 4 und 5 Spieler korrekt unterstützt,
- niemals >5 zugewiesen werden,
- vier mechanisch unterschiedliche Templates funktionieren,
- neue Templates ohne Engine-Codeänderung hinzugefügt werden können,
- private/public State sauber getrennt ist,
- State Machine server-authoritativ ist,
- Runs nach Reload fortgesetzt werden können,
- Matching + Template-Auswahl nachvollziehbar ist,
- Repeat-Vermeidung funktioniert,
- RLS/Cross-user-Tests grün sind,
- WhatsApp Room-Link nur zugewiesenen Spielern gezeigt wird,
- kompletter E2E-Durchlauf funktioniert,
- bestehender Character-Flow nicht regressiert,
- Produktion nicht mit unfertigen Spielmodi belastet wird.

---

# Quellen / Referenzen

## Spiele und Mechaniken

- Just One / Spiel des Jahres 2019: https://www.spiel-des-jahres.de/just-one-ist-das-spiel-des-jahres-2019/
- Repos Production — Just One: https://www.rprod.com/de/games/just-one
- Codenames / Spiel des Jahres 2016: https://www.spiel-des-jahres.de/erjahr/2016/
- Codenames Rules: https://czechgames.com/files/rules/codenames-rules-en.pdf
- Die Crew / Kennerspiel des Jahres 2020: https://www.spiel-des-jahres.de/spiel-des-jahres-2020-kennerspiel-des-jahres/
- Die Crew Rules: https://thamesandkosmos.com/manuals/full/691868_Crew_Manual.pdf
- The Mind: https://pandasaurusgames.com/products/the-mind
- Fiasco: https://bullypulpitgames.com/products/fiasco
- Fiasco DIY License: https://bullypulpitgames.com/pages/fiasco-diy-license
- Alice is Missing: https://www.huntersentertainment.com/alice-is-missing
- For the Queen: https://darringtonpress.com/forthequeen/
- Decrypto: https://www.scorpionmasque.com/en/decrypto
- Decrypto official shop / awards claim: https://shop.scorpionmasque.com/products/decrypto
- Wavelength: https://www.cmyk.games/products/wavelength
- Fun Facts: https://www.rprod.com/en/press/fun-facts

## Game-Design-Rahmen

- MDA: https://aaai.org/papers/ws04-04-001-mda-a-formal-approach-to-game-design-and-game-research/

## Backend / Realtime

- Supabase RLS: https://supabase.com/docs/guides/database/postgres/row-level-security
- Supabase Realtime: https://supabase.com/docs/guides/realtime
- Supabase Realtime Broadcast / DB Changes: https://supabase.com/docs/guides/realtime/subscribing-to-database-changes
- Supabase Realtime Authorization: https://supabase.com/docs/guides/realtime/authorization
- Vercel Functions: https://vercel.com/docs/functions

---

## Kurzfassung für Codex

```text
Baue keine Sammlung zufälliger Szenen.
Baue eine Engine aus wiederverwendbaren Mechanik-Primitiven.

Standardgruppe: 3–4
Maximum: 5

Engine =
  Matching
+ Template Registry
+ State Machine
+ private/public information
+ role assignment
+ timers
+ actions/votes/guesses
+ reveals
+ recovery
+ audit log
+ safety controls

Content = Daten, nicht neue Engine-Branches.

Inspiration aus etablierten Spielen nur auf Mechanikebene.
Keine proprietären Texte kopieren.

WhatsApp ist V1 nur Kommunikationsraum.
Kein Browser-Provisioning ohne Approval.
```