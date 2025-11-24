# Report

## Pipeline

Résumé de la pipeline :
- Encodage des target en convertissant les char en int.
- Entrainement du modèle avec la loss CTC
- Prédiction du modèle puis utilisation du gready CTC pour afficher la prédiction et al comparer à la target.

## Analyse des résultats

Exemples de prédictions pour le MLP après 50 epochs :
  - tgt: i emphasised complacently
  - prd: a  osos t le tt

Le manque de contexte empêche le MLP de comprendre le liens entre les lettres des alentours et donc le concept de mot.

Solution utilisation du CNN pour ajouter une notion de window et donc de contexte et de dépendance temporelle dans l'audio.

Exemples de prédictions pour le CNN après 50 epochs :

- tgt: all my worries about you were foolish
- prd: all my weresebotyo werfelah
- tgt: illustration rusks
- prd: ellustraton resks

La mauvaise prédiction d'espaces donne une fusion de plusieurs mots.

Pistes d'amélioration :
- Normalisation
- Bi-directionnal
- Utilisation de validation
- Optimisation des hyper-paramètres
- Rapprochement avec un dictionnaire de mots