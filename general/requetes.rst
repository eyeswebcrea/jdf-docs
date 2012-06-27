Les des requetes Requete :
=========

Lexique
=======

- ADH = Adhérent + abonnée 
- ADE = Adhérent seul 
- EXP = (Depreced) Expert appartenant à rustica 
- (?) = Ligne de requete dont l'explication n'est pas encore suffisant claire et validée 

Me.SqlSelectCommand1
--------------------

Resumée de la requête:
~~~~~~~~~~~~~~~~~~~~~~

La requete selectionne le nom, prenom , adresse, code postal, ville de la table Client (Jdf)

Des ligne dont un des champs suivant :

	- nom
	- prenom
	- raison sociale
	- nom societe
	

de la table megelan sont en duplicata dans la table client 

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::
	-- *********** Lexique ************
	-- jdf = jardinier de france
	-- m = Table magellan (Information de magelan)
	-- c = Table client (Information de jdf)
	-- CmpAsso = Compte associatif
	-- charindex = strpos (php)
	-- ADH = Adhérent 
	-- soundex = composition phonetique d'un mot resultat sous la forme d'un nombre à quatre chiffre
	-- difference = calcule le soundnex de deux chaine et renvoie 0 à 4 selon la similitude 0 pour pas 4 pour beaucoup
	-- SELECT -- Selectionne les champs de la table 'client' (JDF) et leur applique un alias
	-- dbo.fn_Dmot = Recupere le dernier mot d'une chaine
	-- les champs ctrl_* sont des valeur de controle de comformité des champs ce ne sont pas de vrai champ en bdd
	-- m.Code_P = c.CodeClient , c.CodeClient = CmpAsso.codeclient | Ces champs sont la représentation du code clients
	-- (?) =  Ligne de requete donc l'explication n'est pas encore suffisament claire
	-- ********************************
	
		c.Nom AS JDF_Nom,		-- Champ nom avec pour alias JDF_Nom
		c.Prenom AS JDF_Prenom, -- Champ prenom avec pour alias JDF_Prenom
		c.Adresse2 AS JDF_Adr2, -- Champ adresse 2 avec pour alias JDF_Adr2
		c.Adresse3 AS JDF_adr3, -- Champ adresse 3 avec pour alias JDF_Adr3
		c.CodePostal AS JDF_cp, -- Champ code postal Avec pour alias JDF_cp
		c.Ville AS JDF_Ville,	-- Champ ville avec pour alias JDF_Ville
			CASE WHEN raisoc IS NULL THEN                    									-- 			Si raisoc (Raison sociale) est nul alors ...
				CASE WHEN difference(m.nom, c.nom) > 2       									-- 					Si le nom est ressemblant phonétiquement entre magelan et jdf  ...
					  AND 							 		 									-- 					ET
				      (									     									-- 					(   
				     	(m.prenom IS NULL OR m.prenom = '')  									-- 						(Si le champ nom de magelan est null ou vide) 
				     	AND 							 	 									-- 						ET
				      	(c.prenom IS NULL OR c.prenom = '')  									-- 						(Si Le champ prenom de magelan est nul ou vide)
				      ) 									 									-- 					)
				      OR 									 									-- 					OU
				      difference(m.prenom, c.prenom) > 2 THEN   								-- 					Si le champ prenom est ressemblant phonétiquement entre magelan et jdf
			    'O' ELSE 'N'																	-- 						Alors O sinon N 
			    END																				-- 					Fin 		
				ELSE CASE WHEN difference(m.raisoc, ltrim(c.adresse0)) > 2  					--      			Sinon si 
					AND																			-- 					Et 
					(																			--					(
						(charindex(m.nom, c.nomsociete) > 0 OR difference(m.nom, c.nom) > 2) 	-- 						Si le nom (magelan) est present dans le nom de la societe (jdf) ou si le nom est ressemblant phonétiquement entre magelan et jdf
						OR 																		-- 						Sinon 
						m.nom IS NULL 															--						Si le nom (magelan) est null
					) 																			-- 					)
					THEN 'O' ELSE 'N' 															--						Alors O Sinon N
				END 																			--					Fin					
			END AS ctrl_nom,																	--					On stocke la reponse du nom ctr_nom
			CASE WHEN c.codepostal = m.cp THEN 													--			Si Le code postal (jdf) est égale au code postal (magellan) Alors
			'O' ELSE 'N' 																		-- 			Alors O Sinon N
			END AS ctrl_CP,																		--			On stocke le controle du code postal dans ctrl_cp
			CASE WHEN (																			--			Si (
						 soundex(dbo.fn_Dmot(c.adresse2)) IN									--			La composition phonétique du dernier mot de l'adresse 2 (jdf) se retrouve dans l'une de ces valeur
						 (																		--			( 		
						 	soundex(dbo.fn_Dmot(m.adr1)),										--				La composition phonétique du dernier mot de l'adresse 1 (magelan)
						  	soundex(dbo.fn_Dmot(m.adr2)),										--				La composition phonétique du dernier mot de l'adresse 2 (magelan)
						  	soundex(dbo.fn_Dmot(m.adr3)),										--				La composition phonétique du dernier mot de l'adresse 3 (magelan)
						  	soundex(dbo.fn_Dmot(m.adr4)) 										--				La composition phonétique du dernier mot de l'adresse 4 (magelan)
						 ) 																		--			)
						 OR																		--			Ou
						 (																		--			(
						 	c.adresse2 IS NULL OR  ltrim(c.adresse2) = ''						--				Si l'adresse 2 (jdf) est null ou vide
					 	 )																		--
					   ) 																		--			)
					   AND 																		--			Et
					   (																		--			(
					      soundex(dbo.fn_Dmot(c.adresse3)) IN 									--			Si la composition phonétique du dernier mot de l'adresse 3 (jdf) se retrouve dans l'une de ces valeur
					   (																		--			(
					   	  soundex(dbo.fn_Dmot(m.adr1)),											--				La composition phonétique du dernier mot de l'adresse 1 (magelan)
					   	  soundex(dbo.fn_Dmot(m.adr2)),											--				La composition phonétique du dernier mot de l'adresse 2 (magelan)
					   	  soundex(dbo.fn_Dmot(m.adr3)),											--				La composition phonétique du dernier mot de l'adresse 3 (magelan)
					   	  soundex(dbo.fn_Dmot(m.adr4)) 											--				La composition phonétique du dernier mot de l'adresse 4 (magelan)
					   ) 																		--			)
					   OR																		--			Ou
					   (																		--			(
					   	  c.adresse3 IS NULL OR ltrim(c.adresse3) = '')) THEN					--				Si L'adresse 3 (Jdf) est null et vide 
					   	  'O' ELSE 'N' 															--				Alors O sinon N
					   	  END AS 																--			On Stocke la réponse dans
					   	  		ctrl_adr,														--			ctrl_adr 
					   	  		m.Code_R,														--		Selectionne le Code_R (magelan)
					   	  		m.Code_P,														--		Selectionne le code client (magelan)
					   	  		m.Code_Action,													--		Selection le code action (magelan)
					   	  		RTRIM(m.Titre) AS titre,										--		Selection le titre (magelan) en supprimant les espace de droite avec pour alias titre
					   	  		m.Mnt_Offre,													--		Selection le montant de l'offre choisi par le client (magelan)
					   	  		m.Duree,														--		Selection la durée de l'offre (megelan)
					   	  		m.mnt_Reg,														--		Selection 
					   	  		m.regle,														--		
					   	  		m.Ech_deb,														--		Selection de la date de début de l'écheance (magelan)										
					   	  		m.Ech_fin,														--		Selection de la date de fin de l'écheance (magelan)
					   	  		m.Tirage_deb,													--		Selection de la date de debut du tirage du journal (magelan)
					   	  		m.Tirage_Fin,													--		Selection de la date de fin du tirage du journal (magelan)
					   	  		m.Date_evt,														--		Selection de la date de l'évenement ... (magelan)
					   	  		m.Raisoc,														--		Selection de la raison sociale (magelan)
					   	  		m.civ,															--		Selection de la civilité (magelan)
					   	  		m.Nom,															--		Selection du nom (magelan)
					   	  		m.Prenom,														--		Selection du prenom (magelan)
					   	  		m.Adr1,															--		Selection de l'adresse 1 (magelan)
					   	  		m.Adr2,															--		Selection de l'adresse 2 (magelan)
					   	  		m.Adr3,															--		Selection de l'adresse 3 (magelan)
					   	  		m.Adr4,															--		Selection de l'adresse 4 (magelan)
					   	  		m.CP,															--		Selection du code postal (magelan)
					   	  		m.Ville,														--		Selection de la ville (magelan)
					   	  		m.pays,															--		Selection du pays (magelan)
					   	  		m.ZIP_Code,														--		Selection du code postal (magelan)
					   	  		m.Date_adresse,													--		Selection de la date de changement de l'adresse par magelan (magelan)
					   	  		m.Telephone,													--		Selection du numéro de téléphone (magelan)
					   	  		m.Email,														--		Selection de l'email (magelan)
					   	  		m.Motif_Ann,													--		Selection du motif d'annulation (magelan)
					   	  		m.Motif_Stop_Rel,												--		Selection du motif ... (magelan)
					   	  		RTRIM(m.Sous_type_tiers) AS	Sous_type_tiers,					--		Selection du sous type tiers en supprimer les espace de droite avec pour alias Sous_type_tiers
					   	  		m.synchro,														--		Selection ... (magelan)
					   	  		c.email AS JDF_email,											--		Selection de l'email (jdf)
					   	  		CmpAsso.datedemADH,												--		(?) Selection de la date de demande de l'adhesion (jdf)
					   	  		CmpAsso.datedemclubiste,										--		(?) Selection de la date de demande de clubiste (jdf)
					   	  		c.club,															--		(?) Selection du numéro du club (jdf)
					   	  		ISNULL(CmpAsso.ISADH, 0) AS ISADH,								--		Selection true si l'utilisateur est adhérent et false sinon avec pour alias ISADH (jdf)
					   	  		CmpAsso.IsClubiste,												--		Selection la boolean clubiste ou non (jdf)
					   	  		CmpAsso.Situation,												--		(?) Selection de la situalition de l'adhérent (jdf)
					   	  		CmpAsso.RefSituation,											--		(?) Selection de la référence de la situation de l'adhérent (jdf)
					   	  		CmpAsso.DateSituation,											--		(?) Selection de la date de situation de l'adhérent (jdf)
					   	  		CmpAsso.DateEditionCarte,										--		Seleciton la date d'émission de la carte de l'adhérent (jdf)
					   	  		CmpAsso.IsCL,													--		(?) Selection de la boolean is CL (jdf)
					   	  		c.Adresse1 AS JDF_Adr1,											--		Selection de l'adresse 1 avec pour alias JDF_Adr1(jdf)
					   	  		c.nomsociete AS JDF_Cmpnom,										--		Selection du nom de la societe avec pour alias JDF_Cmpnom(jdf)
					   	  		c.telephone AS JDF_Tel,											--		Selection du téléphone avec pour alias JDF_TEL(jdf)
					   	  		c.type,															--		Selection du type d'adhérent (jdf)
					   	  		c.societe AS JDF_Societe,										--		(?) Selection societe avec pour alias JDF_Societe(jdf)
					   	  		c.adresse0 AS JDF_Adr0,											--		Selection de l'adresse 0 avec pour alias JDF_Adr0(jdf)
					   	  		c.Titre AS JDF_titre, 											--		(?) Selection du titre avec pour alias JDF_titre(jdf)
					   	  		c.CodeClient AS JDF_CC, 										--		Selection du CodeClient avec pour alais JDF_CC (jdf)
					   	  		m.compteur, 													--		Selection du compteur (magelan)
					   	  		c.DateModificationFiche, 										--		Selection de la derniere date de modification de la fiche client (jdf)
					   	  		c.Origine,														--  	(?)	Selection de l'origine (jdf)
					   	  		c.CodeRustica AS JDF_CODER,										--		Selection du code rustica avec pour alias JDF_CODER	 (jdf)
					   	  		CmpAsso.optDistrib,												--		(?) Selection optDistrib compe assosciation (jdf)
					   	  		c.Pays AS Jdf_pays,												--		Selection du pays avec pour alias Jdf_pays (jdf)
					   	  		CmpAsso.datenomADH,												--		(?) Selection de la date nom adhérent (jdf)
					   	  		CmpAsso.Situation_APR,											--		(?) Selection de la situation APR (jdf)
					   	  		CmpAsso.DateSituation_APR,
					   	  		CmpAsso.RefSituation_APR,
					   	  		c.pasclub 
	FROM Magellan m 
		LEFT OUTER JOIN Clients c ON m.Code_P = c.CodeClient 
		LEFT OUTER JOIN CmpAsso ON c.CodeClient = CmpAsso.codeclient 
			WHERE 
				(m.synchro = 0) 
				AND 
				( NOT (m.Code_P IS NULL) ) 
				AND 
				(
					m.compteur NOT IN 
						(
							SELECT compteur FROM magellan_anomalie
						)
				) 
				AND 
				(m.Ech_fin IS NOT NULL) 
				AND 
				(m.compteur BETWEEN @compteur_dep AND @compteur_fin)