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
### Q6. La règle de surréservation par hôtel
### Q7. Les vrais types de séjours

*Wenceslas Bouity — `notebooks/Q7.ipynb`, branche `q7-types-sejours`*

**Compréhension.** Les trois segments déclarés à la réservation (affaires, loisir, groupe) décrivent-ils vraiment la façon dont les clients réservent, et combien de types faut-il distinguer pour piloter une politique commerciale ?

**Méthode.** Classification non supervisée sur les trois variables continues qui décrivent le séjour au moment où il est réservé : nombre de nuits, nombre de personnes, délai de réservation, centrées et réduites pour peser le même poids. Trois exclusions, chacune motivée : le segment déclaré, parce que c'est l'étiquette mise en doute ; l'annulation, parce que c'est un résultat, gardé pour la validation ; le prix de la nuit, parce que 89 % de sa variance vient de la catégorie de l'hôtel (eta² = 0,887) — il décrit l'hôtel, pas le séjour. K-means est comparé à une classification hiérarchique de Ward ; il est retenu pour sa meilleure silhouette (0,437 contre 0,338) et des classes plus équilibrées.

**Choix de k.** Quatre critères construits différemment désignent tous k = 4 : coude de l'inertie (baisse de 300 en passant à 4 groupes, 112 seulement pour le cinquième), silhouette maximale (0,437), Calinski-Harabasz maximal (551), Davies-Bouldin minimal (0,881). Aucun ne désigne 3.

**Vérifications.** Stabilité : sur 50 sous-échantillons de 80 %, la partition est reconstruite à ARI 0,972, et c'est le seul k dont le pire tirage reste au-dessus de 0,93. Les trois variables séparent toutes les groupes (Kruskal-Wallis, p < 10⁻⁸⁰) et aucune silhouette de groupe n'est sous 0,27. Les variables écartées ne cachaient pas de typologie : prix p = 0,30, week-end p = 0,09, petit-déjeuner p = 0,19, canal p = 0,30.

**Résultat.** **Valmont a quatre types de séjours, pas trois.**

| Type | Réservations | Personnes | Nuits | Délai médian | Annulation |
| --- | ---: | ---: | ---: | ---: | ---: |
| Court séjour de dernière minute | 363 (46 %) | 1,3 | 2,1 | 13 j | **18,7 %** |
| Long séjour individuel | 171 (22 %) | 1,8 | 4,9 | 25 j | 22,8 % |
| Groupe et séminaire | 134 (17 %) | 11,5 | 2,3 | 55 j | 33,6 % |
| Réservation très anticipée | 126 (16 %) | 2,7 | 3,3 | **96 j** | **43,7 %** |

Le quatrième type n'existe pas dans la typologie actuelle : il est dilué dans le segment « loisir », dont il porte pourtant le risque le plus élevé. L'accord entre les deux typologies n'est que de 0,37 (ARI) : 28 % des réservations du premier type sont déclarées « loisir » alors qu'elles se comportent comme des réservations d'affaires.

**Décision.** Remplacer le segment déclaré par ces quatre types comme clé de segmentation dans l'outil de réservation. Le type se calcule dès la réservation à partir de trois champs déjà saisis, sans collecte nouvelle.

**Gain.** La segmentation ne rapporte rien par elle-même, elle rend une décision possible : les quatre types étalent le risque d'annulation sur 25 points (18,7 % à 43,7 %) contre 17 points pour les trois segments déclarés, soit 47 % de pouvoir de séparation en plus sur la variable qui décide du revenu perdu. Le chiffrage est en Q8.

**Limites et axes d'amélioration.** La frontière entre court et long séjour individuel est une convention de découpage, que Ward place ailleurs ; elle ne touche pas les deux types à forte annulation. Un historique client plus riche permettrait de distinguer, parmi les réservations très anticipées, celles qui annulent de celles qui tiennent. Un modèle de mélange gaussien donnerait une probabilité d'appartenance, utile pour les cas frontières.

**Fichier produit.** `data/clean/q7_types_sejours.csv` — le type de chacune des 794 réservations analysées, pour que la Q8 et la synthèse repartent de la même affectation sans relancer K-means.

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