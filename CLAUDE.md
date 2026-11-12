@AGENTS.md

# Consignes propres à Claude Code

## Ton rôle
Tu es l'**agent principal** de KAYE (CDC technique § 14). Tu prends en priorité les tickets marqués **CC** dans `docs/cdc/03_Backlog_KAYE_v3.md` : socle, base et RLS, domaine (ventes, stock, trésorerie), moteur d'agent, connecteur FNE, passerelle de modèles. Codex prend les tickets **CX** (écrans, CRUD, gabarits, imports, tests).

## Règles cardinales (CDC fonctionnel § 10) — à respecter dans tout code et toute revue
1. **RG-1** Ne jamais bloquer une vente (DGI, réseau, stock insuffisant, paramétrage incomplet).
2. **RG-2** L'IA propose, l'humain valide : aucune écriture d'argent, de stock ou de facture sans confirmation explicite ; aucune certification d'un montant non confirmé.
3. **RG-3** Aucun message vers un tiers sans « oui » du commerçant (hors réponses clients autorisées dans la fenêtre de 24 h).
4. **RG-4** Ne jamais détenir, transporter ni convertir de fonds.
5. **RG-5** Le carnet de dettes n'est jamais derrière un paiement.
6. **RG-6** Aucun paramétrage préalable à la première vente.
7. **RG-7** L'agent ne sort pas du commerce ; il n'improvise jamais sur l'argent.
8. **RG-8** Pas de crédit, d'avance ni de scoring.
9. **RG-9** Ne jamais couper l'accès aux données à l'échéance d'un pass.
10. **RG-10** Une facture certifiée ne se modifie pas, une écriture validée ne s'efface pas : on contre-passe.
11. **RG-11** WhatsApp uniquement par l'API officielle de Meta.
12. **RG-12** Ne jamais deviner un chiffre : toute lecture incertaine (vocal, photo) est redemandée.

Si une règle cardinale semble gêner l'implémentation : s'arrêter et poser la question dans la PR.

## Git (rappel, détail dans `AGENTS.md`)
- Branche `claude/t-<num>-<slug>` créée depuis `dev` à jour ; PR vers `dev` uniquement.
- Jamais de push direct ni de force-push sur `main`, `recette`, `dev` ; jamais de fusion de ta propre PR.
- Commits en Conventional Commits, en français.
- Ne commence pas un ticket dont le module est déjà pris par une branche `codex/…` ouverte.

## Avant chaque ticket
1. Lis le ticket, ses dépendances et les EF citées dans le CDC fonctionnel.
2. Lis les sections du CDC technique concernées.
3. Vérifie que les dépendances sont fusionnées dans `dev` et qu'aucune branche `codex/…` ouverte ne touche le même module.
4. Propose un plan court dans la PR avant d'écrire beaucoup de code si le ticket est marqué *socle*.

## Revue des PR de Codex
Vérifie dans l'ordre : règles non négociables d'`AGENTS.md`, RLS et `merchant_id`, absence d'écriture hors confirmation, montants en `bigint`, tests de conversation, charte Bissap pour l'UI. Vérifie aussi le nom de branche (`codex/t-<num>-<slug>`), la cible (`dev`), la taille (une tâche, lisible en dix minutes) et la liste des EF couvertes. Laisse des commentaires précis ; ne réécris pas la PR à sa place et ne la fusionne pas.

## Le Conseil des experts
- `/conseil <porte> <objet>` lance les 10 sous-agents `conseil-*` en parallèle et écrit le rapport dans `docs/conseil/`.
- Lance-le toi-même avant de commencer un lot (G0) et quand ta PR ajoute une ADR ou une table (G1).
- Pour une PR sensible, une revue ciblée suffit : « Utilise les sous-agents conseil-staff-engineer, conseil-expert-erp et conseil-securite pour relire cette PR ».
- Tu appliques les décisions des fondateurs, pas celles du Conseil directement.

## À ne jamais faire
- Fusionner une PR, quelle que soit la cible (réservé aux fondateurs).
- Coder en dur un nom de modèle, une URL DGI de production ou une clé.
- Contourner la confirmation pour « simplifier un test ».
