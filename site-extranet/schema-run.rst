========================
Schéma de fonctionnement
========================

Nous avons donc une architecture mvc

- Le modèle pour les données
- Le controleur pour les entrées
- Et la vue pour la sortie

Les modèles se réfèrent à des entités symfony et sont situés dans DataAcces
Il sont nommés de la façon suivante : da_([A-Za-z_-]+) 

$1 = Nom de la table

Les controleurs sont tous rattachés à une vue 

Le controleur et la vue portent le même nom mais ne portent pas la même extension.

Les vues sont en aspx et les controleurs en cs (C#)

Adhérent
--------

