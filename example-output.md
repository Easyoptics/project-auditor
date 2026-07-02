# Exemple d'audit — HelpDesk Pro (SaaS B2B)

> Audit fictif généré pour illustrer le skill `project-auditor` v4, profil `saas-b2b`.

---

## PHASE 0 — FICHE DE CONTEXTE PROJET

```
┌─────────────────────────────────────────────────────────────┐
│  Nom du projet     : HelpDesk Pro                            │
│  Type de projet    : SaaS B2B — gestion de tickets support   │
│  Stack technique   : Ruby on Rails 7 · React · PostgreSQL ·  │
│                      Sidekiq · AWS S3                         │
│  Taille du code    : ~18 000 lignes                           │
│  Taille d'équipe   : 6-20                                     │
│  Contexte d'usage  : prod (clients payants, données perso)   │
│  Code audité       : ~75%                                     │
│  Code AI-généré ?  : non                                      │
│  Domaine sensible  : standard (données clients B2B, tickets)  │
└─────────────────────────────────────────────────────────────┘
```

**Profil appliqué : `saas-b2b`** → active D12 (UX/A11y) et D13 (Conformité/Risque)
Confiance audit : **ÉLEVÉE**

---

## PHASE 1 — AUDIT (extraits pertinents pour les nouvelles dimensions)

### D12 — UX & Accessibilité `5/10` 🎨

**Contraste insuffisant** — Les boutons secondaires (`.btn-ghost`) utilisent un gris `#999` sur fond blanc = ratio 2.8:1, sous le seuil WCAG AA de 4.5:1.

**Navigation clavier cassée** — Le modal de création de ticket (`TicketModal.jsx`) ne piège pas le focus : Tab sort du modal vers le contenu en arrière-plan.

**Labels manquants** — Formulaire de réponse rapide : les champs utilisent `placeholder` seul, sans `<label>` associé. Un lecteur d'écran ne peut pas les identifier.

**Points positifs** : messages d'erreur de formulaire explicites ("L'email doit contenir un @"), pas de dark pattern détecté sur le flow d'abonnement/désabonnement.

⚠️ Règle plancher D12 : le modal de création de ticket est un parcours critique → piège focus absent, mais pas totalement bloquant au clavier (Échap fonctionne). Pas de plafond déclenché, mais correction P2 recommandée.

```bash
npx @axe-core/cli http://localhost:3000/tickets/new
# → 12 violations, dont 3 de niveau "critical" sur le contraste
```

---

### D13 — Conformité & Gestion des Risques `4/10` ⚖️

**🚨 Droit à l'effacement non fonctionnel** — `User.destroy` en Rails ne supprime pas les tickets associés (`has_many :tickets` sans `dependent: :destroy`), ni les pièces jointes sur S3. Suppression de compte = données orphelines conservées indéfiniment.

**Sous-traitant hors UE non documenté** — Le service d'emailing transactionnel (`SendGrid`) est hébergé aux US. Aucune clause contractuelle type (SCC) référencée dans le code ou la doc.

**Logs avec PII en clair** — `Rails.logger.info "User #{user.email} created ticket"` — l'email complet apparaît dans les logs applicatifs, conservés 90 jours sans anonymisation.

**Points positifs** : consentement cookies correctement implémenté (opt-in, pas de case pré-cochée), export de données utilisateur disponible via un endpoint `/api/export`.

⚠️ Règle plancher D13 : droit à l'effacement non fonctionnel = absence de base légale respectée sur ce point → **D13 plafonné à 3/10**, relevé à 4 en contre-audit (le reste du dispositif RGPD — consentement, export — est correctement pensé).

```bash
grep -rIn "logger\.\(info\|debug\)" app/ | grep -i "email"
# → 7 occurrences avec email en clair dans les logs
```

> Ce score signale des zones à faire vérifier par un DPO — ce n'est pas un avis juridique.

---

*(Les dimensions D1-D10 suivent le même format détaillé que dans l'audit standard — omises ici pour se concentrer sur les nouveautés v4)*

---

## PHASE 3 — SCORE DE MATURITÉ GLOBAL (extrait)

```
╔══════════════════════════════════════════════════════════════════╗
║  Profil appliqué    : saas-b2b       Confiance : ÉLEVÉE          ║
╠══════════════════════════════════════════════════════════════════╣
║  D1  Sécurité             6/10  poids:×1.5                       ║
║  D4  Fiabilité             7/10  poids:×1.5                       ║
║  D12 UX/Accessibilité      5/10  poids:×1.2  🎨 NOUVEAU           ║
║  D13 Conformité/Risque     4/10  poids:×1.3  ⚖️ NOUVEAU           ║
╠══════════════════════════════════════════════════════════════════╣
║  SCORE PONDÉRÉ            5.9/10                                  ║
║  NIVEAU                   🟡 MOYEN                                ║
╚══════════════════════════════════════════════════════════════════╝
```

---

## PHASE 4 — PLAN DE REMÉDIATION (extraits P1/P2 liés aux nouvelles dimensions)

**[P1] | D13 | Effort: S | Impact: Critique**
Problème : `User.destroy` ne supprime pas en cascade les tickets et pièces jointes S3.
Impact : Non-conformité RGPD sur le droit à l'effacement — risque de sanction CNIL.
Correction :
```ruby
# app/models/user.rb
has_many :tickets, dependent: :destroy
has_many :attachments, dependent: :destroy_async  # + job de suppression S3
```

**[P2] | D12 | Effort: XS | Impact: Fort**
Problème : Focus non piégé dans `TicketModal.jsx`.
Correction : utiliser `focus-trap-react` ou équivalent autour du modal.

**[P2] | D13 | Effort: S | Impact: Fort**
Problème : Emails en clair dans les logs.
Correction :
```ruby
Rails.logger.info "User #{user.id} created ticket"  # id au lieu d'email
```

---

## PHASE 5 — VERDICT FINAL (extrait)

**Recommandation en une phrase :** Corriger le droit à l'effacement (P1, effort S) avant tout audit de conformité client — le reste est un travail de sprint normal, pas bloquant pour la prod actuelle.
