# Spec — Dashboard des ventes

| | |
|---|---|
| Source | [Dashboard des ventes - lancement](https://microsofteur-my.sharepoint.com/personal/asmaeyadaoui_microsoft_com/Documents/Recordings/Dashboard%20des%20ventes%20-%20lancement-20261123_223224-Meeting%20Recording.mp4?web=1), enregistrement du 23 novembre 2026 ; Asmae Yadaoui joue les rôles de Claire Dubois, Karim Benali, Julie Peters et Maxime Lambert. |
| Statut | Validée par l'utilisatrice ; questions ouvertes à trancher |
| Validée par | L'utilisatrice |

## Contexte
Les données de réquisitions fictives 2024–2025 sont consultables à l'état brut dans le registre. L'objectif est de donner aux responsables une vue des ventes dans l'espace admin / registre de l'application existante, sans modifier la boutique. La priorité exprimée est l'exactitude des chiffres, puis les visuels et les filtres.

## 1. Ce que ça fait
- Dans le registre, afficher le montant des ventes, le nombre de commandes et le panier moyen, pour la période et les filtres sélectionnés. Calculer les ventes à partir de `quantity × unitPriceCents` sur les lignes historiques, remises incluses, jamais à partir du prix actuel du catalogue ; panier moyen = total / nombre de commandes. Les données représentent des commandes fictives et non des paiements réels : le libellé doit le préciser.
- Présenter les ventes par point de vente comparées au portail en ligne, les 10 produits principaux et l'évolution mensuelle. Permettre de filtrer par dates choisies, canal et point de vente ; tous les chiffres et visuels reflètent les mêmes filtres.
- Identifier le canal via `orders.locationId → locations.channel` : `online` pour le portail et `store` pour les magasins (`src/types/sales.ts`). La sélection de deux périodes successives permet leur comparaison ; aucune comparaison simultanée n'a été décidée.
- Afficher les montants en USD à partir de cents entiers ; les dates de commande sont des dates UTC (`src/data/sales.json`, `src/App.vue`). Gérer explicitement l'absence de résultats et les erreurs, sans présenter un chiffre inventé.

## 2. Comment on saura que ça marche
| Contrôle | Comment | Résultat attendu |
|---|---|---|
| Réconciliation | Recalcul indépendant sur les lignes de `src/data/sales.json`, du 1 au 31 mars 2025 inclus, avant ajout de commandes locales | 55 commandes, 646 092 cents, soit 6 460,92 USD ; écart de 0,00 USD. Ce total est une référence sur le jeu initial, pas sur un stockage local modifié. |
| Prix historique et filtres | Jeu de test avec prix catalogue différent, ligne remisée, deux canaux et deux dates ; calculer le résultat à la main | Les KPI et visuels utilisent le prix payé enregistré et excluent les commandes hors filtres ; le portail n'est pas compté comme magasin. |
| Interface et cas limites | Test d'interface des filtres, d'une période vide et de l'accès au registre | Chiffres et visuels cohérents ; état vide explicite, sans division par zéro ; erreur visible si les données sont invalides. |
| Régression | Tests des parcours boutique, panier, commande et registre existants | Les parcours fonctionnent encore et les données de ventes enregistrées ne sont pas altérées par la consultation du dashboard. |

## 3. Ce que ça ne doit pas casser
- Boutique, recherche, panier, commande simulée, stockage local, registre brut et leurs erreurs existantes (`src/App.vue`, `src/services/salesStorage.ts`, `src/components/WorkshopDataView.vue`).
- Données historiques et clés de stockage ; aucun nouveau serveur ni outil externe. L'interface actuelle est locale au navigateur : aucun contrôle d'accès administrateur n'est identifié dans le code examiné, ne pas prétendre que le registre est protégé.

## Hors périmètre
| Demande | Raison |
|---|---|
| Marge | Aucun coût d'achat dans les données. |
| Export Excel ou CSV | Explicitement reporté hors V1 lors du lancement. |
| Paiements réels, taxes, expédition et prévisions | Absents des données / non décidés au lancement. |

## Hypothèses et limites
- La source de référence est le jeu fictif local, enrichi des commandes simulées conservées dans le navigateur ; aucun paiement réel n'est enregistré (`README.md`, `src/data/sales.json`). Les montants ne prouvent donc pas des encaissements réels.
- Le point de vente et le canal proviennent de la localisation de la commande, pas des préférences du client. L'agrégation et la conversion pour l'affichage doivent conserver la précision exacte des cents et signaler un dépassement (`src/services/money.ts`).
- Le classement des produits et l'arrondi du panier moyen nécessitent une validation avant implémentation ; ne pas inventer une règle métier.

## Questions ouvertes (3 maximum)
1. **Métier :** le top 10 est-il classé par chiffre d'affaires ou par quantité vendue ? Quel départage en cas d'égalité ?
2. **Métier / contrôle de gestion :** quelle règle d'arrondi appliquer au panier moyen lorsqu'il comporte une fraction de cent ?
3. **Responsable produit :** le registre local sans authentification suffit-il pour cet « espace admin », ou une protection d'accès est-elle requise ? En cas de protection requise, définir son périmètre avant développement.

## Tâches proposées — après validation humaine
Les tâches sont séquentielles ; chacune vise une PR et les fichiers partagés ne sont pas modifiés en parallèle. Aucune session ni PR n'est lancée.

### 1 — Calculs de ventes fiables
**Prompt pour l'agent :** Lis la spec `docs/specs/2026-11-24-dashboard-ventes.md`. Implémente uniquement les calculs des KPI et des données agrégées filtrées dans un service sous `src/services/`, avec tests associés, à partir de `src/types/sales.ts` et `src/services/money.ts`. Ne modifie ni stockage ni boutique. Respecte les décisions de validation sur les questions ouvertes ; ne devine pas les règles manquantes.
**Dépendance :** validation de la spec et réponses aux questions 1 et 2.
**Test qui doit échouer avant :** calculer la référence indépendante de mars 2025 et un jeu remisé multi-canal filtré.
**Critère :** 55 commandes et 646 092 cents sur le jeu initial ; filtres et regroupements exacts, sans perte de précision.

### 2 — Dashboard dans le registre
**Prompt pour l'agent :** Lis la spec `docs/specs/2026-11-24-dashboard-ventes.md`. Après la tâche 1, construis la vue du dashboard dans `src/components/` et intègre-la à `src/App.vue` / `src/components/WorkshopDataView.vue`, en réutilisant le service de calcul. Préserve les parcours boutique et registre et les états d'erreur ; ajoute les tests d'interface et de régression nécessaires, y compris la consultation après une commande simulée. Ne modifie pas les calculs de la tâche 1 sauf correction explicitement nécessaire.
**Dépendance :** tâche 1 et réponse à la question 3.
**Test qui doit échouer avant :** accéder au dashboard, choisir dates, canal et point de vente, puis vérifier KPI et visuels sur des données connues et une sélection vide.
**Critère :** tous les contrôles d'interface de la section 2 passent ; les commandes locales sont reflétées par les filtres et les parcours existants restent utilisables.
