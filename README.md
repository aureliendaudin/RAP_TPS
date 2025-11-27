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

## Transformer

## Modèle et architecture

On a implémenté un modèle de reconnaissance de parole caractère par caractère basé sur une architecture Transformer encoder–decoder appliquée au corpus LJSpeech.
- Encodeur qui traite des spectrogrammes audio normalisés
- Décodeur qui est un Transformer avec attention sur la sortie de l’encodeur et un classifieur final linéaire sur un vocabulaire de 34 symboles pour prédire la séquence de caractères

## Préparation des données

Les fichiers audio du corpus LJSpeech sont téléchargés, associés à leurs transcriptions et filtrés selon une longueur maximale de texte.
Chaque signal est:
- converti en spectrogramme
- mis à l’échelle
- normalisé par frame
- paddé à une durée fixe pour obtenir des tenseurs de taille constante.
Les transcriptions sont vectorisées caractère par caractère avec ajout de tokens de début et fin de séquence.

## Résultats d’apprentissage

Sur 30 époques, la loss d’entraînement diminue d’environ 1,3 à 0,43 tandis que la loss de validation passe d’environ 1,3 à 0,51, ce qui montre une amélioration continue sans surapprentissage.
Dans les premières époques, les séquences générées sont incorrectes ou vides:
target:     <sattler was tried for murder and convicted#>
prediction: <the an the ore the the the the ore ore the the the the the the the the of the an the the ore the there.>
Mais elles se structurent progressivement en sorties plus cohérentes à mesure que la perte baisse:
target:     <sattler was tried for murder and convicted#>
prediction: <satler was tried for murder, and convicted.>
