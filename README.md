# P35_Traitement_Caryotypes

## Introduction

### Enjeux de la reconnaissance des chromosomes

A l’heure actuelle, les cancers du sang, hémopathies malignes, représentent 30% des cancers de l’enfant et 5% des cancers chez l’adulte. Afin de pouvoir classifier ces cancers, il est nécessaire de prendre en compte le tissu d’origine, la morphologie, les marqueurs protéiques, ainsi que les données chromosomiques de ces cancers. Les cancers du sang sont diagnostiqués grâce au caryotype, car ils sont dus à une translocation de gènes, dans laquelle les chromosomes 9 et 22 ont échangé réciproquement des fragments de chromosomes.

Pour pouvoir mettre en évidence ce type d’anomalie chromosomique, il est donc nécessaire de classer les chromosomes afin d’établir le caryotype, puis de l’analyser. Ce projet est composé de deux parties, la première se charge de la segmentation et de la classification (Part I), et la deuxième se consacre à la détection d’anomalies (Part II). Dans ce brief, on va traiter la première partie.

Un caryotype est réalisé à partir d’une photographie d’une cellule en vue microscopique lors de la métaphase de la mitose. A cette étape, la chromatine est condensée ce qui rend les chromosomes visibles. Dans un caryotype, les chromosomes sont classés par paire, par taille et en fonction de la position du centromère. Il y a 23 paires de chromosomes autosomes (de 1 à 22) et une paire de gonosomes (XX pour les femmes et XY pour les hommes).

Les logiciels de réalisation des caryotypes actuels sont semi-automatiques et nécessitent encore l’intervention des techniciens de laboratoire qui savent reconnaître les chromosomes.<br>
**L'objectif de ce projet est d'automatiser entièrement ce processus**. Pour ce faire, il est nécessaire de procéder en deux étapes. Il s'agit d'abord d'isoler chacun des chromosomes présents dans les vues microscopiques des cellules en phase de mitose (que nous appelerons "mélange"), puis de les classifier, afin d'établir le caryotype final.

### Données disponibles

203 couples d'images mélange/caryotype on été mise à disposition apr le CHRU pour la réalisation de ce projet

![image](https://user-images.githubusercontent.com/73179354/131327930-50a930f3-ec55-4973-acde-50c0ba6355bc.png)


## Classification des images des chromosomes

Le travail de classification se fait en utilisant uniquement les caryotypes.

En premier lieu, un travail est réalisé pour extraire des images de chromosomes de chacun des 203 caryotypes. On obtient donc environ 23x203=4669 images de chromosomes (plus ou moins, en fonction des anomalies qui ont pu êtres rencontrées : trisomies, monosomies, ...).<br>
Les images obtenues, extraites et enregistrées accompagnées de leur label (leur n° chromosomique), permettront d'entrainer un modèle de classification. <br>Ce images sont similaires à celles visibles ci-dessous :

![separated](images/separated.png)

Le code pour obtenir les images et leur label est décrit dnas le notebook [chomosomes_extraction.ipynb](chomosomes_extraction.ipynb)

Grâce à ces images on peut ensuite entrainer un modèle de classification efficace. 

Une data-augmentation a été réalisée pour donner une rotation aléatoire à chaque image, 
les chromosomes détectés dans les mélanges n'étant pas dirigés tous dans le même sens.<br>
Un modèle séquentiel de CNN convolutif est ensuite entrainé sur les images. 

Les résultats en sortie du modèle sont plutôt bons, avec une accuracy finale de 90% sur la base de test.

![cmat](images/confmat.png)


## Segmentation des images des chromosomes

Différentes approches ont été testées pour la segmentation des vues microscopiques des cellules en phase de mitose ("mélanges"). 

Le défi dans cette segmentation est de séparer les chromosomes qui pésentent un chevauchement. Pour les chromosomes isolés, une segmentation avec des outils simples comme les méthodes déjà implémentées dnas scikit-image conviennent (voir [Label Image Regions](https://scikit-image.org/docs/dev/auto_examples/segmentation/plot_label.html#sphx-glr-auto-examples-segmentation-plot-label-py)).


Nous avons d'abord tenté une segmentation sémantique multiclasse en créant nos propres chevauchements de chromosomes avec leurs masques de segmentation. Malheureusement, ces mélanges créés de toutes pièces ne ressemblaient aps assez aux mélanges réels, et les résultats sur les mélanges réels n'étaient pas bons.

Nous avons ensuite essayé une segmentation par instance avec pytorch et détectron, en labelisant manuellement les mélanges réels existants. Les résultats, légèrements meilleurs que les précédents, n'étaient pas non plus convainquants.

Finalement, la solution résidait dans l'article scientifique de Cao X et Al, _ChromSeg: Two-Stage Framework for Overlapping
Chromosome Segmentation and Reconstruction_ .
Cette équipe de recherche est parvenue à segmenter les chevauchements de chromosomes en utilisant un réseau UNet++, qui extrait séparément les croisements et chacun des chromosomes.


