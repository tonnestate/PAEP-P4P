# PAEP / P4P — eigenständiges Produktkonzept

**Version:** Konzept v0.1, 2026-11-23  
**Geltung:** Eigenständiger Kommunikationsdienst. PumpkinMe ist der erste denkbare Anwendungsfall, aber keine Architekturvoraussetzung.

## 1. Problem und Lösung

Menschen und Unternehmen verwenden unterschiedliche KI-Assistenten. Ein gemeinsamer Agenten- und Nachrichtenraum darf weder eine bestimmte KI-Plattform noch MCP noch eine installierte native App voraussetzen.

PAEP definiert einen **dynamischen, strukturierten JSON-Kommunikationsendpunkt** und dessen Operationen. P4P erweitert die direkte Teilnehmerbeziehung um autorisierte Vertreter: zwei menschliche/organisatorische Seiten und ihre jeweiligen Agenten. Der Dienst **vermittelt Endpunkte und Berechtigungen**, betreibt aber nicht die KI-Agenten selbst.

Die frühere Idee individueller Webseiten wird vereinfacht: **Eine Webseite ist eine optionale Ansicht des Endpunkts, nicht der Endpunkt selbst.** Der Kommunikationsraum entsteht aus zwei Endpunktreferenzen, genehmigten Leases, kryptografisch verifizierbaren Empfängerschlüsseln und einem Nachrichtentransport.

## 2. Zwei Seiten, vier mögliche Rollen

| Seite A | Seite B | Prinzip |
|---|---|---|
| Mensch A / Organisation A | Mensch B / Organisation B | menschliche Entscheidung und Verantwortung |
| Agent A (bevollmächtigt) | Agent B (bevollmächtigt) | begrenzte Delegation im Auftrag der jeweiligen Seite |

Vier Rollen sind **keine Vorgabe von vier gleichzeitigen Netzwerkverbindungen**. Ein Mensch kann ohne Agent arbeiten; Agenten können zeitversetzt Nachrichten austauschen. „Organisation“ bedeutet eine geprüfte organisatorische Identität und einen konkreten menschlichen Entscheidungsträger, nicht eine frei behauptete Firmenzugehörigkeit.

Beispiel: Ein persönlicher Agent findet eine berufliche Gelegenheit, der Unternehmensagent fragt nach Verfügbarkeit, eine menschliche Person antwortet direkt, und beide Menschen entscheiden über die Freigabe weiterer Daten. Der PAEP-Kern kennt dafür **keine Job-Semantik**: Andere Dienste können dieselben Kommunikationsräume für Projektarbeit, Kundengespräche oder Kooperation verwenden.

## 3. Der Produktkern

1. **Discovery**: Eine öffentliche, für reguläre Suchmaschinen auffindbare Einstiegsseite verweist auf eine maschinenlesbare Protokollbeschreibung. Untrusted Web-Inhalte sind keine Autorisierung.
2. **Provisioning**: Auf zulässige Anfrage wird ein Endpunkt mit zufälliger opaker ID erzeugt; der JSON-Zustand entsteht dynamisch aus minimalen persistenten Datensätzen.
3. **Delegation**: Der jeweilige Mensch beziehungsweise eine legitimierte Organisation erteilt einem Agenten ausdrücklich begrenzte Lese-/Schreibrechte.
4. **Pairing**: Die Seiten schlagen die Verknüpfung ihrer Endpunkte vor und bestätigen den erlaubten Zweck. Es gibt keinen unkontrollierten Join durch Kenntnis einer GUID.
5. **Lease**: Der Vermittlungsdienst prüft Raum, Teilnehmer, Berechtigungen, Laufzeit, Widerruf und Revision.
6. **Schlüsselaustausch**: Pro teilnahmeberechtigter Identität wird ein überprüfbarer öffentlicher Schlüssel gebunden. Private Entschlüsselungsschlüssel verlassen den vertrauenswürdigen Client nicht.
7. **Nachrichten**: Ein autorisierter Absender verschlüsselt für die erlaubten Empfänger; das Relay stellt Chiffrat und Zustellinformationen bereit.
8. **Menschliche Ansicht**: Optionaler Browser-Chat für Lesen, Antworten, Widerrufen und Rechtefreigaben; er greift auf denselben JSON-Vertrag zu.

## 4. Was bewusst nicht gespeichert wird

Der generische PAEP-Kern braucht **keinen** zentralen Fähigkeitsgraphen, Lebenslauf, Firmenstamm, vollständigen Gesprächsklartext oder proprietären Agenten-Context. Diese gehören – sofern erforderlich – in die Anwendungsebene oder bleiben beim jeweiligen Teilnehmer.

Minimaler persistenter Zustand: Endpunkt-IDs, öffentliche Schlüssel und deren bestätigte Bindungen, Rollen/Autorisierungen, Lease-Zustände, Raumbeziehungen, Nachrichten-Chiffrate mit minimaler Zustellung, sequenzielle Änderungsstände sowie begrenzte technische Nachweise. JSON-Dokumente werden daraus bei Bedarf zusammengesetzt, nicht pro Anfrage vollständig dupliziert.

Die Domain-Applikation kann optionale, freigegebene JSON-Felder referenzieren; kein Profilinhalt wird allein durch eine Endpunkt-ID öffentlich.

## 5. Schnittstellen ohne Bindung an einen Anbieter

- **Web + JSON:** indexierbarer Einstieg, öffentliches Discovery-Dokument, authentifizierte JSON-Operationen.
- **CLI / Code Mode:** äquivalente Aufrufe für zugelassene Claude-Code-, GitHub-Copilot- und andere lokale Agenten.
- **Dateiaustausch:** geprüftes Import-/Exportformat für isolierte Unternehmensumgebungen; gleiche Signatur-/Revisions-/Replay-Regeln.
- **Agent Skills:** erklärende, optional installierbare Instruktionen. Keine Autorität allein durch Installation.
- **MCP:** rein optionaler Adapter, falls organisatorisch erlaubt.
- **Menschliche Oberfläche:** optional, aus demselben Protokollzustand; kein zweites Chat-Backend.

Microsoft Copilot/Copilot Studio und GitHub Copilot sind getrennte Integrationsumgebungen; konkrete Funktionsverfügbarkeit und Unternehmensrichtlinien müssen je Umgebung getestet werden.

## 6. Positionierung

PAEP/P4P ist zunächst ein **Kommunikations- und Berechtigungssystem**, weder Stellenbörse noch genereller Agenten-Host noch soziales Netzwerk. Es kann später in andere Produkte eingebettet oder direkt als eigenständiger Dienst genutzt werden.

Die technischen Bausteine sind nicht neu: A2A, ANP, DIDComm, Matrix, MLS und bestehende Public-Key-Verfahren bilden wichtige Vorarbeiten. Ein möglicher Unterschied liegt in der kombinierten **automatischen Erzeugung zugänglicher JSON-Endpunkte, vier ausdrücklich getrennten Rollen, menschlich kontrollierten Leases und der herstellerunabhängigen Nutzung ohne MCP**. Neuheit, Schutzfähigkeit und Marktpotenzial sind **nicht** bewiesen.

## 7. Nichtziele der ersten Version

- Kein eigener LLM-Host und keine automatische Agentenausführung.
- Kein eigener kryptografischer Algorithmus, kein unprüfbares „Sicherheitsversprechen“.
- Keine Verarbeitung von Bewerberdaten als Voraussetzung des Kommunikationskerns.
- Keine Pflichtregistrierung über einen bestimmten KI-Anbieter.
- Keine Verpflichtung zur permanenten Online-Präsenz eines Fremdagenten.
- Keine vollumfängliche Unternehmensplattform, Zahlungsabwicklung oder Massengruppenchat in v0.1.
- Keine Kopie von PumpkinMe-Modulen; generische Verknüpfung über Schnittstellen.

## 8. Erfolgskriterium des Produkts

Zwei voneinander unabhängige Clients, darunter mindestens ein Agentenadapter und eine menschliche Oberfläche, können eigene Endpunkte provisionieren, getrennte Rechte nachweisen, nach gegenseitiger Freigabe einen Raum erzeugen, authentifizierte Ende-zu-Ende-verschlüsselte Nachrichten austauschen, Nachrichten nach Offline-Zeit abrufen und nach Widerruf keine neuen Zustellungen über den widerrufenen Lease vornehmen. Das Verhalten muss **ohne MCP und ohne PumpkinMe** funktionieren.
