# 🔍 Project Auditor Skill — Claude Cowork

Auditeur & contre-auditeur de projet logiciel avec score de maturité.
Conçu pour [Claude Cowork](https://claude.ai) d'Anthropic.

## Ce que ça fait

Analyse un projet logiciel sur **14 dimensions** (activées selon le profil) et produit :

- Une fiche de contexte calibrée selon le type de projet (web-app, API, lib, embarqué, SaaS B2B, data/ML, secteur réglementé…)
- Un score de maturité pondéré sur 10, référencé ISO 25010 · OWASP SAMM v2 · CMMI · DORA
- Un **contre-audit systématique** : l'IA challenge ses propres conclusions avant de rendre son verdict
- Un plan de remédiation priorisé par matrice Effort × Impact
- Des commandes bash concrètes selon la stack détectée
- Un suivi de maturité dans le temps (delta entre audits)
- Un export JSON/SARIF pour intégration dashboard ou CI/CD

## Les 14 dimensions

| # | Dimension | Référentiel | Toujours actif ? |
|---|-----------|-------------|-------------------|
| D1 | Sécurité | OWASP Top 10, SAMM v2 | Oui |
| D2 | Qualité du code | ISO 25010, Clean Code, SOLID | Oui |
| D3 | Architecture | 12-Factor, Clean/Hexagonal | Oui |
| D4 | Fiabilité & Résilience | SRE Practices | Oui |
| D5 | Performance | ISO 25010 | Oui |
| D6 | Documentation & Maintenabilité | ISO 25010 | Oui |
| D7 | DevOps & Observabilité | DORA Metrics, OpenTelemetry | Oui |
| D8 | Dépendances & Supply Chain | OWASP A06, SLSA | Oui |
| D9 | Compatibilité | ISO 25010 | Oui |
| D10 | Gouvernance | CMMI, OWASP SAMM | Oui |
| D11 | Sûreté fonctionnelle | IEC 61508, ISO 26262, FDA | Profil `embedded-critical` |
| D12 | UX & Accessibilité | WCAG 2.1 AA | Profil `saas-b2b` |
| D13 | Conformité & Risque | RGPD, IA Act | Profils `saas-b2b`, `data-platform`, `regulated` |
| D14 | Qualité Données & Biais | ISO/IEC 5259, NIST AI RMF | Profil `data-platform` |

## Installation

1. Télécharge le fichier `project-auditor.skill`
2. Dans Claude Cowork, glisse le fichier dans la session ou utilise le menu Compétences
3. Dis simplement : `audite mon projet` ou `analyse mon code`

## Utilisation

Déclenché automatiquement sur :
> "audite mon projet" · "analyse mon code" · "score de maturité" · "revue architecture" · "audit sécurité" · "qu'est-ce qui ne va pas dans mon code"

Modes spéciaux :
> "mode CI/CD" ou "audit rapide" → verdict PASS/FAIL allégé sur 4 dimensions
> "export JSON" ou "format SARIF" → sortie machine-readable

## Profils supportés

| Profil | Description |
|--------|-------------|
| `solo-script` | Script/outil personnel, 1 dev |
| `web-app` | Application web avec utilisateurs finaux |
| `api-service` | API / microservice / backend |
| `lib-sdk` | Librairie consommée par d'autres devs |
| `embedded-critical` | Médical, industriel, finance critique |
| `saas-b2b` | SaaS multi-tenant, contrats clients |
| `data-platform` | Pipeline data, plateforme analytics/ML |
| `regulated` | Secteur réglementé hors embedded (santé, finance, secteur public) |

## Exemple

Un audit fictif complet est disponible dans [`example-output.md`](./example-output.md) — projet SaaS B2B avec les dimensions UX et conformité activées.

## Licence

MIT — libre d'utilisation, de modification et de partage.
