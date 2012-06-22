Schema de fonctionnement
========================

.. code-block:: php

  But_phase1_Click() // l’ors du click sur le bouton phase 1 : Nouveau client 
  {
    Preparer_Analyse_Phase1()
  	affecter_doublon_automatique()
  	cm.Position = 0
  	grp_double.BringToFront()
  }

affecter_double_automatique()
-----------------------------

Si pas de doublon potentiel : on crée une nouvelle fiche

Le systeme crée le client avec al methode creer_client()

Il insere le client dans la table client avec

codeclient, origine, type, datecreationfiche, opcrea, nom, prenom, nomsociete, adresse1, adresse2, adresse3, codepostal, ville, adresse0, societe, pays, coderustica, club, titre

On verfie qu’il y a bien une ligne affecter par la requete d’insertion pas 2,0,-1 mais bien 1

Si oui on crée un complément d’asso valide en inserent dans la table cmpasso les donée suivantes :

codeclient = code client généré précédament l’ors de l’insertion du client 

::

	iscl = 0
	isprof = 0 
	isadm = 0
	isadh =  0
	isclubiste = 0
	iscd = 0
	iscad = 0
	isabo = 0


La méthode NouveauCodeClient :
------------------------------

L’information est obtenu dans la table parametre

On utilise la procedure stockée dans jdf_nouveauCodeClient


Si il y a effectivement doublon dans la base jdf alors on 

Affecte a code client au donée du client cotée magelan champ **code_p**  avec **code_r**

Peut etre
---------

Si le doublon est en prospect donc est inscrit dans la base mais n'est aps encore client 
Et qu'il n'a donc pas de code client alors on l'ignore 

Sinon On affecte a la ligne magellan le code client du doublon 
Le doublon est approuvée et une deuxieme analyse sera néscésaire pour synchroniser cette fiche

Lexique : 
=========

.. note::

	ncc = numéro code client