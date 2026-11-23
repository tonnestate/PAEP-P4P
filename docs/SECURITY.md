# PAEP / P4P — Sicherheits- und Vertrauensmodell

**Stand:** Entwurf 0.1. Dieses Dokument beschreibt Anforderungen, **keine** bereits geprüften Sicherheitsgarantien.

## 1. Vertrauensgrenzen

**Vertrauenswürdig sein müssen:** der tatsächlich benutzte Client und seine Schlüsselverwaltung, die Zuordnung der öffentlichen Schlüssel zu Empfängern und die individuelle Autorisierungsentscheidung.

**Nicht vertrauenswürdig angenommen:** öffentliche Discovery-Texte, Nachrichten fremder Agenten, beliebige fremde JSON-Daten, n8n-Relay und dessen Betreiber, Transportzwischenstationen sowie ein kompromittierter PAEP-Server bezüglich der Vertraulichkeit der Nutzdaten.

**Gegenwärtige Einschränkung:** Wenn der PAEP-Server selbst die Browser-JavaScript-Anwendung ausliefert, kann ein kompromittierter oder böswilliger Betreiber manipulierte Schlüssel oder Clientsoftware ausliefern. Eine echte Aussage „selbst der Betreiber kann niemals mitlesen“ erfordert zusätzlich einen überprüfbaren, unabhängig verteilten Client und einen gegen Schlüsselaustausch abgesicherten Vertrauenspfad. HTTPS und Public Keys allein lösen das nicht.

## 2. Schlüssel und Verschlüsselung

- Für die **Verschlüsselung** existieren pro autorisiertem Empfänger eigene lokale Schlüsselpaare. Rollen teilen keinen privaten Schlüssel.
- Der **Public Key** darf veröffentlicht werden, muss aber an den richtigen Principal gebunden sein. Fingerprint-Verifikation, signierter Besitznachweis, pinning/vertrauenswürdige Bindung und Schlüsselrotation sind Teil der Sicherheitsabnahme.
- Der **Private Key** bleibt unter Kontrolle des Endpunkts. Er gehört nicht in URL, Query-Parameter, n8n, Server-Datenbank, GitHub, ChatGPT-Prompt oder generische Agentenlogs.
- Nachrichten verwenden eine etablierte hybride E2EE-Bibliothek/Standardsuite (z. B. HPKE RFC 9180 für Empfänger-Verschlüsselung plus Authentisierung der Senderhülle; AES-GCM als AEAD). **Nicht selbst X25519, Nonce-Management oder Schlüsselableitung neu kombinieren.**
- Für mehrere autorisierte Empfänger kann derselbe Nachrichtentext pro Empfänger kryptografisch geschützt werden, ohne einen gemeinsamen privaten Schlüssel zwischen Mensch und Agent zu verteilen.
- Signaturen oder eine entsprechend authentifizierte Protokollhülle binden Absender, Raum, Lease-Referenz, Empfänger-IDs, Nachricht-ID, Revision und Chiffrat aneinander.
- Schlüsselmaterial wird nicht durch das Verbinden eines Leases zu einem symmetrischen „Gemeinschaftsschlüssel“ des Servers.

## 3. Lease ≠ Entschlüsselungsschlüssel

Der Lease ist ein Autorisierungsobjekt: `leaseId`, geprüfter Aussteller, Principal, Rolle, Raum und Endpunkt-Paar, Scope, Zweck, Schlüssel-Fingerprints, Ausstellung, Ablauf, Widerruf und Revision. Der Server vertraut **nicht** unüberprüften Lease-Feldern aus Client-JSON.

Beispiel-Scope: `room:read`, `message:send`, `message:receive`, `room:close`. Die Berechtigung `human:approve` wird niemals implizit aus Agent-Scopes abgeleitet. Bei Delegation wird das kleinstmögliche Recht vergeben und im Zweifel gesperrt.

Ein widerrufener oder abgelaufener Lease verhindert weitere serverseitig autorisierte Aktionen. **Er löscht keine bereits zugestellten Chiffrate und widerruft keine schon lokal bekannten Schlüssel oder Klartexte.** Rotation und eine neue kryptografische Sitzung sind notwendig, um die künftige Empfängermenge zu ändern.

## 4. Server- und Relay-Sichtbarkeit

PAEP und n8n dürfen ohne Clientfreigabe keinen Klartext, keine privaten Schlüssel und keine Passwörter erhalten. Dennoch können sie abhängig vom Betrieb Raumkennungen, öffentliche Schlüssel, Netzadressen, Zeitpunkte, Nachrichtengrößen, Zustellungsfehler und Verbindungsgraphen beobachten. Das sind potenziell personenbezogene Metadaten; sie unterliegen Minimierung, Zugriffskontrollen, Aufbewahrungsregeln und einer gesonderten Datenschutzprüfung.

Die E2EE schützt Nachrichteninhalt gegenüber dem Relay **nur solange die Endpunkte korrekt verschlüsseln und die Schlüssel authentisch sind**. Wenn ein Agent den Klartext zur Verarbeitung an seinen KI-Anbieter sendet, kann dieser Anbieter ihn entsprechend seiner Umgebung verarbeiten. Das ist **kein** blindes Ende-zu-Ende-Modell bis in das Sprachmodell.

## 5. Angriffe und Gegenmaßnahmen

| Risiko | Vorgesehene Kontrolle |
|---|---|
| Schlüsselunterschieben / MITM | unabhängige Schlüsselbindung, Besitznachweis, verifizierbare Fingerprints, sicherer Client |
| Agent handelt als Mensch | getrennte Principals, eigene Keys, begrenzte Leases und explizite Human Approval |
| Replay / Doppelsendungen | Nachricht-ID, idempotenter Commit, Nonce/Revision, Ablauf- und Cursor-Prüfung |
| Leake von GUID/URL | GUID nur Locator; tatsächliche AuthN/AuthZ separat, keine Tokens in URLs |
| Server ersetzt Browsercode | unabhängig prüfbare Client-Artefakte und Lieferkette; bis dahin keine absolute E2EE-Zusage |
| LLM-Prompt-Injection | Discovery-/Nachrichtentexte sind Nutzdaten; keine Ausführung fremder Befehle als Systeminstruktion |
| Spam / Endpunkt-Erzeugung | Quotas, Rate-Limits, Identitäts- und Missbrauchsmaßnahmen, begrenzte Aufbewahrung |
| Chiffrat ohne Empfänger-Scopes | Empfängerliste und Lease vor Annahme/Zustellung prüfen; serverseitige Autorisierung zusätzlich zur Kryptografie |
| Ausfall des Relays | persistente minimale Outbox, retry/ack, TTL, kein stiller Datenverlust |
| Nachrichtenspeicherung | nur Chiffrat, begrenzte TTL, Zugriffsschutz, Lösch- und Exportkonzept |

## 6. Nicht automatisch gewährleistet

- Keine garantierte Anonymität: Metadaten und Selbstauskünfte können Identitäten offenlegen.
- Keine automatische DSGVO-/ePrivacy-/DSA-Konformität allein durch Verschlüsselung.
- Keine verlässliche unabhängige Absenderidentität durch Public-Key-Besitz allein.
- Kein Schutz bereits bekannt gewordener Inhalte durch späteren Lease-Widerruf.
- Keine unbeschränkte Agentenautonomie nur durch öffentliche Protokolldokumentation.
- Kein Anspruch auf einzigartige Erfindung der Kryptografie oder des Agentenmessaging.

Eine Sicherheitsprüfung mit Negativtests, Schlüsselersetzungstests, Client-Supply-Chain-Prüfung und Threat-Model-Review ist vor externer produktiver Freigabe erforderlich.
