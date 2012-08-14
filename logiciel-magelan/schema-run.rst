========================
Schéma de fonctionnement
========================

Pratiquement la plupart des dispatch de la table ``magelan`` se font dans la table cmpAsso

On rappelle que la table cmpAsso n'est pas une table itérative

.. code-block:: php

  But_phase1_Click() // l’ors du click sur le bouton phase 1 : Nouveau client 
  {
    Preparer_Analyse_Phase1()
  	affecter_doublon_automatique()
  	cm.Position = 0
  	grp_double.BringToFront()
  }

La fonction affecter_double_automatique
---------------------------------------

Le client analysé est celui qui est actuellement affiché, il se base sur son numéro de compteur.

S'il n'y a pas de doublon potentiel alors on crée une nouvelle fiche

Le systeme crée le client via la methode ``creer_client()``

Il insère le client dans la table client avec pour champs

codeclient,
origine,
type,
datecreationfiche,
opcrea,
nom,
prenom,
nomsociete,
adresse1,
adresse2,
adresse3,
codepostal,
ville,
adresse0,
societe,
pays,
coderustica,
club,
titre

On vérifie qu’il y a bien une ligne affectée par la requête d’insertion ni plus ni moins.

Si oui on crée un complément d’association valide en insérant dans la table ``cmpasso`` les données suivantes :

codeclient = code client généré précédament lors de l’insertion du client 

::

	iscl = 0
	isprof = 0 
	isadm = 0
	isadh =  0
	isclubiste = 0
	iscd = 0
	iscad = 0
	isabo = 0


La méthode NouveauCodeClient
----------------------------

L’information est obtenue dans la table ``paramêtre``

On utilise la procédure stockée dans ``jdf_nouveauCodeClient``


Si il y a effectivement doublon dans la base jdf alors on 

affecte un code client aux données du client coté magelan champ **code_p**  avec **code_r**

Peut être
---------

Si le doublon est en mode prospect, il est inscrit dans la base mais n'est pas encore client 
.Il n'a donc pas de code client alors on l'ignore 

Sinon On affecte à la ligne magellan le code client du doublon 
Le doublon est alors approuvé et une deuxieme analyse sera néscésaire pour synchroniser cette fiche

L'affectation d'un code client sur une fiche magelan ce fait sur le champ dont le compteur 
affiché sur la vue est égal à son propre compteur.


Lexique
-------

	Me.DataGrid1.DataSource
	DataView1.table = Me.ds_in.Analyse 
	
	
	Me.SqlSelectCommand => daAnalyse(.SelectCommand) =>[Fill] ds_in.Analyse(In Database) => DataView1(Array Contenue) => DataGrid1 (View)
	
	.. image:: images/ds_in_Analyse.png 

.. note::

	- ncc = numéro code client
	
	