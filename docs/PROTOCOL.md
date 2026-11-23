# PAEP — Protokollentwurf für P4P

**Stand:** Entwurf 0.1, 2026-11-23. **Nicht implementiert.** Die beschriebenen Pfade und Nachrichtentypen sind ein erster Vertrag zur Prüfung, keine registrierten Internetstandards oder produktive URLs.

## 1. Schichten

1. **Discovery**: beschreibt Versionen, Operationen und erlaubte Transportbindungen öffentlich.
2. **Resource Model**: Endpunkt, Akteur, Raum, Lease, Schlüsselbindung, Nachricht, Änderungs-Cursor.
3. **Authority**: überprüft Identität/Delegation, Rollen, Umfang, Ablaufdatum, Widerruf und Versionsbedingungen.
4. **Messaging**: Nachrichtenhülle und opakes Chiffrat; vertrauliche Nutzdaten werden nur am autorisierten Client verarbeitet.
5. **Transport**: zunächst HTTPS/JSON; derselbe Bedeutungsvertrag für CLI, kontrollierten Dateiimport und optional MCP.
6. **Presentation**: optionaler Menschenchat über dieselben Zustände; keine eigene Protokolllogik.

Eine GUID bezeichnet eine Ressource und ist **niemals** allein ein Authentifizierungsmerkmal.

## 2. Entitäten

| Entität | Minimale Bedeutung |
|---|---|
| `endpointId` | opake zufällige Endpunktkennung; öffentlich höchstens ohne vertraulichen Zustand |
| `principalId` | authentifizierter Mensch, organisatorischer Vertreter oder delegierter Agent |
| `role` | `human`, `organization_human`, `agent`; tatsächliche Scope-Prüfung separat |
| `roomId` | Kommunikationsraum, der genau zwei Seiten/Endpunkte verknüpft (mehrere berechtigte Akteure möglich) |
| `leaseId` | zeitlich begrenzte Zuordnung von Berechtigungen zu Raum, Akteur, Zweck und Revision |
| `keyId` | Fingerprint/Referenz eines überprüften öffentlichen Empfängerschlüssels; keine private Schlüsselablage |
| `messageId` | zufällige deduplizierbare Nachrichtenkennung |
| `cursor` | raumgebundene, monotone Änderungsposition; kein global lesbarer Aktivitätsfeed |

Der Client verwahrt private Schlüssel. Eine veröffentlichte Key-ID bestätigt allein **noch nicht** die Identität des Schlüsselinhabers.

## 3. Operationen (plattformunabhängig)

| Aktion | Zweck | Sicherheit |
|---|---|---|
| `DISCOVER` | öffentlich verfügbare Protokollinformationen lesen | kein privater Zustand |
| `ENDPOINT_CREATE` | eigener Endpunkt für authentifizierten Principal | gedrosselt, kein impliziter Agentenzugriff |
| `KEY_REGISTER` | öffentlichen Schlüssel und Besitznachweis hinterlegen | signierter Besitznachweis, Bindung verifizieren |
| `DELEGATE` | Agenten Rechte einräumen | nur legitimierter Mensch/Organisationsvertreter |
| `ROOM_PROPOSE` | zweitem Endpunkt eine Verbindung vorschlagen | nur berechtigter Principal |
| `ROOM_ACCEPT` | Verbindung bestätigen und scoped Lease ausstellen | beide Seiten/Policies müssen zustimmen |
| `MESSAGE_SEND` | verschlüsselte Nachricht an definierte Empfänger übermitteln | Lease, Senderauthentizität, Replay-Schutz |
| `CHANGES_READ` | neue autorisierte Ereignisse nach Cursor abrufen | Empfangsberechtigung, Pagination/Quotas |
| `MESSAGE_ACK` | Zustell-/Abrufstatus melden | keine Aussage über menschliches Lesen |
| `LEASE_REVOKE` | künftigen Zugriff beenden | widerrufsberechtigter Principal/Authority |
| `KEY_ROTATE` | neue Schlüsselbindung mit Nachweis etablieren | keine stillschweigende Substitution |

**Keine Operation `IDENTITY_REVEAL` im generischen Kern.** Anwendungen dürfen zusätzliche menschliche Freigabeprozesse anknüpfen; deren Rechte dürfen nicht aus einer Agent-Delegation abgeleitet werden.

## 4. Discovery über reguläre Webseiten

- `GET /for-agents`: öffentlich indexierbare, erklärende HTML-Einstiegsseite als Auffindbarkeitshilfe; keine private Endpunktseite.
- `GET /.well-known/paep.json`: vorgeschlagene maschinenlesbare Discovery-Adresse. Der Name ist **noch kein registrierter Well-Known-URI-Standard**.
- Discovery liefert Protokollversionen, Schemas/Verträge, erlaubte Transports, dokumentierte Grenzen und die öffentlichen API-Pfade. Keine Geheimnisse, persönlichen Endpunktinhalte oder privaten Schlüssel.

Das Anzeigen oder Lesen dieser Informationen ist keine Einwilligung zu Uploads. Ein Agent benötigt Host-seitig tatsächlich freigegebene HTTP-, Browser-, Datei- oder CLI-Aktionen.

## 5. Vorläufige HTTPS-Bindings

| Methode / Pfad | Aktion |
|---|---|
| `POST /v1/endpoints` | ENDPOINT_CREATE |
| `GET /v1/endpoints/{endpointId}` | autorisierte dynamische JSON-Ansicht |
| `POST /v1/endpoints/{endpointId}/keys` | KEY_REGISTER / KEY_ROTATE |
| `POST /v1/endpoints/{endpointId}/delegations` | DELEGATE |
| `POST /v1/rooms` | ROOM_PROPOSE |
| `POST /v1/rooms/{roomId}/accept` | ROOM_ACCEPT |
| `GET /v1/rooms/{roomId}/changes?after={cursor}` | CHANGES_READ |
| `POST /v1/rooms/{roomId}/messages` | MESSAGE_SEND |
| `POST /v1/rooms/{roomId}/messages/{messageId}/ack` | MESSAGE_ACK |
| `POST /v1/leases/{leaseId}/revoke` | LEASE_REVOKE |

Alle privaten Operationen benötigen **transportgebundene, überprüfte Authentifizierung und Autorisierung**. Es darf keine als Geheimnis missbrauchbare ID oder ein Token in einer öffentlichen URL oder in einem protokollierten JSON-Payload stehen.

## 6. Beispiel einer dynamischen, authentifizierten Endpunktansicht

~~~json
{
  "protocol": "paep",
  "version": "0.1-draft",
  "endpointId": "ep-example-a",
  "revision": 2,
  "keys": [
    {"keyId": "key-human-a", "purpose": "encrypt", "verified": true},
    {"keyId": "key-agent-a", "purpose": "encrypt", "verified": true}
  ],
  "rooms": [
    {"roomId": "room-example", "state": "active", "cursor": "12"}
  ]
}
~~~

Dies ist ein **synthetisches Schema-Beispiel**, nicht die endgültige JSON-Schema-Datei. Eine reale Response würde nur die gemäß Principal/Rechten erlaubten Schlüssel- und Raumdaten zurückgeben. Öffentliche Schlüssel werden separat als überprüfbare kryptografische Daten geliefert; `verified` allein ersetzt keine unabhängige Schlüsselauthentifizierung.

## 7. Nachrichtenhülle (synthetisches Beispiel)

~~~json
{
  "protocol": "paep",
  "version": "0.1-draft",
  "type": "MESSAGE_SEND",
  "messageId": "7ae5eb52-4729-4cf1-9da6-d804a7960dbc",
  "roomId": "room-example",
  "sender": "agent-a",
  "recipientKeyIds": ["key-human-b"],
  "leaseId": "lease-example",
  "expectedRevision": 7,
  "crypto": {
    "format": "hpke-aead-envelope-draft",
    "ciphertext": "<base64url-encrypted-payload>",
    "keyEncapsulations": "<recipient-specific-key-material>",
    "senderProof": "<signed-authenticated-envelope>"
  }
}
~~~

Werte zwischen spitzen Klammern sind Platzhalter, **keine lauffähige Kryptografie**. Die finale Verschlüsselungssuite, der Signaturumfang, die kanonische Serialisierung, Nonce-Regeln und das wire-kompatible JSON Schema sind vor der Implementierung verbindlich festzulegen und durch Vektortests abzusichern. Die `leaseId` ist eine Referenz, nicht der Zugriffsnachweis.

**Kritisch:** Die Empfängerliste bestimmt, welche Rollen Klartext erhalten dürfen. Mensch und Agent besitzen **getrennte Schlüssel**. Agenten sehen menschlich private Nachrichten nur, wenn sie explizit als Empfänger beziehungsweise autorisierte Verarbeitungspartner festgelegt wurden.

## 8. Zustandsautomaten (Zielentwurf)

- Endpunkt: `CREATED → ACTIVE → SUSPENDED / CLOSED`.
- Raum: `PROPOSED → ACCEPTED → ACTIVE → REVOKED / CLOSED`. Vor `ACTIVE` ist `MESSAGE_SEND` verboten.
- Lease: `ISSUED → ACTIVE → EXPIRED / REVOKED`; kein Wiederaktivieren eines widerrufenen Lease.
- Nachricht: `RECEIVED → QUEUED → DELIVERED/EXPIRED`; quittierter Empfang bedeutet **nicht**, dass ein Mensch gelesen oder zugestimmt hat.

Jede zustandsändernde Anfrage benötigt idempotente Behandlung, erwartete Revision und rechtmäßigen Rollen-/Scope-Nachweis. Unbekannte Felder und ungeprüfte Versionen werden abgelehnt.

## 9. Synchronisierung und Fehler

- Monotone, raumbezogene Cursors; inkrementelle Änderungen, keine vollständige Profil- oder Chatreplikation bei jedem Abruf.
- Nachrichten-IDs für Deduplizierung, TTL und begrenzte Retries; bei Verbindungsabbrüchen sicher wiederholbar.
- `401` fehlende/ungültige Authentifizierung; `403` fehlende Autorität; `404` nicht sichtbare Ressource; `409` Versionskonflikt; `410` abgelaufene/gelöschte Ressource; `429` Rate-Limit.
- Dasselbe fachliche Fehlerobjekt gilt für CLI und Datei-Transport, unabhängig vom HTTP-Code.
- Signierte/offline importierte Nachrichten müssen dieselben Rechte- und Replay-Prüfungen bestehen wie Live-HTTP.

## 10. Protokollgrenzen

PAEP transportiert keine allgemeinen LLM-Prompts als ausführbare Systemanweisungen. Sichtbare Nachrichten und Discovery-Texte werden als **untrusted content** behandelt. Das Protokoll löst weder selbst Agentenaktionen aus noch erstellt es ohne Host-Unterstützung ChatGPT-Aufgaben, Push-Nachrichten oder Hintergrundsitzungen.

Für die erste Version bleibt die Raumbindung bei **zwei Endpunkten**; spätere Mehrparteien-/Gerätemodelle erfordern ein eigenständiges, geprüftes Gruppen-Schlüsselmanagement.
