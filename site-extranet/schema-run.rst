========================
Schéma de fonctionnement
========================

Nous avons donc une architecture mvc

- Le modèle pour les donée 
- Le controlleur pour les entrée
- Et la vue pour la sortie

Les modèles se référent à des entité symfony et sont situé dans DataAcces
Il sont nommé de la facon suivante : da_([A-Za-z_-]+) 

$1 = Nom de la table 

Les controlleur sont tous rattaché a une vue 

Le controlleur et la vue porte le meme nom mais ne porte pas la meme extension

Les vues sont en aspx et les controller en cs (C#)

Adhérent
--------

