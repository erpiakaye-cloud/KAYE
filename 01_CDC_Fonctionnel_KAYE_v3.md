# KAYE — Cahier des charges fonctionnel

**Version 3.2 · 1er novembre 2026 · Statut : référence pour le développement, revue par le Conseil des experts (§ 21–22)**

> *« Votre commerce dans WhatsApp, la conformité en plus. »*

| Champ | Valeur |
|---|---|
| Produit | KAYE (anciennement KARNET, renommé le 31 octobre 2026) — ERP IA conversationnel pour TPE/PME (Côte d'Ivoire d'abord) |
| Société porteuse | GAAGLE SAS / KAYE (dénomination à confirmer), SAS de droit français, deux associés |
| Rédaction | Consolidation de tout le dossier `C:\KAYE` et du projet Claude « KAYE » |
| Remplace | CDC_Fonctionnel_Karnet v1.0 (28 août), Cahier des charges Carnet v1 (août) et v2 (26 août), CDC/SFD des prototypes Figma |
| Complété par | `02_CDC_Technique_KAYE_v3.md` (architecture, données, API, déploiement) et `03_Backlog_KAYE_v3.md` (tickets) |
| Charte | Bissap v1.0 (octobre 2026) — rouge bissap, jaune mangue. **Logo officiel (2 nov.) : le « k » dans sa bulle avec les perforations de carnet à gauche**, mot « kaye » en Unbounded 800 minuscules (fichiers : `logo-kaye/`) |

---

## Sommaire

1. Comment lire ce document
2. Journal des décisions (août → octobre 2026)
3. Contexte et problème
4. Vision et positionnement
5. Cibles et personas
6. Acteurs, rôles et droits
7. Canaux et surfaces
8. Périmètre et découpage en lots
9. Exigences fonctionnelles par module
10. Règles cardinales
11. Règles de gestion transverses
12. Référentiel des commandes reconnues
13. Documents générés
14. Exigences non fonctionnelles (résumé)
15. Modèle économique et contraintes produit
16. Critères de recette et indicateurs
17. Calendrier
18. Risques
19. Points à trancher
20. Glossaire
21. Conseil des experts
22. Revue n°1 du Conseil (31 octobre 2026)
Annexe A — Sources consultées

---

## 1. Comment lire ce document

Ce document dit **ce que KAYE doit faire**. Le *comment* (stack, base de données, API, déploiement) est dans le CDC technique. Chaque exigence porte un identifiant stable, à citer dans le code, les tickets et la recette.

| Marqueur | Sens |
|---|---|
| `EF-Mx-nn` | Exigence fonctionnelle du module x |
| `RG-n` | Règle cardinale : non négociable, jamais arbitrée par un développeur ni par un agent de code |
| `RG-XXX-nn` | Règle de gestion d'un domaine (ACH achats, VTE ventes, STK stock, CPT compta…) |
| **MVP** | Indispensable à l'ouverture du pilote (décembre 2026) et au lancement public (janvier 2027) |
| **V1** | Attendu dans les 6 mois qui suivent le lancement (février → juin 2027) |
| **V2** | Après l'équilibre ou sur décision explicite (juillet 2027 et au-delà) |
| **HORS** | Exclu. Écrit pour qu'il ne revienne pas par la petite porte |

Les mots DOIT, NE DOIT PAS, DEVRAIT, PEUT ont le sens de la RFC 2119.

---

## 2. Journal des décisions (août → octobre 2026)

Ce tableau fixe l'état des décisions prises en réunion. En cas de contradiction entre deux documents du dossier, **la décision la plus récente l'emporte**.

| Date | Décision | Source |
|---|---|---|
| 3 août | Projet retenu : système de gestion et de facturation accessible par WhatsApp, marché ivoirien, modèle prépayé, interface épurée | CR 03/08 |
| 19 août | Priorité : facture normalisée (FNE) via WhatsApp ; test manuel (« Wizard of Oz ») avant automatisation | CR 19/08 |
| 23 août | Validation humaine obligatoire avant toute transmission à la DGI ; inscription et NCC gérés directement dans la conversation | CR 23/08 |
| 26 août | CDC v2 : l'application web devient le centre de consultation, WhatsApp reste le canal de saisie | CDC v2 |
| 31 août | **Abandon de l'application mobile native** : web + bot WhatsApp. Clients particulier / professionnel à la création. Authentification multi-utilisateurs et changement de boutique. Aide contextuelle par module (façon `man`). Trois paliers tarifaires TPE/PME. 5 utilisateurs max sur l'offre de base | CR 31/08 |
| 29 août → 3 oct. | ADR-09 : le bot opère sur le WhatsApp Business **du commerçant** (Tech Provider, Embedded Signup, Coexistence, via un BSP). ADR-10 : **deux canaux** — pilotage (numéro KAYE + bot Telegram) et commercial (WhatsApp Business de la boutique). API officielle uniquement | ADR-09, ADR-10 |
| 20 sept. | Architecture produit : **progiciel web SaaS (cœur) + bot WhatsApp + bot Telegram**. Quatre modules : **achats, ventes, stock, trésorerie** + données de base. Interface web : accueil en mode chat, **double mode chat / menus** (test A/B), tableaux structurés, **2 clics maximum**. Branches Git : `dev`, `recette`, `main` | CR 20/09 |
| 3 oct. | Flux métier détaillés (achats, ventes, stock, compta) : photo du BL comme pièce centrale, base article qui apprend, dette fournisseur, comptage guidé, comptabilité « sous-produit » | flux-karnet.html |
| Oct. | **Charte Bissap v1.0** retenue (rouge bissap #B0124A, mangue #FFB627, Unbounded + Nunito Sans). Logo « k » dans sa bulle (29 oct.) | Charte, logos |
| 17 oct. | **Double stratégie** : finir KAYE vite pour la crédibilité locale (TPE Afrique) **et** préparer un agent IA « business » mondial, connecté aux ERP/CRM existants (produit séparé). Modèles de langage : Mistral par défaut, ouverture aux autres fournisseurs, choix automatique selon la tâche. **Capture des factures fournisseurs par e-mail dédié.** Import des référentiels (CSV, modèles d'import, plus tard connecteurs). MVP avec WhatsApp, **lancement officiel en janvier 2027** | CR 17/10 |
| 22 oct. | Validation terrain (Yopougon, Treichville) : la cible MVP est **formelle** ; décideur = patron ou comptable ; douleur forte sur le **référencement des articles** ; segment électronique/informatique le plus réceptif | Validation terrain |
| **31 oct.** | **Changement de nom : KAYE devient KAYE** (prononcé comme « cahier »). Un concurrent, `carnet.net`, lancé le 3 septembre, cible les commerçants d'Afrique de l'Ouest avec un carnet de dettes et des relances WhatsApp. Le déploiement Vercel est désactivé. Hébergement chez **Hostinger** (site, e-mails professionnels, agents). Création de la société via Legal Place, compte joint (Qonto), compte Revolut partagé pour les abonnements. Accès DGI : une **structure locale en Côte d'Ivoire** sera probablement nécessaire (Issouf s'en charge). Priorité à la vitesse d'exécution | CR 31/10 |

---

## 3. Contexte et problème

### 3.1 Une obligation légale
La **facture normalisée électronique (FNE)** est obligatoire pour toutes les entreprises opérant en Côte d'Ivoire depuis le **1er décembre 2025** (arrêté 0337/MFB/DGI du 9 mai 2025, lancement officiel le 21 juillet 2025). Chaque facture est certifiée par la plateforme de la DGI, qui délivre trois éléments : **le numéro normatif, le visuel FNE et le QR code de vérification**. Sans enrôlement, l'entreprise n'obtient pas son **attestation de régularité fiscale (ARF)** et perd l'accès aux marchés publics et à beaucoup de grands clients.

Ordres de grandeur issus des études du dossier (à recouper avant toute communication financière) : environ 52 000 entreprises enrôlées début 2026, dont environ 30 % n'ont jamais émis de facture par le système. Elles se sont inscrites puis sont retournées au cahier : **l'obstacle n'est pas la loi, c'est le logiciel**.

La DGI autorise les entreprises disposant de leur propre système de facturation à faire certifier leurs factures **par API** (procédure d'interfaçage, mai 2025). C'est la voie technique de KAYE.

### 3.2 Un canal déjà occupé par le commerce
Le commerçant ivoirien vit sur WhatsApp : commandes, vocaux, photos de tarifs, captures Wave. Il n'ouvrira pas une application de plus. Les ERP classiques (Sage, Odoo) et les logiciels de caisse sont trop lourds, trop chers, et abandonnés en quelques semaines.

### 3.3 Ce qui se perd aujourd'hui (constats des flux du 3 octobre)
| Domaine | Ce qui se perd | Conséquence |
|---|---|---|
| Ventes | Ventes au comptoir non notées, crédits oubliés | Créances jamais encaissées, caisse fausse |
| Achats | Prix négociés, remises, reliquats, dette fournisseur | Marge inconnue, c'est le cahier du fournisseur qui fait foi |
| Stock | Sorties hors vente, casses, prélèvements | Ruptures découvertes trop tard, soupçons sans preuve |
| Trésorerie | Argent du foyer mêlé à la caisse, soldes mobile money | Bénéfice réel inconnu, pas d'accès au crédit |
| Conformité | Factures conformes impossibles à produire | Perte de clients professionnels, ARF bloquée |

### 3.4 Ce que dit le terrain (septembre 2026, 30 entreprises)
- 27 % ont vu une démo, 17 % ont pris un engagement concret, **0 a payé** (le prix n'a pas été demandé).
- Le **décideur n'est pas au comptoir** : il faut un support à laisser (vidéo courte, lien de test).
- Le **comptable** prescrit ou bloque.
- Le **référencement des articles** est une douleur concrète (Perla) : KAYE doit la résoudre dès l'inscription.
- Signaux sur la voix, la photo et le multi-établissement (Ets Badre).

---

## 4. Vision et positionnement

### 4.1 La promesse
**KAYE est l'ERP IA des boutiques.** Le commerçant parle à son agent sur WhatsApp ou Telegram — par texte, vocal ou photo. Achats, ventes, stock, caisse et comptes se tiennent à partir de la conversation. Chaque vente peut produire une facture FNE certifiée. Une application web légère montre les tableaux, les listes et les chiffres quand un message ne suffit plus.

> « vendu 3 riz à Awa 45 000 wave » → vente enregistrée, stock sorti, facture FNE certifiée, caisse Wave à jour. En une phrase et une confirmation.

### 4.2 Trois surfaces, un seul cerveau
| Surface | Pour qui | Pour quoi |
|---|---|---|
| **Agent KAYE** (WhatsApp KAYE + bot Telegram) | Patron, gérant, vendeurs | Saisir, demander, valider, recevoir les alertes |
| **WhatsApp Business de la boutique** (Coexistence) | Clients et fournisseurs du commerçant | L'agent lit, prépare, et envoie après validation |
| **Application web KAYE** | Patron, gérant, comptable | Consulter, chercher, corriger, exporter, paramétrer |

Les trois surfaces écrivent dans **le même état** : une vente saisie sur WhatsApp apparaît immédiatement dans l'application, et inversement.

### 4.3 Ce qui nous distingue
1. **Zéro formation** : la conversation remplace les écrans de saisie.
2. **Conformité native** : FNE certifiée par l'API officielle de la DGI.
3. **Stock intégré** aux achats et aux ventes (avantage cité face aux concurrents).
4. **L'IA propose, le commerçant valide** : aucune écriture d'argent sans confirmation.
5. **Prépayé mobile money** : on recharge KAYE comme du crédit téléphonique.

### 4.4 Ce que KAYE n'est pas
Ni une banque, ni un portefeuille, ni un logiciel de comptabilité SYSCOHADA complet, ni un assistant généraliste. Voir les exclusions (§ 8.3) et les règles cardinales (§ 10).

### 4.5 Vision long terme (hors périmètre de ce document)
Expansion Côte d'Ivoire → Afrique de l'Ouest (UEMOA) → Afrique subsaharienne → Afrique du Nord → Europe ; à terme un « OS du commerce » multicanal. Le projet d'**agent IA business mondial** connecté aux ERP existants (décidé le 17 octobre) est un **produit distinct**. KAYE lui prépare le terrain par deux choix d'architecture : un moteur d'agent indépendant du canal, et une passerelle de modèles de langage indépendante du fournisseur.

---

## 5. Cibles et personas

| Persona | Profil | Prix acceptable | Traitement |
|---|---|---|---|
| **Amadou** — transition formelle | 38 ans, quincaillerie à Marcory. NCC, enrôlé FNE par obligation, bloqué par l'outil. WhatsApp 50 fois par jour | 5 000 – 12 000 F/mois | **Cible n°1 du MVP** |
| **Koné & Frères** — semi-grossiste | Épicerie semi-grossiste à Port-Bouët, NCC, plusieurs fournisseurs, clients à crédit | 12 000 F/mois | Client type des achats planifiés et du multi-utilisateur |
| **Commerçant tech** — électronique / informatique | Treichville, Adjamé, Plateau. Beaucoup de références, à l'aise avec la technologie | 5 000 – 12 000 F/mois | Segment prioritaire de la prochaine vague terrain (hypothèse H5) |
| **Fatou** — fournisseur B2B / État | 42 ans, services à Cocody. Besoin vital de l'ARF, multi-utilisateurs, exports comptables | 12 000 – 30 000 F/mois | Cible n°2, meilleur panier (V1) |
| **Yao** — vendeur social | 27 ans, Instagram/TikTok, conclut sur WhatsApp. Rapproche à la main les captures Wave | 5 000 F/mois | Cible secondaire : ventes WhatsApp, lecture des captures |
| **Awa** — informel | 31 ans, Adjamé, téléphone ancien, data limitée, vocaux. Veut savoir qui lui doit | 0 F | Palier gratuit, mode sans certification. À servir, pas à facturer |
| PME structurée | 10 salariés et plus, SYSCOHADA, paie, multi-sites | 15 000 F + | **Hors cible** (terrain de Sage, Odoo, Madata) |

**Décideurs et prescripteurs** : le patron signe, le vendeur voit la démo, le **comptable** (ou le centre de gestion agréé) prescrit. Le produit DOIT donc avoir un rôle comptable en lecture et un export propre.

---

## 6. Acteurs, rôles et droits

### 6.1 Acteurs
| Acteur | Description |
|---|---|
| Patron (propriétaire) | Titulaire du compte, tous les droits, valide les actions sensibles avec son **PIN** |
| Gérant | Délégué du patron sur une boutique : ventes, achats, stock, caisse ; pas de paramètres ni d'utilisateurs |
| Vendeur (employé) | Vend, réceptionne, consulte le stock ; ses actions sensibles partent en validation |
| Comptable | Lecture seule sur les pièces, journaux et exports ; aucune écriture |
| Client final | Client de la boutique ; échange sur le WhatsApp Business de la boutique ; n'a jamais de compte |
| Fournisseur | Fournisseur de la boutique ; même principe que le client |
| Agent KAYE | Le moteur IA : comprend, propose, prépare, n'engage jamais seul |
| DGI | Plateforme FNE : certifie les factures et avoirs |
| Opérateur KAYE | Équipe interne : support, supervision, reprise manuelle (back-office) |

### 6.2 Matrice des droits
| Action | Patron | Gérant | Vendeur | Comptable |
|---|---|---|---|---|
| Enregistrer une vente au comptoir ou à crédit | ✔ | ✔ | ✔ | — |
| Vente à crédit au-delà de la limite du client | ✔ (PIN) | ✔ | validation | — |
| Remise au-delà du plafond du rôle | ✔ | ✔ | validation | — |
| Annuler une vente / émettre un avoir | ✔ (PIN) | ✔ | — | — |
| Créer / modifier un client | ✔ | ✔ | ✔ | — |
| Modifier une limite de crédit | ✔ | ✔ | — | — |
| Réceptionner une livraison (photo du BL) | ✔ | ✔ | ✔ (écarts → validation) | — |
| Créer un article / fixer un prix de vente | ✔ (PIN pour le prix) | ✔ | brouillon → validation | — |
| Passer une commande fournisseur | ✔ | ✔ | — | — |
| Régler un fournisseur | ✔ (PIN) | — | — | — |
| Ajuster le stock au-delà du seuil | ✔ (PIN) | ✔ | — | — |
| Décaissement, clôture de caisse | ✔ | ✔ | — | — |
| Envoyer un message groupé (relances, propositions) | ✔ (PIN) | ✔ | — | — |
| Passer une créance en irrécouvrable | ✔ (PIN) | — | — | — |
| Paramètres, NCC, clé FNE, abonnement, utilisateurs | ✔ (PIN) | — | — | — |
| Consulter pièces, journaux, exports | ✔ | ✔ | ventes du jour | ✔ |

---

## 7. Canaux et surfaces

### 7.1 Canal de pilotage — le commerçant parle à son agent
- **Un numéro WhatsApp KAYE unique** (WhatsApp Cloud API) et **un bot Telegram unique**, partagés par tous les commerçants. Chaque message est rattaché à l'utilisateur par son numéro (WhatsApp) ou son `chat_id` (Telegram). Le commerçant a l'impression d'avoir *son* assistant.
- Telegram est lié par un code à usage unique envoyé à l'inscription (`/start <code>`).
- Ce qui y passe : ventes, achats, stock, caisse, photos de BL et de factures, vocaux, validations, alertes, résumés.
- Un expéditeur inconnu reçoit le parcours d'inscription, **jamais de données**.

### 7.2 Canal commercial — l'agent parle aux contacts de la boutique
- Le **WhatsApp Business du commerçant**, branché en **Coexistence** (le commerçant garde son application et son numéro, l'agent agit par l'API sur le même fil).
- Le client ou le fournisseur voit la boutique, **jamais KAYE**.
- Chaque contact porte une étiquette :

| Étiquette | L'agent lit | L'agent répond seul | L'agent envoie après validation |
|---|---|---|---|
| client | oui | oui, dans la fenêtre de 24 h, selon les réglages de la boutique | relances, devis, factures, propositions (opt-in) |
| prospect | oui | **première réponse soumise au commerçant**, puis comme un client | idem client |
| fournisseur | oui (prix, BL, factures, confirmations) | **jamais** | demandes de prix, bons de commande, reçus, relevés |
| personnel | non | non | non |
| inconnu | oui | non | non, jusqu'à étiquetage |

L'agent propose l'étiquette dans le canal de pilotage (« Sow vous envoie des prix : c'est un fournisseur ? »).

### 7.3 Application web
- Accessible depuis un navigateur (mobile et ordinateur), **sans installation** ; connexion par code reçu sur WhatsApp, Telegram ou SMS.
- **Accueil en mode chat** (charte Bissap) : grand bloc bissap avec « Bonjour ! Qu'est-ce qu'on fait pour la boutique ? », champ de saisie, 4 cartes de suggestion (stock, ventes, commandes, caisse).
- **Mode menus** à côté du mode chat : Ventes, Achats, Stock, Caisse, Clients, Fournisseurs, Rapports, Paramètres. Test A/B des deux modes.
- Toute information doit être atteinte **en 2 clics maximum** (ou une phrase).
- Une demande en langage naturel peut produire un **tableau** (« commandes non payées de septembre ») plutôt qu'un texte.

### 7.4 E-mail entrant (V1)
Chaque boutique reçoit une adresse dédiée (ex. `factures-<boutique>@in.kaye.ci`) pour que ses fournisseurs y envoient factures et BL en PDF. Les pièces suivent le même traitement qu'une photo envoyée au bot.

---

## 8. Périmètre et découpage en lots

### 8.1 Modules
| Module | Contenu | Lot |
|---|---|---|
| M0 Compte, établissement, utilisateurs | Inscription conversationnelle, NCC, boutiques, rôles, PIN | MVP |
| M1 Agent conversationnel | Intentions, texte, vocal, photo, confirmation, correction, aide | MVP (photo V1 partielle) |
| M2 Ventes | Comptoir, crédit, retours ; vente WhatsApp, devis, commandes, livraisons | MVP (comptoir, crédit) · V1 (WhatsApp, devis, commandes) |
| M3 Facturation FNE | Certification, PDF, envoi, avoirs, mode dégradé | MVP |
| M4 Clients et créances | Carnet clients, encours, règlements, relances | MVP |
| M5 Achats et fournisseurs | Achat direct par photo du BL, base article, dette fournisseur ; achats planifiés | MVP (achat direct) · V1 (planifié) |
| M6 Stock | Journal, motifs, alertes, comptage guidé, écarts ; péremption, déconditionnement | MVP (journal, alertes) · V1 (comptage, péremption) |
| M7 Caisse et trésorerie | Session du jour, encaissements, décaissements, clôture ; comptes par moyen de paiement, transferts, prélèvements | MVP (caisse) · V1 (comptes, transferts) |
| M8 Comptabilité simplifiée | Dépenses, résultat du mois, exports comptable, échéances | V1 |
| M9 Application web | Accueil chat, menus, tableaux, recherche, tableau de bord | MVP |
| M10 Notifications | Alertes, résumé du soir, rappels | MVP |
| M11 Import et onboarding | Import CSV, catalogue par photo, e-mail entrant | MVP (CSV) · V1 (e-mail) |
| M12 Abonnement | Paliers, pass prépayés Wave / Orange Money, compteurs | MVP (manuel) · V1 (automatisé) |
| M13 Back-office | Supervision, files FNE, reprise, support dans le fil | MVP |
| M14 Canal commercial | Coexistence, étiquetage, réponses clients, envois validés | V1 |

### 8.2 Ce qui est livré au lancement public (janvier 2027)
Canal de pilotage WhatsApp + Telegram, application web, ventes comptoir et crédit, FNE certifiée, clients et créances avec relances, achat direct par photo du BL, stock journalisé avec alertes, caisse du jour, résumé du soir, import CSV, pass prépayés (encaissement manuel au besoin), back-office.

### 8.3 Exclusions définitives (HORS)
| Exclusion | Raison |
|---|---|
| Détenir, transporter, convertir des fonds pour un tiers | Activité réglementée ; KAYE lit les modes de paiement, il ne les opère pas |
| Crédit, avance, scoring, recommandation de prêt | Activité réglementée, autre métier. Préparer un dossier de chiffres pour un prêteur reste possible en V2 (export, pas de décision) |
| Comptabilité SYSCOHADA complète, liasse fiscale, paie, RH | Terrain des ERP installés ; KAYE prépare des données propres pour le comptable |
| Dépôt de déclarations fiscales | KAYE prépare les montants, le comptable ou le commerçant dépose |
| Liaison WhatsApp non officielle (WhatsApp Web, appareils liés) | Risque de bannissement du numéro du commerçant |
| Application mobile native | Abandonnée le 31 août au profit du web et des bots |
| Assistant généraliste | L'agent ne répond qu'au métier du commerce (règles Meta et confiance) |

---

## 9. Exigences fonctionnelles par module

### M0 — Compte, établissement, utilisateurs

| Réf. | Exigence | Lot |
|---|---|---|
| EF-M0-01 | Le compte se crée en écrivant au numéro WhatsApp KAYE ou au bot Telegram, ou depuis l'application web. Aucun mot de passe ni e-mail exigé. | MVP |
| EF-M0-02 | L'inscription collecte au plus : nom de la boutique, activité, NCC (facultatif), nom du patron. Durée cible : **moins de 60 secondes**. | MVP |
| EF-M0-03 | Le NCC est facultatif. Sans NCC, KAYE fonctionne **sans certification** (carnet de ventes, dettes, stock, caisse). Les documents portent alors la mention « Reçu non certifié ». | MVP |
| EF-M0-04 | L'activation de la FNE se fait dans les paramètres : NCC, établissement, point de vente, clé d'API FNE (ou mandat, selon la voie retenue avec la DGI). Un test de certification valide la configuration avant la première vraie facture. | MVP |
| EF-M0-05 | Un établissement (entreprise, un NCC) peut avoir plusieurs **boutiques** (points de vente). Chaque boutique a son stock, sa caisse, ses utilisateurs. | MVP (1 boutique) · V1 (plusieurs) |
| EF-M0-06 | Le patron ajoute un utilisateur en une phrase (« ajoute Koffi 0707123456 vendeur ») ou depuis l'application. L'utilisateur reçoit un message d'activation. | MVP |
| EF-M0-07 | Le patron retire un utilisateur en une phrase ; l'accès est coupé immédiatement sur tous les canaux. | MVP |
| EF-M0-08 | Le patron définit un **PIN à 4 chiffres**, demandé pour les actions sensibles (§ 6.2). Trois erreurs bloquent l'action 15 minutes. | MVP |
| EF-M0-09 | Un utilisateur rattaché à plusieurs boutiques choisit la boutique active (« boutique Treichville ») ; la boutique active est rappelée dans chaque récapitulatif. | V1 |
| EF-M0-10 | Le commerçant peut exporter toutes ses données (CSV/JSON) et demander la suppression de son compte ; les pièces fiscales restent archivées pour la durée légale. | V1 |
| EF-M0-11 | **Numéro recyclé** : après 60 jours sans message d'un numéro, l'accès est suspendu jusqu'à la saisie du PIN (patron) ou la réactivation par le patron (employé). Les opérateurs réattribuent les numéros inactifs. | MVP |
| EF-M0-12 | **Mode démo** : toute personne peut essayer KAYE sur une boutique fictive pré-remplie, sans inscription, depuis un lien partageable (le « lien de test » que le vendeur transmet au patron). Données de démo isolées, effacées après 7 jours, jamais certifiées. | MVP |
| EF-M0-13 | **Comptable multi-dossiers** : un comptable ou un cabinet invité par plusieurs commerçants voit la liste de ses dossiers et passe de l'un à l'autre, en lecture seule. | V1 |

### M1 — Agent conversationnel

| Réf. | Exigence | Lot |
|---|---|---|
| EF-M1-01 | L'agent comprend le **français ivoirien courant** : fautes, abréviations, nouchi (« j'ai gbê »), montants en chiffres ou en lettres, « 45k », « 45 mille », « 45.000 », « 45 000 f cfa ». | MVP |
| EF-M1-02 | Les **notes vocales** sont transcrites puis traitées comme du texte. La transcription est affichée dans le récapitulatif. | MVP |
| EF-M1-03 | Les **photos** sont lues : bon de livraison, facture fournisseur, ticket manuscrit, capture Wave / Orange Money, photo d'un produit. | MVP (BL, facture) · V1 (captures, tickets) |
| EF-M1-04 | Toute demande est classée dans une **intention connue** (§ 12). Une demande hors périmètre reçoit une réponse courte et un menu de boutons ; l'agent n'improvise jamais sur l'argent ni hors du commerce. | MVP |
| EF-M1-05 | Toute écriture qui touche à l'argent, au stock ou à un tiers présente un **récapitulatif** puis des boutons **[Confirmer] [Corriger] [Annuler]**, quel que soit le niveau de confiance. | MVP |
| EF-M1-06 | Le récapitulatif affiche le texte d'origine (ou la transcription), les montants avec séparateurs de milliers et la devise (« 45 000 F CFA »), et les avertissements. | MVP |
| EF-M1-07 | La correction se fait en langage naturel dans le fil (« non c'est 40 000 »), sans tout ressaisir. | MVP |
| EF-M1-08 | Un champ ambigu (deux clients « Koffi », un article inconnu) donne lieu à une question à choix ; **rien n'est deviné en silence**. | MVP |
| EF-M1-09 | La conversation est **reprenable** : une saisie interrompue reste en attente (24 h), le commerçant peut changer de sujet et revenir. | MVP |
| EF-M1-10 | Commande « aide » et aide **contextuelle par module** (« aide ventes ») avec 3 à 5 exemples copiables, sur le modèle d'une page de manuel. | MVP |
| EF-M1-11 | L'agent tient compte du **rôle** : une action hors droits est refusée avant exécution ou envoyée en validation au patron. | MVP |
| EF-M1-12 | **Catalogue qui apprend** : prix unitaires et désignations habituelles sont mémorisés et proposés (« riz » → « Riz 25 kg, 19 500 F »). | MVP |
| EF-M1-13 | Ventes multi-articles dans une même phrase (« vendu 2 riz et 1 huile à Awa 52 000 cash »). | MVP |
| EF-M1-14 | Réponse audio facultative (le commerçant dans le bruit du magasin). | V2 |
| EF-M1-15 | Toute saisie non comprise ou corrigée alimente un corpus de revue pour améliorer l'agent. | MVP |
| EF-M1-16 | Le récapitulatif tient sur un écran de téléphone : au plus 6 lignes, montant total en gras, boutons en un seul tap. Confirmer ne demande jamais de retaper quoi que ce soit. | MVP |
| EF-M1-17 | Langues locales (dioula, baoulé) en saisie vocale. | V2 |

### M2 — Ventes

**Parcours 1 — Vente au comptoir (MVP).** Une phrase couvre commande, sortie de stock, facture et encaissement.

| Réf. | Exigence | Lot |
|---|---|---|
| EF-M2-01 | « vendu [qté] [article] à [client] [montant] [mode] » enregistre une vente. Client facultatif (client de passage). Mode par défaut : espèces. | MVP |
| EF-M2-02 | Modes reconnus : espèces, Wave, Orange Money (« om »), MTN MoMo, Moov Money, virement, chèque, carte, **crédit** (« à crédit », « pas payé », « il paie samedi »). | MVP |
| EF-M2-03 | Paiement **partiel ou mixte** (« 30 000 cash, reste demain ») : la part payée est encaissée, le reste devient une créance. | MVP |
| EF-M2-04 | Si le montant est absent mais le prix connu au catalogue, KAYE calcule et le montre ; si le montant saisi s'écarte fortement du prix habituel, il avertit. | MVP |
| EF-M2-05 | À la confirmation : vente, lignes, sortie de stock, encaissement ou créance, facture (FNE si activée) — **en une seule opération** atomique. | MVP |
| EF-M2-06 | Vente à crédit : contrôle de la limite du client ; alerte à 80 % ; au-delà, validation du patron. | MVP |
| EF-M2-07 | Après la vente, KAYE propose d'envoyer la facture au client (par le WhatsApp de la boutique quand il est connecté, sinon en PDF à transférer). | MVP |
| EF-M2-08 | Retour client : « retour [qté] [article] de [client] [motif] » ; contrepartie échange, avoir ou remboursement, décidée par le commerçant ; marchandise remise en stock ou au rebut. | MVP |
| EF-M2-09 | Consultation : « ventes du jour », « ventes de la semaine », « ventes de Awa », top articles. | MVP |
| EF-M2-10 | **Prestations de service** (livraison, découpe, réparation…) : article de type *service*, sans stock, facturable en FNE. | MVP |
| EF-M2-11 | **Avance et avoir client** : un client qui paie d'avance ou reçoit un avoir non remboursé a un **solde créditeur**, proposé en déduction sur ses achats suivants. | MVP |
| EF-M2-12 | Une vente saisie avec un prix pour un article sans prix de vente propose d'enregistrer ce prix au catalogue (« Je garde 15 000 F comme prix du riz ? »). | MVP |

**Parcours 2 — Vente à distance sur WhatsApp (V1).**

| Réf. | Exigence | Lot |
|---|---|---|
| EF-M2-20 | L'agent lit la demande d'un client sur le WhatsApp de la boutique (texte, vocal, photo d'article), reconnaît les articles, répond avec le **stock disponible** et le **prix du client**. | V1 |
| EF-M2-21 | Article en rupture : alternative proposée ; la demande est enregistrée comme **demande non servie** et alimente le réappro. | V1 |
| EF-M2-22 | Question hors catalogue, négociation, ton tendu ou réclamation : l'agent accuse réception, se tait et **alerte le commerçant**. | V1 |
| EF-M2-23 | **Devis** en une phrase (« devis 40 riz pour Koné remise 5 % »), durée de validité, PDF envoyé après validation ; un « ok » du client le transforme en commande sans ressaisie. | V1 |
| EF-M2-24 | **Commande** avec réservation du stock ; récapitulatif envoyé au client ; acompte possible ; confirmation automatique sous un montant fixé par le patron. | V1 |
| EF-M2-25 | Liste de préparation du jour ; « livré Fatou » sort le stock, lève la réservation, gère le reliquat ; preuve de livraison (« reçu » du client ou photo du BL signé). | V1 |
| EF-M2-26 | Grilles de prix (détail, gros, négocié par client) et plafond de remise par rôle. | V1 |
| EF-M2-27 | Propositions commerciales aux clients ayant donné leur accord (opt-in), par gabarit marketing, avec plafond par contact et par semaine. | V2 |

Cycle d'une commande : `devis → confirmée → préparée → partiellement livrée → livrée → facturée → soldée` (ou `annulée`). Une vente au comptoir parcourt tout le cycle en une opération.

### M3 — Facturation FNE

| Réf. | Exigence | Lot |
|---|---|---|
| EF-M3-01 | Pour un établissement FNE actif, chaque vente confirmée est **certifiée par l'API FNE de la DGI**. KAYE récupère le **numéro normatif**, le **lien de vérification** (converti en QR code) et le solde de stickers. | MVP |
| EF-M3-02 | KAYE ne fabrique jamais le numéro normatif ni le QR : ils viennent de la DGI. KAYE tient en plus sa **numérotation interne continue** (`VTE-AAAA-NNNNNN`) pour rattacher chaque facture même quand la certification est différée. | MVP |
| EF-M3-03 | Type de facturation déduit du client : **B2C** (particulier), **B2B** (professionnel avec NCC — NCC obligatoire), **B2G** (administration), **B2F** (étranger, devise et taux obligatoires). | MVP (B2C, B2B) · V1 (B2G, B2F) |
| EF-M3-04 | Taxe par ligne : TVA 18 %, TVA réduite 9 %, exonérations conventionnelle et légale (0 %), selon le régime de l'établissement et l'article ; taxes additionnelles possibles. Valeur par défaut fixée à l'activation de la FNE. | MVP |
| EF-M3-05 | Le PDF de facture porte toutes les mentions obligatoires, le **visuel FNE**, le **QR code** vérifiable par tout smartphone, la boutique, le client, les lignes, HT, TVA, TTC, le mode de paiement. Poids **< 200 Ko**. | MVP |
| EF-M3-06 | **Ne jamais bloquer une vente** : si la DGI ou le réseau ne répond pas, la vente est enregistrée, un **reçu provisoire horodaté** (empreinte vérifiable) est disponible, la certification est rejouée automatiquement ; le commerçant est prévenu quand elle aboutit. | MVP |
| EF-M3-07 | Une facture certifiée n'est **jamais modifiée ni supprimée**. Toute correction passe par un **avoir certifié** (total ou partiel, par ligne et quantité). | MVP |
| EF-M3-08 | Recherche et renvoi d'une facture par client, date, montant ou numéro (« facture de Awa du 12 »). | MVP |
| EF-M3-09 | Indicateur de conformité : factures certifiées, en attente, rejetées ; solde de stickers ; alerte quand il devient bas. | MVP |
| EF-M3-10 | Pied de page discret « Facture émise avec KAYE » + lien : levier d'acquisition, compatible avec les mentions obligatoires. | MVP |
| EF-M3-11 | Mode **démonstration** explicite (environnement de test DGI) : mention « SPÉCIMEN — NON CERTIFIÉ ». | MVP |
| EF-M3-12 | Bordereau d'achat de produits agricoles (type « purchase » de l'API FNE). | V2 |
| EF-M3-13 | Reçu normalisé (RNE) pour les ventes au détail, si la voie est confirmée avec la DGI. | V2 |

### M4 — Clients et créances

| Réf. | Exigence | Lot |
|---|---|---|
| EF-M4-01 | Création d'un client **particulier** (« client Kouamé Akissi 0707123456 50000 ») ou **professionnel** (« NCC 0247813650 Épicerie Koné & Frères 0101234567 200000 ») ; le type est demandé si absent. | MVP |
| EF-M4-02 | Création à la volée pendant une vente : un nom inconnu propose « Nouveau client Awa ? ». | MVP |
| EF-M4-03 | Fiche client : téléphone, type, NCC, limite de crédit, encours, % de limite utilisé (alerte ≥ 80 %), historique, plus vieille créance. | MVP |
| EF-M4-04 | « Koffi doit combien », « qui me doit », « liste des dettes » : détail par échéance, total, classement des débiteurs. | MVP |
| EF-M4-05 | « Koffi a payé 20 000 wave » : règlement partiel ou total, imputé sur la créance la plus ancienne (sauf indication), encaissement en caisse, reçu proposé. | MVP |
| EF-M4-06 | Relance d'un client ou de tous les débiteurs, **déclenchée par le commerçant** : aperçu du message, nombre de destinataires, coût estimé, puis validation (PIN pour un envoi groupé). | MVP |
| EF-M4-07 | Envoi étalé, plafond de relances par client et par semaine ; un client qui répond STOP n'est plus relancé. | MVP |
| EF-M4-08 | **Balance âgée** : moins de 30 jours, 30 à 60, plus de 60. | V1 |
| EF-M4-09 | Le carnet de dettes est disponible **dans tous les paliers, y compris gratuit**. | MVP |

### M5 — Achats et fournisseurs

**Parcours 1 — Achat direct : une photo suffit (MVP).** La plupart des achats en TPE se font au dépôt, payés sur place, sans commande préalable.

| Réf. | Exigence | Lot |
|---|---|---|
| EF-M5-01 | Le commerçant envoie la **photo du BL** (ou de la facture) au bot. KAYE lit les lignes (désignation, référence, quantité, prix), reconnaît les articles connus, signale les inconnus. | MVP |
| EF-M5-02 | Toute ligne lue avec une confiance insuffisante (BL manuscrit, photo floue) est **redemandée, jamais devinée**. | MVP |
| EF-M5-03 | Une confirmation crée la réception, l'entrée en stock, le **coût d'achat** (coût moyen pondéré) et, sans commande préalable, une commande soldée a posteriori. | MVP |
| EF-M5-04 | Paiement : « payé cash », capture Wave, ou rien → le montant passe en **dette fournisseur**. | MVP |
| EF-M5-05 | **Base article qui apprend** : dans l'ordre, référence fournisseur connue → code-barres → désignation proche (« c'est le même ? ») → proposition de création. Une référence liée n'est plus jamais redemandée. | MVP |
| EF-M5-06 | Création d'un article depuis une ligne de BL : fiche pré-remplie (désignation, réf., unité, prix d'achat) ; le commerçant fixe le **prix de vente** (proposé à partir de sa marge habituelle, jamais appliqué sans validation) et peut ajouter une photo. | MVP |
| EF-M5-07 | Fiche fournisseur (nom, téléphone WhatsApp, NCC, conditions de paiement) ; création en une phrase ou à la volée. | MVP |
| EF-M5-08 | « je dois combien à Sow », « payé 150 000 à Sow par Wave », « relevé Sow ». Règlement réservé au patron, avec PIN. | MVP |
| EF-M5-09 | **Alerte de marge** : quand le coût moyen augmente au-delà du seuil de la boutique, KAYE propose un nouveau prix de vente. | V1 |

**Parcours 2 — Achat planifié (V1).** Pour les semi-grossistes et le crédit fournisseur.

| Réf. | Exigence | Lot |
|---|---|---|
| EF-M5-20 | **Réappro** : quand un article passe sous son seuil, KAYE propose quantité (jusqu'au stock cible, arrondie à l'unité d'achat) et fournisseur habituel. | V1 |
| EF-M5-21 | **Demande de prix** à un ou plusieurs fournisseurs, envoyée après validation par le WhatsApp de la boutique ; réponses lues (texte, vocal, photo de tarif) et comparées en tableau. | V1 |
| EF-M5-22 | **Bon de commande** : prix et remise figés, conversion unité d'achat → unité de stock (1 carton = 12), conditions de paiement, PDF envoyé après validation ; la confirmation du fournisseur met à jour le statut. | V1 |
| EF-M5-23 | **Réception contre commande** : reliquats, quantités refusées, retours fournisseurs (avoir, remplacement, déduction) avec motif codé. | V1 |
| EF-M5-24 | **Contrôle de facture** (commande = réception = facture, tolérance paramétrable) ; un litige ne bloque que le montant contesté. | V1 |
| EF-M5-25 | **Déconditionnement** (« ouvrir 1 sac riz » → 25 kg vendables au kilo). | V1 |
| EF-M5-26 | Réception des factures fournisseurs par **e-mail dédié** (§ 7.4). | V1 |
| EF-M5-27 | Fournisseurs eux-mêmes sur KAYE : la commande d'achat du détaillant arrive comme commande de vente chez le grossiste. | V2 |

Cycle d'une commande d'achat : `brouillon → envoyée → confirmée → partiellement reçue → reçue → facturée → soldée` (ou `annulée`).

### M6 — Stock

| Réf. | Exigence | Lot |
|---|---|---|
| EF-M6-01 | Le stock est un **journal de mouvements** ; la quantité est la somme des mouvements. Aucun mouvement n'est modifié ni supprimé : on le compense. | MVP |
| EF-M6-02 | **Aucun inventaire initial n'est exigé** avant la première vente. Un article se crée au premier usage. | MVP |
| EF-M6-03 | Les ventes et réceptions mouvementent le stock automatiquement. | MVP |
| EF-M6-04 | Sorties hors vente en une phrase, **avec motif** : « cassé 2 huile », « pris 1 sac riz maison », « périmé 3 lait ». | MVP |
| EF-M6-05 | Un stock qui deviendrait négatif **n'empêche pas la vente** : avertissement et comptage programmé de l'article. | MVP |
| EF-M6-06 | Seuil d'alerte par article ; alerte quand qté ≤ seuil, regroupée dans le résumé du jour. | MVP |
| EF-M6-07 | « état stock », « stock riz », « alerte stock », valeur du stock au coût moyen. | MVP |
| EF-M6-08 | Saisie directe « il me reste 12 sacs de riz » = ajustement, soumis à confirmation et au PIN au-delà du seuil. | MVP |
| EF-M6-09 | **Comptage guidé** (inventaire tournant) : chaque jour, à l'heure choisie, le bot demande de compter quelques articles (priorité : valeur élevée, ventes fréquentes, écart récent, stock négatif). La quantité attendue n'est **pas** affichée avant le comptage. | V1 |
| EF-M6-10 | **Écarts valorisés en F CFA** avec motif obligatoire ; validation du patron au-delà du seuil ; écarts répétés signalés sans accuser personne. | V1 |
| EF-M6-11 | Dates de péremption pour les articles périssables ; alerte à 30 jours. | V1 |
| EF-M6-12 | Stock réservé par les commandes clients : disponible = physique − réservé. | V1 |
| EF-M6-13 | Transferts entre boutiques. | V2 |
| EF-M6-14 | **Coût inconnu** : un article vendu avant toute réception n'a pas de coût d'achat. Il est signalé (« coût à compléter ») et le résultat du mois indique la part des ventes dont la marge est inconnue, au lieu d'afficher une marge fausse. | MVP |

### M7 — Caisse et trésorerie

| Réf. | Exigence | Lot |
|---|---|---|
| EF-M7-01 | **Session de caisse journalière** par boutique : solde d'ouverture, encaissements, décaissements, solde théorique. | MVP |
| EF-M7-02 | « caisse du jour », « encaissements », « décaissements », « balance caisse », avec ventilation par mode de paiement. | MVP |
| EF-M7-03 | Une vente à crédit **ne crée pas** d'encaissement : elle crée une créance ; l'encaissement apparaît au règlement. | MVP |
| EF-M7-04 | **Clôture du soir** : le bot demande le comptage des espèces, calcule l'écart, exige un motif (monnaie, prélèvement, erreur, inconnu). Une journée close n'accepte plus d'écriture (correction datée du jour suivant). | MVP |
| EF-M7-05 | Dépense hors achats en une phrase ou une photo (« payé loyer 75 000 cash », photo de la facture d'électricité). | MVP |
| EF-M7-06 | **Un compte de trésorerie par moyen** : espèces, Wave, Orange Money, MTN, Moov, banque ; soldes suivis ; capture de solde mobile money lue à la clôture. | V1 |
| EF-M7-07 | **Transferts** entre comptes (« versé 100 000 sur wave ») : ni recette ni dépense. | V1 |
| EF-M7-08 | **Prélèvements du patron** (« pris 20 000 maison ») : ligne normale, suivie à part, hors charges. | V1 |
| EF-M7-09 | Rapprochement d'un relevé mobile money ou bancaire ; frais de transaction détectés. | V2 |

### M8 — Comptabilité simplifiée

La comptabilité est un **sous-produit** des autres flux : le commerçant ne « fait pas sa compta ».

| Réf. | Exigence | Lot |
|---|---|---|
| EF-M8-01 | Plan de catégories simple (marchandises, loyer, énergie, transport, salaires, frais mobile money, impôts, divers), chaque catégorie reliée à un compte SYSCOHADA pour le comptable. | V1 |
| EF-M8-02 | « résultat septembre » : ventes, coût des marchandises vendues (au coût moyen), charges, résultat ; prélèvements à part. | V1 |
| EF-M8-03 | Export comptable mensuel (journaux ventes, achats, trésorerie) en tableur et PDF ; une période exportée est verrouillée. | V1 |
| EF-M8-04 | Calendrier des échéances fiscales selon le régime de l'établissement, montants préparés (CA, TVA collectée − TVA déductible) ; rappels quelques jours avant. KAYE **ne dépose rien**. | V2 |
| EF-M8-05 | Dossier de chiffres exportable pour un banquier ou une microfinance (sans scoring). | V2 |

### M9 — Application web

| Réf. | Exigence | Lot |
|---|---|---|
| EF-M9-01 | Connexion par **code à usage unique** reçu sur WhatsApp, Telegram ou SMS ; choix de la boutique ; déconnexion et changement de compte. | MVP |
| EF-M9-02 | **Accueil chat** conforme à la charte Bissap (§ 7.3) : même agent que sur WhatsApp, mêmes intentions, mêmes confirmations. | MVP |
| EF-M9-03 | **Mode menus** : listes Ventes, Factures, Clients, Créances, Achats, Fournisseurs, Stock, Caisse, Rapports, avec filtres et recherche ; **2 clics maximum** vers toute donnée. | MVP |
| EF-M9-04 | Sélection d'une rubrique = insertion de la syntaxe adaptée dans la barre de saisie (assistant de saisie). | MVP |
| EF-M9-05 | **Tableau de bord** : solde de caisse et variation du jour, ventes du jour, créances totales, stock en alerte, factures FNE en attente, top débiteurs. | MVP |
| EF-M9-06 | Formulaires courts uniquement comme repli (articles, clients, paramètres) ; boutons +/− pour les quantités. | MVP |
| EF-M9-07 | Une demande en langage naturel peut produire une **liste filtrée** (« commandes non payées de septembre »). | V1 |
| EF-M9-08 | Visualisation et téléchargement des PDF ; partage par WhatsApp. | MVP |
| EF-M9-09 | Paramètres : boutique, NCC et FNE, utilisateurs et rôles, PIN, seuils (alerte stock, ajustement, marge, remise), heure du résumé, réglages de l'agent sur le canal commercial. | MVP |
| EF-M9-10 | Fonctionne sur Android d'entrée de gamme en 3G ; affichage mobile d'abord. | MVP |
| EF-M9-11 | Mode hors ligne : saisie conservée et synchronisée au retour du réseau (sans numéro FNE hors ligne). | V2 |

### M10 — Notifications et résumés

| Réf. | Exigence | Lot |
|---|---|---|
| EF-M10-01 | **Résumé du soir** à l'heure choisie : ventes, encaissements par mode, caisse théorique, créances ouvertes, alertes de stock, factures en attente. | MVP |
| EF-M10-02 | Les notifications partent sur **Telegram si lié** (gratuit), sinon sur WhatsApp (gabarit utilitaire hors fenêtre de 24 h, coût budgété). | MVP |
| EF-M10-03 | Alertes regroupées : au plus un message d'alerte non sollicité par jour, sauf urgence (facture rejetée par la DGI). | MVP |
| EF-M10-04 | Alertes de fin de pass à J-7, J-3, J-1. | V1 |

### M11 — Import et onboarding

| Réf. | Exigence | Lot |
|---|---|---|
| EF-M11-01 | Import d'articles, clients, fournisseurs par **fichier CSV / Excel** avec modèles téléchargeables et rapport d'erreurs ligne à ligne. | MVP |
| EF-M11-02 | Création du catalogue par **photo** (BL, facture, étagère) : propositions de fiches à valider en lot. | V1 |
| EF-M11-03 | Import de soldes d'ouverture : créances clients et dettes fournisseurs existantes, stock de départ (facultatif). | MVP |
| EF-M11-04 | Connecteurs vers d'autres logiciels (Odoo, etc.) et API publique KAYE pour des partenaires intégrateurs. | V2 |

### M12 — Abonnement et pass prépayés

| Réf. | Exigence | Lot |
|---|---|---|
| EF-M12-01 | Paliers : **Découverte** (gratuit), **Boutique**, **Commerce**, **Réseau** (§ 15). Les limites sont appliquées par compteur (factures FNE/mois, utilisateurs, boutiques). | MVP |
| EF-M12-02 | Paiement par **pass prépayé** (3 ou 6 mois) via lien marchand Wave ou Orange Money vers le compte de KAYE ; pas de prélèvement automatique. | MVP (activation manuelle par le back-office) · V1 (automatique) |
| EF-M12-03 | À l'échéance, le compte redescend au palier gratuit : **aucune perte d'accès aux données** ni au carnet de dettes. | MVP |
| EF-M12-04 | Parrainage : « offrez 1 mois à un confrère = 1 mois offert ». | V1 |
| EF-M12-05 | Paiement par un tiers (un grossiste finance les pass de ses fournisseurs). | V2 |

### M13 — Back-office et support

| Réf. | Exigence | Lot |
|---|---|---|
| EF-M13-01 | Console interne : comptes, palier, statut FNE, files de certification, échecs d'API, reprise manuelle d'une facture bloquée. | MVP |
| EF-M13-02 | Revue des saisies non comprises ou corrigées (corpus d'amélioration). | MVP |
| EF-M13-03 | **Support dans le fil** : un opérateur peut reprendre la conversation depuis le canal de pilotage ; le commerçant le voit. | MVP |
| EF-M13-04 | Journal d'audit immuable de toute écriture financière (auteur, date, avant/après, message source). | MVP |
| EF-M13-05 | Activation manuelle d'un pass sur preuve de paiement. | MVP |
| EF-M13-06 | **Activation progressive** : chaque fonction nouvelle peut être ouverte commerçant par commerçant (pilotes d'abord). | MVP |
| EF-M13-07 | Conditions d'utilisation, politique de confidentialité et mentions sur le traitement des données des clients du commerçant, acceptées à l'inscription (lien court dans le fil). | MVP |

### M14 — Canal commercial (WhatsApp Business de la boutique)

| Réf. | Exigence | Lot |
|---|---|---|
| EF-M14-01 | « Connecter mon WhatsApp » depuis l'application web : parcours Meta (Embedded Signup) en 3 à 5 minutes ; Coexistence activée ; le commerçant garde son application. | V1 |
| EF-M14-02 | Contacts importés en « inconnu » ; l'agent propose les étiquettes dans le canal de pilotage. | V1 |
| EF-M14-03 | **Aucun message vers un tiers sans validation** dans le canal de pilotage, sauf réponses automatiques aux clients dans la fenêtre de 24 h autorisées par le patron. | V1 |
| EF-M14-04 | Reprise en main : quand le commerçant écrit lui-même dans un fil, l'agent se met en pause sur ce fil (durée paramétrable). | V1 |
| EF-M14-05 | Réglages de l'agent : répondre toujours, seulement hors heures d'ouverture, ou seulement si le commerçant ne répond pas sous N minutes. | V1 |
| EF-M14-06 | Envois en attente listés (« en attente ») ; expiration à 24 h par défaut. | V1 |
| EF-M14-07 | Indicateur de qualité du numéro (Meta) visible dans le tableau de bord. | V1 |
| EF-M14-08 | Tant que le WhatsApp Business n'est pas connecté, l'agent fournit le PDF ou le texte à transférer à la main. | MVP |

---

## 10. Règles cardinales

1. **RG-1 — Ne jamais bloquer une vente.** Ni la DGI, ni le réseau, ni un stock insuffisant, ni un paramétrage incomplet n'empêchent d'enregistrer une vente.
2. **RG-2 — L'IA propose, l'humain valide.** Aucune écriture d'argent, de stock ou de facture sans confirmation explicite ; aucune certification d'un montant non confirmé.
3. **RG-3 — Aucun message vers un tiers sans « oui » du commerçant**, hormis les réponses aux clients dans la fenêtre de 24 h qu'il a explicitement autorisées.
4. **RG-4 — Ne jamais détenir, transporter ni convertir de fonds.**
5. **RG-5 — Le carnet de dettes n'est jamais derrière un paiement.**
6. **RG-6 — Aucun paramétrage préalable à la première vente** (pas d'inventaire, pas de catalogue obligatoire).
7. **RG-7 — L'agent ne sort pas du commerce** ; il n'improvise jamais sur l'argent.
8. **RG-8 — Pas de crédit, d'avance ni de scoring.**
9. **RG-9 — Ne jamais couper l'accès aux données** à l'échéance d'un pass.
10. **RG-10 — Une facture certifiée ne se modifie pas**, une écriture validée ne s'efface pas : on contre-passe.
11. **RG-11 — WhatsApp uniquement par l'API officielle de Meta.**
12. **RG-12 — Ne jamais deviner un chiffre** : toute lecture incertaine (vocal, photo) est redemandée.

---

## 11. Règles de gestion transverses

| Réf. | Règle |
|---|---|
| RG-GEN-01 | Montants en F CFA (XOF) **entiers** ; affichage « 45 000 F CFA ». |
| RG-GEN-02 | Quantités à 3 décimales (ventes au poids : 2,5 kg). |
| RG-GEN-03 | Dates affichées au fuseau Africa/Abidjan ; stockées en UTC. |
| RG-GEN-04 | Toute écriture porte : auteur, canal (WhatsApp, Telegram, web, e-mail, système), message source. |
| RG-GEN-05 | Documents numérotés par type, par boutique et par année, sans trou : `PREFIXE-AAAA-NNNNNN` (§ 13). |
| RG-VTE-01 | Prix annoncé = prix de la grille du client ; figé sur un devis pendant sa validité. |
| RG-VTE-02 | Une remise au-delà du plafond du rôle part en validation chez le patron. |
| RG-VTE-03 | Une réclamation n'est jamais résolue par l'agent seul. |
| RG-ACH-01 | Prix d'une ligne de commande figé à la validation. |
| RG-ACH-02 | Quantité en unité d'achat convertie en unité de stock par le coefficient de l'article fournisseur. |
| RG-ACH-03 | Seules les quantités acceptées entrent en stock ; une ligne « à référencer » n'entre pas en stock tant que l'article n'est pas créé ou lié. |
| RG-ACH-04 | Coût moyen pondéré recalculé à chaque réception. |
| RG-ACH-05 | Une référence fournisseur est unique par fournisseur et liée à un seul article. |
| RG-ACH-06 | Sur un fil étiqueté fournisseur, l'agent lit et extrait ; il ne répond jamais de lui-même. |
| RG-ACH-07 | Une réception sans commande crée une commande soldée a posteriori ; sans règlement déclaré, le montant passe en dette fournisseur. |
| RG-STK-01 | Stock = somme des mouvements ; toute sortie hors vente et hors retour fournisseur porte un motif. |
| RG-STK-02 | Stock négatif accepté avec avertissement et comptage programmé. |
| RG-CPT-01 | Tout mouvement d'argent est rattaché à un compte de trésorerie et à une source. |
| RG-CPT-02 | Un transfert entre comptes n'est ni une recette ni une dépense ; un prélèvement du patron n'est pas une charge. |
| RG-CPT-03 | Un écart de clôture exige un motif ; au-delà du seuil, le patron valide. |
| RG-FNE-01 | Une vente n'est certifiée qu'une fois ; une double soumission ne crée pas de doublon. |
| RG-FNE-02 | Le numéro FNE n'est jamais attribué hors ligne ni par KAYE. |
| RG-FNE-03 | Une facture en échec de certification n'est jamais oubliée : rejeu automatique, puis alerte au commerçant et au back-office. |

---

## 12. Référentiel des commandes reconnues

Les phrases sont des **exemples** : l'agent accepte les variantes, les fautes, le nouchi, les vocaux.

| Domaine | Exemples | Lot |
|---|---|---|
| Aide | `aide`, `aide ventes`, `?` | MVP |
| Ventes | `vendu 3 riz à Awa 45000 wave` · `vendu 2 huile à Koffi à crédit` · `30 000 cash reste demain` · `ventes du jour` · `retour 1 huile de Fatou cassée` | MVP |
| Ventes | `devis 40 riz pour Koné remise 5 %` · `commandes à préparer` · `livré Fatou` · `demandes non servies` | V1 |
| Factures | `facture de Awa du 12` · `renvoie la facture 849` · `factures en attente` | MVP |
| Clients | `client Kouamé Akissi 0707123456 50000` · `NCC 0247813650 Épicerie Koné 0101234567` · `liste des clients` · `limite Koffi 150000` | MVP |
| Créances | `Koffi doit combien` · `qui me doit` · `Koffi a payé 20 000 wave` · `relance Koffi` · `relance tous` | MVP |
| Achats | `[photo du BL]` · `payé cash` · `fournisseur Sow 0707000000` · `je dois combien à Sow` · `payé 150 000 à Sow par Wave` · `articles à créer` | MVP |
| Achats | `réappro` · `prix riz et sucre chez Sow et Diallo` · `commande 20 sacs riz chez Sow` · `commandes en cours` · `retour 1 sac riz chez Sow déchiré` · `relevé Sow` · `ouvrir 1 sac riz` | V1 |
| Stock | `état stock` · `stock riz` · `alerte stock` · `cassé 2 huile` · `pris 1 riz maison` · `il me reste 12 sacs de riz` | MVP |
| Stock | `compter` · `compter tout` · `écarts du mois` · `périment bientôt` | V1 |
| Caisse | `caisse du jour` · `encaissements` · `décaissements` · `payé loyer 75 000 cash` · `clôture caisse` | MVP |
| Trésorerie | `versé 100 000 sur wave` · `pris 20 000 maison` · `[capture du solde Wave]` | V1 |
| Compta | `résultat septembre` · `export comptable octobre` · `prochaines échéances` | V1 |
| Équipe | `ajoute Koffi 0707123456 vendeur` · `retire Koffi` · `boutique Treichville` | MVP |
| Canal commercial | `Sow est fournisseur` · `en attente` · `envoie` | V1 |

Modes de paiement : `cash/espèces/liquide` → espèces · `wave` → Wave · `om/orange/orange money` → Orange Money · `mtn/momo` → MTN MoMo · `moov/flooz` → Moov Money · `virement` · `chèque` · `carte` · `crédit/pas payé/demain` → crédit.

---

## 13. Documents générés

| Préfixe | Document | Module |
|---|---|---|
| (numéro DGI) + `VTE` | Facture de vente certifiée FNE (numéro interne VTE + numéro normatif DGI) | M3 |
| `AVO` | Avoir certifié FNE | M3 |
| `RCU` | Reçu (non certifié, ou provisoire en attente de certification) | M3 |
| `DEV` | Devis | M2 |
| `CMD` | Commande client | M2 |
| `BL` | Bon de livraison | M2 |
| `RTC` | Retour client | M2 |
| `DPX` | Demande de prix fournisseur | M5 |
| `BCA` | Bon de commande d'achat | M5 |
| `REC` | Bon de réception | M5 |
| `RET` | Retour fournisseur | M5 |
| `REG` | Règlement fournisseur (reçu) | M5 |
| `CPT` / `AJU` | Comptage / ajustement de stock | M6 |
| `CAI` | Rapport de caisse journalier | M7 |
| `DEP` | Dépense | M7 |
| `RES` | Résultat mensuel | M8 |
| `REL` | Relevé de compte client ou fournisseur | M4/M5 |

Tous les PDF : charte Bissap (sobre), < 200 Ko, lisibles sur téléphone.

---

## 14. Exigences non fonctionnelles (résumé)

| Réf. | Exigence | Cible |
|---|---|---|
| ENF-01 | Enregistrement d'une vente disponible même si la DGI ne l'est pas | 99,5 % |
| ENF-02 | Accusé conversationnel | < 3 s (texte) · < 8 s (vocal, photo) |
| ENF-03 | Certification en fonctionnement nominal | < 15 s |
| ENF-04 | Exactitude d'extraction des **montants** sur le corpus de recette | > 99 % |
| ENF-05 | Coût variable par commerçant actif (IA, messages, stockage, certification) | < 2 €/mois |
| ENF-06 | Application web sur Android d'entrée de gamme en 3G | premier affichage < 3 s |
| ENF-07 | Cloisonnement strict des données entre commerçants | contrôlé en base et testé automatiquement |
| ENF-08 | Archivage des factures et pièces | 10 ans |
| ENF-09 | RGPD (structure française) + droit ivoirien applicable | registre, sous-traitants documentés |
| ENF-10 | Accessibilité | contrastes WCAG AA, cibles tactiles ≥ 44 px |

Le détail (mesure, vérification) est dans le CDC technique.

---

## 15. Modèle économique et contraintes produit

### 15.1 Grille de référence (feuille de route d'octobre 2026)
| Palier | Prix | Pour qui | Contenu |
|---|---|---|---|
| **Découverte** | 0 F | Awa, premiers pas | 30 factures FNE/mois, carnet de dettes, caisse, stock simple, 1 utilisateur |
| **Boutique** | 5 000 F/mois · pass 3 mois 13 000 F · 6 mois 24 000 F | Amadou, Yao | FNE illimitée, achats par photo, stock, relances, 2 utilisateurs |
| **Commerce** | 12 000 F/mois | Koné & Frères, Fatou | Canal commercial, achats planifiés, compta simplifiée, exports, 5 utilisateurs |
| **Réseau** | 30 000 F/mois (V2) | Fatou grands comptes | Multi-établissements, tableau de bord consolidé |

Panier moyen visé : ~8 000 F CFA/mois ; marge brute cible 82 % ; conversion gratuit → payant visée 8 %.

### 15.2 Contraintes produit qui en découlent
1. **Prépayé, pas d'abonnement** : pas de prélèvement automatique en mobile money.
2. **Coût variable plafonné** (ENF-05) : pas d'appel systématique à un gros modèle ; règles d'abord, IA ensuite, modèle choisi selon la tâche.
3. **Conception entrante** : le commerçant écrit d'abord, ce qui garde la conversation dans la fenêtre gratuite de 24 h ; tout message initié par KAYE est compté.
4. **Le volume est le métier** : tout ce qui freine l'inscription (NCC exigé, inventaire, mot de passe) coûte des clients.

---

## 16. Critères de recette et indicateurs

### 16.1 Tests d'acceptation
| Domaine | Test | Seuil |
|---|---|---|
| Compréhension | 100 phrases réelles (fautes, nouchi, montants sans séparateur) : montant correct | > 99 % |
| Compréhension | 50 vocaux enregistrés en boutique : montant correct | > 95 % |
| Photo | 30 vrais BL de pilotes : lignes correctes ou signalées comme incertaines | 100 % (aucune ligne fausse acceptée sans question) |
| Confirmation | 200 scénarios : aucune écriture ni certification sans confirmation | 100 % |
| Résilience | API DGI coupée : vente enregistrée, reçu provisoire disponible | 100 % |
| Résilience | Rattrapage de la file après 24 h d'indisponibilité simulée | 100 % |
| Conformité | Spécimens validés par la DGI (vente, avoir) | validation écrite |
| Cloisonnement | Accès croisé entre deux commerçants sur chaque ressource | 0 fuite |
| Performance | Accusé conversationnel en 3G dégradée | < 3 s |
| Coût | Coût variable mesuré sur un mois de pilote | < 2 € par actif |

### 16.2 Scénarios de bout en bout (à automatiser)
| Réf. | Scénario | Vérification |
|---|---|---|
| E2E-01 | Vente texte → confirmation → FNE certifiée → PDF envoyé | Stock, caisse, facture cohérents |
| E2E-02 | Vente vocale à crédit → relance → règlement partiel | Créance, encaissement au règlement seulement |
| E2E-03 | Client professionnel avec NCC → facture B2B | NCC présent, template B2B |
| E2E-04 | Photo de BL avec un article inconnu → création → entrée en stock | Ligne en attente puis entrée, coût moyen |
| E2E-05 | Achat non payé → dette fournisseur → règlement Wave avec PIN | Dette soldée, décaissement |
| E2E-06 | Annulation d'une vente certifiée | Avoir certifié, facture d'origine intacte |
| E2E-07 | DGI indisponible 2 h | Vente acceptée, reçu provisoire, certification rejouée |
| E2E-08 | Clôture de caisse avec écart | Écart motivé, journée verrouillée |
| E2E-09 | Vendeur tente un règlement fournisseur | Refus ou validation patron |
| E2E-10 | Pass expiré | Retour au gratuit, données intactes |

### 16.3 Indicateurs produit
Commerçants actifs 5 jours sur 7 · ventes saisies par jour · délai médian d'une vente (< 15 s) · taux de confirmation du premier coup · taux de correction · part des vocaux et des photos · factures en attente > 6 h · conversion gratuit → payant · réachat de pass.

---

## 17. Calendrier

| Période | Jalon | Critère de sortie |
|---|---|---|
| **Maintenant → 15 nov.** | Lot 0 (en parallèle du code) : demande d'accès API FNE et environnement de test DGI ; Business Verification Meta ; choix du BSP ; corpus de 100 phrases et 30 BL réels | Accès test DGI obtenu ; dossier Meta déposé |
| **Novembre** | Lot 1 — socle : compte, agent texte/vocal, ventes comptoir et crédit, clients et créances, caisse, stock journalisé, FNE en environnement de test, Telegram + application web | Démo complète en recette |
| **Décembre** | Lot 2 — pilote : WhatsApp de pilotage, achat direct par photo du BL, résumé du soir, import CSV, pass (activation manuelle) ; **15 pilotes à Abidjan** ; spécimens FNE envoyés à la DGI ; gel du code fin décembre | 10 pilotes sur 15 actifs 5 j/7 ; 5 paiements réels à 13 000 F sans aide ; accès production DGI |
| **Janvier 2027** | **Lancement public en Côte d'Ivoire** | Go / No-Go (§ 17.1) |
| **Février → juin 2027** | V1 : canal commercial (Coexistence), ventes WhatsApp, devis, achats planifiés, comptage guidé, trésorerie multi-comptes, compta simplifiée, e-mail entrant, pass automatisés | 50 payants sans remise (juin) |
| **Juillet 2027 →** | V2 : multi-établissements, palier Réseau, relais à Abidjan, 2e pays UEMOA | Équilibre visé M18 |

### 17.1 Go / No-Go avant ouverture publique
1. Accès écrit à l'API FNE (production) ou à un éditeur agréé.
2. Test en boutique validé : ventes réelles dictées sans erreur de montant.
3. Rétention pilote : 10/15 actifs au moins 5 jours sur 7.
4. 5 commerçants ont payé 13 000 F sans remise ni aide.
5. Coût d'acquisition < 8 € sur une campagne Click-to-WhatsApp.
6. Moteur anti-blocage FNE opérationnel (file, rejeu, reçu provisoire).
7. Relais local identifié pour le mois 7.
8. Interface conforme à la charte Bissap ; zéro détention de fonds.

---

## 18. Risques

| Réf. | Risque | Niveau | Traitement |
|---|---|---|---|
| R-01 | L'accès API FNE n'est pas ouvert à un éditeur tiers opérant pour le compte de commerçants | Sévère | Lot 0 par écrit ; connecteur FNE derrière une interface (API directe ou éditeur agréé) ; pivot carnet de gestion pur |
| R-02 | Les commerçants ne paient pas seuls un pass | Sévère | Test de paiement réel en décembre ; pivot payeur tiers (grossistes, grands comptes) |
| R-03 | Extraction peu fiable en français ivoirien et nouchi | Sévère | Confirmation systématique, corpus réel, saisie guidée par boutons en repli |
| R-04 | Vérification Meta et Tech Provider longues | Élevé | Démarrer maintenant ; Telegram et web en premier ; numéro de test WhatsApp en pilote |
| R-05 | Dépendance à Meta (tarifs, règles sur les assistants IA) | Moyen | Conception entrante, Telegram en parallèle, repli SMS documenté |
| R-06 | Fraude interne (ventes non saisies par les employés) | Moyen | Comptage guidé, écarts valorisés, clôture de caisse, journal d'audit |
| R-07 | Concurrence gratuite | Moyen | Différenciation : zéro installation, FNE native, stock intégré |
| R-08 | Absence de présence terrain | Élevé | Relais Abidjan budgété ; vidéo et lien de test à laisser au patron |
| R-09 | **Concurrent direct** (`carnet.net`, carnet de dettes + relances WhatsApp, Afrique de l'Ouest, lancé en septembre 2026) | Élevé | Vitesse d'exécution ; se différencier par ce qu'il n'a pas : FNE certifiée, stock et achats intégrés, application web ; ne plus rien publier sur le produit avant le pilote ; surveiller son offre et ses prix |
| R-10 | Fuite d'idées par des outils publics | Moyen | Dépôt privé, pas de documents produit publiés en ligne avant le lancement, déploiements de prévisualisation protégés par mot de passe |

---

## 19. Points à trancher

| Sujet | État | Proposition |
|---|---|---|
| Voie d'accès FNE : clé API de chaque commerçant, mandat, ou éditeur agréé | Non tranché | Interroger `support.fne@dgi.gouv.ci` cette semaine ; concevoir les deux voies |
| Nom | **Tranché le 31 oct. : KAYE** (charte Bissap conservée) | Vérifier la disponibilité INPI / OAPI et les domaines (`kaye.ci`, `kaye.app`…), puis déposer ; refaire le mot-symbole du logo (le « k » dans sa bulle reste valable) |
| Accès DGI depuis une société française | Ouvert (31 oct.) | Structure ou succursale locale en Côte d'Ivoire (Issouf) ; à confirmer avec la réponse de la DGI |
| Grille tarifaire : 2 500 / 7 500 F (pitch deck), 13 000 / 25 000 F (CR 31 août), 5 000 / 12 000 / 30 000 F (roadmap oct.) | Contradiction | Retenir la grille d'octobre jusqu'au test de paiement de décembre |
| Chiffres du pitch deck (12 pilotes, NPS 8,4, 47 s) non confirmés par la validation terrain (0 payant) | Incohérence | Ne plus utiliser ces chiffres tant qu'ils ne sont pas mesurés |
| Réponse automatique aux prospects (ADR-10 l'interdit aux inconnus) | Ouvert | Étiquette « prospect » dès qu'une intention d'achat est détectée ; première réponse validée |
| Moment de la facture FNE pour une commande livrée | Ouvert | À la livraison (B2C), à la commande pour les professionnels qui le demandent |
| Client B2C sans e-mail ni téléphone (champs exigés par l'API FNE) | Ouvert | Valider avec la DGI les valeurs admises pour un client de passage |
| Statut FNE de l'informel (Awa) | Non vérifié | Mode sans certification par défaut |
| Plafond d'utilisateurs : 5 sur l'offre de base (31 août) vs 2 (grille proposée) | Ouvert | Trancher avec la grille définitive |
| Numéros de téléphone de prospects dans les documents de travail | Vigilance | Ne pas les copier dans le code ni dans les jeux de test |
| Vente au détail aux particuliers : facture FNE (B2C) ou **reçu normalisé (RNE)** ? | Ouvert (conseil, expert fiscal) | Question à la DGI dans le même courrier que la voie éditeur |
| **Timbre fiscal** (champ `fiscalStamp` renvoyé par l'API) sur certains paiements en espèces | Ouvert | Vérifier avec un expert-comptable ivoirien avant le pilote |
| **Coût des stickers FNE** payé par le commerçant | Ouvert | Mesurer le coût réel ; l'expliquer dans l'offre et le palier gratuit |
| Encaisser les pass en Côte d'Ivoire (Wave Business, Orange Money marchand) avec une société française | Ouvert | Partenaire local, filiale CI ou agrégateur de paiement ; à trancher avant décembre |
| Déclaration à l'**ARTCI** (données personnelles en Côte d'Ivoire) et transfert vers l'UE | Ouvert | Avis d'un juriste ; démarche avant le pilote |
| Fatigue de confirmation (vendeur à 80 ventes par jour) | Débattu (§ 22) | RG-2 maintenue ; confirmation en un tap (EF-M1-16) ; mesure au pilote |

---

## 20. Glossaire

| Terme | Définition |
|---|---|
| FNE | Facture normalisée électronique, certifiée par la plateforme de la DGI |
| RNE | Reçu normalisé électronique |
| DGI | Direction générale des impôts de Côte d'Ivoire |
| NCC | Numéro de compte contribuable |
| ARF | Attestation de régularité fiscale |
| Sticker | Unité de certification FNE ; le solde est renvoyé par l'API à chaque facture |
| BL | Bon de livraison |
| CMP | Coût moyen pondéré |
| Canal de pilotage | Conversation entre le commerçant et son agent KAYE (WhatsApp KAYE, Telegram) |
| Canal commercial | WhatsApp Business de la boutique, utilisé par l'agent auprès des clients et fournisseurs |
| Coexistence | Mode Meta où l'application WhatsApp Business et l'API partagent le même numéro |
| BSP | Fournisseur de services WhatsApp Business (Solution Partner Meta) |
| Fenêtre de 24 h | Période ouverte par un message de l'utilisateur, pendant laquelle les réponses sont gratuites |
| Pass | Crédit d'utilisation prépayé de 3 ou 6 mois |
| Nouchi | Argot urbain ivoirien |
| SYSCOHADA | Référentiel comptable de l'espace OHADA |

---

## 21. Conseil des experts

### 21.1 Pourquoi
Avant de construire, chaque décision importante est attaquée par dix points de vue différents. Le Conseil cherche les failles (oublis, contradictions, risques, complexité inutile), les met en débat, et laisse une trace écrite. Il ne remplace pas les fondateurs : il les éclaire, eux tranchent.

Dans la pratique, le Conseil est un **ensemble de dix agents IA spécialisés** installés dans le dépôt (CDC technique § 18), lancés en une commande depuis Claude Code (`/conseil`) ou rejoués avec Codex. Les fondateurs peuvent y associer de vrais experts humains quand ils le souhaitent (comptable ivoirien, juriste, commerçant pilote) : ils reçoivent le même dossier et la même grille.

### 21.2 Composition
| # | Siège | Profil | Ce qu'il défend | Il peut bloquer sur |
|---|---|---|---|---|
| 1 | **Fonctionnel** (président de séance) | Analyste fonctionnel / product manager, 10 ans de cahiers des charges | Cohérence des exigences, couverture des parcours, traçabilité EF ↔ tests | Exigence ambiguë, parcours incomplet, contradiction entre documents |
| 2 | **Architecte full stack** | Architecte logiciel TypeScript / Postgres | Frontières de modules, modèle de données, contrats d'API, dette évitable | Modèle de données incohérent, couplage dangereux |
| 3 | **Développeur full stack** | Développeur senior React / NestJS qui livre au quotidien | Faisabilité, simplicité, tests, temps de développement réel | Ticket irréalisable ou non testable tel qu'écrit |
| 4 | **Staff engineer « grande échelle »** | Ingénieur passé par des équipes de type Amazon ou Tesla : systèmes distribués, fiabilité, exploitation | Idempotence, files, reprise sur incident, observabilité, coûts d'infrastructure | Perte ou duplication possible de données, absence de plan de reprise |
| 5 | **UX / UI** | Designer produit mobile, terrain Afrique de l'Ouest, charte Bissap | Zéro formation, 2 clics, lisibilité au soleil, faible alphabétisation, charte | Parcours qui exige une formation ou une saisie longue |
| 6 | **Expert ERP** | Consultant ERP très expérimenté : Odoo, Sage X3, SAP (achats, ventes, stocks, finance) | Intégrité des flux de gestion, valorisation des stocks, rapprochements, clôtures | Flux qui fausse le stock, la marge ou les comptes |
| 7 | **Expert fiscal et comptable ivoirien** | Expert-comptable OHADA / SYSCOHADA, pratique de la FNE et de la DGI | Conformité FNE, TVA, timbre, conservation, rôle du comptable | Facture non conforme, risque fiscal pour le commerçant |
| 8 | **Ingénieur IA** | Ingénieur LLM / OCR / voix, évaluation de modèles | Exactitude des montants, corpus d'évaluation, coût par opération, injection d'instructions | Montant pouvant être deviné ou certifié sans confirmation |
| 9 | **Sécurité et données** | RSSI, RGPD et loi ivoirienne sur les données (ARTCI) | Cloisonnement, secrets, identité par numéro de téléphone, conservation | Fuite possible entre commerçants, secret exposé, base légale absente |
| 10 | **Terrain et croissance** | Ancien responsable commercial PME à Abidjan, connaît WhatsApp Business, Wave, les cabinets comptables | Adoption, prix, décideur réel, coût d'acquisition, contraintes Meta | Fonction que personne n'utilisera, offre invendable |

### 21.3 Quand il se réunit
| Porte | Moment | Dossier examiné | Sortie |
|---|---|---|---|
| **G0 — Revue de lot** | Avant de démarrer chaque lot (lot 1, lot 2, V1…) | Les EF du lot, les tickets, les points à trancher | Liste des failles, tickets ajoutés ou modifiés, décisions |
| **G1 — Revue d'architecture** | À chaque nouvelle ADR ou changement de schéma important | L'ADR et le schéma | Avis par siège, ADR acceptée, amendée ou rejetée |
| **G2 — Revue de mise en production** | Avant chaque fusion `recette → main` | Notes de version, résultats de tests, migrations | Feu vert ou liste de blocages |
| **Revue ciblée** | À la demande, sur une PR sensible (argent, FNE, sécurité, agent) | La PR | Avis des 3 à 4 sièges concernés |

### 21.4 Déroulé d'une séance
1. **Dossier** : le président de séance (siège 1) liste ce qui est examiné et les documents de référence.
2. **Avis indépendants** : chaque siège rend ses constats sans voir ceux des autres. Chaque constat porte : niveau (**Bloquant**, **Majeur**, **Mineur**), référence (EF, RG, section, ticket), description, proposition.
3. **Débat** : les constats contradictoires sont mis face à face (par exemple UX « moins de confirmations » contre IA « confirmer tout montant »). Chaque siège répond une fois.
4. **Synthèse** : le président regroupe, dédoublonne, classe, et propose une décision pour chaque point.
5. **Décision** : les fondateurs acceptent, amendent ou rejettent. Les décisions vont au journal (§ 2) ; les nouveautés deviennent des EF ou des tickets.

Règles : un **Bloquant** non levé arrête le lot. Les fondateurs peuvent passer outre, en écrivant la raison dans le journal. Un rapport de séance est conservé dans `docs/conseil/AAAA-MM-JJ-<porte>.md`.

---

## 22. Revue n°1 du Conseil (31 octobre 2026)

Porte G0, dossier : CDC fonctionnel v3.0, CDC technique v3.0, backlog v3.0. Les corrections décidées sont **déjà intégrées** dans la version 3.1 des trois documents.

### 22.1 Constats et décisions
| Siège | Niveau | Constat | Décision intégrée |
|---|---|---|---|
| 1 Fonctionnel | Majeur | Le décideur (patron) n'est pas au comptoir : rien dans le produit ne permet au vendeur de lui transmettre un essai | **EF-M0-12 mode démo** et lien de test |
| 1 Fonctionnel | Mineur | Les CGU et la politique de confidentialité n'apparaissent nulle part | **EF-M13-07** |
| 2 Architecte | Majeur | Le traitement « séquentiel par conversation » reposait sur les groupes BullMQ, fonction de la version payante | Verrou Redis par conversation (CDC technique § 4.5) |
| 3 Développeur | Majeur | Impossible de tester les webhooks en local sans tunnel ni comptes Meta | Adaptateur `test` + tunnel documenté ; Telegram d'abord |
| 3 Développeur | Mineur | Pas de mécanisme pour ouvrir une fonction aux seuls pilotes | **EF-M13-06** activation progressive (feature flags) |
| 4 Staff engineer | Bloquant | Objectif RPO ≤ 5 min affirmé sans vérifier que l'hébergeur fait de la restauration à la minute | Exigence « restauration à un instant donné » imposée au choix de l'hébergeur (CDC technique § 16.4) |
| 4 Staff engineer | Majeur | Double certification possible en cas de coupure (pas d'idempotence côté DGI) | Statut `ambiguous`, revue manuelle, question à la DGI (déjà prévu, confirmé) |
| 5 UX / UI | Majeur | 80 ventes par jour × une confirmation : risque de lassitude et d'abandon | Débat ci-dessous ; **EF-M1-16** confirmation en un tap |
| 5 UX / UI | Mineur | Commerçants peu à l'aise à l'écrit | Vocaux en MVP (déjà) ; langues locales **EF-M1-17** (V2) |
| 6 Expert ERP | Majeur | Vente avant toute réception = coût inconnu : la marge serait fausse | **EF-M6-14** coût inconnu signalé |
| 6 Expert ERP | Majeur | Avances et avoirs non remboursés laissent un client créditeur sans traitement | **EF-M2-11** solde créditeur |
| 6 Expert ERP | Mineur | Les prestations (livraison, découpe) n'ont pas de stock | **EF-M2-10** articles de type service |
| 6 Expert ERP | Mineur | Le premier prix tapé doit pouvoir devenir le prix catalogue | **EF-M2-12** |
| 7 Fiscal | Bloquant | Pour la vente au détail aux particuliers, le bon document est peut-être le **reçu normalisé (RNE)** et non la facture B2C | Ajouté aux points à trancher, question à la DGI avant tout code de production FNE |
| 7 Fiscal | Majeur | Timbre fiscal possible sur paiements en espèces (`fiscalStamp`) ; stickers payants | Points à trancher ; coût des stickers à intégrer à l'offre |
| 8 IA | Majeur | Les numéros de téléphone et NCC partent dans les prompts | Masquage avant envoi au modèle (CDC technique § 7.3) |
| 8 IA | Majeur | Sans version de prompt, impossible de savoir quelle version a produit une erreur | `prompt_version` journalisé, prompts versionnés dans le dépôt |
| 9 Sécurité | Bloquant | L'identité repose sur le numéro : un numéro inactif réattribué par l'opérateur donnerait accès à la boutique | **EF-M0-11** suspension après 60 jours, PIN pour réactiver |
| 9 Sécurité | Majeur | Données de commerçants ivoiriens hébergées dans l'UE : déclaration ARTCI à vérifier | Point à trancher + tâche lot 0 |
| 10 Terrain | Bloquant | Une société française ne peut pas forcément ouvrir un compte marchand Wave ou Orange Money en Côte d'Ivoire : sans lui, pas d'encaissement des pass | Point à trancher + tâche lot 0 (partenaire local ou agrégateur) |
| 10 Terrain | Majeur | Le comptable prescrit ; il suit plusieurs clients | **EF-M0-13** comptable multi-dossiers (V1) |

### 22.2 Débat consigné : faut-il confirmer chaque vente ?
- **UX (5)** : un vendeur au comptoir fait des dizaines de ventes ; deux messages par vente ralentissent et lassent. Proposition : confirmation automatique des petites ventes de clients connus.
- **IA (8)** et **Fiscal (7)** : une facture certifiée avec un mauvais montant est un problème fiscal pour le commerçant et ne s'efface pas. L'extraction ne sera jamais fiable à 100 % sur des vocaux en boutique.
- **Terrain (10)** : les commerçants veulent avant tout être sûrs du montant ; la confirmation rassure si elle est instantanée.
- **Décision** : RG-2 maintenue. La confirmation tient en **un seul tap**, le récapitulatif en 6 lignes maximum (EF-M1-16). Le taux d'abandon après récapitulatif est mesuré pendant le pilote ; si la gêne est démontrée, le Conseil réexamine une confirmation groupée (« confirmer les 5 ventes de la dernière heure »), jamais une absence de confirmation.

### 22.3 Prochaine séance
G0 du lot 1, lancée par `/conseil G0 lot 1` dès que le dépôt est créé (voir le dossier `A_FAIRE`), puis à chaque porte du § 21.3.

---

## Annexe A — Sources consultées

**Comptes rendus de réunion** : 2, 3, 19, 23, 28, 31 août ; 20 et 24 septembre ; 17 octobre 2026.
**Conception** : Checkpoints 1 à 5, Concept Sheet, Carte d'empathie, Scénario de validation, REX prototype.
**Spécifications** : Cahier des charges Carnet v1 (août) et v2 (26 août), CDC_Fonctionnel_Karnet v1.0, STD_Karnet v1.0, ADR-09, ADR-10, CDC/SFD des trois prototypes Figma.
**Sprint 2** : Détails des flux, Achats dans KAYE v3, flux-karnet (27 oct.), Roadmap KAYE, Validation terrain (sept. 2026), Charte Bissap v1.0, logos (29 oct.).
**Réglementaire** : DGI — Procédure d'interfaçage des entreprises par API (mai 2025).
**Commercial** : Pitch deck, landing page v1, études mini-ERP TPE Côte d'Ivoire et France.
**Juridique** (non repris ici) : statuts, pacte d'associés, convention de vesting GAAGLE v2.
