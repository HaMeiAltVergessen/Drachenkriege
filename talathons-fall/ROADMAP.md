# Talathons Fall: The Dragonwars — Roadmap

Lebendes Planungsdokument und **Single Source of Truth** für die Projektrichtung.
Wird beim Abarbeiten gepflegt: erledigte Punkte abhaken, Neues ergänzen. Größere Einzelaufgaben
werden bei Bedarf separat geplant, bevor Code entsteht.

**Owner-Legende:** 🤖 Claude (Code/Systeme/Tests/Docs) · 🧑 Sebastian (Assets, kreative
Entscheidungen, Playtest-Feedback, Konten/Hosting, Business) · 🤝 gemeinsam (Design/Balance-Feinschliff).

> Reihenfolge der Meilensteine = empfohlene Bearbeitung. Die Querschnitt-Tracks unten laufen parallel.
> **Erstes großes Ziel: spielbare Demo inkl. Meta-Loop mit der Fraktion Zireal (M0–M6).**

**Schwesterprojekt:** `E:\Fork\Eldmyrdur\eldmyrdur` — **gleiches Universum** (Myrkur ist dieselbe Macht).
Viele Systeme werden von dort kopiert und angepasst (siehe „Übernahme aus Eldmyrdur").

---

## 📌 Design-Fundament (entschieden)

| Bereich | Entscheidung |
|---|---|
| Engine / Darstellung | Godot 4.4, 2D **isometrisch**, `TileMapLayer` + Y-Sorting |
| Renderer | **GL Compatibility** (schwache Android-Geräte, Web-Build als Option offen) |
| Plattform | PC **und** Android parallel, Landscape, Touch-first mit Maus-Parität |
| Repo | Godot-Projekt in `talathons-fall/`, Docs (ROADMAP, STORY, FACTIONS, ASSETS) daneben |
| Content-Format | **`.tres`-Ressourcen** — im Inspector und im Map-/Wellen-Editor bearbeitbar, kein im Code gebautes ContentDB |
| Universum | Gleiche Welt wie Eldmyrdur; Lore-Bezüge und Cameos erlaubt |
| Kernmechanik | **Blocking** wie Arknights: Gegner bleiben am Blocker stehen, fester Pfad, kein Re-Pathing |
| Platzierung | Kachelbasiert auf markierten Feldern, **auch während der Wellen**, kostet Match-Ressource |
| Vorbereitung | 60 s vor Wellenstart; Wellen sind vorziehbar, keine Zeitstopps dazwischen |
| Rückruf | Held kann zurückgezogen und neu gesetzt werden, **behält seine HP** (Redeploy-Cooldown nötig) |
| Held bei 0 HP | Fällt aus, Schaden **bleibt bis zur bezahlten Heilung** in der Basis; Items/Fähigkeiten können heilen |
| Zielregeln | **Klassenabhängig** als Datenfeld (Matrix, siehe M2) — nicht als Sonderfall im Code |
| Ebenen | Eine Ebene, keine Hoch/Tief-Trennung — Schutz von Fernkampf läuft über die Zielregel-Matrix |
| Truppen | Eigene **und** gegnerische Einheiten aus demselben Datensatz; Quelle: Kaserne, Turm oder Heldenfähigkeit |
| Truppen-Zyklus | Kaserne hält einen Trupp, **kostenloser Respawn** nach Cooldown |
| Fraktionen | Zireal, Gol Dargath, Azar spielbar · Myrkur nur Gegner · **Zireal ist der erste Feldzug (Demo)** |
| Helden-Tiers | Champion/Legende/Mythos **sind** die Heldenrarität (Items haben eine eigene Item-Rarität) |
| Helden-Erwerb | Story-Belohnungen **+ Anwerben gegen Ingame-Währung in der Basis** — kein Gacha/Summon |
| Helden-Stufen | Stufe 1–5 **nur über Story-Fortschritt**; Veredelung/Verderbnis als zweite Säule (M4) |
| Gruppe | Helden-Limit **je Map** als Datenfeld (`hero_limit`), nicht global fest |
| Ausrüstung | Nach Eldmyrdur-Schema (Slots, Rarität, Instanzen mit gerollten Werten, Live-Vorschau) + Loot-Drops nach Maps |
| Story-Struktur | Pro Feldzug 3 Akte, **Seitenwahl** am Ende von Akt 2 → zwei Akt-3-Varianten (wie Eldmyrdur) |
| Maps | **24 Layouts** (Akt 1: 9, Akt 2: 8, Akt 3: 7), pro Fraktion neu geskinnt statt neu gebaut |
| Leben | 3 pro Map, als Modifier offen gehalten |
| Idle | Offline-/Idle-Ertrag zwischen den Sessions (Umfang siehe M6) |
| Assets | **Flux-Standbilder** + Nachbearbeitung; Bewegung zuerst per Tween (Bob/Lunge/Flash), Spritesheets optional |
| Monetarisierung | **Free + Werbung/Unlock**: Demo gratis, Vollversion per Einmal-Kauf, optionale Rewarded Ads |
| PvP | Für v1 **gestrichen**, später asynchron gegen gespeicherte Layouts |

**Architektur-Konsequenz aus der Map-Entscheidung:** Ein Level besteht aus zwei getrennten Ressourcen.
`MapLayout` (Pfad, Platzierungsfelder, Turm-Slots, Tileset-Referenz, `hero_limit`) ist fraktionsneutral und wird geteilt,
`WaveSet` (Wellenzusammensetzung, Gegnerfraktion, Belohnungen, optional `hero_limit`-Override) hängt an Kampagne und Akt.
Das Editor-Tool in M5 muss beide separat bearbeitbar machen.

**Architektur-Konsequenz aus der Asset-Entscheidung:** Die Einheiten-Darstellung ist eine eigene Komponente
(`UnitVisual`), die ein Standbild + Tween-Animationen abspielt und automatisch auf `SpriteFrames` umschaltet,
sobald welche hinterlegt sind. So können echte Animationen später einzeln nachgeliefert werden.

## 🔁 Übernahme aus Eldmyrdur

Quelle: `E:\Fork\Eldmyrdur\eldmyrdur`. Strategie: **kopieren und anpassen** (kein gemeinsames Addon).

| System | Quelldatei(en) | Anpassung für Talathon |
|---|---|---|
| Save/Load | `scripts/autoload/save_manager.gd` | 1:1 inkl. Versionierung/Migration; neue Felder: Helden-HP, Map-Fortschritt, Idle-Zeitstempel, Unlock-Flag |
| Szenenwechsel | `scripts/autoload/scene_router.gd` | 1:1 (Fades, Musik je Szene), Szenen-Liste neu |
| Einstellungen | `scripts/autoload/settings.gd` | Landscape statt Portrait, Kamera-/Zoom-Optionen ergänzen |
| Audio | `scripts/autoload/audio_manager.gd`, `audio/default_bus_layout.tres` | 1:1 |
| Dialoge | `scripts/resources/dialog_data.gd`, `dialog_line.gd`, `scripts/ui/dialog_box.gd`, `scenes/dialog/` | 1:1 für Hub-Dialoge, Story und In-Match-Overlays; Layout auf Landscape |
| Lokalisierung | `localization/text.csv`, `tests/loc_test.gd`, `tests/content_loc_test.gd` | Workflow „`[PH]`-Platzhalter → finaler Text" übernehmen; Content-Test auf `.tres` umstellen |
| Roster/Team | `scripts/autoload/game_state.gd` (`owned_ids`, `team_ids`, `set_team`, `get_team`) | `MAX_TEAM` → `hero_limit` der Map; Fortschritt je Fraktion statt je Story-Pfad |
| Ausrüstung | `game_state.gd` (`item_instances`, `equip`, `unequip`, `equipment_bonus`), `scripts/resources/equipment_data.gd` | Stats auf TD-Werte umstellen; **gerollte Werte je Instanz** (in Eldmyrdur als M3.5 offen) nachholen |
| Loot | `scripts/autoload/content_db.gd` (`roll_loot`, `_roll_loot_rarity`) | Tabelle je Akt/Map als Ressource statt Code |
| Progression | `scripts/combat/progression.gd` | Stufen 1–5 aus der Story statt Fusion; Transform-Bonus bleibt |
| Veredelung/Verderbnis | `game_state.gd` (`transform`, `energy_for`, `_grant_chapter_energy`), `content_db.gd` (`transform_ability`) | Energie aus dem gewählten Akt-3-Zweig; Transform-Fähigkeit als TD-Fähigkeit |
| Seitenwahl | `scenes/side_choice/`, `game_state.gd` (`choose_side`, `ch3_id_for_side`) | 1:1 im Prinzip, je Fraktion eigene Texte |
| Helden-Daten | `scripts/resources/character_data.gd` | → `HeroData`: Tier, Klasse, Reichweite (Kachelmuster), Angriffstempo, Blockkapazität, Deploy-Kosten, Redeploy-Cooldown, aktive Fähigkeit |
| Gegner-Daten | `scripts/resources/enemy_data.gd` | → gemeinsamer `UnitData`-Datensatz (eigene Truppen + Gegner), Zielregel-Klasse |
| Juice/FX | `scripts/ui/fx.gd`, `scripts/ui/ui_theme.gd` | Grundlage für Tween-Animationen der Flux-Standbilder + UI-Theme |
| Tests | `tests/scene_harness.gd`, `tests/smoke_test.gd`, `tests/save_test.gd` | Headless-Harness übernehmen, Inhalte neu |
| Doc-Workflows | `STORY.md`, `ASSETS.md`, `MUSIC.md`, `STORY_STATIONS.md` | Aufbau als Vorlage für die Talathon-Docs |

**Nicht übernommen:** kartenbasierter Energiekampf (`scenes/battle/battle.gd`, `scripts/combat/ability_resolver.gd`,
`combatant.gd`), Beschwörung (`summon_manager.gd`, `scenes/summon/`), Splitter-Fusion, im Code gebautes `ContentDB`,
Fraktale/Endlos-Runs (Endlosmodus wird in M9 eigenständig für TD gebaut).

## ❓ Bewusst offene Design-Entscheidungen
*Keine Blocker für den Start — werden im jeweiligen Meilenstein entschieden.*
- 🤝 Match-Ökonomie: Startguthaben, Regenerationsrate, Kosten je Tier, Rückerstattung beim Rückzug, Redeploy-Cooldown. *(M2)*
- 🤝 Truppen-Sammelpunkt: fix am Gebäude oder frei verschiebbar in einem Radius. *(M3)*
- 🤝 Loadout-Zeitpunkt: Truppentyp vor dem Match wählen oder beim Bauen auf der Map. *(M3/M6)*
- 🤝 Turmfähigkeit: manuelle oder automatische Auslösung. *(M3)*
- 🤝 **Sicherheitsventil gegen die Heilungs-Sackgasse**: langsame Gratis-Regeneration, kostenlose Heilung nach Niederlage oder Grundeinnahme. Idle-Ertrag kann das mit abdecken. *(M6)*
- 🤝 Währungen: welche, woraus, wofür (Heilung, **Anwerben**, Stadt-Upgrades, Truppen-Verbesserung, Ritual, Veredelung). *(M6)*
- 🤝 **Anwerben**: Welche Helden sind kaufbar, welche nur über die Story, Preise je Tier. *(M6)*
- 🤝 **Idle-Ökonomie**: Was tickt offline, wie schnell, Soft-Cap. *(M6)*
- 🤝 Ritual: Materialquelle für Verschmelzung, werden Ausgangshelden verbraucht. *(M9)*
- 🤝 Wellen pro Map (Zielkorridor) und Wellen-Archetypen je Akt. *(M5/M7)*
- 🤝 **Demo-Grenze**: Wo endet die Gratis-Version (z. B. nach Akt 1 von Zireal)? *(vor M7)*
- 🤝 Rewarded Ads: Wofür gibt es Belohnungen (Heilung, Idle-Verdopplung, Loot-Reroll) — ohne Pay-to-Win-Gefühl. *(M10)*

---

## 🧱 M0 — Projektfundament
*Ziel: Ein Repo, in dem gebaut werden kann, ohne zweimal anzufangen.*
- [ ] 🤖 `project.godot`: GL Compatibility, Landscape, Stretch-Modus, Main-Scene (Boot), Touch-Emulation per Maus.
- [ ] 🤖 Kopier- und Anpassrunde aus Eldmyrdur: `SaveManager`, `SceneRouter`, `Settings`, `AudioManager`, Dialogsystem, Lokalisierung, `fx.gd`/`ui_theme.gd`, Test-Harness.
- [ ] 🤖 Autoloads: `ContentDB` (lädt `.tres` aus Ordnern), `GameState`, `SaveManager`, `SceneRouter`, `Settings`, `AudioManager`.
- [ ] 🤖 Resource-Klassen: `UnitData`, `HeroData`, `TowerData`, `WaveSet`, `MapLayout`, `EquipmentData`, `DialogData`.
- [ ] 🤖 Save/Load (JSON, versioniert von Anfang an) + Lokalisierung DE mit EN-Spalte.
- [ ] 🤖 Eingabe-Abstraktion (Tap/Klick, Pinch/Scroll, Long-Press/Rechtsklick) statt verstreuter Input-Abfragen.
- [ ] 🤖 Headless-Test-Harness + `CLAUDE.md` mit Projektkonventionen.
- [ ] 🤖 Umzug `neues-spiel/` → `talathons-fall/` committen (nach Freigabe).
- [ ] 🧑 Ordnerstruktur und Namenskonventionen absegnen.
- *Done:* Leere Szene startet auf PC und dem Android-Testgerät, Tests laufen headless grün.

## 🗺️ M1 — Iso-Grid, Kamera, Bewegung
*Ziel: Einheiten laufen sichtbar korrekt über eine isometrische Karte.*
- [ ] 🤝 **Asset-Pipeline festzurren** (vorgezogen): Flux-Standbild je Einheit (1–2 Winkel, gespiegelt = 4 Facings), Bildgröße, Freistellen, Ablagepfade. Referenz-Einheit als Stiltest.
- [ ] 🤖 Isometrisches `TileMapLayer`-Setup, Kachelmaß festlegen, Y-Sorting für Einheiten/Gebäude/Deko.
- [ ] 🤖 Pfad-Definition auf Kachelbasis, Einheiten folgen ohne Navigation-Mesh.
- [ ] 🤖 Kamera: Pan und Zoom für Touch und Maus, Grenzen, Verhalten bei 16:9 bis 20:9.
- [ ] 🤖 `UnitVisual`: Standbild + Tween-Animationen (Laufen-Bob, Angriffs-Lunge, Treffer-Flash, Tod-Fade), Richtungswahl aus Bewegungsvektor; `SpriteFrames` optional.
- [ ] 🤖 Performance-Szene (N Einheiten spawnen, FPS anzeigen) für Messungen.
- [ ] 🧑 Performance auf dem **alten Android-Testgerät** messen und Ergebnis zurückmelden.
- [ ] 🤝 Pfadführung entlang der Iso-Diagonalen als Map-Regel festhalten (sonst zeigen Sprites sichtbar falsch).
- *Done:* Testkarte mit laufender Gegnerkette, sauber sortiert, flüssig auf dem alten Testgerät.

## ⚔️ M2 — Kampfkern (Vertical Slice)
*Ziel: Eine Map, eine Fraktion (Zireal), aber vollständig und schon spaßig.*
- [ ] 🤖 Match-Ressource: Startguthaben, Regeneration, Kosten beim Setzen.
- [ ] 🤖 Helden-Limit je Map (`hero_limit`) in Loadout und Platzierung durchsetzen.
- [ ] 🤖 Blocking-Komponente (geteilt von Helden und eigenen Truppen): Kapazität, Aggro-Zuweisung, Auflösung bei Tod.
- [ ] 🤖 **Zielregel-Matrix** als Datenfeld je Gegnerklasse (Nahkampf → nur Blocker, Fernkampf/Artillerie → alles in Reichweite, Flieger → ignoriert Blocken, Attentäter → ungeschützte Ziele). Vollständigkeit per Test erzwingen.
- [ ] 🤖 Reichweiten als Kachelmuster, Angriff, Schaden, Tod, Trefferfeedback.
- [ ] 🤖 Rückruf mit HP-Erhalt + Redeploy-Cooldown + Kostenregel beim Neusetzen.
- [ ] 🤖 Wellen-Runner: Ankündigung, Vorziehen, Leaks, 3 Leben, Sieg/Niederlage, Tempo 1×/2×.
- [ ] 🤖 Headless-Tests: Blocking-Zuweisung, Zielregeln je Klasse, Leak-Zählung, Rückruf/HP-Persistenz.
- [ ] 🤝 Erste Werte für Kosten, Cooldowns und Blockkapazitäten.
- [ ] 🧑 Playtest auf PC und Handy, Feedback zum Spielgefühl.
- *Done:* Eine Map ist gewinn- und verlierbar, Rotation von Blockern fühlt sich taktisch an und ist nicht degeneriert.

## 🏹 M3 — Türme, Kasernen, Truppen
*Ziel: Die zweite und dritte Entity-Kategorie stehen.*
- [ ] 🤖 Turm-Slots, Turmklassen, aufladende Turmfähigkeit.
- [ ] 🤖 Kaserne: hält einen Trupp, kostenloser Respawn nach Cooldown, Sammelpunkt-Logik.
- [ ] 🤖 Truppen aus Heldenfähigkeiten (zeitlich begrenzt, zählen beim Blocken mit).
- [ ] 🤖 Teamneutraler Einheiten-Datensatz: gleiche Werte und Darstellung, Verhalten je Seite (halten vs. zum Ziel laufen).
- [ ] 🤖 Performance-Budget messen: maximale gleichzeitige Einheiten auf dem alten Testgerät, Budget in `CLAUDE.md` festhalten.
- [ ] 🤝 Truppen- und Turm-Auswahl für Zireal festlegen.
- *Done:* Kaserne + Turm + Held verteidigen gemeinsam, das alte Testgerät hält die Bildrate bei voller Welle.

## 🦸 M4 — Helden-System, Ausrüstung & Loot
*Ziel: Helden sind mehr als Blocker mit HP.*
- [ ] 🤖 Klassen (Nahkampf/Fernkampf/Support + Mischklassen) und Tiers Champion/Legende/Mythos als Heldenrarität.
- [ ] 🤖 Aktive Fähigkeit mit Cooldown pro Held, UI für Auslösung auf Touch und Maus.
- [ ] 🤖 Persistente Helden-HP über Maps hinweg im Save, Heilung über Basis, Items und Fähigkeiten.
- [ ] 🤖 Ausrüstung aus Eldmyrdur übernehmen (Slots, Item-Rarität, Instanzen) + **gerollte Werte je Drop**.
- [ ] 🤖 **Loot-Drops nach Maps** (Tabelle je Akt/Map als Ressource), Anzeige im Sieg-Screen.
- [ ] 🤖 Stufen 1–5, an Story-Fortschritt gekoppelt.
- [ ] 🤖 **Veredelung/Verderbnis** (Eldmyrdur-Transform): Energie verbrauchen → Stat-Bonus + Alignment-Fähigkeit + optisch anderer Zustand.
- [ ] 🤖 Helden-/Ausrüstungs-Screen (Landscape) mit Live-Vorschau.
- [ ] 🤖 Headless-Tests: Stat-Summierung, Equip/Unequip, gerollte Werte, Transform, HP-Persistenz, Save-Roundtrip.
- [ ] 🤝 Fähigkeiten-Fantasy je Klasse und für die Zireal-Helden.
- [ ] 🧑 Zireal-Helden-Roster grob skizzieren (Namen, Klasse, Tier, Fantasy) — Vorlage in `FACTIONS.md`.
- *Done:* Ein Held wird ausgerüstet, verletzt, geheilt, aufgestuft, veredelt — und alles bleibt nach dem Speichern erhalten.

## 🛠️ M5 — Map- und Wellen-Editor
*Ziel: Sebastian baut Maps und Wellen selbst, ohne auf Code zu warten.*
- [ ] 🤖 Editor-Plugin: Pfad zeichnen, Platzierungsfelder und Turm-Slots setzen, `hero_limit` setzen, Layout als `.tres` speichern.
- [ ] 🤖 Wellen-Editor: Gegnertypen, Mengen, Abstände, Pfadzuweisung, Vorschau ohne Spielstart.
- [ ] 🤖 Validierung (Pfad vollständig, Slots erreichbar, Welle spielbar) + Schnelltest-Button.
- [ ] 🤖 Kurzanleitung für das Tool (`docs/EDITOR.md`).
- [ ] 🧑 Erste eigene Map damit bauen als Praxistest des Tools.
- *Done:* Eine komplette Map inklusive Wellen entsteht ohne eine Zeile Code.

## 🏰 M6 — Basis, Stadt & Meta-Loop → **Demo**
*Ziel: Zwischen den Maps passiert etwas — und das Ganze ist als Demo teilbar.*
- [ ] 🤖 Kontinent-Basis für Zireal mit Stadtauswahl (für weitere Fraktionen vorbereitet).
- [ ] 🤖 Heilung, Truppen-Verbesserung, Währungsfluss.
- [ ] 🤖 **Anwerben von Helden** in der Basis gegen Ingame-Währung.
- [ ] 🤖 **Offline-/Idle-Ertrag** (Zeit seit letztem Login → Ressourcen, mit Soft-Cap).
- [ ] 🤖 Loadout-Auswahl vor dem Match (Helden bis `hero_limit`, Truppen, Ausrüstung).
- [ ] 🤖 Held-Dialoge im Hub (Eldmyrdur-Dialogsystem).
- [ ] 🤖 Demo-Build: Windows-Export + Android-APK, Anleitung in `README.md`.
- [ ] 🤝 Ökonomie: Einnahmen pro Map, Preise, Anwerbe-Kosten, Idle-Rate, Sicherheitsventil gegen Sackgassen.
- [ ] 🧑 Erste Maps für die Demo im Editor bauen (Zireal, Akt-1-Anfang).
- [ ] 🧑 Demo an Tester:innen geben, Feedback sammeln.
- *Done:* Niederlage → heilen → verbessern → erneut versuchen fühlt sich fair an. **Demo ist auf PC und Android spielbar.**

## 📖 M7 — Kampagne 1: Zireal (Akt 1–3)
*Ziel: Ein Feldzug vollständig spielbar.*
- [ ] 🧑 24 `MapLayout`-Ressourcen (9/8/7) im Editor bauen (🤖 unterstützt bei Tool-Problemen).
- [ ] 🤖 Wellensets für Akt 1–3, Gegner-Roster Myrkur + eine Feindfraktion.
- [ ] 🤖 Fortschritt, Freischaltungen, Akt-Bosse (Mythos-Gegner als Finale).
- [ ] 🤖 **Seitenwahl** nach dem Akt-2-Boss → zwei Akt-3-Varianten mit eigener Energie (Veredelung/Verderbnis).
- [ ] 🤖 Cutscenes (Dialog + Hintergrund + Musik), Kampagnenkarte mit Stationen.
- [ ] 🤖 Demo-Grenze als Flag im Save, Unlock-Pfad vorbereitet.
- [ ] 🧑 Story-Beats und Dialogtexte (Platzhalter zuerst in `STORY.md` sammeln, dann patchen).
- [ ] 🤝 Balance über den gesamten Akt-Bogen.
- *Done:* Zireal ist von Akt 1 bis zu beiden Akt-3-Enden durchspielbar.

## 🔁 M8 — Kampagnen 2 & 3 (Gol Dargath, Azar)
*Ziel: Die Breite, ohne den Aufwand zu verdreifachen.*
- [ ] 🤖 Reskin-Pipeline: gleiche Layouts, anderes Tileset, andere Wellensets.
- [ ] 🤖 Helden-, Truppen- und Turm-Rosters für die beiden übrigen Fraktionen.
- [ ] 🧑 Assets und Story je Fraktion.
- [ ] 🤝 Fraktionsidentität schärfen: gleiche Rollen, spürbar andere Werkzeuge.
- *Done:* Alle drei Feldzüge spielbar, jeder fühlt sich eigen an.

## 🌌 M9 — Ritual & Endgame
- [ ] 🤖 Verschmelzungs-Ritual (Champion → Legende → Mythos), gegen Story-Ende freigeschaltet.
- [ ] 🤖 Schwellenwelt: endloser Modus gegen Myrkur, Skalierung, Score, Belohnungen.
- [ ] 🤖 Datenmodell so anlegen, dass ein Verteidigungs-Layout später serialisierbar ist (Vorbereitung auf asynchrones PvP).
- *Done:* Nach der Story gibt es mindestens einen Loop, der trägt.

## 🔊 M10 — Audio, Monetarisierung, Release
- [ ] 🧑 Musik und SFX beschaffen (Lizenz beachten), Liste in `MUSIC.md`.
- [ ] 🤖 Audio einbinden (Busse aus Eldmyrdur stehen schon), Lautstärke in den Einstellungen.
- [ ] 🤖 Unlock-Kauf (Google Play Billing) + Rewarded-Ads-Plugin, beides hinter einer Schnittstelle, damit PC ohne läuft.
- [ ] 🤝 Rewarded-Ads-Belohnungen und Preis der Vollversion festlegen.
- [ ] 🤖 Export-Presets PC + Android final, Build-Anleitung im `README.md`.
- [ ] 🧑 Store-Konten (Play Console, ggf. Steam/itch.io), Werbe-Konto, Store-Assets, Texte, Datenschutzerklärung.

---

## 🔄 Querschnitt-Tracks (laufen parallel, nie „fertig")
- **Lore & Story** 🧑 liefert Lore zu Talathon und den Fraktionen; 🤖 legt `STORY.md` (Dialog-Ledger mit CSV-Keys) und `FACTIONS.md` (Fragen + Platzhalter je Fraktion) als Gerüst an und verknüpft die Eldmyrdur-Lore (Myrkur).
- **Assets** 🧑 generiert mit Flux nach `ASSETS.md` (Stilblöcke je Fraktion, wie bei Eldmyrdur); 🤖 schreibt die Prompt-Listen und verdrahtet die Texturfelder. Platzhalter bis dahin.
- **Performance** 🤖 fortlaufendes Budget für gleichzeitige Einheiten, 🧑 misst auf dem alten Testgerät.
- **Balancing** 🤝 bei jedem neuen Inhalt.
- **Tests** 🤖 jede neue Mechanik headless absichern.
- **Lokalisierung EN** 🤖 sobald die DE-Texte stabil sind.

## ⚠️ Hauptrisiken
1. **Content-Menge.** 24 Layouts × 3 Feldzüge sind vor allem hunderte komponierte Wellen. Das Editor-Tool (M5) ist die Gegenmaßnahme, nicht ein Extra.
2. **Mobile-Budget.** Blocking + Kasernen-Trupps + Horden erzeugen viele gleichzeitige Einheiten, und das Testgerät ist schwach. Ab M1 messen, sonst wird spät umgebaut.
3. **Attrition-Sackgasse.** Persistente Helden-HP plus dünne Heldenbank plus knappe Währung kann Spieler feststecken lassen. Anwerben und Idle-Ertrag helfen; das Sicherheitsventil wird in M6 festgelegt.
4. **Asset-Konsistenz.** Flux liefert Standbilder, keine konsistenten Animationen; vier Fraktionen in einem Stil zu halten ist schwer. Gegenmaßnahme: Tween-first-Darstellung, Referenzbilder + Seeds je Fraktion, `SpriteFrames` nur dort, wo es sich lohnt.
5. **Monetarisierungs-Technik.** Werbe- und Billing-Plugins für Godot/Android sind Drittanbieter-Code; früh (spätestens M7) mit einem Prototyp prüfen, dass sie mit Compatibility-Renderer und der Godot-Version laufen.

## Aufgabenteilung auf einen Blick
| | 🤖 Claude | 🧑 Sebastian | 🤝 Gemeinsam |
|---|---|---|---|
| **Fokus** | Code, Systeme, UI, Editor-Tool, Tests, Docs, Builds, Asset-Verdrahtung, Übernahme aus Eldmyrdur | Lore, Assets (Flux), kreative/narrative Entscheidungen, Maps im Editor, Playtest auf dem Handy, Konten, Business | Design- und Balance-Feinschliff, Ökonomie, Fraktionsidentität, Asset-Pipeline |
| **Nächste Schritte** | M0 Fundament (Kopierrunde, Compatibility, Landscape), dann M1 Iso-Grid | Lore-Material liefern, Stil-Referenz für Zireal, Ordnerstruktur absegnen | Asset-Pipeline (Flux → Iso-Standbild), Match-Ökonomie, Zielregel-Matrix |

## 🙋 Wo ich (Claude) deinen Input brauche
| Wann | Was | Form |
|---|---|---|
| M0 | Ordnerstruktur/Namenskonventionen absegnen, Umzug committen lassen | kurzes OK |
| M1 | Referenzbild für den Iso-Stil (eine Zireal-Einheit), Performance-Messung auf dem alten Handy | Bild + Zahlen |
| M2 | Playtest-Feedback zum Blocking-Gefühl, erste Werte gemeinsam | Session/Notizen |
| M3 | Truppen- und Turmtypen für Zireal | Liste in `FACTIONS.md` |
| M4 | Zireal-Helden-Roster (Name, Klasse, Tier, Fähigkeits-Fantasy) | `FACTIONS.md` |
| M6 | Ökonomie-Rahmen (wie „grindy" darf es sein?), Demo-Tester:innen | Gespräch |
| M7 | Story-Beats, Dialogtexte, Seitenwahl-Rahmung, Demo-Grenze | `STORY.md` |
| laufend | Lore zu Talathon, den Fraktionen und dem Bezug zu Eldmyrdur | `FACTIONS.md`/`STORY.md` |
