========================
Schéma de fonctionnement
========================

Nous avons donc une architecture mvc

- Le modèle pour les données
- Le controlleur pour les entrées
- Et la vue pour la sortie

Les modèles se référent à des entités symfony et sont situés dans DataAcces
Il sont nommés de la facon suivante : da_([A-Za-z_-]+) 

$1 = Nom de la table

Les controlleurs sont tous rattaché à une vue 

Le controlleur et la vue porte le même nom mais ne porte pas la même extension.

Les vues sont en aspx et les controlleur en cs (C#)

Adhérent
--------

