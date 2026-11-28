# Groupe Valmont : du revenu à la règle

Projet de fin de module Datamining (B3, ECE Paris), sujet 9.

**Équipe** : Kadidiatou Ibrahima Bagayoko, Aristote Izamo, Anaelle Tuebou Kamga, Elvis Thym, Wenceslas Bouity

## 1. Contexte

Le Groupe Valmont exploite 32 hôtels de 3 à 5 étoiles en France. La direction veut remplacer l'intuition de son revenue manager par des règles chiffrées : tarification, surréservation, conditions par type de séjour.

## 2. Données et décisions de nettoyage

Les deux fichiers fournis ont été audités dans `notebooks/00_analyse_exploratoire.ipynb`. Le fichier propre commun est `data/clean/valmont_clean.csv` : 800 réservations, 23 colonnes, aucune valeur manquante.

| Constat | Volume | Décision |
| --- | --- | --- |
| Lignes strictement identiques | 140 | Supprimées : le fichier passe de 940 à 800 réservations |
| Nombre de personnes manquant | 20 | Imputé par la médiane du segment déclaré (1 en affaires, 2 en loisir, 11 en groupe), avec une colonne indicatrice |
| Petit-déjeuner manquant | 20 | Modalité « inconnu », avec une colonne indicatrice |
| Séjours de 365 nuits | 3 | Saisie impossible : marqués, exclus des modèles, étudiés en question 9 |
| Prix de la nuit à 0 € | 3 | Tarif non commercial : marqués, exclus de la grille tarifaire, étudiés en question 9 |
| Délai de réservation de 0 jour | 4 | Réservations le jour même, plausibles : conservées |
| Nuitées cumulées inférieures aux nuits du séjour | 78 | 10 % des lignes, trop pour une erreur : la colonne décrit l'historique du client. Conservée |

**Unités vérifiées** : le prix ne varie ni avec le nombre de nuits ni avec la taille des groupes (corrélations de Spearman de −0,02). C'est donc un prix par nuit et par chambre. Les délais ont le même ordre de grandeur sur les quatre canaux (en jours) et les distances à la gare vont de 0,5 à 9,8 km. Aucune conversion n'a été nécessaire.

## 3. Réponses aux neuf questions

Pour chaque question : compréhension, méthode retenue, pourquoi celle-là, vérifications effectuées, résultat chiffré, décision et gain.

### Q1. Ce qui distingue une réservation annulée d'une réservation honorée
### Q2. Réserver tôt, est-ce un client sûr ?
### Q3. Un prix de nuitée défendable
### Q4. Les périodes où la grille se trompe
### Q5. Prédire l'annulation à la réservation

**Méthode.** Quatre modèles du cours (régression logistique, kNN, arbre de décision, forêt aléatoire) comparés en validation croisée stratifiée à 5 plis, mesurés par l'AUC et le score de Brier (la justesse est trompeuse : 74 % sans rien apprendre). Réglage du kNN et de l'arbre par validation croisée, puis test de parcimonie.

**Résultat.** Le délai de réservation seul prédit le mieux (AUC de 0,652, contre 0,609 à 0,633 pour les modèles à 16 variables). Chaque mois d'avance multiplie la cote d'annulation par 1,5. Probabilités bien calibrées hors échantillon (26,1 % prédit contre 26,1 % observé).

**Décision.** Un risque de 30 % correspond à 61 jours : au-delà de 60 jours avant l'arrivée, tarif avec acompte. Ce seuil rejoint ceux des questions 2 et 8.

**Gain.** Environ 14 300 € conservés sur l'échantillon, soit près de 1 200 € par mois, avec un acompte de 30 % sur les réservations alertées.

Détail : `notebooks/Q5.ipynb`. Probabilités exportées : `data/clean/proba_annulation.csv`.

### Q6. La règle de surréservation par hôtel

**Méthode.** Taux d'annulation de chaque hôtel issu du modèle de la question 5, puis règle par la loi binomiale : on retient le plus grand nombre de chambres vendues en plus tel que le risque de refuser au moins un client reste sous 20 % (un refus coûte 4 fois le prix de la nuit). Hypothèse prudente : 25 % des annulations sont tardives.

**Vérification clé.** Les écarts de taux bruts entre hôtels (de 7 % à 50 %) sont exactement ceux du hasard (écart-type de 0,087 observé contre 0,088 attendu) : surréserver sur les taux bruts reviendrait à parier sur du bruit.

**Résultat.** 317 chambres surréservables sur 6 003 (5,3 % de la capacité), de 2 à 22 chambres selon l'hôtel. La part d'annulations tardives pèse bien plus que le coût d'un refus : de 25 % à 100 % d'annulations tardives, la surréservation possible passe de 5 % à 31 %.

**Décision et gain.** Un plafond par hôtel (`data/clean/regle_surreservation.csv`). Sur une nuit complète, 286 des 389 chambres laissées vides sont récupérées, pour environ 10 clients refusés sur le groupe : gain net de 46 200 €. Première amélioration : enregistrer la date de chaque annulation.

Détail : `notebooks/Q6.ipynb`.

### Q7. Les vrais types de séjours
### Q8. Une politique par type de séjour
### Q9. Les réservations à bloquer

## 4. Synthèse : la nouvelle politique commerciale

À compléter.

## 5. Structure du dépôt et reproduction

- `data/raw/` : fichiers d'origine, jamais modifiés
- `data/clean/` : fichier nettoyé commun
- `notebooks/` : un notebook par bloc de questions
- `slides/` : support de soutenance en PDF

Installation : `pip install -r requirements.txt`