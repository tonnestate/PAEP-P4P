# PAEP / P4P — erster Umsetzungsschritt und Abnahme

**Status:** geplanter Prototyp; kein Code als umgesetzt behauptet.

## 1. Ziel des ersten funktionsfähigen Nachweises

**Zwei unabhängig gestartete Clients verbinden zwei dynamische JSON-Endpunkte ohne MCP, PumpkinMe oder fremde LLM-API.** Ein Mensch und ein delegierter Agent können auf jeder Seite unterschiedliche Rechte besitzen. Eine Nachricht wird lokal verschlüsselt, über den Relay-Kanal ausgetauscht, nur am richtigen Empfänger entschlüsselt und nach Widerruf dürfen keine neuen Nachrichten über den widerrufenen Lease zugestellt werden.

Das ist bewusst kleiner als ein vollständiges Agentennetzwerk. Erst danach werden echte KI-Umgebungen integriert.

## 2. Technische Richtung

- **TypeScript:** eine schlanke, typisierte API und ein geteiltes Protokoll-/Clientmodul. Kein zusätzlicher Flask-/Python-Dienst.
- **DuckDB:** minimaler lokaler Zustand für Endpunkte, Schlüsselreferenzen, Rechte, Leases, Räume, Chiffrate und Zustellungs-Cursors. Keine separate Graphdatenbank und kein externes Konto als Betriebsvoraussetzung.
- **n8n:** geplanter, austauschbarer Transportadapter für Chiffrat; ein lokaler Zustellweg muss denselben Vertrag testbar machen. Kein ungeschützter Webhook darf Berechtigungsprüfungen umgehen.
- **Browser/CLI:** zuerst einfache, prüfbare Clients. Menschliche Oberfläche und Agentenadapter greifen auf denselben Kern zu.
- **Kryptografie:** etablierte, geprüfte Implementierung verwenden; genaue Algorithmen/Suite und Schlüsselspeicherung vor Implementierung verbindlich auswählen und testen.
- **Betrieb:** lokal nachvollziehbare Befehle und Tests, **keine GitHub Actions/CI**. Keine teure Pflichtinfrastruktur.

## 3. Umsetzung in prüfbaren Schritten

1. **Vertrag festlegen:** JSON-Schema 0.1, Fehlerformat, Versionierung, Zustandsübergänge, öffentliche Discovery-Daten und nichtöffentliche Operationen; vollständige Negativschemas.
2. **Kernmodell:** Endpunkt, Principal, Agentendelegation, Schlüsselbindung, Raum, Lease und atomare Revision in DuckDB; getrennte Ressource-IDs und Rechte.
3. **Lokaler Ende-zu-Ende-Test:** Zwei Clients erzeugen ihre eigenen Schlüssel; kontrollierte Verifikation; Verschlüsselung, Transport eines opaken Chiffrats und Entschlüsselung nur beim autorisierten Empfänger.
4. **Nachrichtenbetrieb:** deduplizierbare Nachrichten, Polling/Cursor, Acknowledgement, Ablauf, Wiederanlauf und gesicherte Outbox.
5. **Relay-Adapter:** n8n transportiert ausschließlich authentifizierte, autorisierte Chiffrate/Metadaten; kein Verzicht auf Serverprüfungen und keine Klartext-Transformation.
6. **Vier-Rollen-Nachweis:** Menschen- und Agentenidentitäten getrennt, Delegation, Antwort Mensch→Agent und Agent→Mensch; Mensch-only-Nachrichten bleiben Agenten verschlossen.
7. **Discovery/Interoperabilität:** öffentliche Agent-Anleitung, dynamische JSON-Sicht, ein CLI/Code-Mode-Beispiel ohne MCP, optionaler Dateiaustausch nach denselben Regeln.
8. **Externer Agentenversuch:** mit tatsächlich erlaubtem ChatGPT-Work-, Claude-Code- oder Copilot-Zugriff. Funktionen **pro Host** dokumentieren, nicht voraussetzen.

## 4. Verbindliche Abnahmeszenarien

| ID | Prüffall | Erwartung |
|---|---|---|
| A01 | Leerer Endpunkt, ohne Auth eine private JSON-Ansicht abrufen | verweigert, keine Informationen |
| A02 | Zwei unabhängige Endpunkte erstellen | verschiedene GUIDs, eigene Principal- und Key-Bindungen |
| A03 | Raum beantragen, nur eine Seite bestätigt | kein aktiver Kanal |
| A04 | Beide Seiten bestätigen | raumgebundener, zeitlich begrenzter Lease |
| A05 | Client A verschlüsselt an B | Relay/Server können keinen Klartext rekonstruieren, sofern Client und Schlüssel echt sind |
| A06 | Eine Rolle ohne Empfängerfreigabe versucht entschlüsseln | kein Zugriff auf Nachrichtenschlüssel |
| A07 | Agent versucht menschliche Zustimmung zu ersetzen | Autorisierung scheitert |
| A08 | Replay derselben Nachricht/Revision | idempotent; keine zweite Zustellung |
| A09 | Lease läuft ab / wird widerrufen | keine neuen autorisierten Transporte; vorhandene Kopien werden nicht fälschlich als gelöscht behauptet |
| A10 | Client offline, später Abruf | Nachricht einmalig über Cursor/Outbox sichtbar |
| A11 | n8n nicht verfügbar | kontrollierter Retry/Fehler; Daten bleiben konsistent |
| A12 | Fremder ersetzt registrierten Public Key | Verifikationspfad erkennt oder blockiert Austausch |
| A13 | Generierter JSON-Status ändert sich | nur Revision/Delta geändert; keine unnötige Profilkopie |
| A14 | API/CLI ohne MCP | identisches Berechtigungsverhalten |

**Abnahmegrenze:** Ein lokaler Demo-Erfolg beweist noch keine Produktionssicherheit. Vor öffentlichen E2EE-Zusagen müssen Browsercode-/Clientvertrauen und unabhängige Schlüsselprüfung belastbar gelöst sein.

## 5. Nicht Teil dieses ersten Meilensteins

Keine Jobbörse, kein Matching, keine bezahlten Unternehmensfunktionen, kein zweites Datenbanksystem, keine Echtzeitgarantie, keine Push-Benachrichtigung durch ChatGPT, kein großflächiges Crawling, keine automatische Registrierung bei Drittplattformen, keine Massengruppenverschlüsselung und keine CI-Pipelines.

## 6. Beobachtbare Ergebnisse

Ein lokales, reproduzierbares Ergebnisprotokoll soll zeigen: zwei Endpunkte, zwei validierte Rollenpaare, eindeutige Room-/Lease-Revisionen, die Zustellreihenfolge, erfolgreich entschlüsselte Nachricht nur am erlaubten Empfänger und die genannten Negativfälle. Keine Beispielschlüssel oder echten privaten Personendaten im Repository.
