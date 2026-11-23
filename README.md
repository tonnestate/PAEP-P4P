# PAEP / P4P

**Ein eigenständiges, herstellerunabhängiges Kommunikationsprotokoll für Menschen, Organisationen und ihre KI-Agenten.**

**Status:** Produkt- und Protokollkonzept v0.1, 23. November 2026. Noch kein lauffähiger Server, kein bereitgestellter Tunnel, keine geprüfte Ende-zu-Ende-Verschlüsselung und keine nachgewiesene Integration in ChatGPT, Claude oder Copilot.

## Idee in einem Satz

Zwei Teilnehmer erhalten jeweils einen dynamisch abrufbaren **JSON-Kommunikationsendpunkt**. PAEP verbindet diese Endpunkte über autorisierte, zweckgebundene Leases zu einem Kommunikationsraum. Menschen und ihre bevollmächtigten Agenten können dort mit getrennten Rechten kommunizieren; ein Transportadapter (zunächst n8n als geplanter Relay-Kandidat) befördert verschlüsselte Nachrichten, ohne die Klartexte verarbeiten zu müssen.

**P4P** bezeichnet das Kommunikationsmodell mit vier möglichen Rollen: Mensch A, Agent A, Mensch B, Agent B. Nicht alle vier Rollen müssen gleichzeitig besetzt oder online sein. Eine Seite kann auch eine Organisation mit menschlichem Vertreter und Agent sein.

## Produktprinzipien

- **Eigenständiges Produkt:** Kein Teil von PumpkinMe oder FruitStack; diese können PAEP/P4P später als Clients beziehungsweise Adapter nutzen. Auch BlueberryMe ist keine zwingende Laufzeitabhängigkeit.
- **JSON zuerst:** Keine Pflicht, pro Nutzer eine Webseite anzulegen. JSON-Ansichten werden aus minimalen persistenten Zuständen dynamisch erzeugt. Eine menschliche Chatansicht bleibt optional.
- **Weg über das offene Web:** Öffentlich auffindbare Anleitung und maschinenlesbare Discovery; die Fähigkeit eines konkreten Agenten, Aktionen auszuführen, hängt von seinem Host und dessen Freigaben ab.
- **Kein MCP-Zwang:** HTTPS/JSON, freigegebene CLI/Code-Mode-Aktionen und kontrollierter JSON-Dateiaustausch nutzen denselben Vertrag. MCP und Agent Skills sind optionale Adapter beziehungsweise Anleitungen.
- **Getrennte Autorität:** Mensch und Agent besitzen getrennte Berechtigungen; ein Agent kann keine menschliche Zustimmung oder Identitätsfreigabe ersetzen.
- **Vertrauliche Kommunikation:** Nachrichten werden vor dem Transport clientseitig Ende-zu-Ende-verschlüsselt. Der Dienst sieht weiterhin technische Metadaten. E2EE-Sicherheit setzt vertrauenswürdige Clients und unabhängig überprüfbare Schlüsselbindungen voraus.
- **Sparsamer Betrieb:** Keine Kopien von Profilen oder Nachrichten-Klartexten als Datenhaltung. GitHub ist Quellcodehost, kein operatives Ledger.
- **Keine eigene Kryptografie erfinden:** Etablierte, überprüfte Verfahren und Bibliotheken verwenden.

## Minimales Ablaufmodell

1. Ein Mensch beauftragt einen vorhandenen Agenten, beispielsweise eine neue berufliche Möglichkeit zu suchen.
2. Der Agent entdeckt eine öffentliche PAEP-Anleitung und verwendet – **nur sofern seine Laufzeit es erlaubt und der Mensch zustimmt** – den freigegebenen Zugang.
3. PAEP erzeugt einen Endpunkt mit öffentlicher Kennung, minimalem Zustand und gesonderten Berechtigungen. Private Schlüssel bleiben in vertrauenswürdigen Clients.
4. Ein anderer Endpunkt wird gefunden oder ausdrücklich adressiert. Eine Verbindung wird beantragt und von berechtigten Seiten akzeptiert.
5. Ein widerrufbarer Lease autorisiert die Teilnahme an einem Kommunikationsraum.
6. Endpunkte tauschen geprüfte öffentliche Schlüssel und verschlüsselte Nachrichten über den konfigurierten Transport aus.
7. Menschen können über eine optionale Oberfläche mitlesen und antworten, **wenn sie im jeweiligen Nachrichtenkontext als Empfänger autorisiert sind**.

## Begriffe

- **PAEP:** vorläufiger Eigenname für das Austauschprotokoll; die bisherige projektgebundene Langform ist **nicht** die Definition dieses eigenständigen Produkts.
- **P4P:** Arbeitsbezeichnung für das Menschen-und-Agenten-Kommunikationsmodell mit vier möglichen Rollen.
- **Endpoint:** adressierbarer Kommunikationsendpunkt mit ID, öffentlichen Schlüsselinformationen und Berechtigungszustand; keine zwingende HTML-Seite.
- **Room:** freigegebene Beziehung zweier Endpunkte; speichert Teilnehmer, Leases, Ereignisstände und Chiffrate.
- **Lease:** überprüfte, zeitlich und funktional begrenzte Berechtigung, nicht selbst ein Verschlüsselungsschlüssel.
- **Relay:** Transport für Chiffrat und minimale Zustellmetadaten; darf nicht die Schlüsselhoheit erhalten.

## Dokumentation

- [Produktkonzept und Abgrenzung](docs/PRODUCT_CONCEPT.md)
- [PAEP-Protokollentwurf und JSON-Vertrag](docs/PROTOCOL.md)
- [Sicherheits- und Vertrauensmodell](docs/SECURITY.md)
- [Umsetzungsplan und messbare Abnahme](docs/MVP.md)
- [Getroffene Entscheidungen und offene Punkte](docs/DECISIONS.md)

## Technische Richtung

**TypeScript** für Protokollkern und JSON-API, **DuckDB** als sparsamer Persistenzkandidat für den ersten eigenständigen Dienst, **n8n** als austauschbarer Relay-Adapter. Die Software muss lokal ohne externe KI und ohne MCP testbar sein. Keine GitHub Actions/CI als Voraussetzung.

## Grenzen

Dieses Repository beschreibt zunächst ein Zielbild. Keine fertige Verschlüsselung, produktive Sicherheit, unabhängige Protokoll-Neuheit, gesetzliche Konformität, automatische ChatGPT-Entdeckung oder garantierte Plattformkompatibilität wird behauptet.

Lizenz: [MIT](LICENSE).
