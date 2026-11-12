# KAYE — Backlog de développement

**Version 3.2 · 1er novembre 2026 · revue par le Conseil (CDC fonctionnel § 22)** · Référence : `01_CDC_Fonctionnel_KAYE_v3.md`, `02_CDC_Technique_KAYE_v3.md`

Chaque ticket : identifiant, agent conseillé (**CC** = Claude Code, **CX** = Codex, **H** = humain), dépendances, exigences couvertes, critères d'acceptation. Branche : `claude/t-<num>-<slug>` ou `codex/t-<num>-<slug>` selon l'agent → PR vers `dev`. Les tickets **socle** se font dans l'ordre ; les autres peuvent avancer en parallèle dès que leurs dépendances sont fusionnées.

Statuts possibles : `à faire` · `en cours` · `en revue` · `fait`.

---

## Lot 0 — Verrous (humains, en parallèle du code, à lancer cette semaine)

| ID | Tâche | Qui | Sortie attendue |
|---|---|---|---|
| L0-01 | Inscription à l'environnement de test FNE ; e-mail à `support.fne@dgi.gouv.ci` : un éditeur peut-il certifier pour ses clients avec leur clé ? idempotence ? valeurs `clientPhone`/`clientEmail` pour un client de passage ? | H (Issouf) | Réponse écrite, accès test |
| L0-02 | Business Verification Meta de la société ; application WhatsApp ; numéro KAYE dédié | H | Dossier déposé, numéro de test actif |
| L0-03 | Comparatif BSP (360dialog, Infobip, Twilio) : prix au message en Côte d'Ivoire, Coexistence, délais | H | Choix écrit |
| L0-04 | Corpus : 100 phrases réelles, 50 vocaux, 30 photos de BL et factures (anonymisés) auprès des prospects engagés (Perla, Ets Badre, Ilux, Taha, MEGABAT) | H (Issouf) | `datasets/` alimenté |
| L0-05 | Banc d'essai modèles (extraction, vision, voix) sur le corpus : coût, latence, exactitude des montants | CC | Rapport + valeurs de `LLM_MODEL_*` |
| L0-06 | Comptes : GitHub (organisation + dépôt privé `kaye`), **Hostinger** (hébergement web, e-mails pro, VPS), Cloudflare (R2), Telegram BotFather, Mistral, Anthropic, Sentry ; abonnements Claude et ChatGPT payés depuis le compte Revolut partagé | H (Jacino) | Accès partagés, secrets dans les coffres |
| L0-11 | Société : création via Legal Place (mercredi 4 nov.), compte bancaire joint, compte Revolut partagé pour les abonnements ; relecture des statuts et du pacte avec Claude puis un avocat | H (les deux) | Kbis, IBAN |
| L0-12 | Structure locale en Côte d'Ivoire pour l'accès DGI et l'encaissement des pass | H (Issouf) | Solution écrite |
| L0-13 | Nouveau nom KAYE : disponibilité INPI / OAPI, domaines, réseaux sociaux ; refaire le mot-symbole du logo ; renommer le Drive et les supports | H | Nom déposé, domaine acheté |
| L0-07 | Encaissement des pass en Côte d'Ivoire : compte marchand Wave / Orange Money via partenaire local, filiale ou agrégateur | H (Issouf) | Solution écrite (T13) |
| L0-08 | Avis juridique ARTCI (déclaration, transfert des données vers l'UE) + modèles CGU / confidentialité | H | Avis écrit, CGU v1 |
| L0-09 | Avis d'un expert-comptable ivoirien : RNE vs facture B2C, timbre fiscal, régimes TEE/RME, coût des stickers | H (Issouf) | Note écrite (T11, T12) |
| L0-10 | Gabarits WhatsApp du numéro KAYE soumis à Meta (code de connexion, résumé du soir, alerte, fin de pass) | H | Gabarits approuvés |

---

## Porte G0 — Conseil des experts (avant le premier ticket)

**T-000 · Installer le Conseil et tenir la séance G0 du lot 1** — CC + fondateurs
Dépend de : dépôt créé
Contenu : copier `A_FAIRE/modele-depot/.claude/` et `docs/conseil/` ; lancer `/conseil G0 lot 1` ; les fondateurs tranchent ; les décisions modifient les CDC ou le backlog par PR.
Acceptation : rapport `docs/conseil/2026-11-xx-G0-lot1.md` fusionné ; aucun Bloquant ouvert, ou dérogation écrite.

---

## Lot 1 — Socle et cœur métier (novembre)

### Socle (séquentiel)

**T-001 · Monorepo et outillage** — CC · socle
Dépend de : —
Contenu : pnpm + Turborepo, `apps/{api,worker,web}`, `packages/*` vides, TS strict, ESLint/Prettier, Vitest, `infra/docker-compose.yml` (Postgres 16, Redis 7, MinIO, Mailpit), scripts `pnpm dev|test|lint|typecheck`, CI GitHub Actions (étapes 1–4 et 8).
Acceptation : `pnpm i && pnpm dev` démarre api + worker + web ; CI verte sur une PR vide.

**T-002 · Base de données, migrations, RLS** — CC · socle
Dépend de : T-001
Contenu : Drizzle, rôles `kaye_app`/`kaye_migrator`/`kaye_ops`, tables MVP du § 5.3, script générant les politiques RLS pour toute table avec `merchant_id`, helper `withTenant(merchantId, fn)`, seed de démonstration (§ 15.4).
Acceptation : migrations rejouables ; test prouvant qu'une requête sans `app.merchant_id` ne voit rien ; test d'accès croisé A/B sur 3 tables.

**T-003 · Domaine pur : argent, quantités, TVA, CMP, numérotation** — CC · socle
Dépend de : T-001
Contenu : `packages/domain` : `Xof`, `Qty` (milli), `formatXof`, calcul HT/TVA/TTC par code DGI, CMP, allocation FIFO des règlements, générateur de numéros `PREFIX-AAAA-NNNNNN`.
Acceptation : ≥ 95 % de couverture ; cas limites (arrondis, quantités 2,5, stock ≤ 0 pour le CMP).

**T-004 · Outbox, files BullMQ, worker** — CC · socle
Dépend de : T-002
Contenu : table `outbox_events`, relais, files du § 4.5, helper de job avec contexte locataire, tableau Bull Board en local.
Acceptation : un événement écrit dans une transaction annulée n'est jamais publié (test).

**T-005 · Identité : utilisateurs, rôles, PIN, OTP, JWT** — CC · socle
Dépend de : T-002
EF : M0-06, M0-07, M0-08, M9-01
Contenu : module `identity`, OTP (envoi via `NotificationPort` simulé), JWT EdDSA + refresh rotatif, rôles par boutique, PIN argon2id avec verrouillage.
Acceptation : tests du parcours OTP, rotation et détection de réutilisation, 3 erreurs PIN → verrouillage 15 min.

### Agent et canaux

**T-010 · nlu-core : normalisation** — CX
Dépend de : T-001
EF : M1-01
Contenu : nombres (« 45k », « 45 mille », « 45.000 », « quarante-cinq mille »), opérateurs (om, wave, mtn, moov, flooz, cash), téléphones E.164 CI, élisions, nouchi courant (« gbê » = vendu), unités.
Acceptation : 150 cas de test unitaires.

**T-011 · nlu-core : analyseur à règles MVP** — CC
Dépend de : T-010
EF : M1-04, M1-13, M2-01, M2-02, M2-03
Contenu : règles pour `sale.record` (multi-lignes, crédit, paiement mixte), `customer.create`, `receivable.*`, `stock.*`, `cash.*`, `help` ; score de confiance.
Acceptation : corpus initial (≥ 200 phrases synthétiques + réelles disponibles) : intention ≥ 95 %, montants ≥ 99 %.

**T-012 · Passerelle de modèles** — CC
Dépend de : T-001
Contenu : `packages/llm-gateway` (Mistral par défaut, Anthropic en secours), `extract`, `readDocument`, `transcribe`, validation Zod, timeouts, compteur de coûts, plafond par commerçant, interrupteur global, fournisseur `fake` pour les tests.
Acceptation : tests avec fournisseur `fake` ; test de contrat sur réponses enregistrées ; aucune clé en dur.

**T-013 · Moteur d'agent : registre d'intentions, propositions, confirmation** — CC
Dépend de : T-004, T-005, T-011, T-012
EF : M1-04 à M1-11
Contenu : chaîne § 6.1, `defineIntent`, `pending_actions` (nonce, expiration), corrections, contrôle des rôles/PIN/approbation patron, `nlu_logs`.
Acceptation : tests de conversation (adaptateur `test`) : confirmer, corriger, annuler, expirer, refus de droits, approbation patron.

**T-014 · Gabarits de messages FR** — CX
Dépend de : T-013
EF : M1-06, M1-10
Contenu : `packages/messages` : récapitulatifs, résultats, erreurs, aide générale et par module, structure neutre `{text, buttons, list}`.
Acceptation : snapshots ; montants toujours formatés « 45 000 F CFA ».

**T-015 · Adaptateur Telegram** — CX
Dépend de : T-013, T-014
Contenu : webhook signé, `/start <code>`, claviers inline, vocaux et photos (`getFile`), envoi via file `outbound`.
Acceptation : conversation complète de vente sur un bot de dev ; signature invalide rejetée.

**T-016 · Parcours d'inscription conversationnel** — CC
Dépend de : T-013, T-015
EF : M0-01, M0-02, M0-03
Contenu : § 6.7 ; création merchant/shop/owner/comptes de trésorerie ; vente d'exemple proposée.
Acceptation : inscription < 60 s en test ; NCC « passer » → mode sans certification.

**T-017 · Transcription des vocaux** — CX
Dépend de : T-012, T-013
EF : M1-02
Acceptation : vocal → transcription affichée dans le récapitulatif ; échec → « Je n'ai pas bien entendu, pouvez-vous écrire ou répéter ? ».

### Métier

**T-020 · Catalogue et articles** — CX
Dépend de : T-002, T-003
EF : M1-12, M6-02
Contenu : CRUD articles, alias, recherche trigrammes, création implicite au premier usage (brouillon si prix absent).
Acceptation : recherche « riz » trouve « Riz 25 kg » ; seuil 0,85 respecté.

**T-021 · Stock : journal et projections** — CC
Dépend de : T-020, T-004
EF : M6-01, M6-03 à M6-08
Contenu : `stock_movements`, `stock_levels`, motifs, stock négatif + avertissement, alertes de seuil, intentions `stock.*`.
Acceptation : réconciliation Σ mouvements = projection (test) ; « cassé 2 huile » crée une sortie motivée.

**T-022 · Clients** — CX
Dépend de : T-002
EF : M4-01 à M4-03
Contenu : particulier / professionnel (NCC obligatoire), création à la volée, limite de crédit, fiche.
Acceptation : contrainte NCC si professionnel ; doublon de téléphone refusé.

**T-023 · Caisse : session, encaissements, décaissements, clôture, dépenses** — CC
Dépend de : T-003, T-004
EF : M7-01 à M7-05
Acceptation : vente à crédit sans encaissement ; clôture avec écart motivé ; journée close verrouillée.

**T-024 · Ventes : écriture composite** — CC
Dépend de : T-021, T-022, T-023, T-013
EF : M2-01 à M2-07, M2-09, RG-1
Contenu : § 5.5, intention `sale.record`, paiement mixte, crédit, limite, idempotence par `pending_action.id`.
Acceptation : E2E-01 et E2E-02 (sans FNE) ; aucune double vente sur double clic.

**T-025 · Créances, règlements, relances** — CX
Dépend de : T-024
EF : M4-04 à M4-07, M4-09
Contenu : `receivable.show/list/pay`, allocation FIFO, relance (aperçu, coût, validation, PIN si groupée, STOP).
Acceptation : règlement partiel met à jour le solde et la caisse ; client STOP jamais relancé.

**T-026 · Retours et annulation de vente** — CC
Dépend de : T-024
EF : M2-08, M3-07
Acceptation : annulation = mouvements compensatoires ; facture certifiée → avoir demandé (outbox).

**T-027 · Services, solde créditeur client, coût inconnu, prix appris** — CC
Dépend de : T-024, T-025
EF : M2-10, M2-11, M2-12, M6-14
Acceptation : un article service ne crée jamais de mouvement de stock ; une avance crée un solde créditeur proposé à la vente suivante ; une vente sans coût connu est marquée « coût à compléter » et le résultat affiche la part de marge inconnue.

### FNE et documents

**T-030 · Connecteur FNE simulé + mapping DGI** — CC
Dépend de : T-003
EF : M3-01 à M3-04, M3-11
Contenu : `FneProvider`, `SimulatedFneProvider` (fixtures de la procédure DGI), mapping § 9.3, avoirs § 9.4.
Acceptation : tests de mapping B2C/B2B, codes TVA, `deferred` ; simulateur 400/401/500/timeout.

**T-031 · Worker de certification et rejeu** — CC
Dépend de : T-030, T-024
EF : M3-06, RG-FNE-01 à 03
Contenu : états § 9.5, backoff § 9.6, disjoncteur, `ambiguous` § 9.7, stickers § 9.9, notification au commerçant.
Acceptation : E2E-07 (DGI coupée 2 h → rattrapage) ; aucun rejeu automatique d'un `ambiguous`.

**T-032 · Client HTTP DGI (test)** — CC
Dépend de : T-031, L0-01
Contenu : `DgiHttpFneProvider`, clé par commerçant chiffrée (§ 13.3), activation FNE § 9.11.
Acceptation : facture certifiée dans l'environnement de test DGI ; réponse brute archivée ; `dgi_item_id` stockés.

**T-033 · PDF : facture, reçu provisoire, avoir, rapport de caisse** — CX
Dépend de : T-003, T-030
EF : M3-05, M3-08, M3-10
Contenu : gabarits React-PDF, charte Bissap sobre, QR (ECC M), empreinte du reçu provisoire, page publique `/v/<numéro>`.
Acceptation : PDF < 200 Ko ; QR lisible ; mention « SPÉCIMEN » en mode simulé.

**T-034 · Envoi des documents dans le fil** — CX
Dépend de : T-033, T-015
Acceptation : après une vente, le PDF arrive dans le canal de pilotage ; « renvoie la facture de Awa » fonctionne.

### Achats (achat direct)

**T-040 · Fournisseurs et dettes fournisseurs** — CX
Dépend de : T-002, T-023
EF : M5-07, M5-08
Acceptation : « je dois combien à Sow » ; « payé 150 000 à Sow par Wave » avec PIN → décaissement + dette soldée.

**T-041 · Lecture de BL / facture et réception** — CC
Dépend de : T-012, T-013, T-020, T-021, T-040
EF : M5-01 à M5-06, RG-12
Contenu : `receipt.from_document`, confiance par ligne, base article qui apprend (réf. fournisseur → code-barres → désignation → création), lignes `to_reference`, CMP, réception sans commande.
Acceptation : E2E-04 et E2E-05 sur 10 BL de test ; aucune ligne incertaine acceptée sans question.

### Application web

**T-050 · Design system Bissap** — CX
Dépend de : T-001
Contenu : tokens § 11.2, polices, logo SVG, composants (bouton pilule, carte, badge, champ de saisie, bulle de chat, tableau, carte de récapitulatif avec boutons).
Acceptation : page Storybook ou `/ui` ; contrastes AA vérifiés.

**T-051 · Connexion et coquille** — CX
Dépend de : T-005, T-050
Acceptation : connexion par code ; choix de boutique ; menu latéral ; recherche globale.

**T-052 · Accueil chat web** — CX
Dépend de : T-013, T-050, T-051
EF : M9-02, M9-04
Acceptation : même vente que sur Telegram, même récapitulatif, mêmes boutons ; cartes de suggestion insèrent la syntaxe.

**T-053 · Listes et fiches (ventes, factures, clients, créances, stock, caisse, achats)** — CX
Dépend de : T-024, T-025, T-021, T-023, T-041
EF : M9-03, M9-08
Acceptation : toute donnée en ≤ 2 clics depuis l'accueil ; filtres ; téléchargement PDF.

**T-054 · Tableau de bord** — CX
Dépend de : T-053
EF : M9-05
Acceptation : `GET /dashboard` en une requête < 4 Ko ; cartes conformes à la charte.

**T-055 · Paramètres (boutique, FNE, équipe, PIN, seuils, notifications)** — CX
Dépend de : T-051, T-032
EF : M0-04, M9-09
Acceptation : clé FNE saisie masquée, jamais réaffichée ; PIN exigé.

### Transverse

**T-060 · Résumé du soir et alertes** — CX
Dépend de : T-004, T-021, T-023, T-025
EF : M10-01 à M10-03
Acceptation : un seul message par jour ; Telegram prioritaire.

**T-061 · Import CSV/XLSX** — CX
Dépend de : T-020, T-022, T-040
EF : M11-01, M11-03
Acceptation : modèles téléchargeables ; rapport d'erreurs ligne à ligne ; soldes d'ouverture.

**T-062 · Back-office /ops** — CX
Dépend de : T-031, T-013
EF : M13-01, M13-02, M13-04, M13-05
Acceptation : liste des factures en échec avec reprise ; corpus des corrections ; activation manuelle d'un pass.

**T-063 · Paliers et compteurs** — CC
Dépend de : T-024, T-031
EF : M12-01, M12-03, RG-5, RG-9
Acceptation : 31e facture FNE du mois en palier gratuit → reçu non certifié + message d'offre ; dettes toujours accessibles.

**T-064 · Test d'isolation généré + Playwright E2E** — CC
Dépend de : T-053
Acceptation : CI échoue si une route expose une donnée d'un autre commerçant ; E2E-01 à E2E-10 automatisés.

**T-065 · Mode démo et lien de test** — CX
Dépend de : T-016, T-052
EF : M0-12
Acceptation : un lien ouvre une boutique fictive pré-remplie (web et Telegram) sans inscription ; FNE simulée ; purge à 7 jours.

**T-066 · Activation progressive (feature flags)** — CC
Dépend de : T-002
EF : M13-06
Acceptation : une fonction marquée est invisible pour un commerçant non pilote, sur tous les canaux.

**T-067 · Numéros inactifs** — CX
Dépend de : T-005, T-004
EF : M0-11
Acceptation : 60 jours sans message → suspension ; réactivation par PIN (patron) ou par le patron (employé).

**T-068 · CGU et confidentialité dans l'inscription** — CX
Dépend de : T-016, L0-08
EF : M13-07
Acceptation : lien court envoyé à l'inscription ; acceptation horodatée en base.

**T-069 · Mise en ligne du site vitrine** — CX
Dépend de : L0-06
Acceptation : site publié sur l'hébergement web Hostinger avec le domaine KAYE ; adresse `contact@` créée et testée ; aucun chiffre non sourcé ; démo du téléphone animée.

---

## Lot 2 — Pilote (décembre)

**T-070 · WhatsApp Cloud API — canal de pilotage** — CC
Dépend de : T-013, L0-02
Contenu : § 8.3 (webhook signé, boutons, listes, médias, fenêtre de 24 h, gabarits hors fenêtre).
Acceptation : parcours vente, BL, relance, résumé du soir sur le numéro KAYE.

**T-071 · Pass prépayés (déclaration + activation manuelle)** — CX
Dépend de : T-063, T-062
EF : M12-02
Acceptation : le commerçant envoie la capture du paiement Wave/OM ; l'opérateur active ; alerte J-7/J-3/J-1.

**T-072 · Lecture des captures Wave / Orange Money** — CC
Dépend de : T-041
EF : M1-03 (V1 avancée), M4-05
Acceptation : montant, date, destinataire proposés puis confirmés.

**T-073 · Durcissement : observabilité, sauvegardes, charge, runbooks** — CC
Dépend de : T-070
Acceptation : alertes § 16.4 actives ; restauration testée ; k6 : 200 commerçants en pic.

**T-074 · Recette pilote et gel du code** — H + CC
Acceptation : 15 pilotes équipés ; spécimens FNE envoyés à la DGI ; critères Go/No-Go § 17.1 du CDC fonctionnel évalués.

---

## V1 — Février → juin 2027 (à détailler après le lancement)

| ID | Thème | EF |
|---|---|---|
| T-100 | Canal commercial : Embedded Signup, Coexistence, `whatsapp_accounts`, gabarits par WABA | M14-01, M14-07 |
| T-101 | Contacts et étiquetage ; `outbound_requests` ; pause sur reprise en main | M14-02 à M14-06 |
| T-102 | Ventes WhatsApp : lecture des demandes clients, stock disponible, demandes non servies | M2-20 à M2-22 |
| T-103 | Devis, commandes avec réservation, livraisons, grilles de prix | M2-23 à M2-26 |
| T-104 | Achats planifiés : réappro, demandes de prix, bons de commande, reliquats, retours, contrôle facture | M5-20 à M5-25 |
| T-105 | E-mail entrant pour les factures fournisseurs | M5-26, M7.4 |
| T-106 | Comptage guidé, écarts valorisés, péremption, déconditionnement | M6-09 à M6-12, M5-25 |
| T-107 | Trésorerie multi-comptes, transferts, prélèvements | M7-06 à M7-08 |
| T-108 | Comptabilité simplifiée : catégories, résultat du mois, export comptable, rôle comptable | M8-01 à M8-03 |
| T-109 | Multi-boutiques | M0-05, M0-09 |
| T-110 | Pass automatisés (liens marchands), parrainage | M12-02, M12-04 |
| T-111 | Liste filtrée générée depuis une demande en langage naturel | M9-07 |
| T-112 | Catalogue par photo à l'inscription | M11-02 |
| T-113 | Alerte de marge | M5-09 |
