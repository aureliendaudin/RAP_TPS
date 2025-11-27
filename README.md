# Rapport


Ce TP met en place un système de reconnaissance de parole de bout en bout sur le corpus LibriSpeech et LJSpeech en utilisant des caractéristiques MFCC comme entrée et différentes architectures entraînées avec une loss CTC.

## MLP

### Pipeline

Les signaux audio du corpus LibriSpeech sont rééchantillonnés à 16 kHz, convertis en MFCC, normalisés et regroupés en batches via une fonction de collate qui gère le padding temporel et l’empilement des séquences. Les transcriptions sont normalisées en minuscules, restreintes à un vocabulaire simple (espace, apostrophe, lettres a–z) puis encodées en entiers pour servir de cibles au critère CTC. Nous avons aussi ajouté l'encodage pour le CTC-blank, un caractère spécial pour permettre la prédiction de deux lettres consécutives comme dans le mot "hello".

Résumé de la pipeline :
- Convertion du signal en MFCC.
- Encodage des target en convertissant les char en int.
- Entrainement du modèle avec la loss CTC
- Prédiction du modèle puis utilisation du gready CTC pour afficher la prédiction et la comparer à la target.

### Analyse des résultats

Exemples de prédictions pour le MLP après 50 epochs :
  - tgt: i emphasised complacently
  - prd: a  osos t le tt

Le manque de contexte empêche le MLP de comprendre le liens entre les lettres des alentours et donc le concept de mot.

## CNN

Solution utilisation du CNN pour ajouter une notion de window et donc de contexte et de dépendance temporelle dans l'audio. De plus, le MFCC sont une version compressée via une transformée en cosinus alors que MelSpectrogram représente directement l’énergie du signal sur l’échelle de Mel et est donc plus adapté aux CNN, car il préserve davantage la structure spatiale du spectre que les MFCC, qui l’aplatissent et réduisent l’information.

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

## GRU

En utilisant la même pipeline, on a implémenté en premier, un modèle GRU unidirectionnel qui a été testé pour prédire les séquences de caractères à partir des séquences MFCC, mais les performances restaient limitées.

On l'a remplacé par une architecture GRU bidirectionnelle, qui traite chaque frame en tenant compte à la fois des informations passées et futures dans la séquence audio. Cette modification améliore la qualité des transcriptions, tout en restant consciente de sa limite pratique : en conditions réelles, il n’est pas possible de récupérer des « futurs » échantillons audio au moment de la prédiction.

## Entraînement et évaluation

Le modèle bidirectionnel BidirGRU_CTC est composé de plusieurs couches GRU empilées avec dropout, d’une couche linéaire projetant vers le nombre de classes, puis entraîné avec l’optimiseur Adam et la loss CTC standard. Les performances sont suivies à l’aide d’un Character Error Rate simple, ce qui permet de quantifier la réduction des erreurs de transcription apportée par le passage du GRU unidirectionnel au GRU bidirectionnel.

Exemples de séquences générées après 50 epochs:

- tgt: he buried his biscuit under a layer of jam over which he spread a thick coating of honey
- prd: he beried his bis kit underaly ov jan over woich i srit y thik comneg o hany

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

## Résultats d’apprentissage

Sur 30 époques, la loss d’entraînement diminue d’environ 1,3 à 0,43 tandis que la loss de validation passe d’environ 1,3 à 0,51, ce qui montre une amélioration continue sans surapprentissage.

Dans les premières époques, les séquences générées sont incorrectes ou vides:
- target: sattler was tried for murder and convicted#
- prediction: the an the ore the the the the ore ore the the the the the the the the of the an the the ore the there.

Mais elles se structurent progressivement en sorties plus cohérentes:

- target:     sattler was tried for murder and convicted#
- prediction: satler was tried for murder, and convicted.

# Conclusion

Ce projet montre bien les forces et faiblesses des différentes briques testées pour la reconnaissance de parole de bout en bout. Le MLP, sans mémoire ni contexte, reste bloqué à des sorties très bruitées au niveau caractère, quand le CNN exploite mieux la structure temps–fréquence mais peine encore sur les espaces et la segmentation des mots. Le GRU bidirectionnel tire pleinement parti du contexte passé et futur pour stabiliser les transcriptions, au prix d’une moindre applicabilité en temps réel, tandis que le Transformer encoder–decoder capte le mieux les dépendances longues et produit les phrases les plus cohérentes, au prix d’une complexité et de besoins en données plus élevés. 

Au vu des résultats, le compromis le plus adapté dans ce TP est le GRU bidirectionnel : il améliore nettement la qualité des transcriptions par rapport au MLP et au CNN, tout en restant plus simple à entraîner et moins exigeant qu’un Transformer complet pour de la reconnaissance de parole caractère par caractère. Si l'exigeance d'entrainement et du nombre de données n'est pas un problème alors le Transformer est celui qui arrivera aux meilleurs résultats au prix du coup d'entrainement.