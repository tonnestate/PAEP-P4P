# Architekturentscheidungen — PAEP / P4P

**Datum:** 2026-11-23. Diese Entscheidungen übertragen das Konzept **in ein eigenständiges Produkt**. Der ursprüngliche PumpkinMe-PAEP-Entwurf bleibt historischer Anwendungskontext, nicht technische Abhängigkeit.

| Bereich | Beschluss |
|---|---|
| Eigenständigkeit | PAEP/P4P ist eigenständiges Repo/Produkt; PumpkinMe ist optionaler Client |
| Protokoll | PAEP als hersteller- und transportunabhängiger, JSON-zentrierter Austauschvertrag |
| Kommunikationsmodell | P4P: zwei Seiten und bis zu vier getrennte menschliche/agentische Rollen |
| Repräsentation | dynamische JSON-Endpunkte; Webseiten nur optional für menschliche Nutzung |
| Lebenszyklus | zwei Endpunkte, Paarung durch gegenseitige Autorisierung, Lease, verschlüsselter Kanal |
| Identität / Berechtigung | Public/Private Key als kryptografischer Baustein; getrennte Rollen und Schlüsselhoheit; Lease kontrolliert Aktionen, ist aber kein AES-Schlüssel |
| Datenschutz | Clientseitige E2EE als Ziel; Relay bekommt keinen Klartext, Metadaten sind nicht automatisch anonym |
| Transport | n8n als geplanter austauschbarer Chiffrat-Relay, nicht als Schlüsselverwalter |
| Technologie | TypeScript, schlanker JSON-Vertrag, DuckDB für minimalen Persistenzzustand |
| Integrationen | Web/HTTPS, CLI/Code Mode, JSON-Datei; Skills und MCP optional; keine MCP-Pflicht |
| GitHub | Code und Spezifikation, kein operatives Nachrichten- oder Profilledger |
| Anwendung | keine Job-, Profil- oder Skill-Pflicht im generischen Kern |
| Entwicklung | zuerst nachvollziehbarer lokaler Prototyp, ohne GitHub Actions/CI |

## Offen und vor Implementierung zu entscheiden

- Verbindliche wire-kompatible JSON-Schemas und endgültige Protokollversionierung.
- Präzise Kryptografiesuite, unabhängige Prüfung von Schlüsselbindungen und sichere Clientdistribution.
- Identitätsprüfung von menschlichen/organisatorischen Principals und Verfahren zur Agentendelegation.
- Belastbare Client-Schlüsselverwaltung im Browser und in KI-Agenten ohne Offenlegung an deren Prompts.
- Aufbewahrungsfristen, Metadatenminimierung, Umgang mit Missbrauch und rechtliche Verantwortlichkeiten.
- Konkrete Nutzungsmöglichkeiten der einzelnen Agenten-Hosts (ChatGPT, Claude Code, GitHub Copilot, Microsoft Copilot).
- Ob die Arbeitsabkürzung PAEP später einen projektnamenunabhängigen ausgeschriebenen Namen erhält.

**Nicht behaupten:** neuartige Kryptografie, exklusive Protokoll-Neuheit, produktiver E2EE-Dienst, automatische ChatGPT-Workflows oder bereits bestandene Konformitätstests.
