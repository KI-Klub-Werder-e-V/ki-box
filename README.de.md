# KI-Box: privater KI-Arbeitsplatz auf Deutsch für einen PC mit 16 GB RAM ohne GPU

[English](README.md) · Deutsch

**Die KI-Box ist ein geplanter Installer, der mit einem Befehl aus einem kleinen Linux-Rechner mit 16 GB RAM, ohne Grafikkarte und mit etwa sechs Jahre alter CPU einen privaten KI-Arbeitsplatz macht: Chat, deutsche Dokumentensuche mit Quellen, Nutzerkonten und verschlüsseltes Backup.** Sie bündelt bestehende Komponenten (Ollama, Open WebUI, Caddy, restic) in einem getesteten Profil für wenig Speicher. Die KI-Box ist ein Projekt des gemeinnützigen [KI Klub Werder e.V.](https://ki-klub-werder.de) aus Werder (Havel).

> **Stand November 2026: nur Spezifikation.** Es gibt noch keinen Installer, und auf der Zielhardware ist noch nichts gemessen. Die vollständige Planung steht in [`docs/spec.de.md`](docs/spec.de.md).

## Steckbrief

| | |
|---|---|
| **Hardware** | Linux-PC, 16 GB RAM, keine GPU, CPU etwa 6 Jahre alt mit AVX2, SSD |
| **Betriebssystem** | Ubuntu Server 24.04 LTS |
| **Für wen** | KI-Enthusiasten mit einer gemeinsamen Box zu Hause, im Verein oder im kleinen Betrieb |
| **Netz** | Nur lokal, in v1 kein Zugriff aus dem Internet |
| **Sprache** | Chat und Dokumentensuche auf Deutsch |
| **Lizenz** | Apache-2.0 für den eigenen Code, Komponenten unter ihren eigenen Lizenzen |
| **Cloud** | Im Betrieb keine, Internet nur für Installation und Updates |

## Geplanter Funktionsumfang

- **Installation mit einem Befehl**, nach Abbruch gefahrlos wiederholbar
- **Lokaler Chat** mit einem kleinen offenen Modell, ausgewählt per deutschem Testsatz
- **Deutsche Dokumentensuche** mit Hybridsuche und Quellenangabe
- **OCR bei Bedarf** für gescannte PDFs, einmalig statt als Dauerdienst
- **Nutzerkonten und Gruppen** mit Dokumentenrechten je Gruppe
- **Verschlüsseltes Backup jede Nacht** mit getestetem Restore
- **Sichere Updates:** feste Versionen, Backup davor, Smoke-Test danach
- **OpenAI-kompatible API** im lokalen Netz für eigene Experimente

## Häufige Fragen

### Läuft ein brauchbares lokales Sprachmodell auf 16 GB RAM ohne Grafikkarte?

Ja, mit Grenzen. Ein kleines quantisiertes Modell passt in 16 GB, die Ausgabe ohne GPU ist aber langsam. Fremdberichte nennen für einen Intel N100 etwa 6–9 Token pro Sekunde bei einem 8B-Modell. Zu erwarten ist eine aktive Anfrage zur Zeit. Das passt für Enthusiasten, nicht für ein Team mit Bedarf an Sofortantworten. Eigene Messwerte folgen vor dem Release.

### Welches Modell ist auf einem kleinen PC am besten für Deutsch?

Noch offen. Gemma 4 E4B, Gemma 4 E2B und Qwen 3.5 4B treten gegen einen deutschen Testsatz aus echten kommunalen Satzungen an. Die Ergebnisse werden veröffentlicht.

### Warum braucht die deutsche Dokumentensuche eine eigene Einrichtung?

Zwei Voreinstellungen von Open WebUI passen nicht zu deutschen Texten. Das Embedding-Modell `all-MiniLM-L6-v2` ist englisch ausgerichtet. Die Stichwortsuche trennt nur an Leerzeichen, sodass „Hundesteuer,“ und „Hundesteuer“ verschiedene Begriffe sind und Komposita wie „Hundesteuersatzung“ nicht zerlegt werden (geprüft in v0.13.3). Die KI-Box stellt auf BGE-M3 um und misst das Stichwortproblem.

### Was unterscheidet die KI-Box von Harbor, Open WebUI oder AnythingLLM?

[Harbor](https://github.com/av/harbor) startet viele verschiedene lokale LLM-Dienste und ist bereits verfügbar. Wer ausprobieren will, nimmt heute Harbor. Die KI-Box soll genau ein getestetes Profil für 16 GB ohne GPU liefern, mit deutscher Suche, Backup und Restore. Ob sie auf Harbor aufbaut, wird im [ersten Meilenstein](docs/spec.de.md#meilensteine) entschieden.

### Verlassen Daten den Rechner?

Planmäßig nicht. Das Image von Open WebUI schaltet Telemetrie ab, die KI-Box setzt den Offline-Modus und schaltet die Websuche aus. Die Abnahme prüft das per Mitschnitt des ausgehenden Verkehrs. Lokaler Betrieb allein macht niemanden DSGVO-konform; Betreibende bleiben Verantwortliche.

### Ist die KI-Box Open Source?

Der eigene Code steht unter Apache-2.0. Open WebUI nutzt eine eigene, nicht OSI-anerkannte Lizenz, die das Entfernen des Brandings oberhalb von 50 Nutzenden einschränkt. Die KI-Box verwendet das offizielle Image unverändert.

## Mitmachen

Am meisten hilft gerade ein Messwert: CPU-Modell, RAM, Modell und Token pro Sekunde. Bitte als [Issue](https://github.com/KI-Klub-Werder-e-V/ki-box/issues) einstellen.

## Lizenz

[Apache-2.0](LICENSE). Ein Projekt des [KI Klub Werder e.V.](https://ki-klub-werder.de)

*Angaben zuletzt geprüft: 9. November 2026.*
