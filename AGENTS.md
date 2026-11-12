# AGENTS.md — KAYE

Instructions pour tout agent de code (Codex, Claude Code, autres). À placer à la racine du dépôt.

## Le produit en une phrase
KAYE est un ERP IA conversationnel pour les boutiques ivoiriennes : le commerçant parle à son agent sur WhatsApp, Telegram ou dans l'application web ; ventes, achats, stock, caisse et factures FNE certifiées par la DGI se tiennent à partir de la conversation.

## Documents de référence (à lire avant tout ticket)
- `docs/cdc/01_CDC_Fonctionnel_KAYE_v3.md` — ce que le produit doit faire (exigences `EF-Mx-nn`, règles `RG-n`).
- `docs/cdc/02_CDC_Technique_KAYE_v3.md` — comment le construire (architecture, données, API, agent, FNE).
- `docs/cdc/03_Backlog_KAYE_v3.md` — la liste des tickets. **Un ticket = une branche = une PR vers `dev`** (voir « Règles Git et collaboration »).
- `docs/adr/` — décisions d'architecture.

## Règles non négociables
1. **L'IA propose, l'humain valide.** Le module `agent` crée des `pending_actions` ; seuls les services du domaine écrivent, après confirmation.
2. **Ne jamais bloquer une vente** (DGI absente, stock insuffisant, paramétrage incomplet).
3. **Aucun appel externe dans une transaction.** Effets externes via `outbox_events` puis BullMQ.
4. **Argent en `bigint` XOF entiers, quantités en milli-unités `bigint`.** Jamais de `number` flottant pour l'argent.
5. **Toute table métier a `merchant_id` + RLS.** Toute nouvelle table : politique RLS + test d'accès croisé.
6. **Rien n'est supprimé** : statuts et contre-passation. Une facture certifiée ne se modifie jamais (avoir).
7. **Le numéro FNE vient de la DGI.** KAYE tient seulement sa numérotation interne (`VTE-AAAA-NNNNNN`).
8. **Messages avec des chiffres = gabarits** (`packages/messages`), jamais du texte généré par un modèle.
9. **Sortie LLM = JSON validé par Zod**, qui ne déclenche jamais d'action seule.
10. **Pas de secrets** dans le code, les tests, les logs, les prompts. NCC et téléphones masqués dans les logs.
11. **Aucune donnée réelle de prospect** dans le dépôt (seed de démonstration uniquement).
12. **WhatsApp uniquement par l'API officielle Meta.** Jamais de WhatsApp Web ni de bibliothèque non officielle.

## Conventions
- TypeScript strict, ESM, Node 22. Code, base et API en **anglais** ; interface et messages en **français** (glossaire : annexe B du CDC technique).
- API : `snake_case`, erreurs RFC 7807 avec `code`, `Idempotency-Key` sur les écritures d'argent ou de stock.
- Modules NestJS = domaines ; un module n'accède pas aux tables d'un autre, il appelle son service.
- Logique pure dans `packages/domain` (sans I/O), testée unitairement.
- Interfaces remplaçables : `FneProvider`, `LlmGateway`, `ChannelAdapter`, `StorageProvider`, `SmsProvider`.
- Migrations : nouvelles migrations uniquement, jamais modifier une migration fusionnée.
- Charte Bissap pour toute interface (`packages/ui`) : un seul grand aplat bissap et un seul bouton principal par écran, pas de dégradé, pas d'émoji en guise d'icône.

## Commandes
```bash
pnpm i                 # installation
pnpm dev               # api + worker + web (docker compose doit tourner)
docker compose -f infra/docker-compose.yml up -d
pnpm db:migrate        # migrations
pnpm db:seed           # données de démonstration
pnpm lint && pnpm typecheck && pnpm test
pnpm test:nlu          # corpus NLU (obligatoire si nlu-core, agent ou prompt modifié)
pnpm test:e2e          # Playwright
```

## Définition de « terminé »
- Critères d'acceptation du ticket couverts par des tests.
- `pnpm lint && pnpm typecheck && pnpm test` verts ; corpus NLU sans régression si concerné.
- Nouvelle intention : test de conversation via l'adaptateur `test` (cas nominal, correction, annulation, droits insuffisants).
- Nouvelle table : `merchant_id`, RLS, test d'accès croisé.
- Description de PR : ticket, EF couvertes, choix faits, points ouverts.

## Quand s'arrêter et demander
- Le ticket touche un point de la section « Points techniques à trancher » (CDC technique § 17) ou « Points à trancher » (CDC fonctionnel § 19).
- Une règle cardinale semble gêner l'implémentation.
- Une exigence est ambiguë ou contradictoire entre les deux CDC.
Dans ce cas : écrire la question dans la PR (ou dans `docs/questions.md`), ne pas inventer de règle métier.

## Conseil des experts
Dix agents critiques (fonctionnel, architecte, dev full stack, staff engineer, UX/UI, expert ERP, fiscal ivoirien, IA, sécurité, terrain) relisent les décisions avant qu'on construise. Définitions : `.claude/agents/conseil-*.md` ; règles : `docs/conseil/CONSEIL.md`.
- Avant le premier ticket d'un lot : séance **G0**. Nouvelle ADR, table ou contrat d'API : **G1**. Mise en production : **G2**.
- PR étiquetée `argent`, `fne`, `securite` ou `agent` : joindre une revue ciblée.
- Un constat **Bloquant** non levé interdit la fusion, sauf dérogation écrite d'un fondateur.
- Avec Codex : suivre `docs/conseil/CODEX_CONSEIL.md`.

## Règles Git et collaboration

### Branches permanentes
| Branche | Rôle | Qui fusionne |
|---|---|---|
| `main` | production | un fondateur, depuis `recette` |
| `recette` | tests avec l'environnement DGI de test | un fondateur, depuis `dev` |
| `dev` | intégration quotidienne | un fondateur, depuis une branche de tâche |

Flux : `main` ← `recette` ← `dev` ← `claude/t-<num>-<slug>` ou `codex/t-<num>-<slug>`.
Personne ne pousse directement sur ces trois branches : tout passe par une pull request.

### Règles
1. Une tâche du backlog (`docs/cdc/03_Backlog_KAYE_v3.md`) = une branche = une pull request vers `dev`.
2. Toujours partir de `dev` à jour (`git fetch origin dev && git switch -c <branche> origin/dev`). Nom de branche : `claude/t-<num>-<slug>` pour Claude Code, `codex/t-<num>-<slug>` pour Codex (ex. `claude/t-004-stock-mouvements`).
3. Jamais de branche permanente par agent. Une branche de tâche vit quelques heures ou quelques jours, puis est supprimée après la fusion.
4. Jamais de push direct ni de force-push sur `main`, `recette` ou `dev`. Jamais de fusion de sa propre pull request : la fusion est faite par un fondateur.
5. Deux agents ne travaillent jamais en même temps sur le même module. Répartition par défaut :
   - **Claude Code** : socle, base de données, RLS, `packages/domain` (ventes, stock, trésorerie), moteur d'agent, connecteur FNE.
   - **Codex** : écrans web, points d'API CRUD, gabarits de messages et PDF, imports, tests, corpus NLU.

   Les tickets marqués *socle* sont traités l'un après l'autre.
6. Petites pull requests : une seule tâche, lisible en dix minutes. Si une tâche grossit, la découper.
7. Une pull request contient le code, les tests, la mise à jour de la documentation si un contrat change, et la liste des EF couvertes.
8. Avant d'ouvrir la pull request : `pnpm lint && pnpm typecheck && pnpm test` verts (dès que le monorepo existe).
9. Relecture croisée : chaque pull request est relue par l'autre agent (Codex relit Claude, Claude relit Codex), puis validée par un fondateur.
10. Point ambigu ou touchant au § 17 du CDC technique : poser la question dans la pull request et s'arrêter, ne pas deviner.
11. Migrations : créées par l'agent qui possède le ticket ; en cas de conflit de numérotation, celui qui fusionne en second régénère la sienne.
12. Aucun secret, aucune donnée réelle de prospect, aucun `console.log` de données personnelles dans le dépôt.
13. Messages de commit au format Conventional Commits, en français : `feat(stock): …`, `fix(fne): …`.

Seule exception connue : la tâche `t-000-regles-git`, qui installe ces règles, vise `main` directement.
