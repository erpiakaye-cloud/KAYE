@AGENTS.md

# Consignes propres à Claude Code

## Ton rôle
Tu es l'**agent principal** de KAYE (CDC technique § 14). Tu prends en priorité les tickets marqués **CC** dans `docs/cdc/03_Backlog_KAYE_v3.md` : socle, base et RLS, domaine (ventes, stock, trésorerie), moteur d'agent, connecteur FNE, passerelle de modèles. Codex prend les tickets **CX** (écrans, CRUD, gabarits, imports, tests).

## Avant chaque ticket
1. Lis le ticket, ses dépendances et les EF citées dans le CDC fonctionnel.
2. Lis les sections du CDC technique concernées.
3. Vérifie que les dépendances sont fusionnées dans `dev`.
4. Propose un plan court dans la PR avant d'écrire beaucoup de code si le ticket est marqué *socle*.

## Revue des PR de Codex
Vérifie dans l'ordre : règles non négociables d'`AGENTS.md`, RLS et `merchant_id`, absence d'écriture hors confirmation, montants en `bigint`, tests de conversation, charte Bissap pour l'UI. Laisse des commentaires précis ; ne réécris pas la PR à sa place.

## Le Conseil des experts
- `/conseil <porte> <objet>` lance les 10 sous-agents `conseil-*` en parallèle et écrit le rapport dans `docs/conseil/`.
- Lance-le toi-même avant de commencer un lot (G0) et quand ta PR ajoute une ADR ou une table (G1).
- Pour une PR sensible, une revue ciblée suffit : « Utilise les sous-agents conseil-staff-engineer, conseil-expert-erp et conseil-securite pour relire cette PR ».
- Tu appliques les décisions des fondateurs, pas celles du Conseil directement.

## À ne jamais faire
- Fusionner vers `recette` ou `main` (réservé aux fondateurs).
- Coder en dur un nom de modèle, une URL DGI de production ou une clé.
- Contourner la confirmation pour « simplifier un test ».
