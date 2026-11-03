# Übergabe: Goose Run (Gänse-Browserspiel)

Stand: 3. November 2026 · Für die Weiterarbeit mit einem anderen Modell

## Worum es geht

Ein kleines Arcade-Browserspiel „Goose Run“: Man steuert eine Gans am Himmel/Feld entlang, weicht Hindernissen aus und sammelt Gänseblumen. Es ist als **Goose-App** (HTML-Fenster) entstanden und zusätzlich als eigenständiges Web-Projekt auf GitHub veröffentlicht.

## Wo alles liegt

| Ort | Was |
|---|---|
| `~/Documents/Goose/goose-run/` | **Arbeitskopie / Quelle der Wahrheit.** Git-Repo, Branch `main`, Remote `origin` = `https://github.com/DerFJ28/goose-run.git` |
| `~/Documents/Goose/goose-run/index.html` | das komplette Spiel, ~1.890 Zeilen, eine Datei (HTML+CSS+JS) |
| `~/Documents/Goose/goose-run/README.md` | öffentliche Beschreibung (Spielprinzip, Steuerung, Technik) |
| `~/Documents/Goose/goose-run/screenshot.png` | Spielszene, im README verlinkt |
| `~/.local/share/goose/apps/goose-run.html` | **Goose-App-Kopie** der HTML-Datei (wird über das `apps`-Extension-Tool `iterate_app`/`list_apps` verwaltet) |

Achtung: Es gibt **zwei** Kopien der HTML-Datei. Änderungen in der Goose-App landen nicht automatisch im Repo und umgekehrt. Nach jeder Änderung die andere Kopie angleichen (`cp index.html ~/.local/share/goose/apps/goose-run.html`) oder bewusst festlegen, dass nur noch das Repo gepflegt wird. Ebenso existiert in `~/.local/share/goose/apps/` noch eine ältere Datei `goose-runner.html` – **ignorieren, nicht verwechseln**, sie gehört nicht zu diesem Projekt.

## Git- und GitHub-Situation

- Repo `DerFJ28/goose-run` ist **öffentlich**, per GitHub-API angelegt.
- Letzter Commit: `Goose Run: Gänse-Arcade als einzelne HTML-Datei` (4 Dateien: `index.html`, `README.md`, `screenshot.png`, `.gitignore`).
- Stand lokal = Stand auf GitHub (`## main...origin/main`, nichts offen). Zu Beginn der nächsten Session trotzdem `git status -sb` und `git pull` prüfen; neue Dateien explizit `git add`.
- **Push-Zugang:** Es gibt hier keine `gh`-CLI. Der Push läuft über die Git-Credentials des Kontos `DerFJ28` (im Klon `~/Documents/Goose/bounty-code` per `git credential fill` hinterlegt und im macOS-Schlüsselbund gespeichert). Das funktioniert für dieses Repo automatisch.
- Der Token **darf nicht** ausgegeben, geloggt, angezeigt oder in andere Projekte kopiert werden. Bei 401/403: nicht umgehen, sondern Frederick bitten, Markus um einen neuen Token zu bitten.
- GitHub-API-Aufrufe (nur wenn nötig) mit `curl -H "Authorization: token $TOKEN" https://api.github.com/...`, dabei Token aus `git credential fill` holen und **nie ausdrucken**.
- Nichts Geheimes committen.

## GitHub Pages – offener Punkt

Damit man das Spiel direkt im Browser spielen kann, fehlt noch: *Repo → Settings → Pages → Source: „Deploy from a branch“ → Branch `main`, Ordner `/ (root)` → Save*. Danach: `https://derfj28.github.io/goose-run/`.
**Wichtig:** GitHub Pages für öffentliche Repos erfordert je nach Kontoeinstellung Plan/Verifizierung; wenn die Aktivierung scheitert (z. B. 422 „Pages is not available for this account“), kann man es auch per API versuchen (`POST /repos/DerFJ28/goose-run/pages`, Body `{"source":{"branch":"main","path":"/"}}`) – nur auf ausdrücklichen Wunsch von Frederick. Die Einstellung wurde in dieser Session bewusst nicht angefasst (Frederick wollte selbst klicken bzw. erst gefragt werden).

## Das Spiel: Aufbau des Codes (wichtige Stellen in `index.html`)

- **Konstanten** ab Zeile ~312: `BASE_W = 900`, `BASE_H = 700`, `GROUND_Y = 596`, `MAX_DT = 0.05` (50-ms-Deckel), `FLOWER_PER_LIFE = 5` (Bonus), `MAX_LIVES = 5`, `INVULN_TIME = 1.5`.
- **Canvas/Skalierung:** `resize()` passt die Bühne proportional an das Fenster an, mit `devicePixelRatio` (max. 2). Canvas innen immer 900×700 logische Pixel.
- **`AudioEngine`** (~358): synthetische WebAudio-Sounds, `AudioEngine.muted` schaltet stumm (Lautsprecherknopf im HUD).
- **Zustände:** `STATE = {START, PLAY, PAUSE, OVER}`, Variable `state`. Der Game-Loop (`loop`, ~1795) aktualisiert nur in `PLAY`, im Startbildschirm nur Hintergrund/Partikel; in Pause/Ende nur Screen-Shake und Blitzeffekt.
- **Zentrale Spielobjekte:** `game` (Punkte, Leben, Combo, Power-up, Shake, Flash …), `goose`, `obstacles`, `flowersArr`, `powerups`, `particles`, `floatTexts`, Hintergründe (`clouds`, `hillsFar`, `hillsNear`, `trees`, `birdsBg`).
- **Spawnen:** `spawnObstacle()` wählt zwischen `spawnGate()` (Tor mit Lücke), `spawnHay()` (Heuballen), `spawnGull()` (Möwe); `laneFreeY(minGap, excludeY, radius)` sorgt dafür, dass Lücken groß genug bleiben (Ziel: keine unlösbaren Muster). Zusätzlich `spawnFlower()` und `spawnPowerup(kind)` (Korn/Feder/Magnet).
- **Steuerung:** Tasten (`keys`, Arrow/WASD/Space, `P`/`Esc` Pause, `R` Neustart) plus Zeiger (`pointer`, Maus/Touch) mit gedämpfter Verfolgung in `updateGoose(dt)`.
- **Kollision:** `checkCollisions()` → `damageGoose()` (Leben, Unverwundbarkeit mit Blinken, Shake, Blitz), `gameOver()`.
- **Sammeln:** `collectFlower()` (Combo bis ×5, Punkte-Popup, alle 5 Blumen Bonus), `applyPowerup(kind)` (nur eines gleichzeitig).
- **Rendering:** `drawSky/drawCloud/drawHills/drawGround/drawGate/drawHay/drawGull/drawFlower/drawPowerup/drawGoose/drawHUD` + `render()`.
- **Overlays:** HTML-Elemente `startScreen`, `pauseScreen`, `overScreen`, `infoScreen` mit Buttons; Highscore in `localStorage` unter `gooseRun.best`.
- **Robustheit:** `window.onerror` fängt Fehler ab, der Game-Loop ist mit try/catch umgeben, damit die Schleife nie stirbt.

## Was bereits geprüft wurde

- JS-Syntax per `node --check` auf dem `<script>`-Block: OK.
- Alle `getElementById`-Referenzen haben passende Elemente im HTML; kein externes Script/CDN, keine Netzwerk-Requests.
- Echte Runde im **Headless-Chrome** über das DevTools-Protokoll gestartet, ~12 s mit Tastatur/Maus gespielt: **keine Konsolenmeldungen, keine Exceptions**.

## Bekannte Lücken / mögliche nächste Schritte

1. **Spielgefühl ist von Menschen noch nicht bewertet.** Frederick sollte Rückmeldung geben zu Tempo, Lückengröße, Steuerpräzision (zu träge/flott), Schweregrad. Danach gezielt an Konstanten schrauben: Startgeschwindigkeit, Beschleunigung in `update()`, Lückengröße in `spawnGate()`.
2. GitHub Pages aktivieren (siehe oben), optional eine kleine Landingpage/Bilder.
3. **Leertaste = Flattern: geprüft und bestätigt (3.11.2026).** Headless-Test (Chrome-CDP gegen eine Testkopie mit `__test`-Hook): Ohne Space bleibt die Gans in der Schwebe (0 px Steigung über 0,5 s), mit gehaltener Space steigt sie ~179 px (mehrere Flügelschläge à −330 vy, Cooldown 0,14 s). Der Boost ist also ein echter Mechanismus, keine Deko. Ob die Stärke (~7 Aufwärtsimpulse/Sekunde) sich gut anfühlt, bleibt Fredericks Spielgefühl-Frage (siehe Punkt 1).
4. Mögliche Erweiterungen: täglicher Highscore-Vergleich, Schwierigkeitsmodi, Soundtrack, „Endless-Zonen“ mit Themen (Nacht, Regen).
5. Wartung: README bei größeren Änderungen aktualisieren, `screenshot.png` neu aufnehmen.

## Arbeitsweise / Regeln für die nächste Session

- Zuerst `cd ~/Documents/Goose/goose-run && git status -sb && git pull`.
- Änderungen in `index.html` machen, dann prüfen: Script-Block mit `node --check` extrahieren (Block `<script>` ohne `type`), optional Smoke-Test mit Headless-Chrome (DevTools-Protokoll, `--remote-debugging-port`), dann committen und **sofort nach GitHub pushen**.
- Der Google-Chrome-Pfad ist `/Applications/Google Chrome.app/Contents/MacOS/Google Chrome`; `node` kommt aus der Hermit-Umgebung von Goose und funktioniert normal.
- Nach dem Push muss der gepushte Stand dem dargestellten Stand entsprechen (README, Screenshot, funktionierende Datei).
- Kein Deploy auf den Minecraft-Server – das Spiel hat damit nichts zu tun. Die Regeln aus `.goosehints` zum Server gelten hier nicht, aber die Git-Regeln (jede Änderung committen und pushen) schon.
