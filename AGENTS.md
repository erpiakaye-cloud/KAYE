# AGENTS.md — KAYE

Instructions pour tout agent de code (Codex, Claude Code, autres). À placer à la racine du dépôt.

## Le produit en une phrase
KAYE est un ERP IA conversationnel pour les boutiques ivoiriennes : le commerçant parle à son agent sur WhatsApp, Telegram ou dans l'application web ; ventes, achats, stock, caisse et factures FNE certifiées par la DGI se tiennent à partir de la conversation.

## Documents de référence (à lire avant tout ticket)
- `docs/cdc/01_CDC_Fonctionnel_KAYE_v3.md` — ce que le produit doit faire (exigences `EF-Mx-nn`, règles `RG-n`).
- `docs/cdc/02_CDC_Technique_KAYE_v3.md` — comment le construire (architecture, données, API, agent, FNE).
- `docs/cdc/03_Backlog_KAYE_v3.md` — la liste des tickets. **Un ticket = une branche = une PR.**
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

## Branches
`main` (production) ← `recette` ← `dev` ← `t-<num>-<slug>`. Les agents ouvrent des PR vers `dev` uniquement.
