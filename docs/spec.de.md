# KI-Box – Spezifikation v1

**Status:** Entwurf vom 9. November 2026. Nichts davon ist gebaut oder auf der Zielhardware gemessen.

## Kurzfassung

Ein Befehl macht aus einem Linux-Rechner mit 16 GB RAM, ohne GPU und mit etwa sechs Jahre alter CPU einen privaten KI-Arbeitsplatz im lokalen Netz: Chat, Suche in eigenen Dokumenten, Nutzerkonten, verschlüsseltes Backup. Das Projekt schreibt nur Installer, Konfiguration, Update-Weg, Tests und Anleitung.

**Das eigentliche Produkt ist die Pflege:** getestete Versionsstände, ein sicherer Update-Weg und ein nachweislich funktionierendes Restore.

## Ziel und Abgrenzung

**Für wen:** KI-Enthusiasten, die eine gemeinsame Box betreiben. Sie kommen mit der Kommandozeile zurecht, tauschen gern Modelle und akzeptieren Wartezeiten.

**Erfolg heißt:** Vom leeren Rechner zur ersten belegten Antwort aus eigenen Dokumenten mit einem Befehl, stabil ohne Speicherabstürze, und nach einem Plattenschaden sind Konten, Chats und Dokumente wieder da.

**In v1:**
- Installation auf Ubuntu Server 24.04 LTS, nur im lokalen Netz
- Chat auf Deutsch mit einem vorausgewählten, austauschbaren Modell
- Dokumentensuche mit Quellenangabe, Scans per OCR-Befehl
- Lokale Konten und Gruppen
- Backup, Restore, Update, Status und OCR über ein kleines CLI
- OpenAI-kompatible API im LAN

**Nicht in v1:** Zugriff aus dem Internet, öffentliche Chatbots, Transkription, Websuche, Cloud-Modelle, eigene Admin-Oberfläche, GPU-Profile, weitere Betriebssysteme, SSO.

## Zielhardware

| | Minimum |
|---|---|
| RAM | 16 GB |
| CPU | x86-64 mit AVX2, etwa 6 Jahre alt |
| GPU | keine |
| Speicher | SSD, mindestens 256 GB |
| Backup-Ziel | USB-Platte oder NAS |

Erwartung: einstellige Token pro Sekunde und eine aktive Anfrage zur Zeit. Der Installer zeigt das vorab an und bricht ohne AVX2 oder bei weniger als 16 GB ab.

## Bausteine

| Aufgabe | Komponente | Lizenz | Anmerkung |
|---|---|---|---|
| Modelle | Ollama | MIT | Nur im internen Docker-Netz |
| Chat, Konten, Suche | Open WebUI (geprüft: v0.13.3) | eigene Lizenz | Vektordatenbank ist enthalten |
| OCR | OCRmyPDF | MPL-2.0 | Einmaliger Container, belegt danach keinen RAM |
| HTTPS | Caddy | Apache-2.0 | Einziger Dienst mit offenen Ports |
| Backup | restic | BSD-2-Clause | Verschlüsselt |
| Embeddings | BGE-M3 | MIT | Über Ollama |

Bewusst nicht dabei: Docling als Dauerdienst, Reranker und Qdrant, weil sie auf 16 GB mit dem Chat-Modell um Speicher konkurrieren.

**Eigener Code (Apache-2.0):** `install.sh`, `compose.yml`, CLI `ki-box`, Smoke-Test, Anleitung.

**Lizenz von Open WebUI:** BSD-3-artig mit Branding-Klausel, nicht OSI-anerkannt. Das Branding darf nur entfernt werden bei höchstens 50 Endnutzenden in 30 Tagen, mit schriftlicher Erlaubnis oder mit Enterprise-Lizenz. Die Box nutzt das offizielle Image unverändert. Ausweichoption ist AnythingLLM (MIT), dann müssten Chats und Konten migriert werden.

## Speicherbudget

| Posten | Speicher | Grundlage |
|---|---|---|
| Linux und Docker | ca. 1,5 GB | Schätzung |
| Open WebUI | ca. 1,5 GB | Schätzung |
| BGE-M3 | 1,2 GB | Modellgröße Ollama |
| `gemma4:e4b-it-qat` | 6,1 GB | Modellgröße Ollama |
| Kontext, Cache, Reserve | ca. 5,7 GB | Rest |

Modellgrößen sind Downloadgrößen; der echte Bedarf liegt höher und wächst mit der Kontextlänge. Darum:

- `OLLAMA_CONTEXT_LENGTH=8192`, denn die mögliche Kontextlänge der Modelle von 128.000 Token würde den Speicher sprengen
- `OLLAMA_NUM_PARALLEL=1`, weitere Anfragen warten
- Fester Speicherdeckel pro Container, damit ein Überlauf nicht das System reißt
- Ausweichmodelle: `gemma4:e2b-it-qat` (4,3 GB), `qwen3.5:4b` (3,3 GB)
- Beim Indexieren größerer Bestände wird das Chat-Modell entladen

## Deutsche Dokumentensuche

Die Standardwerte von Open WebUI passen nicht für deutsche Texte:

1. **Embedding:** Voreingestellt ist das englisch ausgerichtete `all-MiniLM-L6-v2`. Die Box setzt `bge-m3` über Ollama. Ein Modellwechsel erfordert ein neues Indexieren aller Dokumente.
2. **Stichwortsuche:** Die Hybridsuche zerlegt Text nur an Leerzeichen (`text.split()`). „Hundesteuer,“ und „Hundesteuer“ gelten als verschiedene Wörter, Komposita werden nicht zerlegt. Das wird gemessen. Reicht es nicht, wird zuerst `enable_hybrid_search_enriched_texts` eingeschaltet, danach ein Patch upstream vorgeschlagen.

Antworten nennen immer die Quelle. Das Standard-Prompt verlangt ein klares „steht nicht in den Dokumenten“, wenn nichts gefunden wurde.

## Installation

```bash
# Geplant, noch kein Release vorhanden
curl -fsSL https://github.com/KI-Klub-Werder-e-V/ki-box/releases/download/v1.0.0/install.sh | sudo bash
```

Die URL ist auf eine Version festgelegt, nie auf `main`.

**Ablauf** (mehrfach ausführbar):
1. Prüfen: Betriebssystem, RAM, AVX2, freier Platz, Port 443, Festplattenverschlüsselung
2. Docker installieren
3. Geheimnisse erzeugen, `.env` mit Rechten `600`
4. Images nach festem Digest und Modelle laden
5. Dienste nur auf localhost starten
6. Admin-Konto anlegen, Registrierung schließen, erst dann Caddy freigeben
7. Backup-Ziel einrichten, restic-Passwort einmalig anzeigen
8. Smoke-Test, Adresse `https://ki-box.local` anzeigen

**Sicherheitsfestlegungen:**
- `ENABLE_SIGNUP` steht standardmäßig auf `True`, und die erste Registrierung wird Admin. Deshalb ist die Reihenfolge in Schritt 6 fest.
- Nur Caddy veröffentlicht Ports, weil Docker ufw-Regeln umgeht.
- `OFFLINE_MODE=true`, Websuche und Direktverbindungen aus. Telemetrie ist im Image bereits aus.
- `WEBUI_SECRET_KEY` fest gesetzt.
- `ENABLE_ADMIN_CHAT_ACCESS=false`.
- Neue Konten starten als `pending`.
- HTTPS im LAN über Caddys interne CA, mit Anleitung zum Zertifikatsimport.

Viele Open-WebUI-Einstellungen wirken nur beim ersten Start und liegen danach in dessen Datenbank. Spätere Änderungen laufen deshalb über das CLI.

## Betrieb

```
ki-box status            # Dienste, Speicher, letztes Backup, Version
ki-box backup            # sofort sichern
ki-box restore [datum]   # Stand wiederherstellen
ki-box update            # Backup, neue Version, Smoke-Test
ki-box ocr <datei.pdf>   # Scan mit deutscher Textebene versehen
```

**Backup:** Nachts per systemd-Timer. Open WebUI wird kurz gestoppt, damit Datenbank, Vektorindex und Uploads zueinander passen. Gesichert werden Datenvolumes, `.env` und Caddy-CA, nicht die Modelle. Aufbewahrung: 7 Tage, 4 Wochen, 6 Monate.

**Update:** Immer manuell, immer mit Backup davor. Datenbankmigrationen sind nicht rückwärts sicher, Zurückrollen heißt daher alte Images plus Restore. Sicherheitsupdates des Betriebssystems über `unattended-upgrades`.

**Pflege:** Quartalsweise ein getesteter Release, bei Sicherheitslücken sofort. Renovate schlägt Versionen vor, der Smoke-Test entscheidet.

## Datenschutz (Eckpunkte, keine Rechtsberatung)

- Die betreibende Person oder Organisation bleibt Verantwortliche. Lokal heißt nicht DSGVO-frei.
- Gelöschtes bleibt bis zu 6 Monate in Backups; das gehört ins Löschkonzept, oder die Aufbewahrung wird gekürzt.
- Festplattenverschlüsselung lässt sich nicht per Skript nachrüsten. Der Installer warnt, die Anleitung verlangt sie bei personenbezogenen Daten schon bei der Ubuntu-Installation. Nach jedem Neustart muss dann jemand das Passwort eingeben.

## Anschluss an andere Projekte

**MarieKondoForAgents:** In v1 von Hand: erst aufräumen, dann freigegebene Dokumente hochladen. Die direkte MCP-Anbindung folgt später, denn `konmari serve --http` verlangt einen OAuth-2.1-Autorisierungsserver und der STDIO-Modus läuft mit einer festen Identität, was Rechte pro Person aushebeln würde.

**Weitere Projekte:** Die Box bietet eine OpenAI-kompatible API im LAN. Auf 16 GB ohne GPU ist sie für Einzelanfragen gedacht, nicht als Backend für öffentliche Dienste oder Transkription.

## Meilensteine

| | Inhalt | Fertig, wenn |
|---|---|---|
| **M0 Klären** | Verantwortliche Person, erster Nutzer, Harbor prüfen | Entscheidung „auf Harbor aufbauen oder eigenständig“ ist dokumentiert |
| **M1 Messen** | `compose.yml` von Hand auf 16-GB-Rechner, Testsatz | Modell, Kontextlänge und Suche anhand von Messwerten festgelegt |
| **M2 Installer und Betrieb** | `install.sh`, CLI, Backup, Restore, Update | Alle Abnahmekriterien bestanden |
| **M3 Pilot** | 4 Wochen echte Nutzung | Release v1.0.0 mit veröffentlichten Messwerten |

**Testsatz M1:** 20–30 Fragen auf öffentlichen Satzungen der Stadt Werder (Havel), davon mindestens 5 mit Komposita und 3 auf Scans, außerdem 5 Fragen ohne Antwort in den Dokumenten.

## Abnahmekriterien

Schwellen sind Vorschläge und werden nach M1 bestätigt.

1. Frische Ubuntu-Installation bis zur ersten Antwort mit einem Befehl.
2. Zweiter Installer-Lauf ändert und zerstört nichts.
3. Portscan zeigt nur 80 und 443; ohne Internet funktionieren Chat und Suche.
4. Mitschnitt zeigt keinen ausgehenden Verkehr im Betrieb.
5. Ohne Konto kein Zugriff; Gruppe A findet keine Dokumente von Gruppe B.
6. Restore auf einem zweiten, leeren Rechner bringt Konten, Chats, Dokumente und Suche zurück.
7. Fehlgeschlagenes Update lässt sich auf den vorherigen Stand zurückführen.
8. Testsatz: mindestens 75 % richtig mit korrekter Quelle, kein erfundener Betrag, bei allen 5 Fragen ohne Antwort ein ehrliches „steht nicht drin“.
9. Auf der Zielhardware: mindestens 5 Token/s, erstes Token einer Dokumentenfrage nach unter 60 Sekunden, 24 Stunden Dauerbetrieb mit gleichzeitigem Indexieren ohne Speicherabsturz.

## Offene Entscheidungen

1. Wer verantwortet die Pflege, und wer ist der erste Nutzer?
2. Auf Harbor aufbauen oder eigenständig bleiben?
3. Welches Modell besteht den Testsatz auf 16 GB?
4. Reicht die Stichwortsuche für deutsche Komposita?

## Quellen

Geprüft am 9. November 2026: [Open-WebUI-Lizenz](https://github.com/open-webui/open-webui/blob/main/LICENSE) und Quellcode v0.13.3, [Open-WebUI-MCP-Dokumentation](https://docs.openwebui.com/features/extensibility/mcp/), [MCP-Spezifikation von MarieKondoForAgents](https://github.com/KI-Klub-Werder-e-V/MarieKondoForAgents/tree/main/openspec/changes/c08-connect-agent-interfaces), Lizenzangaben der Upstream-Repositories und Modellkarten, [Modellgrößen bei Ollama](https://ollama.com/library/gemma4/tags).

Sekundär: [Mini-PC-Durchsatz](https://www.mayhemcode.com/2026/02/best-mini-pc-for-ollama-and-local-llms.html).
