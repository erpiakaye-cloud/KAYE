# KAYE — Cahier des charges technique

**Version 3.2 · 1er novembre 2026 · Destinataires : Claude Code, Codex, développeurs, DevOps · revue par le Conseil des experts (CDC fonctionnel § 22)**

| Champ | Valeur |
|---|---|
| Référence fonctionnelle | `01_CDC_Fonctionnel_KAYE_v3.md` (exigences `EF-Mx-nn`, règles `RG-n`) |
| Plan de travail | `03_Backlog_KAYE_v3.md` (tickets `T-nnn`) |
| Remplace | STD_Karnet v1.0 (29 août) ; reprend et met à jour ADR-01 à ADR-10 |
| Corrections majeures vs STD v1.0 | Numérotation FNE attribuée **par la DGI** (§ 9) ; agent indépendant du canal (§ 6) ; passerelle de modèles multi-fournisseurs, Mistral par défaut (§ 7) ; deux canaux WhatsApp (§ 8) ; périmètre achats, stock, trésorerie (§ 5) |

> **Pour les agents de code.** Lisez d'abord § 1 (règles de travail), § 2 (principes) et § 14 (mode opératoire). En cas de doute sur une règle métier, le CDC fonctionnel fait foi. Si un ticket touche un « point à trancher » (§ 17), arrêtez-vous et demandez.

---

## Sommaire

1. Règles de travail non négociables
2. Principes d'architecture
3. Décisions d'architecture (ADR)
4. Pile technique et organisation du dépôt
5. Modèle de données
6. Moteur d'agent conversationnel
7. Passerelle de modèles (LLM, transcription, lecture de documents)
8. Canaux : WhatsApp, Telegram, web, e-mail
9. Connecteur FNE (API DGI)
10. API REST
11. Application web
12. Documents PDF et fichiers
13. Sécurité, données personnelles, conformité
14. Mode opératoire Claude Code + Codex
15. Tests et qualité
16. Environnements, CI/CD, déploiement, exploitation
17. Points techniques à trancher
18. Conseil des experts : mise en œuvre
Annexes : A. Variables d'environnement · B. Glossaire FR ↔ EN · C. Traçabilité EF → modules · D. Exemples de payloads

---

## 1. Règles de travail non négociables

Ces règles s'appliquent à **tout** code produit, par un humain ou un agent.

1. **Aucune écriture métier sans confirmation** (RG-2). L'agent IA crée des *propositions* ; seul un *command handler* du domaine écrit, et seulement après confirmation ou depuis l'API web authentifiée.
2. **Le domaine ne dépend d'aucun service externe.** Pas d'appel HTTP (DGI, Meta, LLM) dans une transaction base de données. Les effets externes passent par l'outbox (§ 4.4).
3. **Argent en entiers XOF** (`bigint`), jamais de flottant. Quantités en **milli-unités entières** (`qty_milli`, 2,5 kg = 2500).
4. **Toute table métier porte `merchant_id`** et est protégée par RLS (§ 13.2).
5. **Rien n'est supprimé** sur les entités métier : statuts, contre-passation, `archived_at`.
6. **Les messages contenant des chiffres sont produits par des gabarits**, jamais rédigés librement par un modèle.
7. **Secrets** uniquement par variables d'environnement ; jamais dans le dépôt, les logs, les prompts.
8. **Code, base, API en anglais ; interface et messages en français** (glossaire en annexe B).
9. **Une PR = un ticket**, tests inclus, CI verte, relue par un autre agent ou un humain.
10. **Ne jamais modifier une migration déjà fusionnée** ; en créer une nouvelle.

---

## 2. Principes d'architecture

| Réf. | Principe | Conséquence |
|---|---|---|
| PR-01 | L'IA propose, l'humain valide | Table `pending_actions` ; exécution uniquement sur confirmation |
| PR-02 | Déterministe avant probabiliste | Analyseur à règles d'abord ; LLM en repli ; règles métier en code pur |
| PR-03 | Un cerveau, plusieurs canaux | Moteur d'agent indépendant du canal ; adaptateurs WhatsApp, Telegram, web, test |
| PR-04 | Ne jamais bloquer une vente | Écriture synchrone, certification FNE asynchrone avec rejeu |
| PR-05 | Conformité non négociable | Facture certifiée immuable ; numéro normatif fourni par la DGI ; numérotation interne continue |
| PR-06 | Isolation stricte | RLS PostgreSQL + contrôle applicatif + tests d'accès croisé générés |
| PR-07 | Journaux comme source de vérité | Stock et trésorerie dérivés de mouvements immuables ; soldes = projections |
| PR-08 | Frugalité | Coût variable < 2 € par commerçant actif par mois ; payloads compacts ; pas de polling |
| PR-09 | Fournisseurs remplaçables | Interfaces `FneProvider`, `LlmProvider`, `ChannelAdapter`, `StorageProvider`, `SmsProvider` |
| PR-10 | Monolithe modulaire | Un seul service API + un service worker ; frontières de modules strictes ; pas de microservices |

---

## 3. Décisions d'architecture (ADR)

Chaque ADR a un fichier dans `docs/adr/`. Statut : **A** = accepté, **M** = modifié par cette version.

| ADR | Décision | Statut |
|---|---|---|
| ADR-01 | Monolithe modulaire (NestJS) + workers BullMQ | A |
| ADR-02 | TypeScript de bout en bout, types partagés (Zod) | A |
| ADR-03 | PostgreSQL managé (transactions multi-tables indispensables) | A |
| ADR-04 | Analyseur à règles en première ligne, LLM en repli | A |
| ADR-05 | **Numérotation** : numéro normatif attribué par la DGI ; KAYE tient une numérotation interne continue par type de document (compteur transactionnel). *Remplace* l'ancienne allocation `FNE-AAAA-NNN` par KAYE | **M** |
| ADR-06 | Journal de mouvements + soldes projetés (stock, trésorerie) | A |
| ADR-07 | **Pas d'application native** ; application web responsive (PWA légère) + bots | **M** |
| ADR-08 | RLS PostgreSQL | A |
| ADR-09 | Bot sur le WhatsApp Business du commerçant : Tech Provider, Embedded Signup, Coexistence, via BSP | A |
| ADR-10 | Deux canaux : pilotage (numéro KAYE + Telegram) et commercial (WABA du commerçant) | A |
| ADR-11 | Moteur d'agent indépendant du canal ; **Telegram et web livrés en premier**, WhatsApp de pilotage dès que le numéro est vérifié | Nouveau |
| ADR-12 | Passerelle de modèles multi-fournisseurs ; **Mistral par défaut**, Anthropic et OpenAI en secours ; routage par tâche | Nouveau |
| ADR-13 | **Hébergement chez Hostinger** (décision du 31 oct.) : offre d'hébergement web pour le site vitrine, l'application web (fichiers statiques) et les e-mails professionnels ; **VPS Hostinger** (Docker Compose) pour l'API, le worker, Postgres et Redis ; fichiers sur Cloudflare R2. Vercel abandonné. Détail § 16.1 | **M** |
| ADR-14 | Code et base en anglais, interface en français | Nouveau |
| ADR-15 | Messages chiffrés produits par gabarits ; le LLM ne remplit que des champs typés | Nouveau |

---

## 4. Pile technique et organisation du dépôt

### 4.1 Pile
| Couche | Choix | Remarque |
|---|---|---|
| Langage | TypeScript 5.x, `strict: true`, Node.js 22 LTS | ESM |
| Monorepo | pnpm workspaces + Turborepo | |
| API | NestJS 11 (adaptateur Fastify) | modules = domaines |
| Workers | BullMQ 5 sur Redis 7 | même code que l'API, autre point d'entrée |
| Base | PostgreSQL 16 + `pg_trgm` + `unaccent` | RLS |
| ORM / migrations | Drizzle ORM + drizzle-kit (migrations SQL versionnées) | |
| Validation | Zod (entrées API, sorties LLM, payloads des canaux) | schémas dans `packages/contracts` |
| Web | React 19 + Vite + Tailwind CSS v4 + TanStack Router + TanStack Query | reprend la base des prototypes Figma |
| PDF | `@react-pdf/renderer` + `qrcode` | polices Unbounded / Nunito Sans sous-ensemblées |
| Fichiers | S3 compatible (Cloudflare R2 ; MinIO en local) | URL présignées 15 min |
| Logs / erreurs | pino (JSON) + Sentry ; OpenTelemetry en V1 | |
| Tests | Vitest, Testcontainers (Postgres, Redis), Playwright | |
| Lint / format | ESLint (typescript-eslint) + Prettier | |
| CI/CD | GitHub Actions | |

### 4.2 Arborescence
```
kaye/
├── AGENTS.md                 # règles pour Codex et tous les agents (lu aussi par Claude Code)
├── CLAUDE.md                 # @AGENTS.md + consignes propres à Claude Code
├── docs/
│   ├── cdc/                  # 01_CDC_Fonctionnel, 02_CDC_Technique, 03_Backlog
│   ├── adr/                  # ADR-01 … ADR-15
│   └── runbooks/             # incidents FNE, Meta, restauration
├── apps/
│   ├── api/                  # NestJS : REST, webhooks, auth
│   │   └── src/modules/      # identity, merchants, catalog, customers, sales, invoicing,
│   │                         # receivables, purchasing, suppliers, inventory, cash, accounting,
│   │                         # agent, channels, notifications, billing, imports, ops
│   ├── worker/               # consommateurs BullMQ (inbound, media, fne, pdf, outbound, cron)
│   └── web/                  # application commerçant + console /ops
├── packages/
│   ├── domain/               # entités, règles, calculs purs (CMP, TVA, soldes) — aucune I/O
│   ├── contracts/            # schémas Zod partagés (API, intents, events)
│   ├── db/                   # schéma Drizzle, migrations, RLS, seeds
│   ├── nlu-core/             # normalisation + analyseur à règles (isomorphe web/serveur)
│   ├── agent/                # orchestrateur : registre d'intents, propositions, rendu
│   ├── llm-gateway/          # fournisseurs LLM / ASR / vision, routage, coûts
│   ├── channels/             # adaptateurs WhatsApp Cloud, Telegram, Web, Test
│   ├── fne/                  # connecteur DGI (simulé, test, production)
│   ├── documents/            # gabarits PDF + numérotation
│   ├── messages/             # gabarits de messages FR (texte, boutons) par canal
│   └── ui/                   # design system Bissap (tokens, composants)
├── datasets/
│   ├── nlu/                  # corpus annotés (.jsonl) : texte, vocaux transcrits
│   ├── documents/            # BL et factures de test (anonymisés)
│   └── fne/                  # réponses DGI enregistrées (fixtures)
├── infra/
│   ├── docker-compose.yml    # postgres, redis, minio, mailpit
│   └── railway/ · vercel/
└── .github/workflows/
```

### 4.3 Frontières de modules
- Un module expose un **service applicatif** (commandes + requêtes) ; les autres modules l'appellent, jamais ses tables directement.
- `packages/domain` contient la logique pure testable sans base.
- Le module `agent` n'écrit **jamais** en base métier : il appelle les services applicatifs.
- Les modules `channels`, `fne`, `llm-gateway` sont des adaptateurs derrière des interfaces.

### 4.4 Outbox et événements
Toute écriture métier qui doit déclencher un effet externe insère, **dans la même transaction**, une ligne `outbox_events` (`sale.confirmed`, `invoice.certify_requested`, `message.send_requested`, …). Un relais (worker) lit l'outbox et pousse dans les files BullMQ. Ainsi, aucune file ne reçoit un identifiant d'une transaction annulée.

### 4.5 Files BullMQ
| File | Rôle | Concurrence / reprise |
|---|---|---|
| `inbound` | Traiter un message entrant (déjà persisté) | séquentiel par conversation grâce à un **verrou Redis** (`lock:conv:<id>`, 30 s) ; un job qui ne l'obtient pas est re-planifié 500 ms plus tard (les groupes BullMQ sont une fonction payante, non utilisée) |
| `media` | Transcription audio, lecture de documents | 4 ; 3 tentatives |
| `fne` | Certification et avoirs | 2 par commerçant ; backoff § 9.6 |
| `pdf` | Génération et stockage des PDF | 4 ; 3 tentatives |
| `outbound` | Envoi de messages (WhatsApp, Telegram) | limité par numéro ; étalement 2 s pour les envois groupés |
| `scheduler` | Résumé du soir, comptage du jour, relances d'expiration, réconciliations nocturnes | cron |
| `email-in` (V1) | Pièces reçues par e-mail | 2 |

---

## 5. Modèle de données

### 5.1 Conventions
- Clés primaires `uuid` (v7 générées côté application, triables) ; journaux à fort volume en `bigserial`.
- Colonnes `created_at`, `updated_at` (`timestamptz`), `created_by` (user id ou `system`/`agent`).
- Montants : `bigint` suffixe `_xof`. Quantités : `bigint` suffixe `_milli`. Taux : `integer` en points de base (`1800` = 18 %).
- Enums en `text` + contrainte `CHECK` (plus simple à faire évoluer).
- `merchant_id` sur **toutes** les tables métier (même quand `shop_id` est présent) → politique RLS unique.

### 5.2 Vue d'ensemble
```
merchant (établissement, NCC, régime fiscal, FNE)
 ├── shop (boutique / point de vente)
 │    ├── cash_session ── treasury_movement ── treasury_account
 │    ├── stock_movement ── article
 │    ├── sale ── sale_line            ── invoice ── invoice_line ── credit_note
 │    ├── purchase_receipt ── receipt_line ── purchase_order (V1)
 │    └── stock_count (V1)
 ├── user ── user_shop_role ── channel_identity (whatsapp, telegram)
 ├── customer ── receivable ── customer_payment
 ├── supplier ── article_supplier ── supplier_invoice ── supplier_payment
 ├── contact (WABA, V1) ── conversation ── message
 ├── pending_action · outbound_request · nlu_log · audit_log
 ├── document_counter · file · fne_credential · whatsapp_account (V1)
 └── subscription · pass · usage_counter
```

### 5.3 Schéma SQL de référence (MVP)

> Le schéma Drizzle (`packages/db/src/schema/*.ts`) est la source ; ce SQL décrit l'intention. Les colonnes `created_at/updated_at` sont omises pour la lisibilité.

```sql
CREATE EXTENSION IF NOT EXISTS pg_trgm;
CREATE EXTENSION IF NOT EXISTS unaccent;

-- Identité et locataires -----------------------------------------------
CREATE TABLE merchants (
  id               uuid PRIMARY KEY,
  legal_name       text NOT NULL,
  trade_name       text NOT NULL,
  activity         text,
  ncc              text UNIQUE,                       -- NULL = mode sans certification
  tax_regime       text NOT NULL DEFAULT 'RME'
                   CHECK (tax_regime IN ('TEE','RME','RSI','RNI','AUTRE')),
  default_vat_code text NOT NULL DEFAULT 'TVAD'
                   CHECK (default_vat_code IN ('TVA','TVAB','TVAC','TVAD')),
  fne_status       text NOT NULL DEFAULT 'disabled'
                   CHECK (fne_status IN ('disabled','sandbox','pending_validation','active','suspended')),
  plan             text NOT NULL DEFAULT 'decouverte'
                   CHECK (plan IN ('decouverte','boutique','commerce','reseau')),
  plan_expires_at  timestamptz,
  settings         jsonb NOT NULL DEFAULT '{}'        -- seuils, heure du résumé, plafonds de remise…
);

CREATE TABLE shops (
  id           uuid PRIMARY KEY,
  merchant_id  uuid NOT NULL REFERENCES merchants(id),
  name         text NOT NULL,
  fne_point_of_sale   text,          -- valeur 'pointOfSale' attendue par la DGI
  fne_establishment   text,          -- valeur 'establishment' attendue par la DGI
  address      text,
  timezone     text NOT NULL DEFAULT 'Africa/Abidjan',
  archived_at  timestamptz
);

CREATE TABLE users (
  id           uuid PRIMARY KEY,
  merchant_id  uuid NOT NULL REFERENCES merchants(id),
  display_name text NOT NULL,
  phone_e164   text NOT NULL,                       -- +2250707123456
  pin_hash     text,                                -- argon2id
  pin_failed   int NOT NULL DEFAULT 0,
  pin_locked_until timestamptz,
  last_seen_at timestamptz,                         -- EF-M0-11 : suspension après 60 jours
  suspended_reason text,                            -- 'inactive_number', 'removed_by_owner'…
  active       boolean NOT NULL DEFAULT true,
  UNIQUE (phone_e164)                               -- un numéro = un utilisateur KAYE
);

CREATE TABLE user_shop_roles (
  user_id   uuid NOT NULL REFERENCES users(id),
  shop_id   uuid NOT NULL REFERENCES shops(id),
  merchant_id uuid NOT NULL REFERENCES merchants(id),
  role      text NOT NULL CHECK (role IN ('owner','manager','seller','accountant')),
  PRIMARY KEY (user_id, shop_id)
);

CREATE TABLE channel_identities (
  id          uuid PRIMARY KEY,
  user_id     uuid NOT NULL REFERENCES users(id),
  merchant_id uuid NOT NULL REFERENCES merchants(id),
  channel     text NOT NULL CHECK (channel IN ('whatsapp','telegram','web')),
  external_id text NOT NULL,          -- wa_id, telegram chat_id
  last_inbound_at timestamptz,        -- pour la fenêtre de 24 h WhatsApp
  active_shop_id uuid REFERENCES shops(id),
  UNIQUE (channel, external_id)
);

-- Catalogue et stock ----------------------------------------------------
CREATE TABLE articles (
  id              uuid PRIMARY KEY,
  merchant_id     uuid NOT NULL REFERENCES merchants(id),
  name            text NOT NULL,
  name_norm       text NOT NULL,                     -- unaccent(lower(name))
  aliases         text[] NOT NULL DEFAULT '{}',      -- « riz », « riz 25 », « sac riz »
  stock_unit      text NOT NULL DEFAULT 'unite',     -- sac, kg, litre, bidon, carton, unite…
  sale_price_xof  bigint,                            -- NULL tant que non fixé (RG-ACH-13)
  vat_code        text,                              -- NULL = défaut du merchant
  barcode         text,
  photo_file_id   uuid,
  alert_threshold_milli bigint NOT NULL DEFAULT 0,
  avg_cost_xof    bigint,                            -- coût moyen pondéré (projection)
  is_perishable   boolean NOT NULL DEFAULT false,
  status          text NOT NULL DEFAULT 'active' CHECK (status IN ('draft','active','archived')),
  origin          text NOT NULL DEFAULT 'manual' CHECK (origin IN ('manual','sale','receipt','import')),
  kind            text NOT NULL DEFAULT 'goods' CHECK (kind IN ('goods','service'))   -- service : jamais de mouvement de stock
);
CREATE INDEX articles_name_trgm ON articles USING gin (name_norm gin_trgm_ops);

CREATE TABLE stock_levels (                          -- projection par boutique
  shop_id uuid NOT NULL REFERENCES shops(id),
  article_id uuid NOT NULL REFERENCES articles(id),
  merchant_id uuid NOT NULL,
  qty_milli bigint NOT NULL DEFAULT 0,
  reserved_milli bigint NOT NULL DEFAULT 0,
  PRIMARY KEY (shop_id, article_id)
);

CREATE TABLE stock_movements (                       -- journal immuable
  id           bigserial PRIMARY KEY,
  merchant_id  uuid NOT NULL,
  shop_id      uuid NOT NULL REFERENCES shops(id),
  article_id   uuid NOT NULL REFERENCES articles(id),
  kind         text NOT NULL CHECK (kind IN ('sale','sale_return','receipt','supplier_return',
                 'loss','breakage','expiry','internal_use','owner_withdrawal','adjustment',
                 'unpack_out','unpack_in','opening')),
  qty_milli    bigint NOT NULL CHECK (qty_milli <> 0),   -- signé
  unit_cost_xof bigint,
  source_type  text,                                     -- sale, receipt, count, …
  source_id    uuid,
  reason       text,
  user_id      uuid,
  occurred_at  timestamptz NOT NULL DEFAULT now()
);

-- Clients et créances ---------------------------------------------------
CREATE TABLE customers (
  id            uuid PRIMARY KEY,
  merchant_id   uuid NOT NULL REFERENCES merchants(id),
  kind          text NOT NULL CHECK (kind IN ('individual','business','government','foreign')),
  name          text NOT NULL,
  name_norm     text NOT NULL,
  phone_e164    text,
  email         text,
  ncc           text,
  credit_limit_xof bigint NOT NULL DEFAULT 0,          -- 0 = pas de limite
  marketing_opt_in boolean NOT NULL DEFAULT false,
  opted_out     boolean NOT NULL DEFAULT false,         -- STOP
  archived_at   timestamptz,
  CONSTRAINT ncc_if_business CHECK (kind <> 'business' OR ncc IS NOT NULL)
);
CREATE INDEX customers_name_trgm ON customers USING gin (name_norm gin_trgm_ops);
CREATE UNIQUE INDEX customers_phone ON customers (merchant_id, phone_e164) WHERE phone_e164 IS NOT NULL;

CREATE TABLE receivables (
  id             uuid PRIMARY KEY,
  merchant_id    uuid NOT NULL,
  shop_id        uuid NOT NULL REFERENCES shops(id),
  customer_id    uuid NOT NULL REFERENCES customers(id),
  sale_id        uuid,
  amount_xof     bigint NOT NULL CHECK (amount_xof > 0),
  remaining_xof  bigint NOT NULL CHECK (remaining_xof >= 0),
  due_date       date,
  status         text NOT NULL DEFAULT 'open'
                 CHECK (status IN ('open','partial','settled','written_off'))
);

CREATE TABLE customer_payments (
  id            uuid PRIMARY KEY,
  merchant_id   uuid NOT NULL,
  shop_id       uuid NOT NULL,
  customer_id   uuid NOT NULL REFERENCES customers(id),
  amount_xof    bigint NOT NULL CHECK (amount_xof > 0),
  method        text NOT NULL,
  allocations   jsonb NOT NULL,                        -- [{receivable_id, amount_xof}]
  treasury_movement_id bigint,
  received_at   timestamptz NOT NULL
);

-- Ventes et factures ----------------------------------------------------
CREATE TABLE sales (
  id              uuid PRIMARY KEY,
  merchant_id     uuid NOT NULL,
  shop_id         uuid NOT NULL REFERENCES shops(id),
  internal_number text NOT NULL,                        -- VTE-2026-000123
  customer_id     uuid REFERENCES customers(id),        -- NULL = client de passage
  seller_user_id  uuid NOT NULL REFERENCES users(id),
  status          text NOT NULL CHECK (status IN ('confirmed','cancelled')),
  channel         text NOT NULL CHECK (channel IN ('whatsapp','telegram','web','api')),
  input_mode      text NOT NULL CHECK (input_mode IN ('text','voice','photo','form')),
  total_ht_xof    bigint NOT NULL,
  total_vat_xof   bigint NOT NULL,
  total_ttc_xof   bigint NOT NULL,
  paid_xof        bigint NOT NULL,                      -- partie encaissée
  credit_xof      bigint NOT NULL,                      -- partie à crédit
  payments        jsonb NOT NULL,                       -- [{method, amount_xof}]
  pending_action_id uuid,
  idempotency_key text,
  occurred_at     timestamptz NOT NULL,
  CONSTRAINT totals_ok CHECK (total_ttc_xof = total_ht_xof + total_vat_xof
                              AND paid_xof + credit_xof = total_ttc_xof),
  UNIQUE (shop_id, internal_number)
);
CREATE UNIQUE INDEX sales_idem ON sales (merchant_id, idempotency_key) WHERE idempotency_key IS NOT NULL;

CREATE TABLE sale_lines (
  id              uuid PRIMARY KEY,
  merchant_id     uuid NOT NULL,
  sale_id         uuid NOT NULL REFERENCES sales(id),
  article_id      uuid REFERENCES articles(id),
  label           text NOT NULL,                        -- figé à la vente
  qty_milli       bigint NOT NULL CHECK (qty_milli > 0),
  unit            text NOT NULL,
  unit_price_ttc_xof bigint NOT NULL,
  discount_bp     int NOT NULL DEFAULT 0,
  vat_code        text NOT NULL,
  line_ht_xof     bigint NOT NULL,
  line_vat_xof    bigint NOT NULL,
  line_ttc_xof    bigint NOT NULL
);

CREATE TABLE invoices (                                -- facture FNE (ou reçu non certifié)
  id               uuid PRIMARY KEY,
  merchant_id      uuid NOT NULL,
  sale_id          uuid NOT NULL UNIQUE REFERENCES sales(id),
  kind             text NOT NULL CHECK (kind IN ('fne_invoice','receipt')),
  template         text CHECK (template IN ('B2C','B2B','B2G','B2F')),
  status           text NOT NULL DEFAULT 'pending'
                   CHECK (status IN ('not_applicable','pending','sending','certified',
                                     'rejected','ambiguous','failed')),
  attempts         int NOT NULL DEFAULT 0,
  next_attempt_at  timestamptz,
  dgi_invoice_id   text,                                -- invoice.id renvoyé par la DGI
  dgi_reference    text,                                -- numéro normatif (ex. 9606123E25000000019)
  dgi_verify_url   text,                                -- 'token' : URL de vérification → QR
  sticker_balance  int,
  dgi_warning      boolean,
  dgi_response     jsonb,                               -- réponse brute (archive)
  last_error       text,
  provisional_hash text,                                -- empreinte du reçu provisoire
  pdf_file_id      uuid,
  certified_at     timestamptz
);
CREATE UNIQUE INDEX invoices_dgi_ref ON invoices (dgi_reference) WHERE dgi_reference IS NOT NULL;

CREATE TABLE invoice_lines (                            -- correspondance ligne KAYE ↔ item DGI
  invoice_id   uuid NOT NULL REFERENCES invoices(id),
  sale_line_id uuid NOT NULL REFERENCES sale_lines(id),
  merchant_id  uuid NOT NULL,
  dgi_item_id  text,                                   -- requis pour les avoirs
  PRIMARY KEY (invoice_id, sale_line_id)
);

CREATE TABLE credit_notes (                             -- avoirs
  id             uuid PRIMARY KEY,
  merchant_id    uuid NOT NULL,
  invoice_id     uuid NOT NULL REFERENCES invoices(id),
  internal_number text NOT NULL,                       -- AVO-2026-000004
  items          jsonb NOT NULL,                       -- [{sale_line_id, dgi_item_id, qty_milli}]
  reason         text NOT NULL,
  status         text NOT NULL DEFAULT 'pending',      -- mêmes statuts que invoices
  dgi_reference  text, dgi_verify_url text, dgi_response jsonb,
  pdf_file_id    uuid
);

-- Achats et fournisseurs (MVP : achat direct) ----------------------------
CREATE TABLE suppliers (
  id uuid PRIMARY KEY, merchant_id uuid NOT NULL,
  name text NOT NULL, name_norm text NOT NULL,
  phone_e164 text, ncc text, payment_terms_days int NOT NULL DEFAULT 0,
  archived_at timestamptz
);

CREATE TABLE article_suppliers (
  id uuid PRIMARY KEY, merchant_id uuid NOT NULL,
  article_id uuid NOT NULL REFERENCES articles(id),
  supplier_id uuid NOT NULL REFERENCES suppliers(id),
  supplier_ref text,                                  -- référence sur le BL
  supplier_label text,
  purchase_unit text NOT NULL,
  units_per_purchase_unit_milli bigint NOT NULL DEFAULT 1000,  -- 1 carton = 12 → 12000
  last_price_xof bigint,
  discount_bp int NOT NULL DEFAULT 0,
  valid_until date,
  UNIQUE (supplier_id, supplier_ref)
);

CREATE TABLE purchase_receipts (
  id uuid PRIMARY KEY, merchant_id uuid NOT NULL,
  shop_id uuid NOT NULL, supplier_id uuid NOT NULL REFERENCES suppliers(id),
  internal_number text NOT NULL,                      -- REC-2026-000001
  purchase_order_id uuid,                             -- V1
  source_file_id uuid,                                -- photo du BL
  status text NOT NULL CHECK (status IN ('confirmed','partially_referenced','cancelled')),
  total_xof bigint NOT NULL,
  paid_xof bigint NOT NULL DEFAULT 0,
  idempotency_key text,
  occurred_at timestamptz NOT NULL
);

CREATE TABLE receipt_lines (
  id uuid PRIMARY KEY, merchant_id uuid NOT NULL,
  receipt_id uuid NOT NULL REFERENCES purchase_receipts(id),
  article_id uuid REFERENCES articles(id),            -- NULL tant que 'to_reference'
  supplier_ref text, supplier_label text NOT NULL,
  qty_accepted_milli bigint NOT NULL, qty_refused_milli bigint NOT NULL DEFAULT 0,
  purchase_unit text, unit_price_xof bigint NOT NULL,
  ocr_confidence numeric(4,3),
  line_status text NOT NULL CHECK (line_status IN ('to_reference','in_stock','refused'))
);

CREATE TABLE payables (                                 -- dette fournisseur par pièce
  id uuid PRIMARY KEY, merchant_id uuid NOT NULL,
  supplier_id uuid NOT NULL REFERENCES suppliers(id),
  source_type text NOT NULL, source_id uuid NOT NULL,  -- receipt, supplier_invoice
  amount_xof bigint NOT NULL, remaining_xof bigint NOT NULL,
  due_date date,
  status text NOT NULL DEFAULT 'open' CHECK (status IN ('open','partial','settled','disputed'))
);

CREATE TABLE supplier_payments (
  id uuid PRIMARY KEY, merchant_id uuid NOT NULL, shop_id uuid NOT NULL,
  supplier_id uuid NOT NULL, internal_number text NOT NULL,   -- REG-2026-000001
  amount_xof bigint NOT NULL, method text NOT NULL,
  allocations jsonb NOT NULL, treasury_movement_id bigint,
  proof_file_id uuid, paid_at timestamptz NOT NULL
);

-- Caisse et trésorerie --------------------------------------------------
CREATE TABLE treasury_accounts (
  id uuid PRIMARY KEY, merchant_id uuid NOT NULL, shop_id uuid NOT NULL,
  kind text NOT NULL CHECK (kind IN ('cash','wave','orange_money','mtn_momo','moov_money','bank','other')),
  label text NOT NULL, UNIQUE (shop_id, kind, label)
);

CREATE TABLE cash_sessions (
  id uuid PRIMARY KEY, merchant_id uuid NOT NULL, shop_id uuid NOT NULL,
  business_date date NOT NULL,
  opening_cash_xof bigint NOT NULL,
  counted_cash_xof bigint, variance_xof bigint, variance_reason text,
  closed_at timestamptz, closed_by uuid,
  UNIQUE (shop_id, business_date)
);

CREATE TABLE treasury_movements (                       -- journal immuable
  id bigserial PRIMARY KEY,
  merchant_id uuid NOT NULL, shop_id uuid NOT NULL,
  account_id uuid NOT NULL REFERENCES treasury_accounts(id),
  cash_session_id uuid REFERENCES cash_sessions(id),
  direction text NOT NULL CHECK (direction IN ('in','out')),
  nature text NOT NULL CHECK (nature IN ('sale','customer_payment','supplier_payment','expense',
                 'transfer','owner_withdrawal','owner_contribution','refund','adjustment')),
  amount_xof bigint NOT NULL CHECK (amount_xof > 0),
  label text, category_id uuid,
  source_type text, source_id uuid,
  transfer_group uuid,                                  -- relie les deux jambes d'un transfert
  user_id uuid, occurred_at timestamptz NOT NULL
);

CREATE TABLE expenses (
  id uuid PRIMARY KEY, merchant_id uuid NOT NULL, shop_id uuid NOT NULL,
  internal_number text NOT NULL,                       -- DEP-2026-000001
  category_id uuid, label text NOT NULL, amount_xof bigint NOT NULL,
  method text NOT NULL, proof_file_id uuid, without_proof boolean NOT NULL DEFAULT false,
  occurred_at timestamptz NOT NULL
);

-- Conversation et agent -------------------------------------------------
CREATE TABLE conversations (
  id uuid PRIMARY KEY, merchant_id uuid,               -- NULL pendant l'inscription
  channel text NOT NULL, external_id text NOT NULL,     -- wa_id / chat_id / web session
  kind text NOT NULL CHECK (kind IN ('pilot','commercial')),
  contact_id uuid,                                      -- canal commercial (V1)
  state jsonb NOT NULL DEFAULT '{}',                    -- étape d'inscription, contexte court
  UNIQUE (channel, external_id, kind)
);

CREATE TABLE messages (
  id bigserial PRIMARY KEY,
  merchant_id uuid, conversation_id uuid NOT NULL REFERENCES conversations(id),
  direction text NOT NULL CHECK (direction IN ('in','out')),
  provider_message_id text,                             -- dédoublonnage
  type text NOT NULL,                                   -- text, audio, image, document, button, template
  body text, media_file_id uuid, payload jsonb,
  transcript text,                                      -- vocal transcrit
  status text,                                          -- sent, delivered, read, failed
  created_at timestamptz NOT NULL DEFAULT now(),
  UNIQUE (provider_message_id)
);

CREATE TABLE pending_actions (                          -- propositions en attente de confirmation
  id uuid PRIMARY KEY,
  merchant_id uuid NOT NULL, shop_id uuid NOT NULL,
  conversation_id uuid NOT NULL, user_id uuid NOT NULL,
  intent text NOT NULL,
  payload jsonb NOT NULL,                               -- commande typée prête à exécuter
  warnings jsonb NOT NULL DEFAULT '[]',
  requires_pin boolean NOT NULL DEFAULT false,
  requires_approval_by text,                            -- 'owner' si soumis au patron
  status text NOT NULL DEFAULT 'awaiting'
         CHECK (status IN ('awaiting','confirmed','cancelled','expired','executed','failed')),
  nonce text NOT NULL,                                  -- présent dans les boutons
  nlu_log_id bigint,
  expires_at timestamptz NOT NULL,
  executed_ref jsonb                                    -- {type, id} de l'objet créé
);

CREATE TABLE outbound_requests (                        -- envois vers des tiers à valider (ADR-10)
  id uuid PRIMARY KEY, merchant_id uuid NOT NULL,
  requested_by uuid NOT NULL, kind text NOT NULL,       -- reminder, quote, invoice, purchase_order…
  recipients jsonb NOT NULL, preview text NOT NULL,
  estimated_cost_xof bigint, status text NOT NULL DEFAULT 'to_validate'
    CHECK (status IN ('to_validate','validated','refused','expired','sent','failed')),
  expires_at timestamptz NOT NULL
);

CREATE TABLE nlu_logs (
  id bigserial PRIMARY KEY, merchant_id uuid, user_id uuid,
  raw_text text NOT NULL, normalized text, input_mode text,
  intent text, entities jsonb, final_entities jsonb,
  path text CHECK (path IN ('rules','llm','hybrid')),
  confidence numeric(4,3), model text, latency_ms int, cost_micro_eur int,
  corrected boolean NOT NULL DEFAULT false,
  prompt_version text,                                  -- version du gabarit d'invite utilisé
  created_at timestamptz NOT NULL DEFAULT now()
);

-- Transverse -------------------------------------------------------------
CREATE TABLE document_counters (                        -- numérotation interne sans trou
  merchant_id uuid NOT NULL, shop_id uuid NOT NULL,
  doc_type text NOT NULL, year int NOT NULL,
  last_value bigint NOT NULL DEFAULT 0,
  PRIMARY KEY (shop_id, doc_type, year)
);

CREATE TABLE files (
  id uuid PRIMARY KEY, merchant_id uuid,
  kind text NOT NULL,                                   -- invoice_pdf, delivery_note_photo, voice, …
  storage_key text NOT NULL, mime text NOT NULL, bytes int NOT NULL, sha256 text NOT NULL,
  retention_until date
);

CREATE TABLE fne_credentials (
  merchant_id uuid PRIMARY KEY REFERENCES merchants(id),
  environment text NOT NULL CHECK (environment IN ('test','production')),
  api_key_enc bytea NOT NULL,                           -- chiffré (AES-256-GCM, § 13.3)
  validated_at timestamptz
);

CREATE TABLE outbox_events (
  id bigserial PRIMARY KEY, merchant_id uuid,
  type text NOT NULL, payload jsonb NOT NULL,
  created_at timestamptz NOT NULL DEFAULT now(), dispatched_at timestamptz
);

CREATE TABLE audit_log (
  id bigserial PRIMARY KEY, merchant_id uuid NOT NULL, user_id uuid,
  action text NOT NULL, entity text NOT NULL, entity_id text,
  before jsonb, after jsonb, channel text, message_id bigint,
  created_at timestamptz NOT NULL DEFAULT now()
);

-- Abonnement -------------------------------------------------------------
CREATE TABLE passes (
  id uuid PRIMARY KEY, merchant_id uuid NOT NULL,
  plan text NOT NULL, months int NOT NULL, amount_xof bigint NOT NULL,
  method text NOT NULL, provider_ref text, proof_file_id uuid,
  status text NOT NULL CHECK (status IN ('pending','active','expired','refunded')),
  starts_at timestamptz, ends_at timestamptz, activated_by uuid
);
CREATE TABLE usage_counters (
  merchant_id uuid NOT NULL, period char(7) NOT NULL,   -- '2026-11'
  metric text NOT NULL, value int NOT NULL DEFAULT 0,   -- fne_invoices, llm_calls, wa_templates…
  PRIMARY KEY (merchant_id, period, metric)
);
```

### 5.4 Tables V1 (à créer avec leurs lots)
| Table | Clés principales | Lot |
|---|---|---|
| `contacts` | `wa_id`, `label` (client, prospect, supplier, staff, unknown), `customer_id`, `supplier_id`, `marketing_opt_in` | V1 canal commercial |
| `whatsapp_accounts` | `waba_id`, `phone_number_id`, `business_token_enc`, `mode`, `quality`, `status` | V1 |
| `quotes`, `quote_lines` | `DEV`, validité, statut | V1 |
| `sales_orders`, `sales_order_lines` | `CMD`, `qty_ordered/reserved/delivered_milli`, acompte | V1 |
| `deliveries` | `BL`, preuve | V1 |
| `customer_returns` | `RTC`, destination stock/rebut, contrepartie | MVP (simple) |
| `price_lists` | article, type (détail, gros), client, prix, validité | V1 |
| `unserved_requests` | article, qté, contact | V1 |
| `purchase_orders`, `purchase_order_lines` | `BCA`, prix figé, qté commandée/reçue | V1 |
| `price_requests` | `DPX`, réponses | V1 |
| `supplier_returns` | `RET`, motif, contrepartie | V1 |
| `supplier_invoices` | n° fournisseur, HT, TVA, TTC, échéance, statut contrôle | V1 |
| `unpack_rules` | article achat → article vente, rapport | V1 |
| `stock_counts`, `stock_count_lines`, `stock_adjustments` | `CPT`, `AJU`, attendu / compté, motif, validé par | V1 |
| `expiry_lots` | article, date, qté restante | V1 |
| `categories` | libellé, compte SYSCOHADA | V1 |
| `reconciliations` | compte, période, solde relevé / KAYE | V2 |
| `inbound_emails` | adresse, expéditeur, pièces, statut | V1 |

### 5.5 Écritures composites (transactions uniques)

**Vente confirmée** (`SalesService.confirmSale`) :
1. Contrôle d'idempotence (`merchant_id`, `idempotency_key` = `pending_action.id`).
2. Numéro interne `VTE` via `document_counters` (`UPDATE … RETURNING`, verrou de ligne).
3. Insertion `sales` + `sale_lines` (calcul HT/TVA/TTC par `packages/domain/tax`).
4. Un `stock_movements` négatif par ligne avec article ; mise à jour de `stock_levels` (même transaction). Stock négatif autorisé (RG-1).
5. Pour chaque paiement immédiat : `treasury_movements` (`in`, `sale`) sur le compte du mode, rattaché à la `cash_session` du jour (créée si absente).
6. Partie à crédit : `receivables`.
7. `invoices` : `pending` si FNE active, sinon `kind='receipt'`, `status='not_applicable'`.
8. `audit_log` + `outbox_events` (`sale.confirmed`, `invoice.certify_requested`, `document.render_requested`).

**Réception fournisseur** (`PurchasingService.confirmReceipt`) : idempotence → `REC` → lignes → mouvements positifs convertis en unité de stock pour les lignes `in_stock` → recalcul du CMP (`domain/costing`) → `payables` pour la part non payée → `treasury_movements` pour la part payée → outbox.

**Règlement client** : allocation FIFO sur les créances ouvertes (sauf allocation explicite) → `customer_payments` → `treasury_movements` → mise à jour des `receivables`. La part non allouée (avance) et les avoirs non remboursés forment le **solde créditeur** du client (EF-M2-11) : vue `customer_balances` = Σ créances restantes − Σ non alloué − Σ avoirs non remboursés ; à la vente suivante, l'agent propose de l'imputer (paiement de nature `customer_credit`, sans mouvement de trésorerie).

**Annulation d'une vente** : statut `cancelled`, mouvements de stock compensatoires, mouvement de trésorerie `refund` si remboursement, créance soldée en `written_off` = non ; si facture certifiée → `credit_notes` + outbox `credit_note.certify_requested`.

### 5.6 Coût moyen pondéré
`nouveau_CMP = (stock_avant × CMP_avant + qté_reçue × coût_unitaire) / (stock_avant + qté_reçue)` ; si `stock_avant ≤ 0`, `nouveau_CMP = coût_unitaire`. Arrondi à l'entier XOF le plus proche. Implémenté et testé dans `packages/domain/costing.ts`.

### 5.7 TVA et montants
- Le commerçant saisit des **prix TTC** (ce que paie le client).
- Codes DGI : `TVA` (18 %), `TVAB` (9 %), `TVAC` (0 %, exonération conventionnelle), `TVAD` (0 %, exonération légale — TEE et RME). Défaut par régime : `TEE`/`RME` → `TVAD` ; `RSI`/`RNI` → `TVA`.
- `HT unitaire = TTC unitaire / (1 + taux)` calculé en décimal exact puis transmis à la DGI ; les totaux KAYE sont recalculés à partir de la réponse DGI après certification (`dgi_response.invoice.amount`, `vatAmount`) et un écart > 1 F par facture déclenche une alerte back-office (point § 17).

### 5.8 Projections et réconciliations nocturnes
- `stock_levels` = Σ `stock_movements` ; `treasury` = Σ `treasury_movements` ; `receivables.remaining` = montant − allocations.
- Tâche nocturne : recalcul et alerte sur tout écart ; contrôle de continuité des `document_counters` ; factures `pending` > 6 h.

---

## 6. Moteur d'agent conversationnel

### 6.1 Chaîne de traitement
```
Webhook canal ──► persistance brute + dédoublonnage (provider_message_id) ──► 200 OK
      │
      ▼ file `inbound` (séquentielle par conversation)
1. Résolution d'identité : channel_identity → user → merchant, shop actif, rôle
   (inconnu → parcours d'inscription, aucune donnée)
2. Réponse à une proposition ? (bouton avec nonce, « oui », « non », « corrige … »)
   → confirmer / annuler / corriger la pending_action (aller en 8)
3. Pré-traitement média : audio → transcription ; image/PDF → extraction de document
4. Normalisation (nlu-core) : minuscules, accents, nombres (« 45k », « 45 mille »),
   opérateurs (« om » → orange_money), téléphones (E.164), élisions
5. Classification + extraction : analyseur à règles ; si confiance < seuil ou intention
   non couverte → LLM (sortie JSON contrainte par le schéma Zod de l'intention)
6. Résolution d'entités : clients, articles, fournisseurs par trigrammes (§ 6.5)
7. Validation métier (domain) → avertissements ; contrôle des droits (rôle, PIN)
8. Si l'intention écrit : création d'une pending_action + rendu du récapitulatif avec boutons
   Si l'intention lit : exécution de la requête + rendu du résultat
9. Sur confirmation : exécution du command handler du domaine (transaction § 5.5),
   puis rendu du résultat ; journalisation nlu_logs (entités finales, corrigé ou non)
```

### 6.2 Registre des intentions
Chaque intention est déclarée une seule fois dans `packages/agent/src/intents/<nom>.ts` :

```ts
export const recordSale = defineIntent({
  name: "sale.record",
  lot: "MVP",
  kind: "write",                       // write | read | navigation
  roles: ["owner", "manager", "seller"],
  requiresPin: (ctx, slots) => false,
  slots: SaleSlotsSchema,              // Zod : lines[], customer?, payments[], credit?
  rules: [saleRules],                  // nlu-core : motifs « vendu », « j'ai gbê », « vente »
  llmHint: "Vente au comptoir ou à crédit…",  // description pour le repli LLM
  resolve: resolveSaleEntities,        // clients, articles, prix catalogue
  validate: validateSale,              // VM-01…VM-07 → erreurs bloquantes / avertissements
  propose: renderSaleProposal,         // gabarit du récapitulatif
  execute: (svc, cmd) => svc.sales.confirmSale(cmd),
  renderResult: renderSaleResult,
});
```

Intentions MVP : `help`, `onboarding.*`, `sale.record`, `sale.return`, `sale.list`, `invoice.find`, `invoice.resend`, `invoice.pending`, `customer.create`, `customer.list`, `customer.set_limit`, `receivable.show`, `receivable.list`, `receivable.pay`, `reminder.send`, `receipt.from_document`, `receipt.pay`, `supplier.create`, `payable.show`, `supplier.pay`, `article.create_from_line`, `article.link_line`, `stock.show`, `stock.alerts`, `stock.out` (casse, perte, prélèvement), `stock.set`, `cash.today`, `cash.in_out`, `cash.close`, `expense.record`, `team.add`, `team.remove`, `shop.switch`, `unknown`.

### 6.3 Seuils et repli
| Paramètre | Valeur par défaut |
|---|---|
| Confiance minimale du chemin règles | 0,85 |
| Champ obligatoire manquant | confiance plafonnée à 0,4 → LLM |
| Timeout LLM extraction | 4 s, 1 nouvelle tentative, puis question guidée |
| Expiration d'une proposition | 24 h (puis `expired`) |
| Correction | « non c'est 40 000 » → re-extraction **sur la proposition en cours** (slots fusionnés) |

### 6.4 Contrôles de validation (domain)
| Code | Contrôle | Niveau |
|---|---|---|
| VM-01 | Montant > 0 | bloquant |
| VM-02 | Au moins une ligne | bloquant |
| VM-03 | Stock insuffisant | avertissement (vente acceptée) |
| VM-04 | Limite de crédit dépassée | avertissement + approbation selon rôle |
| VM-05 | Unité saisie ≠ unité de l'article | avertissement |
| VM-06 | Prix > 3× ou < 1/3 du prix habituel | avertissement |
| VM-07 | Doublon probable (même client, montant, article < 60 s) | avertissement |
| VM-08 | Client professionnel sans NCC en FNE B2B | bloquant pour la certification, vente acceptée en reçu |
| VM-09 | Article sans prix de vente | question (prix demandé) |
| VM-10 | Ligne de document lue sous le seuil de confiance | question (jamais deviné) |

### 6.5 Résolution d'entités
```sql
SELECT id, name, similarity(name_norm, $1) AS score
FROM customers WHERE merchant_id = $2 AND archived_at IS NULL AND name_norm % $1
ORDER BY score DESC LIMIT 5;
```
| Score | Comportement |
|---|---|
| ≥ 0,80 et écart ≥ 0,15 avec le 2e | appariement automatique (affiché dans le récapitulatif) |
| ≥ 0,45 avec plusieurs candidats | question à choix (boutons ou liste) |
| < 0,45 | proposition de création |

Articles : seuil relevé à 0,85, recherche aussi dans `aliases` et `article_suppliers.supplier_ref`.

### 6.6 Rendu des messages
- `packages/messages` contient des gabarits FR typés : `saleProposal(data)`, `saleConfirmed(data)`, …
- Un gabarit produit une structure neutre `{ text, buttons[], list?, attachments[] }` ; chaque adaptateur de canal la traduit (WhatsApp : 3 boutons max ou liste de 10 ; Telegram : clavier inline ; web : carte).
- Formatage des montants : `formatXof(45000)` → « 45 000 F CFA » (espace insécable).
- Ton : tutoiement ou vouvoiement selon le réglage de la boutique (défaut : vouvoiement), phrases courtes, pas d'émoji en guise d'icône.

Exemple de récapitulatif :
```
Vente à confirmer
• 3 sacs Riz 25 kg × 15 000 F = 45 000 F CFA
• Client : Awa Koné
• Paiement : Wave
⚠ Stock riz : 2 sacs enregistrés, la vente passera quand même.
[Confirmer] [Corriger] [Annuler]
```

### 6.7 Parcours d'inscription (canal de pilotage)
1. Message d'un numéro inconnu → « Bienvenue sur KAYE… Comment s'appelle votre boutique ? »
2. Activité → nom du patron → NCC (« passer » autorisé) → création `merchant`, `shop`, `user (owner)`, `channel_identity`, `treasury_accounts` par défaut.
3. Proposition immédiate d'une première vente d'exemple (RG-6 : aucun paramétrage obligatoire).
4. Code de liaison Telegram et lien vers l'application web envoyés à la fin.
Durée cible < 60 s ; état dans `conversations.state`.

### 6.8 PIN et approbations
- Actions avec PIN : règlement fournisseur, prix de vente, ajustement de stock au-delà du seuil, envoi groupé, annulation de facture, paramètres.
- Sur WhatsApp, le PIN est demandé dans un message séparé et **supprimé du stockage** après vérification (le corps du message n'est pas conservé ; seul `pin_ok` est journalisé).
- Une action d'employé hors droits crée une `pending_action` avec `requires_approval_by='owner'` ; le patron reçoit la demande et valide.

---

## 7. Passerelle de modèles (LLM, transcription, lecture de documents)

### 7.1 Interface
```ts
interface LlmGateway {
  extract<T>(task: "intent" | "slots", input: ExtractInput, schema: ZodType<T>): Promise<LlmResult<T>>;
  readDocument(input: { fileId: string; hint: "delivery_note" | "invoice" | "payment_capture" | "receipt" | "product_photo" }): Promise<LlmResult<DocumentExtraction>>;
  transcribe(input: { fileId: string; language: "fr" }): Promise<LlmResult<{ text: string; confidence?: number }>>;
}
```
`LlmResult` contient : `data`, `provider`, `model`, `latencyMs`, `costMicroEur`, `raw` (non journalisé en clair si données sensibles).

### 7.2 Routage par tâche (configurable par variable d'environnement)
| Tâche | Fournisseur par défaut | Secours | Remarque |
|---|---|---|---|
| Extraction d'intention et de champs | Mistral (petit modèle, sortie JSON) | Anthropic (palier rapide) | température 0, schéma imposé |
| Lecture de BL, factures, captures | Mistral (OCR / vision) | Anthropic ou OpenAI (vision) | sortie : lignes + confiance par ligne |
| Transcription vocale | Mistral (Voxtral) ou Whisper | l'autre | français, corpus ivoirien à mesurer |

Les noms exacts de modèles sont **des paramètres** (`LLM_MODEL_EXTRACT`, `LLM_MODEL_VISION`, `ASR_MODEL`) et sont choisis après le banc d'essai du lot 0 sur le corpus réel (coût, latence, exactitude des montants). Ne pas les coder en dur.

### 7.3 Règles
- Le texte utilisateur est passé dans un **bloc de données délimité** ; le prompt système interdit toute action ; la sortie est validée par Zod et **ne peut rien déclencher seule** (protection contre l'injection d'instructions).
- Contexte injecté : au plus 200 articles et 200 clients les plus probables (pré-filtrés par trigrammes), jamais le NCC ni les téléphones complets.
- Coût : compteur `usage_counters` (`llm_calls`, `llm_cost_micro_eur`) ; plafond mensuel par commerçant (`LLM_MONTHLY_CAP_EUR`, défaut 1,20 €) au-delà duquel seul le chemin règles + saisie guidée est utilisé.
- Interrupteur global `NLU_LLM_ENABLED`.
- **Masquage avant envoi** : numéros de téléphone, NCC et e-mails sont remplacés par des marqueurs (`<TEL_1>`, `<NCC_1>`) avant l'appel au modèle, puis restaurés dans la sortie.
- **Prompts versionnés** : chaque gabarit d'invite vit dans `packages/llm-gateway/prompts/<tâche>.v<N>.md` ; la version est journalisée (`nlu_logs.prompt_version`) ; changer un prompt = nouvelle version + passage du corpus.
- Les échanges avec les fournisseurs de modèles sont listés comme sous-traitants (RGPD) ; option « pas de conservation » activée quand le fournisseur la propose.

### 7.4 Lecture d'un BL (sortie attendue)
```json
{
  "document_type": "delivery_note",
  "supplier": { "name": "SOW DISTRIBUTION", "phone": null, "ncc": null },
  "number": "0457", "date": "2026-10-28",
  "lines": [
    { "ref": "SW-RZ25", "label": "RIZ 25 KG", "qty": 18, "unit": "sac", "unit_price": 16500, "confidence": 0.94 },
    { "ref": "SW-LN400", "label": "LAIT NIDO 400G", "qty": 24, "unit": "pot", "unit_price": null, "confidence": 0.71 }
  ],
  "total": null
}
```
Toute ligne `confidence < OCR_LINE_THRESHOLD` (0,85) ou à champ nul est présentée comme question.

---

## 8. Canaux : WhatsApp, Telegram, web, e-mail

### 8.1 Interface commune
```ts
interface ChannelAdapter {
  channel: "whatsapp" | "telegram" | "web" | "test";
  parseWebhook(req: RawRequest): InboundEvent[];      // messages, boutons, statuts
  verifySignature(req: RawRequest): boolean;
  send(to: Recipient, msg: RenderedMessage): Promise<{ providerMessageId: string }>;
  downloadMedia(ref: MediaRef): Promise<Buffer>;
}
```
L'adaptateur `test` (en mémoire) permet d'écrire des tests de conversation de bout en bout sans réseau : **obligatoire pour toute nouvelle intention**.

### 8.2 Telegram (canal de pilotage — livré en premier)
- Bot API en webhook (`setWebhook` avec `secret_token`, vérifié via l'en-tête `X-Telegram-Bot-Api-Secret-Token`).
- Liaison : `/start <code>` (code à 8 caractères, valable 15 min, à usage unique) → `channel_identities`.
- Boutons : `inline_keyboard` avec `callback_data = "<pending_action_id>:<nonce>:<action>"` (≤ 64 octets).
- Vocaux : `getFile` (OGG/Opus) → stockage → transcription.

### 8.3 WhatsApp — canal de pilotage (numéro KAYE)
- WhatsApp Business **Cloud API** (Graph API) ; version fixée par `WA_GRAPH_VERSION`.
- Webhook : vérification `hub.challenge` + signature `X-Hub-Signature-256` (HMAC SHA-256 du corps avec le secret d'application).
- Messages interactifs : `button` (≤ 3 boutons, titre ≤ 20 caractères) et `list` (≤ 10 lignes) ; `id` du bouton = `"<pending_action_id>:<nonce>:<action>"`.
- Médias : `GET /{media-id}` puis téléchargement avec le jeton ; types audio (OGG/Opus), image, document.
- **Fenêtre de 24 h** : `channel_identities.last_inbound_at` ; hors fenêtre, seuls les gabarits approuvés partent (`kaye_resume_soir`, `kaye_alerte`, `kaye_code_connexion`, `kaye_fin_pass`).
- En attendant la vérification Meta : numéro de test Cloud API (destinataires limités) pour le pilote interne.

### 8.4 WhatsApp — canal commercial (V1, ADR-09)
- Programme **Tech Provider**, **Embedded Signup** (SDK JavaScript Meta dans l'application web) avec **Coexistence** ; échange du `code` côté serveur contre un business token **chiffré** (`whatsapp_accounts`).
- Webhook mutualisé, routage par `phone_number_id` ; un `phone_number_id` inconnu est ignoré et journalisé.
- Gabarits dupliqués par WABA à l'onboarding (`facture_client`, `relance_creance`, `relance_echeance`, `accuse_reglement`, `devis_client`, `bon_commande`, `releve_compte`) ; suivi de leur approbation.
- Étiquetage des contacts et matrice de réponse (CDC fonctionnel § 7.2) appliqués **avant** tout envoi ; `outbound_requests` pour toute validation.
- Pause de l'agent sur un fil quand le commerçant y écrit (`echo` des messages envoyés depuis l'application).
- Facturation des messages via le BSP (360dialog, Infobip ou Twilio — choix au lot 0) ; coût estimé affiché avant envoi groupé.

### 8.5 Web (canal de pilotage dans l'application)
- `POST /v1/agent/messages` (texte, fichier) → réponse rendue (`RenderedMessage`) ; `POST /v1/agent/actions/{id}` pour confirmer/annuler.
- Même moteur, mêmes intentions, mêmes gabarits que WhatsApp/Telegram.

### 8.6 E-mail entrant (V1)
- Adresse par boutique `factures-<slug>@in.kaye.ci` via un service d'e-mail entrant (Postmark Inbound ou équivalent) → webhook signé → `inbound_emails` → pièces PDF/images vers la file `media` (même traitement qu'une photo de BL) → proposition envoyée au patron sur son canal de pilotage.
- Expéditeur inconnu : pièce mise en attente, jamais traitée automatiquement.

### 8.7 SMS (secours)
`SmsProvider` pour les codes de connexion quand ni WhatsApp ni Telegram ne sont joignables ; fournisseur local à choisir (lot 0).

---

## 9. Connecteur FNE (API DGI)

Source : DGI, *Procédure d'interfaçage des entreprises par API*, mai 2025.

### 9.1 Ce que dit la procédure
- API REST, JSON, méthode POST, authentification `Authorization: Bearer <API_KEY>` ; la clé est affichée dans l'espace FNE de l'entreprise (onglet *Paramétrage*) **après validation par la DGI**, visible du gestionnaire principal.
- Environnement de test : `http://54.247.95.108/ws` (inscription préalable sur la plateforme de test). URL de production transmise par la DGI après validation des spécimens envoyés à `support.fne@dgi.gouv.ci`.
- Trois opérations : facture de vente (`invoiceType: "sale"`), **avoir**, bordereau d'achat agricole (`"purchase"`).
- La réponse contient : `ncc`, `reference` (**numéro normatif**), `token` (**URL de vérification à convertir en QR code**), `warning` (alerte stock de stickers), `balance_sticker`, `invoice` (détail, avec les `id` d'items).

### 9.2 Interface
```ts
interface FneProvider {
  certifySale(ctx: FneContext, req: FneSaleRequest): Promise<FneCertification>;
  certifyRefund(ctx: FneContext, dgiInvoiceId: string, items: { dgiItemId: string; quantity: number }[]): Promise<FneCertification>;
}
// Implémentations : SimulatedFneProvider (local, CI) · DgiHttpFneProvider (test, production)
// Réservé : AccreditedEditorFneProvider (si la voie retenue passe par un éditeur agréé)
```
`FneContext` = `{ merchantId, environment, apiKey (déchiffrée en mémoire), baseUrl }`.

### 9.3 Correspondance KAYE → DGI (facture de vente)
| Champ DGI | Source KAYE | Règle |
|---|---|---|
| `invoiceType` | — | `"sale"` |
| `paymentMethod` | `sales.payments` | `cash` (espèces) · `mobile-money` (Wave, OM, MTN, Moov) · `transfer` · `check` · `card` · `deferred` (crédit). Paiement mixte : mode du montant le plus élevé ; si une part à crédit existe → `deferred` (à valider § 17) |
| `template` | `customers.kind` | individual → `B2C` · business → `B2B` · government → `B2G` · foreign → `B2F` |
| `isRne` / `rne` | — | `false` (RNE en V2) |
| `clientNcc` | `customers.ncc` | obligatoire si `B2B` |
| `clientCompanyName` | `customers.name` ou « Client comptant » | |
| `clientPhone` | `customers.phone_e164` (10 chiffres locaux) | valeur par défaut à valider avec la DGI pour un client de passage |
| `clientEmail` | `customers.email` | idem |
| `clientSellerName` | `users.display_name` du vendeur | |
| `pointOfSale` / `establishment` | `shops.fne_point_of_sale` / `shops.fne_establishment` | obligatoires ; renseignés à l'activation FNE |
| `commercialMessage` / `footer` | paramètres boutique | `footer` = mention KAYE (EF-M3-10) |
| `foreignCurrency` / `foreignCurrencyRate` | — | `""` / `0` sauf `B2F` |
| `items[].description` | `sale_lines.label` | |
| `items[].reference` | `articles.id` court ou code interne | |
| `items[].quantity` | `qty_milli / 1000` | |
| `items[].amount` | prix unitaire **HT** | § 5.7 |
| `items[].discount` | `discount_bp / 100` | en % |
| `items[].measurementUnit` | `sale_lines.unit` | |
| `items[].taxes` | `[vat_code]` | `TVA`, `TVAB`, `TVAC`, `TVAD` |
| `items[].customTaxes` | — | vide en MVP |
| `discount` (global) | — | `0` en MVP |

### 9.4 Avoir
`POST {baseUrl}/external/invoices/{dgi_invoice_id}/refund` avec `{ "items": [{ "id": "<dgi_item_id>", "quantity": n }] }` → réponse `201` avec `reference` (préfixe `A…`), `token`, `balance_sticker`. D'où l'obligation de stocker `invoice_lines.dgi_item_id` à la certification.

### 9.5 États d'une facture
```
pending ──(worker)──► sending ──200──► certified   (immuable)
   ▲                     │
   │           400 ──────┼──► rejected  (motif affiché, correction → nouvelle vente/annulation)
   │           401 ──────┼──► pending + fne_status='suspended' (clé invalide → alerte patron)
   └── 5xx / réseau ─────┘    (rejeu avec backoff)
              timeout après envoi ──► ambiguous (voir § 9.7)
```

### 9.6 Rejeu
Backoff : 30 s, 1 min, 4 min, 15 min, 1 h, 4 h, 12 h, 24 h (gigue ± 20 %). Après épuisement : `failed`, alerte back-office et commerçant ; reprise manuelle depuis `/ops`. Disjoncteur : au-delà de 50 % d'échecs sur 20 appels, pause 60 s de la file `fne`.

### 9.7 Risque de double certification
L'API ne documente **ni clé d'idempotence ni recherche de facture**. Un timeout *après* envoi peut signifier une facture certifiée côté DGI mais inconnue de KAYE. Règles :
- Timeout de requête long (30 s) ; un seul envoi en vol par facture (verrou Redis).
- Timeout ambigu → statut `ambiguous`, **pas de rejeu automatique** ; contrôle du `balance_sticker` au prochain appel réussi (une baisse de 2 au lieu de 1 signale un doublon) et file de revue back-office.
- Question écrite à la DGI : existe-t-il un endpoint de recherche ou une clé d'idempotence ? (§ 17).

### 9.8 Reçu provisoire
Tant que la facture n'est pas certifiée, le PDF est un **reçu provisoire** : numéro interne, date, lignes, total, mention « Certification FNE en cours », et une empreinte `SHA-256(merchant_id|internal_number|total_ttc|occurred_at|server_secret)` tronquée à 16 caractères, vérifiable via `https://kaye.ci/v/<internal_number>` (page publique minimale). À la certification, le commerçant reçoit « Facture certifiée » et le PDF définitif.

### 9.9 Stickers
`balance_sticker` et `warning` mis à jour à chaque réponse ; alerte au patron sous un seuil (`FNE_STICKER_ALERT`, défaut 50).

### 9.10 Mode simulé
`SimulatedFneProvider` renvoie des réponses conformes aux exemples de la procédure (fixtures `datasets/fne/*.json`), un `reference` fictif préfixé `SIM`, et la mention « SPÉCIMEN — NON CERTIFIÉ » sur le PDF. Il sait simuler 400, 401, 500 et timeout (pour les tests).

### 9.11 Activation FNE d'un commerçant
1. Le patron renseigne NCC, établissement, point de vente, régime fiscal.
2. Il saisit sa clé API (environnement de test puis production) dans l'application web (champ masqué, PIN). Chiffrement immédiat.
3. Facture de test certifiée en environnement de test ; affichage du résultat.
4. `fne_status` : `sandbox` → `pending_validation` (spécimens envoyés à la DGI) → `active`.

> La question de savoir si chaque commerçant doit faire valider son propre interfaçage, ou si la validation de KAYE (éditeur) suffit, est **le verrou n°1** (§ 17). Le modèle ci-dessus fonctionne dans les deux cas.

---

## 10. API REST

### 10.1 Conventions
| Aspect | Règle |
|---|---|
| Base | `https://api.kaye.ci/v1` (domaine à confirmer) |
| Format | JSON UTF-8, champs `snake_case` |
| Auth | `Authorization: Bearer <JWT>` (15 min) + refresh token rotatif (30 j) lié à l'appareil |
| Contexte | en-tête `X-Shop-Id` (boutique active), vérifié contre les rôles |
| Idempotence | `Idempotency-Key` obligatoire sur toute création qui écrit de l'argent ou du stock |
| Pagination | curseur : `?limit=50&cursor=…` → `{ data, next_cursor }` |
| Erreurs | RFC 7807 `application/problem+json` + `code` + `trace_id` |
| Montants | entiers XOF ; quantités en décimal (chaîne) côté API, converties en milli à l'entrée |
| Documentation | OpenAPI générée depuis les schémas Zod, servie en `/v1/docs` (hors production) |

### 10.2 Authentification
| Méthode | Chemin | Description |
|---|---|---|
| POST | `/auth/otp/request` | `{ phone }` → code 6 chiffres envoyé par WhatsApp KAYE, Telegram lié, ou SMS ; 5 demandes/h/numéro |
| POST | `/auth/otp/verify` | `{ phone, code, device_id }` → `{ access_token, refresh_token, user, shops }` |
| POST | `/auth/refresh` · `/auth/logout` | rotation ; réutilisation détectée → révocation de la famille |
| POST | `/auth/telegram-link` | génère le code `/start` |

### 10.3 Points d'entrée MVP
| Domaine | Méthode et chemin |
|---|---|
| Agent | `POST /agent/messages` · `POST /agent/actions/{id}` (`confirm`, `cancel`, `correct`) |
| Boutique | `GET/PATCH /merchant` · `GET/POST /shops` · `GET/POST/DELETE /users` · `PUT /me/pin` |
| FNE | `PUT /fne/credentials` · `POST /fne/test` · `GET /fne/status` |
| Articles | `GET/POST /articles` · `PATCH /articles/{id}` · `GET /articles/search?q=` |
| Stock | `GET /stock?alert=true` · `POST /stock/movements` (motif obligatoire) · `GET /stock/movements?article_id=` |
| Clients | `GET/POST /customers` · `GET/PATCH /customers/{id}` |
| Créances | `GET /receivables` · `GET /receivables/summary` · `POST /customer-payments` · `POST /reminders` |
| Ventes | `POST /sales` · `GET /sales` · `GET /sales/{id}` · `POST /sales/{id}/cancel` · `POST /sales/{id}/returns` |
| Factures | `GET /invoices` · `GET /invoices/{id}` · `GET /invoices/{id}/pdf` (URL signée) · `POST /invoices/{id}/retry` · `POST /invoices/{id}/send` |
| Fournisseurs | `GET/POST /suppliers` · `GET /suppliers/{id}/statement` |
| Réceptions | `POST /receipts/from-document` (fichier → proposition) · `POST /receipts` · `GET /receipts/{id}/unmatched` · `POST /receipts/{id}/lines/{line}/link` · `POST /articles/from-receipt-line` |
| Dettes fournisseurs | `GET /payables` · `POST /supplier-payments` |
| Caisse | `GET /cash/session` · `POST /cash/movements` · `POST /cash/session/close` · `POST /expenses` |
| Tableau de bord | `GET /dashboard` (une requête, < 4 Ko compressés, cache 60 s invalidé à l'écriture) |
| Imports | `POST /imports/{articles|customers|suppliers|opening-balances}` (CSV/XLSX) · `GET /imports/{id}` |
| Fichiers | `POST /files` (URL d'upload présignée) · `GET /files/{id}` |
| Abonnement | `GET /billing` · `POST /passes` (déclaration de paiement + preuve) |
| Webhooks | `POST /webhooks/whatsapp` · `POST /webhooks/telegram` · `POST /webhooks/email` (V1) |
| Back-office | `/ops/*` (rôle opérateur KAYE, hors RLS via rôle dédié, journalisé) |

Les points V1 (devis, commandes, livraisons, commandes d'achat, comptages, trésorerie, comptabilité, canal commercial) sont listés dans `flux-karnet.html` et repris dans le backlog ; ils suivent les mêmes conventions.

### 10.4 Codes d'erreur
`ERR_AUTH_OTP_INVALID` 401 · `ERR_AUTH_TOKEN_EXPIRED` 401 · `ERR_FORBIDDEN_ROLE` 403 · `ERR_PIN_REQUIRED` 403 · `ERR_PIN_LOCKED` 423 · `ERR_NOT_FOUND` 404 · `ERR_IDEMPOTENCY_CONFLICT` 409 · `ERR_SALE_ALREADY_CANCELLED` 409 · `ERR_INVALID_AMOUNT` 422 · `ERR_CREDIT_LIMIT` 422 · `ERR_CASH_SESSION_CLOSED` 422 · `ERR_NLU_AMBIGUOUS` 422 · `ERR_PLAN_LIMIT` 402 · `ERR_RATE_LIMIT` 429 · `ERR_FNE_UNAVAILABLE` 503.

---

## 11. Application web

### 11.1 Écrans MVP
| Route | Écran | Notes |
|---|---|---|
| `/connexion` | Numéro → code | |
| `/` | **Accueil chat** | grand bloc bissap « Bonjour ! Qu'est-ce qu'on fait pour la boutique ? », champ « Écris ta demande… », micro mangue, 4 cartes (stock, ventes, commandes, caisse) ; colonne gauche : nouvelle conversation, récents, canaux connectés (WhatsApp, Telegram) |
| `/tableau-de-bord` | Indicateurs | caisse, ventes du jour, créances, stock en alerte, FNE en attente, top débiteurs |
| `/ventes` · `/ventes/:id` | Liste + détail | filtres période, client, mode, statut FNE |
| `/factures` | Liste | statut DGI, renvoi, téléchargement |
| `/clients` · `/clients/:id` | Carnet + fiche | encours, % limite, historique, relance |
| `/creances` | Débiteurs | tri par montant / ancienneté, relance groupée |
| `/achats` · `/fournisseurs` | Réceptions, dettes | lignes à référencer |
| `/stock` | Inventaire | alertes, mouvements, ajustement |
| `/caisse` | Session du jour | encaissements, décaissements, clôture |
| `/import` | Imports CSV | modèles téléchargeables |
| `/parametres` | Boutique, FNE, équipe, PIN, seuils, notifications, abonnement | |
| `/ops/*` | Back-office | comptes, files FNE, corpus, support |

Règle des **2 clics** : barre latérale (mode menus) + recherche globale (`Ctrl K` / bouton loupe) sur clients, articles, factures, ventes.

### 11.2 Design system Bissap (`packages/ui`)
```css
:root {
  --k-bissap:  #B0124A;  /* signature : grands aplats, bouton principal, logo */
  --k-mangue:  #FFB627;  /* touches : badges, micro, puces d'état (jamais de texte blanc dessus) */
  --k-petale:  #F6D3DA;  /* surfaces, lignes, état sélectionné */
  --k-rose-the:#FFF0F2;  /* fond général */
  --k-grenat:  #2A0A14;  /* texte, bouton secondaire foncé */
  --k-radius-pill: 999px; --k-radius-xl: 28px; --k-radius-lg: 24px; --k-radius-md: 18px;
  --k-font-title: "Unbounded", sans-serif;   /* 600, 800 ; approche -3 % */
  --k-font-body:  "Nunito Sans", sans-serif; /* 400, 600, 800 */
}
```
Proportions : fond 55 %, bissap 20 %, pétale 12 %, grenat 8 %, mangue 5 %. **Un seul grand bloc bissap et un seul bouton principal par écran.** Pas de dégradé ni de violet « IA », pas d'émoji en guise d'icône, icônes au trait 2 px à bouts arrondis dans une pastille pétale. Phrases courtes, deuxième personne. Mode sombre : fond grenat, rond du logo en mangue (charte § 04). Logos : `assets/brand/` = contenu de `logo-kaye/` (SVG et PNG). **Logo officiel : le symbole avec les perforations à gauche** (`kaye-symbole.svg`, `kaye-logo-horizontal.svg`, `kaye-logo-vertical.svg`, version sombre et blanche) ; ne jamais utiliser la variante sans perforations ; le favicon reprend le même symbole.

### 11.3 Performance
Lot initial ≤ 180 Ko compressés ; premier affichage ≤ 3 s en 3G sur Android d'entrée de gamme ; polices en `font-display: swap`, sous-ensemble latin ; vérifié par Lighthouse CI.

---

## 12. Documents PDF et fichiers

- Gabarits React-PDF dans `packages/documents` : facture FNE, reçu (provisoire ou non certifié), avoir, rapport de caisse, état des créances, relevé, bon de réception, bon de commande (V1), devis (V1).
- Facture certifiée : visuel FNE et QR code (`dgi_verify_url`, correction d'erreur M), numéro normatif DGI en évidence, numéro interne en petit, mentions obligatoires, pied de page KAYE.
- Taille < 200 Ko (polices sous-ensemblées, pas d'image décorative).
- Stockage : `merchants/{merchant_id}/{year}/{doc_type}/{internal_number}.pdf` ; accès par URL présignée 15 min ; un PDF perdu est régénérable à l'identique depuis les données.
- Médias entrants (vocaux, photos) : `merchants/{merchant_id}/inbound/{yyyy-mm}/{uuid}` ; conservation 90 jours (vocaux), pièces justificatives 10 ans.

---

## 13. Sécurité, données personnelles, conformité

### 13.1 Mesures
| Domaine | Mesure |
|---|---|
| Transport | TLS 1.2+ partout, HSTS |
| Authentification | OTP, JWT EdDSA 15 min, refresh rotatif lié à l'appareil, détection de réutilisation |
| Autorisation | rôles (API) + RLS (base) + PIN (actions sensibles) |
| Webhooks | signatures vérifiées (Meta, Telegram, e-mail) ; rejet sinon |
| Entrées | Zod strict, champs inconnus refusés ; taille des médias ≤ 16 Mo |
| Injection SQL | requêtes paramétrées uniquement |
| Injection d'instructions | données utilisateur délimitées ; sortie LLM validée ; aucune action sans proposition confirmée |
| Limitation de débit | 300 lectures/min, 60 écritures/min par utilisateur ; OTP 5/h/numéro, 10/h/IP |
| Journaux | pas de secret, de jeton, de code OTP, de PIN ; NCC et téléphones masqués (`07******56`) |
| Dépendances | audit à chaque build ; vulnérabilité critique = déploiement bloqué |
| Sauvegardes | quotidiennes chiffrées, rétention 30 j, restauration testée chaque mois |

### 13.2 RLS
```sql
-- Au début de chaque transaction applicative :
SELECT set_config('app.merchant_id', $1, true);

ALTER TABLE sales ENABLE ROW LEVEL SECURITY;
ALTER TABLE sales FORCE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON sales
  USING (merchant_id = current_setting('app.merchant_id', true)::uuid)
  WITH CHECK (merchant_id = current_setting('app.merchant_id', true)::uuid);
-- Politique générée pour toutes les tables portant merchant_id (script de migration).
```
Rôles PostgreSQL : `kaye_app` (sans `BYPASSRLS`, sans DDL), `kaye_migrator` (DDL), `kaye_ops` (lecture transverse pour le back-office, journalisée).

### 13.3 Chiffrement des secrets par commerçant
Clés API FNE et business tokens WhatsApp : AES-256-GCM, clé de données dérivée de `DATA_ENCRYPTION_KEY` (32 octets, coffre du fournisseur d'hébergement), IV aléatoire par valeur, rotation annuelle avec re-chiffrement.

### 13.4 Données personnelles
- Responsable de traitement pour les comptes commerçants ; **sous-traitant** pour les données des clients et fournisseurs des commerçants.
- Registre des traitements, liste des sous-traitants (hébergeur, Meta/BSP, Telegram, fournisseurs de modèles, e-mail, SMS), DPA signés.
- Conservation : factures et pièces 10 ans ; journaux NLU en clair 90 jours puis anonymisés ; journaux techniques 30 jours ; audit 5 ans ; vocaux 90 jours.
- Export complet (CSV/JSON) et suppression de compte (sauf pièces à conservation légale, isolées).
- Le droit à l'effacement s'arrête à l'obligation d'archivage fiscal (à écrire dans les CGU).

---

## 14. Mode opératoire Claude Code + Codex

### 14.1 Répartition des rôles
| Rôle | Qui | Responsabilités |
|---|---|---|
| Responsable produit | Fondateurs | Priorités, arbitrage des points à trancher, recette fonctionnelle, fusion vers `recette` et `main` |
| **Agent principal** | **Claude Code** | Socle et modules critiques : base, RLS, domaine (ventes, stock, trésorerie), moteur d'agent, connecteur FNE, revue des PR de Codex |
| **Agent parallèle** | **Codex** | Tickets bien délimités : écrans web, points d'API CRUD, gabarits de messages et PDF, imports, tests, corpus ; revue des PR de Claude Code |
| Relecture croisée | l'autre agent | Chaque PR est relue par l'agent qui ne l'a pas écrite, puis validée par un humain |

### 14.2 Règles de collaboration
1. Le backlog (`docs/cdc/03_Backlog_KAYE_v3.md`) est la seule liste de travail. Un ticket = une branche `claude/t-<num>-<slug>` (Claude Code) ou `codex/t-<num>-<slug>` (Codex), créée depuis `dev` à jour = une PR vers `dev`.
2. Avant de coder : lire le ticket, les EF citées, les sections du présent document citées ; si un point est ambigu ou touche § 17, **ouvrir une question dans la PR et s'arrêter**.
3. Deux agents ne travaillent jamais en même temps sur le même module ; les tickets marqués *socle* sont séquentiels.
4. Toute migration est créée par l'agent qui possède le ticket ; conflit de numérotation de migration → l'agent qui fusionne en second régénère la sienne.
5. Une PR contient : code, tests, mise à jour de la doc si le contrat change, et la liste des EF couvertes.
6. Une PR étiquetée `argent`, `fne`, `securite` ou `agent` joint un rapport de revue ciblée du Conseil (§ 18.3). Une nouvelle ADR ou table passe par la porte G1.

### 14.3 Définition de « terminé »
- Critères d'acceptation du ticket vérifiés par des tests automatisés.
- `pnpm lint && pnpm typecheck && pnpm test` verts ; corpus NLU sans régression (§ 15.3) si `nlu-core`, `agent` ou un prompt change.
- Nouvelle intention : test de conversation via l'adaptateur `test` (heureux + correction + annulation + droits).
- Nouvelle table métier : `merchant_id`, RLS, test d'accès croisé.
- Aucun secret, aucun `console.log` de données personnelles.

### 14.4 Fichiers d'instructions
- `AGENTS.md` (racine) : règles communes, commandes, conventions — lu par Codex et, via `@AGENTS.md`, par Claude Code.
- `CLAUDE.md` : `@AGENTS.md` + consignes propres à Claude Code (rôle d'agent principal, revue).
- Modèles fournis dans le dossier de livraison (`AGENTS.md`, `CLAUDE.md`).

---

## 15. Tests et qualité

### 15.1 Pyramide
| Niveau | Outil | Portée | Seuil |
|---|---|---|---|
| Unitaire | Vitest | `domain`, `nlu-core`, gabarits, mappings FNE | ≥ 85 % de couverture sur `domain` et `nlu-core` |
| Intégration | Vitest + Testcontainers | transactions composites, RLS, outbox, files | tous les parcours d'écriture |
| Conversation | adaptateur `test` | intentions de bout en bout | 100 % des intentions |
| Contrat | fixtures enregistrées | DGI, Meta, Telegram, fournisseurs LLM | tous les appels externes |
| Corpus NLU | exécuteur dédié | `datasets/nlu/*.jsonl` | § 15.3 |
| Bout en bout web | Playwright | parcours E2E-01 à E2E-10 du CDC fonctionnel | 100 % |
| Charge | k6 | 200 commerçants, pic de fin de journée | avant le lancement |

### 15.2 Test d'isolation généré
Un test parcourt la table des routes, crée deux commerçants A et B, et tente avec le jeton de A chaque ressource de B : toute réponse autre que 403/404 fait échouer la CI.

### 15.3 Corpus NLU
- Format : `{"text": "...", "mode": "text|voice", "intent": "sale.record", "slots": {...}, "source": "terrain|synthetic|incident"}`.
- Objectif MVP : 500 phrases dont 100 réelles de terrain et 50 vocaux.
- Seuils bloquants : exactitude d'intention ≥ 97 %, exactitude des **montants** ≥ 99 %, exactitude des champs obligatoires ≥ 92 % ; régression > 2 points = fusion refusée.

### 15.4 Jeux de données
Seed de démonstration (repris du prototype) : Épicerie Koné & Frères, clients Koffi Brou, Mariam Coulibaly, Ibrahim Traoré, Fatou Sanogo, Seydou Bamba ; articles riz, huile, sucre, savon, lait concentré, farine. **Aucune donnée réelle de prospect** (les numéros figurant dans les documents de terrain ne doivent jamais apparaître dans le dépôt).

---

## 16. Environnements, CI/CD, déploiement, exploitation

### 16.1 Environnements
| Env. | Branche | Hébergement | FNE | WhatsApp | Données |
|---|---|---|---|---|---|
| local | — | Docker Compose (Postgres, Redis, MinIO, Mailpit) | simulée | adaptateur test / tunnel | seed |
| dev | `dev` | VPS Hostinger « hors production » (projet Docker Compose `dev`) | simulée | numéro de test | seed |
| recette | `recette` | même VPS, projet Docker Compose `recette`, sous-domaine protégé par mot de passe | **environnement de test DGI** | numéro de test / numéro KAYE | générées + pilotes internes |
| production | `main` | **VPS Hostinger dédié à la production** (Docker Compose : api, worker, postgres, redis, caddy) ; site et application web sur l'hébergement web Hostinger | production DGI | numéro KAYE vérifié | réelles |

**Ce que fait chaque offre Hostinger.** L'offre d'hébergement web retenue le 31 octobre (sites et boîtes e-mail illimités, 50 Go NVMe, 3 Go de RAM, 2 processeurs) convient au **site vitrine**, aux **e-mails professionnels** (`contact@`, `support@`, `factures-…@`) et à l'**application web** compilée (fichiers statiques). Elle ne permet pas de faire tourner durablement une API Node.js avec ses workers, PostgreSQL et Redis : pour cela il faut un **VPS** (à partir de 2 vCPU et 8 Go de RAM conseillés pour la production). Vérifier l'offre exacte avant l'achat.

**Région** : centre de données dans l'Union européenne (RGPD). **Pilotage** : Claude peut administrer le VPS et les déploiements par dialogue (serveur MCP / API Hostinger), mais toute action en production reste validée par un fondateur.

**Sauvegardes du VPS de production** : instantané quotidien du VPS + archivage continu des journaux Postgres (WAL-G vers Cloudflare R2) pour la restauration à un instant donné (RPO ≤ 5 min) ; copie quotidienne `pg_dump` chiffrée hors du VPS ; restauration testée chaque mois.

**Déploiements de prévisualisation** : protégés par mot de passe et non indexés (risque R-10 du CDC fonctionnel).

### 16.2 Chaîne CI (GitHub Actions, sur chaque PR)
1. Installation (cache pnpm) · 2. Lint + format · 3. `tsc --noEmit` · 4. Tests unitaires · 5. Intégration (Testcontainers, migrations appliquées) · 6. Tests de conversation · 7. Corpus NLU (si concerné) · 8. Build api/worker/web · 9. Lighthouse CI (web) · 10. Audit des dépendances · 11. Image Docker signée (sur `dev`, `recette`, `main`).

### 16.3 Déploiement
- `dev` : déploiement automatique à chaque fusion.
- `recette` : PR `dev → recette`, validée par un fondateur ; démonstrations et pilotes.
- `main` : PR `recette → main`, fenêtre **hors 8 h–20 h heure d'Abidjan** ; migrations en expansion/contraction ; retour arrière par redéploiement de l'image précédente (< 5 min).
- API et worker : image Docker construite par GitHub Actions, poussée sur le registre GitHub (GHCR), déployée sur le VPS par SSH (`docker compose pull && docker compose up -d`) ; Caddy assure le HTTPS.
- Application web et site vitrine : build statique déployé sur l'hébergement web Hostinger (SFTP ou intégration Git d'Hostinger).

### 16.4 Exploitation
| Signal | Alerte |
|---|---|
| Erreurs 5xx API | > 1 % sur 5 min |
| Latence p95 écriture | > 400 ms sur 5 min |
| File `fne` | factures `pending` plus vieilles que 6 h ; `ambiguous` > 0 |
| File `inbound` | âge du plus vieux message > 30 s |
| Webhooks Meta | échecs de signature, chute du volume entrant > 50 % vs veille |
| NLU | taux de correction > 15 % sur 24 h ; repli LLM > 40 % |
| Coût | coût LLM d'un commerçant > 80 % du plafond |
| Réconciliation | tout écart stock/trésorerie |

Runbooks (`docs/runbooks/`) : DGI indisponible, clé FNE révoquée, numéro WhatsApp suspendu ou qualité rouge, fuite de secret, restauration de base.

Objectifs : disponibilité de l'enregistrement 99,5 %, RPO ≤ 5 min, RTO ≤ 2 h. **Le RPO de 5 min exige une restauration à un instant donné (PITR)** : vérifier que l'offre Postgres retenue la fournit ; sinon utiliser un Postgres managé qui la fournit (Neon, Supabase, Scaleway) ou ajouter un archivage WAL. Restauration testée une fois par mois.

**Activation progressive** (EF-M13-06) : table `feature_flags (merchant_id, flag, enabled)` lue par un service unique ; toute fonction nouvelle est d'abord ouverte aux pilotes.

**Mode démo** (EF-M0-12) : commerçant fictif `demo-<uuid>` créé à la volée depuis le lien de test, rempli par le seed, FNE forcée en `simulated`, purge après 7 jours par le `scheduler`.

**Numéros inactifs** (EF-M0-11) : tâche quotidienne qui suspend les utilisateurs sans message depuis 60 jours ; réactivation par PIN (patron) ou par le patron (employé).

### 16.5 Mise en route (ordre conseillé, à faire par les fondateurs)
1. **Organisation GitHub** `kaye` + dépôt privé ; branches `main`, `recette`, `dev` protégées.
2. **Domaine** (`kaye.ci` et/ou `kaye.app`) et DNS chez Cloudflare.
3. **Hostinger** : offre d'hébergement web (site, application web, e-mails professionnels, domaine) + **VPS** pour l'API, le worker, Postgres et Redis (un VPS hors production, un VPS de production avant le pilote).
4. **Cloudflare R2** : trois buckets privés (fichiers + sauvegardes Postgres).
5. Désactiver et supprimer les anciens déploiements Vercel.
6. **Telegram** : créer le bot avec @BotFather (un bot de dev, un de prod).
7. **Meta** : Business Manager de la société, **Business Verification**, application WhatsApp, numéro KAYE ; candidature Tech Provider (pour V1) ; choix du BSP.
8. **DGI** : inscription à l'environnement de test FNE ; e-mail à `support.fne@dgi.gouv.ci` sur la voie éditeur (§ 17).
9. **Fournisseurs IA** : clés Mistral (défaut) et Anthropic (secours), plafonds de dépense.
10. **Sentry**, service d'e-mail entrant, fournisseur SMS.
11. Tunnel local pour les webhooks (Cloudflare Tunnel ou ngrok) afin de tester Telegram et WhatsApp depuis un poste de développement.
12. Copier le modèle de dépôt (`A_FAIRE/modele-depot/` : `AGENTS.md`, `CLAUDE.md`, `.claude/`, `docs/`) dans le dépôt, lancer `/conseil G0 lot 1`, puis le ticket `T-001`. Le guide pas à pas est dans `A_FAIRE/00_LISEZ_MOI.md`.

---

## 17. Points techniques à trancher

| # | Question | Impact | Proposition par défaut |
|---|---|---|---|
| T1 | Un éditeur (KAYE) peut-il certifier pour le compte de chaque commerçant avec la clé API du commerçant ? Chaque commerçant doit-il faire valider son interfaçage ? | Verrou n°1 | Écrire à la DGI cette semaine ; coder `DgiHttpFneProvider` avec clé par commerçant |
| T2 | Idempotence / recherche de facture à la DGI | Doublons possibles | Statut `ambiguous` + revue manuelle |
| T3 | Valeurs admises pour `clientPhone` / `clientEmail` d'un client de passage B2C | Ventes comptoir | Tester en environnement de test ; demander à la DGI |
| T4 | Paiement mixte et crédit → `paymentMethod` | Exactitude fiscale | `deferred` dès qu'une part est à crédit |
| T5 | Arrondis HT/TVA entre KAYE et la DGI | Écarts de 1 F | Totaux finaux = ceux de la DGI ; alerte si écart |
| T6 | Moment de la certification pour une commande livrée | V1 | À la livraison |
| T7 | Choix du BSP WhatsApp | Coût, délais | Comparer 360dialog, Infobip, Twilio au lot 0 |
| T8 | Modèles exacts (extraction, vision, voix) | Coût, exactitude | Banc d'essai sur le corpus réel |
| T9 | Taille du VPS Hostinger et séparation production / hors production | Coût, fiabilité | Tranché sur Hostinger (31 oct.) ; 1 VPS hors production dès novembre, 1 VPS production avant le pilote |
| T10 | Fournisseur SMS en Côte d'Ivoire | Connexion de secours | À choisir au lot 0 |
| T11 | Facture B2C ou reçu normalisé (RNE) pour la vente au détail | Type de document certifié | Question DGI ; `isRne`/`rne` prévus dans le mapping |
| T12 | Timbre fiscal (`fiscalStamp`) : calculé par la DGI ou à transmettre ? | Montant total | Lire la réponse en environnement de test ; avis d'un comptable |
| T13 | Compte marchand Wave / Orange Money pour encaisser les pass | Encaissement | Partenaire local ou agrégateur ; activation manuelle en attendant |
| T14 | Offre Postgres avec restauration à un instant donné | RPO | Vérifier avant la mise en production |

---

## 18. Conseil des experts : mise en œuvre

Le Conseil décrit au CDC fonctionnel § 21 est installé **dans le dépôt** pour être lancé par Claude Code et rejoué par Codex.

### 18.1 Fichiers
```
.claude/
├── agents/                         # 10 sous-agents Claude Code, en lecture seule
│   ├── conseil-fonctionnel.md      # siège 1, président de séance
│   ├── conseil-architecte.md       # siège 2
│   ├── conseil-dev-fullstack.md    # siège 3
│   ├── conseil-staff-engineer.md   # siège 4
│   ├── conseil-ux-ui.md            # siège 5
│   ├── conseil-expert-erp.md       # siège 6
│   ├── conseil-fiscal-ci.md        # siège 7
│   ├── conseil-ia.md               # siège 8
│   ├── conseil-securite.md         # siège 9
│   └── conseil-terrain.md          # siège 10
└── skills/
    └── conseil/SKILL.md            # commande /conseil : orchestre une séance
docs/conseil/
├── CONSEIL.md                      # composition, grille, règles (copie du CDC fonctionnel § 21)
├── CODEX_CONSEIL.md                # même séance, rejouée siège par siège avec Codex
└── AAAA-MM-JJ-<porte>.md           # rapports de séance
```
Chaque sous-agent a les outils `Read, Grep, Glob` uniquement : **le Conseil lit et critique, il n'écrit jamais de code**. Les modèles de ces fichiers sont livrés dans `A_FAIRE/modele-depot/`.

### 18.2 Déroulé technique de `/conseil <porte> <objet>`
1. Le skill lit le dossier visé (EF du lot, ADR, PR ou notes de version).
2. Il lance les dix sous-agents **en parallèle**, chacun avec la même consigne et sa grille.
3. Chaque sous-agent répond au format imposé :
   ```
   ### Siège N — <nom>
   | Niveau | Référence | Constat | Proposition |
   ```
4. Le skill met face à face les constats contradictoires et demande à chaque siège concerné une réponse courte (une seule passe).
5. Le président (siège 1) rédige la synthèse et les décisions proposées.
6. Le rapport est écrit dans `docs/conseil/AAAA-MM-JJ-<porte>.md` ; les décisions acceptées par les fondateurs deviennent des modifications des CDC ou des tickets (par une PR humaine ou de l'agent principal, jamais par le Conseil lui-même).

### 18.3 Portes obligatoires dans le cycle de développement
| Porte | Déclencheur | Qui lance | Bloquant |
|---|---|---|---|
| G0 | Avant le premier ticket d'un lot | Fondateur ou Claude Code | Oui |
| G1 | Nouvelle ADR, nouvelle table, changement de contrat d'API | Auteur de la PR | Oui |
| G2 | PR `recette → main` | Fondateur | Oui |
| Ciblée | PR étiquetée `argent`, `fne`, `securite` ou `agent` | Relecteur | Sièges 4, 6, 7, 8, 9 selon l'étiquette |

Une PR étiquetée sensible ne peut pas être fusionnée sans un rapport de revue ciblée joint (vérifié par une case du modèle de PR).

### 18.4 Coût et limites
Une séance complète consomme l'équivalent de dix lectures du dossier : la réserver aux portes ci-dessus. Les sous-agents sont des simulations d'experts : ils trouvent beaucoup d'oublis, mais ne remplacent ni un expert-comptable ivoirien, ni un juriste, ni un test avec de vrais commerçants. Les points `Bloquant` des sièges 7 (fiscal) et 9 (sécurité, juridique) doivent être confirmés par un humain qualifié avant la production.

---

## Annexe A — Variables d'environnement

| Variable | Exemple | Rôle |
|---|---|---|
| `APP_ENV` | `local` / `dev` / `recette` / `production` | profil |
| `DATABASE_URL` | `postgres://kaye_app@…` | rôle applicatif (sans BYPASSRLS) |
| `DATABASE_MIGRATION_URL` | `postgres://kaye_migrator@…` | migrations |
| `REDIS_URL` | `redis://…` | files, cache, verrous |
| `S3_ENDPOINT` / `S3_BUCKET` / `S3_ACCESS_KEY` / `S3_SECRET_KEY` | R2 | fichiers |
| `JWT_PRIVATE_KEY` / `JWT_PUBLIC_KEY` | Ed25519 PEM | jetons |
| `DATA_ENCRYPTION_KEY` | 32 octets base64 | chiffrement des secrets par commerçant |
| `RECEIPT_HASH_SECRET` | aléatoire | empreinte des reçus provisoires |
| `PUBLIC_BASE_URL` / `WEB_BASE_URL` | `https://kaye.ci` | liens |
| `FNE_MODE` | `simulated` / `dgi` | connecteur |
| `FNE_TEST_BASE_URL` | `http://54.247.95.108/ws` | test DGI |
| `FNE_PROD_BASE_URL` | (fourni par la DGI) | production |
| `FNE_HTTP_TIMEOUT_MS` | `30000` | § 9.7 |
| `FNE_STICKER_ALERT` | `50` | § 9.9 |
| `WA_GRAPH_VERSION` | `v21.0` (à mettre à jour) | Cloud API |
| `WA_PILOT_PHONE_NUMBER_ID` / `WA_PILOT_TOKEN` | — | numéro KAYE |
| `WA_APP_SECRET` / `WA_VERIFY_TOKEN` | — | webhooks |
| `WA_EMBEDDED_SIGNUP_CONFIG_ID` | — | V1 |
| `TELEGRAM_BOT_TOKEN` / `TELEGRAM_WEBHOOK_SECRET` | — | bot |
| `LLM_PROVIDER_DEFAULT` | `mistral` | § 7 |
| `MISTRAL_API_KEY` / `ANTHROPIC_API_KEY` / `OPENAI_API_KEY` | — | fournisseurs |
| `LLM_MODEL_EXTRACT` / `LLM_MODEL_VISION` / `ASR_MODEL` | — | choisis au lot 0 |
| `NLU_LLM_ENABLED` / `NLU_CONFIDENCE_THRESHOLD` | `true` / `0.85` | repli |
| `OCR_LINE_THRESHOLD` | `0.85` | lecture de documents |
| `LLM_MONTHLY_CAP_EUR` | `1.20` | plafond par commerçant |
| `SMS_PROVIDER` / `SMS_API_KEY` | — | secours OTP |
| `INBOUND_EMAIL_SECRET` | — | V1 |
| `SENTRY_DSN` | — | erreurs |

## Annexe B — Glossaire FR ↔ EN (code)

| Français (interface) | Anglais (code) |
|---|---|
| Établissement / commerçant | `merchant` |
| Boutique / point de vente | `shop` |
| Patron, gérant, vendeur, comptable | `owner`, `manager`, `seller`, `accountant` |
| Client / fournisseur | `customer` / `supplier` |
| Créance / dette fournisseur | `receivable` / `payable` |
| Règlement client / fournisseur | `customer_payment` / `supplier_payment` |
| Vente / ligne | `sale` / `sale_line` |
| Facture FNE / avoir / reçu | `invoice` / `credit_note` / `receipt` (kind) |
| Réception / bon de livraison | `purchase_receipt` / `delivery_note` |
| Commande d'achat / devis / commande client | `purchase_order` / `quote` / `sales_order` |
| Mouvement de stock / comptage / ajustement | `stock_movement` / `stock_count` / `stock_adjustment` |
| Caisse du jour / clôture | `cash_session` / `close` |
| Compte de trésorerie / mouvement | `treasury_account` / `treasury_movement` |
| Dépense / prélèvement du patron / transfert | `expense` / `owner_withdrawal` / `transfer` |
| Proposition en attente | `pending_action` |
| Envoi à valider | `outbound_request` |
| Coût moyen pondéré | `avg_cost` |

## Annexe C — Traçabilité exigences → modules de code

| Exigences | Modules / packages |
|---|---|
| EF-M0-* | `identity`, `merchants`, `agent/intents/onboarding*`, `channels` |
| EF-M1-* | `nlu-core`, `agent`, `llm-gateway`, `messages`, `channels` |
| EF-M2-* | `sales`, `domain/tax`, `inventory`, `cash`, `receivables` |
| EF-M3-* | `invoicing`, `fne`, `documents`, worker `fne`, `pdf` |
| EF-M4-* | `customers`, `receivables`, `notifications` |
| EF-M5-* | `purchasing`, `suppliers`, `catalog`, `llm-gateway.readDocument` |
| EF-M6-* | `inventory`, `domain/costing` |
| EF-M7-* / EF-M8-* | `cash`, `accounting` |
| EF-M9-* | `apps/web`, `packages/ui` |
| EF-M10-* | `notifications`, worker `scheduler` |
| EF-M11-* | `imports` |
| EF-M12-* | `billing` |
| EF-M13-* | `ops`, `audit` |
| EF-M14-* | `channels/whatsapp-business`, `contacts`, `outbound` |

## Annexe D — Exemple de requête de certification (vente B2C, RME)

```json
POST {FNE_BASE_URL}/external/invoices/sign
Authorization: Bearer <clé du commerçant>
Content-Type: application/json

{
  "invoiceType": "sale",
  "paymentMethod": "mobile-money",
  "template": "B2C",
  "isRne": false,
  "clientCompanyName": "Awa Koné",
  "clientPhone": "0707123456",
  "clientEmail": "<à valider avec la DGI>",
  "clientSellerName": "Amadou",
  "pointOfSale": "<shops.fne_point_of_sale>",
  "establishment": "<shops.fne_establishment>",
  "footer": "Facture émise avec KAYE · kaye.ci",
  "foreignCurrency": "",
  "foreignCurrencyRate": 0,
  "items": [
    { "taxes": ["TVAD"], "reference": "RIZ25", "description": "Riz 25 kg",
      "quantity": 3, "amount": 15000, "discount": 0, "measurementUnit": "sac" }
  ],
  "discount": 0
}
```
Réponse attendue (extrait) : `{ "ncc": "…", "reference": "…", "token": "http://…/fr/verification/<uuid>", "warning": false, "balance_sticker": 179, "invoice": { "id": "…", "items": [{ "id": "…" }] } }`.
