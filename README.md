# moove

Umzugsplaner für Dijana & Tobi. Statische Seite, Daten in Supabase,
Änderungen erscheinen beim anderen sofort.

## Aufbau

| Datei        | Zweck |
|--------------|-------|
| `index.html` | die gesamte App, eine Datei |
| `daten.json` | euer aktueller Stand, wird einmalig importiert |

`index.html` enthält keine Inhalte, nur Programmcode — das Repository darf also
öffentlich sein.

## Einrichtung

### 1. Anon Key eintragen

In `index.html` ganz oben im Skriptblock:

```js
var SUPA_URL = "https://upzausewdoeadldfuyav.supabase.co";   // steht schon drin
var SUPA_KEY = "HIER_ANON_KEY_EINFUEGEN";                     // ← ersetzen
```

Der Key steht in Supabase unter Project Settings → API → *anon public*.
Er darf öffentlich sein; geschützt wird der Zugriff durch die Regel in der Datenbank.

Alternativ ohne Codeänderung: Seite öffnen, unten *Projektzugang* ausfüllen —
gilt dann nur für dieses eine Gerät.

### 2. Tabelle in Supabase

SQL Editor, beide E-Mail-Adressen ersetzen, ausführen:

```sql
create table if not exists public.planner (
  id         text primary key,
  data       jsonb not null default '{}'::jsonb,
  rev        text,
  updated_at timestamptz not null default now(),
  updated_by text
);

insert into public.planner (id) values ('umzug')
  on conflict (id) do nothing;

alter table public.planner enable row level security;

create policy "nur wir zwei" on public.planner
  for all to authenticated
  using      (auth.jwt() ->> 'email' in ('tobi@example.com','dijana@example.com'))
  with check (auth.jwt() ->> 'email' in ('tobi@example.com','dijana@example.com'));

alter publication supabase_realtime add table public.planner;
```

Die letzte Zeile schaltet die Live-Übertragung frei. Fehlt sie, wird korrekt
gespeichert, aber ihr seht die Änderungen des anderen erst beim Neuladen.

### 3. Hosten

`index.html` ins Repository, Settings → Pages → Branch `main`, Ordner `/ (root)`.

Auf dem Handy die Seite öffnen und *Zum Home-Bildschirm hinzufügen* — dann läuft
sie ohne Browserleiste wie eine App.

### 4. Daten einspielen

Einmalig, von einer Person: anmelden → Menü (☰ oben rechts) → **Import** →
`daten.json` auswählen. Wird sofort hochgeladen und ist bei beiden da.

## Die fünf Bereiche

**Kalender** — alles mit Datum an einem Ort: Aufgabentermine, Bestellfristen,
Liefer- und Abholtermine, feste Termine. Tag antippen zeigt, was ansteht;
jeder Eintrag lässt sich direkt dort bearbeiten. Darunter die nächsten zwei Wochen.

**Aufgaben** — nur To-dos. Drei Zustände (offen, in Arbeit, erledigt), Filter nach
Person, Überfälliges und Heutiges ganz oben, der Rest nach Bereich. Das Feld
*Wartet auf* ist für Dinge wie „bestellt, kommt am 30.07.".

**Einrichtung** — Entscheidungen pro Raum. Jede Position hat zwei Häkchen: **D**
und **T**. Sobald beide gesetzt sind, springt der Status von *Idee* auf
*Entschieden*. Der Filter **Uneinig** zeigt genau die Positionen, bei denen nur
einer zugestimmt hat — das ist die Liste, über die ihr reden müsst.
Das Notizfeld ist für Anmerkungen aneinander gedacht.

**Einkauf** — dieselben Positionen, nur nach Laden gruppiert und auf das reduziert,
was entschieden und noch nicht da ist. Mit Summe pro Laden. Status antippen
schaltet Entschieden → Bestellt → Da; sobald etwas *Da* ist, verschwindet es
aus der Liste.

**Kosten** — Plan, tatsächlich Bezahltes und der Ausgleich zwischen euch beiden.
Jede Aufgabe und jede Position hat Felder für Betrag und Zahler.

Alle Bereiche arbeiten auf denselben Daten. Ein Möbelstück, das ihr in der
Einrichtung auf *Bestellt* setzt und mit Liefertermin versehen, erscheint
automatisch im Kalender und verschwindet aus dem Einkauf.

## Im Betrieb

- Gespeichert wird gut eine Sekunde nach jeder Änderung.
- Oben rechts steht der Zustand: *live*, *ungespeichert*, *speichert…*, *Konflikt*.
- Ändert ihr zufällig gleichzeitig, kommt *Konflikt* statt eines Sprungs mitten
  im Tippen. Dann im Menü *Neu laden* (andere Fassung gewinnt) oder
  *Jetzt speichern* (eure gewinnt).
- **Export** im Menü zieht eine Sicherungskopie. Die Tabelle hält nur den
  aktuellen Stand, keine Versionsgeschichte.

## Zu beachten

- Ohne Internet läuft nichts. Die Seite ist keine Offline-App.
- Supabase pausiert Projekte im kostenlosen Tarif nach längerer Inaktivität.
  Während des Umzugs kein Thema; die genaue Frist steht im Dashboard.
- Der *service_role key* gehört niemals ins Repository. Nur der anon key.
- Produktbilder: Klick auf das leere Bildfeld versucht, das Bild über
  microlink.io von der Produktseite zu holen. Klappt nicht bei jedem Shop —
  zuverlässiger ist Rechtsklick aufs Produktbild → Bildadresse kopieren →
  im Stift-Menü ins Feld *Bild-URL*.
