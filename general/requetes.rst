Les requetes :
==============

Lexique
=======

- ADH = Adhérent + abonnée 
- ADE = Adhérent seul 
- EXP = (Depreced) Expert appartenant à rustica 
- (?) = Ligne de requete dont l'explication n'est pas encore suffisant claire et validée 

Me.SqlSelectCommand1 (#R1)
--------------------------

Resumée de la requête:
~~~~~~~~~~~~~~~~~~~~~~

La requête sélectionne le nom, prenom , adresse, code postal, ville de la table Client (Jdf), la table CmpAsso (Jdf) et la table magelan (magelan)

Elle aligne les lignes de ces tables avec les conditions suivantes :

m.Code_P = c.CodeClient , c.CodeClient = CmpAsso.codeclient

Elle ne prend que les lignes dont les champs suivant de magelan ne ressemblent pas phonétiquement à ceux jdf

Il y a un champ dont elle ne prend pas toute la valeur phonétique mais seulement le dernier mot grâce à la fonction scalaire Dmot

Si la ligne n'est pas déja synchronisé avec jdf (m.synchro) et n'est pas dans les anomalie ( si le compteur n'est pas dans les anomalies )

Des lignes dont un des champs suivant :

	- nom
	- prenom
	- raison sociale
	- nom societe
	

de la table magelan sont présents en double dans la table client 

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
	-- difference = calcule le soundex de deux chaines et renvoie 0 à 4 selon la similitude 0 pour pas 4 pour beaucoup
	-- SELECT -- Sélectionne les champs de la table 'client' (JDF) et leur applique un alias
	-- dbo.fn_Dmot = Recupere le dernier mot d'une chaine
	-- les champs ctrl_* sont des valeur de controle de comformité des champs ce ne sont pas de vrai champ en bdd
	-- m.Code_P = c.CodeClient , c.CodeClient = CmpAsso.codeclient | Ces champs sont la représentation du code clients
	-- (?) =  Ligne de requête donc l'explication n'est pas encore suffisament claire
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
						(charindex(m.nom, c.nomsociete) > 0 OR difference(m.nom, c.nom) > 2) 	-- 						Si le nom (magelan) est présent dans le nom de la sociéte (jdf) ou si le nom est ressemblant phonétiquement entre magelan et jdf
						OR 																		-- 						Sinon 
						m.nom IS NULL 															--						Si le nom (magelan) est null
					) 																			-- 					)
					THEN 'O' ELSE 'N' 															--						Alors O Sinon N
				END 																			--					Fin					
			END AS ctrl_nom,																	--					On stocke la reponse du nom ctr_nom
			CASE WHEN c.codepostal = m.cp THEN 													--			Si Le code postal (jdf) est égal au code postal (magellan) Alors
			'O' ELSE 'N' 																		-- 			Alors O Sinon N
			END AS ctrl_CP,																		--			On stocke le controle du code postal dans ctrl_cp
			CASE WHEN (																			--			Si (
						 soundex(dbo.fn_Dmot(c.adresse2)) IN									--			La composition phonétique du dernier mot de l'adresse 2 (jdf) se retrouve dans l'une de ces valeurs
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
					      soundex(dbo.fn_Dmot(c.adresse3)) IN 									--			Si la composition phonétique du dernier mot de l'adresse 3 (jdf) se retrouve dans l'une de ces valeurs
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
					   	  		m.Code_R,														--		Sélectionne le Code_R (magelan)
					   	  		m.Code_P,														--		Sélectionne le code client (magelan)
					   	  		m.Code_Action,													--		Sélection le code action (magelan)
					   	  		RTRIM(m.Titre) AS titre,										--		Sélection le titre (magelan) en supprimant les espaces de droite avec pour alias titre
					   	  		m.Mnt_Offre,													--		Sélection le montant de l'offre choisi par le client (magelan)
					   	  		m.Duree,														--		Sélection la durée de l'offre (megelan)
					   	  		m.mnt_Reg,														--		Sélection 
					   	  		m.regle,														--		
					   	  		m.Ech_deb,														--		Sélection de la date de début de l'écheance (magelan)										
					   	  		m.Ech_fin,														--		Sélection de la date de fin de l'écheance (magelan)
					   	  		m.Tirage_deb,													--		Sélection de le numéro de debut du tirage du journal (magelan)
					   	  		m.Tirage_Fin,													--		Sélection de le numéro de fin du tirage du journal (magelan)
					   	  		m.Date_evt,														--		Sélection de la date de l'évenement ... (magelan)
					   	  		m.Raisoc,														--		Sélection de la raison sociale (magelan)
					   	  		m.civ,															--		Sélection de la civilité (magelan)
					   	  		m.Nom,															--		Sélection du nom (magelan)
					   	  		m.Prenom,														--		Sélection du prenom (magelan)
					   	  		m.Adr1,															--		Sélection de l'adresse 1 (magelan)
					   	  		m.Adr2,															--		Sélection de l'adresse 2 (magelan)
					   	  		m.Adr3,															--		Sélection de l'adresse 3 (magelan)
					   	  		m.Adr4,															--		Sélection de l'adresse 4 (magelan)
					   	  		m.CP,															--		Sélection du code postal (magelan)
					   	  		m.Ville,														--		Sélection de la ville (magelan)
					   	  		m.pays,															--		Sélection du pays (magelan)
					   	  		m.ZIP_Code,														--		Sélection du code postal (magelan)
					   	  		m.Date_adresse,													--		Sélection de la date de changement de l'adresse éffectuée par magelan (magelan)
					   	  		m.Telephone,													--		Sélection du numéro de téléphone (magelan)
					   	  		m.Email,														--		Sélection de l'email (magelan)
					   	  		m.Motif_Ann,													--		Sélection du motif d'annulation (magelan)
					   	  		m.Motif_Stop_Rel,												--		Sélection du motif ... (magelan)
					   	  		RTRIM(m.Sous_type_tiers) AS	Sous_type_tiers,					--		Sélection du sous type tiers en supprimant les espaces de droite avec pour alias Sous_type_tiers
					   	  		m.synchro,														--		Sélection ... (magelan)
					   	  		c.email AS JDF_email,											--		Sélection de l'email (jdf)
					   	  		CmpAsso.datedemADH,												--		(?) Sélection de la date de demande de l'adhésion (jdf)
					   	  		CmpAsso.datedemclubiste,										--		(?) Sélection de la date de demande de clubiste (jdf)
					   	  		c.club,															--		(?) Sélection du numéro du club (jdf)
					   	  		ISNULL(CmpAsso.ISADH, 0) AS ISADH,								--		Sélection true si l'utilisateur est adhérent et false sinon avec pour alias ISADH (jdf)
					   	  		CmpAsso.IsClubiste,												--		Sélection la boolean clubiste ou non (jdf)
					   	  		CmpAsso.Situation,												--		(?) Sélection de la situalition de l'adhérent (jdf)
					   	  		CmpAsso.RefSituation,											--		(?) Sélection de la référence de la situation de l'adhérent (jdf)
					   	  		CmpAsso.DateSituation,											--		(?) Sélection de la date de situation de l'adhérent (jdf)
					   	  		CmpAsso.DateEditionCarte,										--		Seleciton la date d'émission de la carte de l'adhérent (jdf)
					   	  		CmpAsso.IsCL,													--		(?) Sélection de la boolean is CL (jdf)
					   	  		c.Adresse1 AS JDF_Adr1,											--		Sélection de l'adresse 1 avec pour alias JDF_Adr1(jdf)
					   	  		c.nomsociete AS JDF_Cmpnom,										--		Sélection du nom de la societe avec pour alias JDF_Cmpnom(jdf)
					   	  		c.telephone AS JDF_Tel,											--		Sélection du téléphone avec pour alias JDF_TEL(jdf)
					   	  		c.type,															--		Sélection du type d'adhérent (jdf)
					   	  		c.societe AS JDF_Societe,										--		(?) Sélection societe avec pour alias JDF_Societe(jdf)
					   	  		c.adresse0 AS JDF_Adr0,											--		Sélection de l'adresse 0 avec pour alias JDF_Adr0(jdf)
					   	  		c.Titre AS JDF_titre, 											--		(?) Sélection du titre avec pour alias JDF_titre(jdf)
					   	  		c.CodeClient AS JDF_CC, 										--		Sélection du CodeClient avec pour alais JDF_CC (jdf)
					   	  		m.compteur, 													--		Sélection du compteur (magelan)
					   	  		c.DateModificationFiche, 										--		Sélection de la derniere date de modification de la fiche client (jdf)
					   	  		c.Origine,														--  	(?)	Sélection de l'origine (jdf)
					   	  		c.CodeRustica AS JDF_CODER,										--		Sélection du code rustica avec pour alias JDF_CODER	 (jdf)
					   	  		CmpAsso.optDistrib,												--		(?) Sélection optDistrib compe assosciation (jdf)
					   	  		c.Pays AS Jdf_pays,												--		Sélection du pays avec pour alias Jdf_pays (jdf)
					   	  		CmpAsso.datenomADH,												--		(?) Sélection de la date nom adhérent (jdf)
					   	  		CmpAsso.Situation_APR,											--		(?) Sélection de la situation APR (jdf)
					   	  		CmpAsso.DateSituation_APR,										--		(?) Sélection de la date situation APR (jdf)
					   	  		CmpAsso.RefSituation_APR,										--		(?) Sélection de la référence de la situation (jdf)
					   	  		c.pasclub 														--		(?) Sélection de la boolean appartien ou est un club (jdf)
	FROM Magellan m 																			--		Sur la table magelan avec pour alias m
		LEFT OUTER JOIN Clients c ON m.Code_P = c.CodeClient 									--		Ainsi que la table Clients avec pour alias c et dont la ligne du code client magelan doit être la même que le code client jdf 
		LEFT OUTER JOIN CmpAsso ON c.CodeClient = CmpAsso.codeclient 							--		Ainsi que la table CmpAsso et dont la ligne du code client compte asso doit être la même que la ligne du code client clients 
			WHERE 																				--		Si
				(m.synchro = 0) 																--		La ligne coté magelan n'est pas encore synchronisée avec jdf
				AND 																			--		Et
				( NOT (m.Code_P IS NULL) ) 														--		(?) Et que le code P n'est pas nul (magelan)
				AND 																			--		Et
				(																				--		(
					m.compteur NOT IN 															--			Le Compteur n'est pas dans (magelan)
						(																		--			(
							SELECT compteur FROM magellan_anomalie								--				les compteur considerés comme anomalie (magelan)
						)																		--			)
				) 																				--		)
				AND 																			--		Et
				(m.Ech_fin IS NOT NULL) 														--		L'écheance de fin de magelan est nulle (magelan)
				AND 																			--		Et
				(m.compteur BETWEEN @compteur_dep AND @compteur_fin)							--		Le compteur se trouve entre le compteur de début et fin spécifier (magelan)
				
Requete (#R2)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

Cette requête permet de trouver les données magellan qui n'on pas encore été importé

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	SELECT 						-- On selectionne ces champs dans les lignes
		'N' AS ctrl_nom,		-- 
		'N' AS ctrl_CP,			--
		'N' AS ctrl_adr,		--
		Code_R,					-- Le code client magélan
		Code_P,					-- Le code client jdf
		Code_Action,			-- Le code Action
		Titre,					-- Le titre
		Mnt_Offre,				-- Le montant de l'offre
		Duree,					-- La durée de l'offre
		mnt_Reg,				-- Le montant réglé
		regle,					-- Si reglé ou pas
		Ech_deb,				-- L'échéance de début
		Ech_fin,				-- L'échéance de fin
		Tirage_deb,				-- Le numéro de début de tirage
		Tirage_Fin,				-- Le numéro de fin de tirage
		Date_evt,				-- La date de l'evenement
		Raisoc,					-- La raison sociale
		civ,					-- La civilité
		Nom,					-- Le Nom
		Prenom,					-- Le prénom
		Adr1,					-- L'adresse 
		Adr2,					-- L'adresse suite
		Adr3,					-- L'adresse suite
		Adr4,					-- L'adresse suite
		CP,						-- Le code postal
		Ville,					-- La ville
		pays,					-- Le pays
		ZIP_Code,				-- Le code postal
		Date_adresse,			-- La date de modification de l'adresse postale
		Telephone,				-- Le téléphone
		Email,					-- L'email
		Motif_Ann,				-- Le motif de l'annulation
		Motif_Stop_Rel,			-- (?) Le Motif 
		Sous_type_tiers,		-- Le sous type tiers
		synchro,				-- Si dispatché dans la base ou 
		0 AS taux               -- (?)
		compteur 				-- Le numéro de compteur
	FROM Magellan m WHERE 		-- Dans la table magelan s'il remplisse les condition suivante...
		(synchro = 0) AND (Code_P IS NULL) AND (compteur BETWEEN @compteur_dep AND @compteur_fin) -- La ligne n'est pas encore dispatché dans la base et le code client JDF est nul et donc pas encore identifié ainsi que si le compteur se situe entre le parametre @compteur_dep (debut) et @compteur_fin (fin)

Requete (#R3)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

Cette commande permet de trouver les prospect par son codeClient

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

Cmd_SearchPropect()

::

	SELECT -- Selection des champs dans les lignes 
		CodeClient,		-- Code client 
		type,			-- Type
		Nom,			-- Nom
		Prenom,			-- Prenom
		CodePostal,		-- Code postal
		Ville 			-- Ville
	FROM Prospects WHERE 	-- Dans la table 'Prospects' dans la conditions
		(CodeClient = @codeclient)	-- Ou le code client est égale au parametre CodeClient


Requete (#R4)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	SELECT c.nom                    AS JDF_Nom,  -- Le nom avec pour alias JDF_NOM
	       c.prenom                 AS JDF_Prenom, -- Le prenom avec pour alias JDF_Prenom
	       c.adresse2               AS JDF_Adr2, 	-- L'adresse avec pour alias JDF_Adr2
	       c.adresse3               AS JDF_adr3, 	-- L'adresse avec pour alias JDF_Adr3
	       c.codepostal             AS JDF_cp, 		-- Le code postal avec pour alias JDF_cp
	       c.ville                  AS JDF_Ville,   -- La ville avec pour alias JDF_Ville
	       CASE 									-- Quand...
	         WHEN raisoc IS NULL THEN 				-- La raison sociale est nul alors ..
	           CASE 								-- Quand...
	             WHEN Difference(m.nom, c.nom) > 2 	-- Le nom chez magelan et jdf sont different
	                  AND ( ( m.prenom IS NULL      -- Et le prénom(magelan) est nul
	                           OR m.prenom = '' ) 	-- Ou le prenom(magelan) est vide
	                        AND ( c.prenom IS NULL 	-- Et le prénom(jdf) est nul
	                               OR c.prenom = '' ) ) -- Or le prénom(jdf) est vide
	                   OR Difference(m.prenom, c.prenom) > 2 THEN 'O' -- Le préson chez magélan et jdf sont différent
	             ELSE 'N' 											-- Sinon N
	           END 												 	-- Fin
	         ELSE 													-- Sinon
	           CASE 												-- Quand
	             WHEN Difference(m.raisoc, Ltrim(c.adresse0)) > 2 	-- La raison sociale et le début de l'adresse sont différent
	                  AND ( ( Charindex(m.nom, c.nomsociete) > 0 	-- 
	                           OR Difference(m.nom, c.nom) > 2 )    -- Ou Si le nom de chez magelan est différent du nom de chez jdf
	                         OR m.nom IS NULL ) THEN 'O' 			-- Ou si le nom de chez magelan est null Alors O
	             ELSE 'N' 											-- Sinon N
	           END 													-- Fin
	       END                      AS ctrl_nom, 					-- Avec pour alias ctrl_nom
	       CASE 													-- Quand...
	         WHEN c.codepostal = m.cp THEN 'O' 						-- Le Code postal de jdf est euivalent au code postal de magelan alors 0
	         ELSE 'N' 												-- Sinon N
	       END                      AS ctrl_CP, 					-- On stocke le résultat dans le champ ctrl_CP
	       CASE 													-- Quand...
	         WHEN ( Soundex(dbo.Fn_dmot(c.adresse2)) IN ( 			-- Si la prononciation phonétique de l'adresse (jdf) équivaut à 
	                         Soundex(dbo.Fn_dmot(m.adr1)), Soundex( -- La prononciation phonétique du dernier mot de adresse 2 (magelan)
	                         dbo.Fn_dmot(m.adr2)), 					--
	Soudex( 														-- 
	dbo.Fn_dmot(m.adr3)), 											-- La composition phonétique du dernier mot de l'adresse3 de magelan  
	Soundex( 														-- + 
	dbo.Fn_dmot(m.adr4)) ) 											-- La composition phénotique du dernier mot de l'adresse4 de magelan  
	OR ( c.adresse2 IS NULL 										-- Ou l'adresse 2 est nul 
	OR Ltrim(c.adresse2) = '' ) ) 									-- ou vide
	AND ( Soundex(dbo.Fn_dmot(c.adresse3)) IN ( 					-- La composition phonétique du dernier mot de l'adresse 
	Soundex(dbo.Fn_dmot(m.adr1)), Soundex(dbo.Fn_dmot(m.adr2)),   	-- La composition phonétique du dernier mot de l'adrese 1 et 2 magelan
	 Soundex( 														-- + 
	 dbo.Fn_dmot(m.adr3)), 											-- La composition phonétique du dernier mot de l'adresse3 de magelan 
	Soundex( 														-- + 
	 dbo.Fn_dmot(m.adr4)) ) 										-- La composition phénotique du dernier mot de l'adresse4 de magelan  
	OR ( c.adresse3 IS NULL 										-- Ou si l'adresse 3 jdf est nul
	OR Ltrim(c.adresse3) = '' ) ) THEN 'O' 							-- ou vide alors O 
	ELSE 'N' 														-- Sinon N
	END                      AS ctrl_adr, 							-- Et on stocke le résultat dans le champ 'ctrl_adr'
	m.code_r, 														-- [Magelan] code P (magelan)
	m.code_action, 													-- [Magelan] code action (magelan)
	Rtrim(m.titre)           AS titre, 								-- [Magelan] titre (magelan)
	m.mnt_offre, 													-- [Magelan] montant offre
	m.duree, 														-- [Magelan] duree
	m.mnt_reg, 														-- [Magelan] Montatn réglé
	m.regle, 														-- [Magelan] Si le client à réglé ou pas sa commande
	m.ech_deb, 														-- [Magelan] L'echeance de début
	m.ech_fin, 														-- [Magelan] L'échance de fin
	m.tirage_deb, 													-- [Magelan] Le numéro de début du tirage
	m.tirage_fin, 													-- [Magelan] Le numéro de fin du tirage
	m.date_evt, 													-- [Magelan] (?) La date d'execution de l'action
	m.raisoc, 														-- [Magelan] La raison sociale
	m.civ, 															-- [Magelan] La civilité
	m.nom, 															-- [Magelan] Le nom
	m.prenom, 														-- [Magelan] Le prenom
	m.adr1, 														-- [Magelan] L'adresse
	m.adr2, 														-- [Magelan] L'adresse
	m.adr3, 														-- [Magelan] L'adresse
	m.adr4, 														-- [Magelan] L'adresse
	m.cp, 															-- [Magelan] Le code postal
	m.ville, 														-- [Magelan] La ville
	m.pays, 														-- [Magelan] Le pays
	m.zip_code, 													-- [Magelan] LE zip code
	m.date_adresse, 												-- [Magelan] La date du dernier changement d'adresse
	m.telephone, 													-- [Magelan] Le numéro de téléphone
	m.email, 														-- [Magelan] L'email
	m.motif_ann, 													-- [Magelan] Le motif d'annlulation
	m.motif_stop_rel, 												-- [Magelan] Le motif stop rel
	Rtrim(m.sous_type_tiers) AS Sous_type_tiers, 					-- [Magelan] Le souu type riers avec pour alias Sou
	m.synchro, 														-- [Magelan] Si la ligne à été dispatché dans la base
	c.email                  AS JDF_email, 							-- [Magelan] L'email avec pour alias JDF_email
	cmpasso.datedemadh, 											-- [JDF] La date de demande d'adhésion					
	cmpasso.datedemclubiste, 										-- [JDF] La date de demande clubiste
	c.club, 														-- [JDF] Le numéro de club
	cmpasso.isadh, 													-- [JDF] Si le client est adhérent
	cmpasso.isclubiste,												-- [JDF] Si le client est clubiste
	cmpasso.situation, 												-- [JDF] La situation du client
	cmpasso.refsituation, 											-- [JDF] La référence de la situation
	cmpasso.datesituation, 											-- [JDF] La date de la situation
	cmpasso.dateeditioncarte, 										-- [JDF] La date d'edition de la carte
	cmpasso.iscl, 													-- [JDF] Si le client est un clubiste
	c.adresse1               AS JDF_Adr1, 							-- [JDF] L'adresse du client avec pour alias JDF_Adr1
	c.nomsociete             AS JDF_Cmpnom, 						-- [JDF] Le nom de la societe du client avec pour alias JDF_Cmpnom
	c.telephone              AS JDF_Tel, 							-- [JDF] Le téléphone du client avec pour alias JDF_Tel
	c.type, 														-- [JDF] Le type de client
	c.societe                AS JDF_Societe, 						-- [JDF] La société du client avec pour alias JDF_Societe
	c.adresse0               AS JDF_Adr0, 							-- [JDF] L'adresse 0 ou Raison sociale du client avec pour alias JDF_Adr0
	c.titre                  AS JDF_titre, 							-- [JDF] Le titre du client avec pour alias JDF_titre
	c.codeclient             AS JDF_CC, 							-- [JDF] Le code client avec pour alias JDF_CC
	m.compteur, 													-- [Magelan] Le code client
	c.datemodificationfiche, 										-- [JDF] La date de modification de la fiche client
	c.origine, 														-- [JDF] L'origine du client 
	c.coderustica            AS JDF_CODER 							-- [JDF] Le code rustica du client avec pour alias JDF_CODER
	FROM   magellan m 												-- Dans la date magelan avec pour alias m
	       LEFT OUTER JOIN clients c 								-- Ainsi que dans la table **client** avec pour alias
	                    ON m.code_p = c.codeclient 					-- On aligne la table avec le code **client** de la table client sur le code p de **rustica**
	       LEFT OUTER JOIN cmpasso 									-- Ainsi que dans la table **cmpasso**
	                    ON c.codeclient = cmpasso.codeclient 		-- On aliigne la table avec le code client de la table **cmpasso** sur le code client de la table **client**
	WHERE  ( m.synchro = 0 ) 										-- Si la ligne n'est pas dispatché dans al base
	       AND ( NOT ( m.code_p IS NULL ) ) 						-- Et que le Code client n'est pas nul
	       AND ( m.compteur IN (SELECT compteur 					-- Et que le compteur de la table **magelan** est la collone compteur d'une des ligne de la table **magellan_anomalie**
	                            FROM   magellan_anomalie) ) 		-- ()
	       AND ( m.ech_fin IS NOT NULL ) 							-- Et que l'echeance fin n'est pas nul
 
Requete (#R4)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

On selectionne les lignes à dispatcher (?)

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	SELECT m.compteur 											-- Selectione Compteur
	FROM   magellan m 											-- Dans la table magelan avec pour alias m 
	       LEFT OUTER JOIN clients c 							-- Joint à la table client avec pour alias c
	                    ON m.code_p = c.codeclient 				-- On aligne le champ code client de la table ``client`` au code p de la table ``magelan``
	       LEFT OUTER JOIN cmpasso 								-- Joint à la table cmpasso 
	                    ON c.codeclient = cmpasso.codeclient 	-- On aligne le champ code client de la table ``client`` au code client de la table ``cmpasso``
	WHERE  ( m.synchro = 0 ) 									-- Quand la ligne n'est pas encore dispatché en base
	       AND ( NOT ( m.code_p IS NULL ) ) 					-- Et que le code p n'est pas nul
	       AND ( m.compteur NOT IN (SELECT compteur 			-- Ainsi que le compteur ne se trouve pas dans les anomalie
	                                FROM   magellan_anomalie) ) -- De la table ``magellan_anomalie``
	       AND ( m.ech_fin IS NOT NULL ) 						-- Et que le champ ech_fin n'est pas nul
       
        
Requete (#R5)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

:: info:

	Magellan_Affecter_Code_Client

On renseigne le code p des ligne de la table magelan dont le compteur est égale au parametre compteur


Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE magellan cc 			-- Mise à jour de la table magelan avec pour alias cc
	SET    code_p = '' 			-- Le code p est égale au parametre code_p
	WHERE  code_p IS NULL 		-- Dans les lignes ou le code p est null
	       AND compteur = "" 	-- Et ou le compteur est égale au parametre compteur
     
Requete (#R6)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

:: info:

	Magellan_Supprimer_Code_P


On supprime le code p pour les ligne selectioner

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~   

::

	UPDATE magellan 			-- Mise à jour de la table magelan
	SET    code_p = NULL 		-- Le code p est égale à nul
	WHERE  compteur = "" 		-- Si le compteur est égale à au compteur de analyse


Requete (#R7)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

(?)


Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~     

::

	update CmpAsso  			-- Mise à jour de la table CmpAsso
			set 				-- Modification
	            datedemadh=''	-- La date de demission de l'adhérent
			    datenomadh='', -- [IIF] La date de nomination de l'adhérent est égale à 
	            optdistrib=1 ou optdistrib=0 -- [IIF] 
	            isadh=1, -- On le définit comme adhérent
	            situation = '',	-- La situation est égale à
	            refsituation=''	-- La référence situation est égale à 
	        where 				-- Si
	        codeclient=			-- Le code client est égale à 
        
Requete (#R8)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

On récupere le code r et l'échéance de début des ligne de la table magelan dont le compteur est spécifié

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

   select 			   -- On sélectionne...
   		code_r,		   -- Le code R   
   		ech_fin 	   -- L'écheance début
   		from magellan  -- sur la table magelan
   where 			   -- Quand...
   		compteur = 	   -- Le compteur est égale au parametre compteur

Requete (#R9)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~
:: info:

	Modifier_CmpAsso_ADH_DEMISSION

La personen démisionne de son status d'adhérent

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~
       
::

	-- Se référér à la doc pour l'existation de la situation et du code Situation
		UPDATE cmpasso 					-- Mise à jour de la table cmpasso
		SET    datedemadh = Getdate(), 	-- La date de demission de l'adhérent est égale à la date du jour
		       isadh = 0, 				-- On le définit comme n'étant pas un adhérent
		       situation = 'X', 		-- La situation est égale à 'X' 
		       refsituation = 'A:' 		-- La référence situation est égale à 'A:'
		WHERE  codeclient = "" 			-- Si le code client est égale au code p de la vue en cours
        
Requete (#R10)
--------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~
:: info:

	Modifier_CmpAsso_ADR_DEMISSION

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

La personne démisione de son status de clubiste

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::
	-- Rappel le code client est égale au code p

	UPDATE cmpasso 							-- Mise à jour de la table 'cmpasso'
	SET    datedemclubiste = Getdate(), 	-- La date de démission du clubiste est égale à la date du jour
	       isclubiste = 0, 					-- La personne n'est plus clubiste
	       situation_apr = 'X', 			-- Sa situation apr est égale à 'X'
	       refsituation_apr = '> Magellan' 	-- Et sa référence situation apr est égale à '> Magellan'
	WHERE  codeclient = "" 					-- Si le code client est égale a au code p de la vue en cours


Requete (#R11)
--------------

Resumé de la requete :
~~~~~~~~~~~~~~~~~~~~~~
:: info:

	Modifier_CompAsso_Clubiste

On met a jour le compte asso du clubiste vis a vis de la vue actuelle

Schematique de la requete :
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE cmpasso 					-- Mise à jour de la table 'cmpasso'
	SET    datedemclubiste = '', 	-- La date de démission du clubiste est égale à la vue actuelle
	       datenomclubiste = '', 	-- La date de nomination du clubiste est égale à la vue actuelle
	       isclubiste = 1, 			-- La personne est définit comme étant clubiste
	       situation = '', 			-- La situation est égale au parametre codes
	       refsituation = '' 		-- La référence situation est égale au aprametres codes suivie du numéro de compteur séparé par ':' (ex : A:454147)
	WHERE  codeclient = "" 			-- Si le code client est égale à la vue courante du code p


Requete (#R12)
--------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~
:: info:

	Modifier_CmpAsso_Clubiste

On met a jour le compte asso du clubiste vis a vis de la vue actuelle
La différence ici est qu'on RAZ sa situation à 0

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE cmpasso 					-- Mise à jour de al table cmpasso
	SET    datedemclubiste = '', 	-- La date de demision du clubiste est égale à l'échéance fin de la vue
	       datenomclubiste = Isnull(datenomclubiste, ''), -- La date de nomination du clubiste est égale soit la date de nomitation du clubiste soit à '' si celle ci est null
	       isclubiste = 1, 			-- La personne est défini comme étant clubiste
	       situation_apr = '', 		-- La situation apr est égale à (?)
	       refsituation_apr = '', 	-- La référence situation apr est égale à (?)
	       datenomadh = NULL, 		-- La date de nomination de l'adhérent est défini comme null
	       datedemadh = NULL, 		-- La date de démission de l'adhérent est défini comme null
	       situation = NULL, 		-- La situation est défini comme null
	       refsituation = 'RAZ Mage.' -- La référence situation est définir comme 'RAZ Mage.'
	WHERE  codeclient = "p" 		-- Si le code client est égale à p

Requete (#R13)
--------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~
:: info:

	Modifier_CmpAsso_Exper_Clubiste

On modifie le compte asso du clubiste exper (?)

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE cmpasso 					-- Mise à jour de la table 'cmpasso'
	SET    datedemclubiste = '',    -- La date de demision du clubiste est égale à l'échéance fin de la vue
	       datenomclubiste = Isnull(datenomclubiste, ''), -- La date de nomination du clubiste est égale soit la date de nomitation du clubiste soit à '' si celle ci est null
	       isclubiste = 1, 				-- On définit la personen comem étatn clubiste
	       situation_apr = '', 			-- (?)
	       refsituation_apr = '', 		-- (?) 
	       datenomadh = NULL, 			-- La date de nomination de l'adhérent est défini comme null
	       datedemadh = NULL, 			-- La date de démission de l'adhérent est défini comme null
	       situation = NULL, 			-- La situation est défini comme null
	       refsituation = 'RAZ Mage.' 	-- La référence situation est définir comme 'RAZ Mage.'
	WHERE  codeclient = "" 				-- Si le code client est égale à p

Requete (#R14)
--------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

	Modifier_CmpAsso_abonnepur

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

On modifie un comtpe asso pour le déc larter en temps qu'abonner pur

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE cmpasso 				-- Mise à jour de la table cmpasso
	SET    datedemabo = '', 	-- La date de démission de l'abonner est égale à l'echeance fin de la vue
	       datenomabo = '', 	-- La date dé nomitation de l'abonner est égale à l'échéance début de la vue
	       isabo = 1, 			-- La personne est définit comme étant abonner 
	       situation = '', 		-- La situation est égale à au paramètre codes
	       refsituation = '' 	-- La référence situation est égale au paramètre codes plus au compteur spéraré par ':' (ex : A:86544)
	WHERE  codeclient = "" 		-- Le code client est égale au code p de al vue

Requete (#R15)
--------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~
:: info:

	Modifier_CmpAsso_abonnePur_Demission

On met fin à l'abonnement d'un abonnée pur

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE cmpasso 							-- Mise à jour de la table cmpasso
	SET    datedemabo = Getdate(), 			-- La date de demission de l'abonné est égale à la date du jour
	       isabo = 0, 						-- On définit l'aboner comme n'étant plus un abonnée
	       situation = 'X', 				-- La situation est égale à 'X'
	       refsituation = 'Magellan:000' 	-- La référence situation est égale à 'Magelan:000'
	WHERE  codeclient = "" 					-- Le code client est égale au code p de la vue

Requete (#R16)
--------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

Suspens un compte utilisateur pas son code client

:: info:

	Modifier_CmpAsso_Ref_SUSP

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	Type: PNJ,VPC
	
	Sous type : ADH, ADE
	UPDATE cmpasso 						-- Mise à jour de la table cmpasso 
	SET    situation = 'S', 			-- On met ``S`` pour valeur au champ situation (S = SUSPENDU)
	       refsituation = 'SUSPENDU' 	-- On met ``SUSPENDU`` pour valeur au champ refsitation 
	WHERE  codeclient = "" 				-- Pour les lignes donc le code client est égale au parametre ``codeclient``
	
	Sous type : APR, EXP
	UPDATE cmpasso 						-- Mise à jour de la table cmpasso 
	SET    situation = 'S', 			-- On met ``S`` pour valeur au champ situation (S = SUSPENDU)
	       refsituation = 'SUSPENDU' 	-- On met ``SUSPENDU`` pour valeur au champ refsitation 
	WHERE  codeclient = "" 				-- Pour les lignes donc le code client est égale au parametre ``codeclient``
	
	Type : PNJ_ABO_PUR
	UPDATE cmpasso 						-- Mise à jour de la table cmpasso 
	SET    situation = 'S', 			-- On met ``S`` pour valeur au champ situation (S = SUSPENDU)
	       refsituation = 'SUSPENDU' 	-- On met ``SUSPENDU`` pour valeur au champ refsitation 
	WHERE  codeclient = "" 				-- Pour les lignes donc le code client est égale au parametre ``codeclient``


Requete (#R1l)
--------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~
:: info:

	Modifier_CmpAsso_ADH

On met a jour le compte asso de l'adhérent 

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	update CmpAsso set datedemadh='' -- On met a jour la table CmpAsso avec le champ datedemadh (équivalent de ech_fin)
				datenomadh='' 	-- [OPTIONELLE] Date dmh = ech_fin
	            optdistrib=1 optdistrib=0 -- (?)
	            isadh=1, 		-- Definir que le client est un adhérent
	            situation = '',	-- La situation est égale à 
	            refsituation=':'
			where 				-- Pour les lignes ou ..
				codeclient="" 	-- Le code client est égale au parametre code client

Requete (#R12)
--------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~
:: info:

	IsAnnuCorrespondSynchroEnCours

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

	On verifie si (?)

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	SELECT code_r, 			-- On séléction le code r
	       ech_fin 			-- Et l'échéance fin
	FROM   magellan 		-- Dans la table magelan
	WHERE  compteur = "" 	-- Dont le compteur est égale au Parametre Compteur1

Requete (#R13)
--------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~
:: info:

	Modifier_CmpAsso_ADH_DEMISSION

L'adhérent démissione

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE cmpasso 						-- On modifie la table 'cmpassco'
	SET    datedemadh = Getdate(), 		-- La date de démission de l'adhérent est égale à la date du jour
	       isadh = 0, 					-- La personne n'est plus déclaré comme étant abonnée
	       situation = 'X', 			-- La situation est égale à 'X'
	       refsituation = 'A:0001' 		-- La référence situation est égale a 'A:0001'
	WHERE  codeclient = "" 				-- Pour les ligne dont le code client est égale au code p de al vue ene cours


Requete (#R14)
--------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~
:: info:
	
	Modifier_CmpAsso_APR_DEMISSION

(?)

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE cmpasso 							-- On met à jour la table cmpasso
	SET    datedemclubiste = Getdate(), 	-- On definit la daté de démission du clubiste à la date du jour
	       isclubiste = 0, 					-- La personne n'est plus définit comme clubiste
	       situation_apr = 'X', 			-- La situation apr est égale à 'X'
	       refsituation_apr = '> Magellan' 	-- La référence situation apr est égale à '> Magellan'
	WHERE  codeclient = "" 					-- Le code client est égale au code p de la vue en cours

Requete (#R15)
--------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~
:: info:

	Modifier_CmpAsso_Clubiste

On modifie la situation du clubiste

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::
		
	UPDATE cmpasso 							-- Mise à jour de la table 'cmpasso'
	SET    datedemclubiste = '', 			-- La date de démission du clubiste est égale à l'echéance fin de la vue en cours
	       datenomclubiste = '', 			-- La date de nomination du clubiste est égale à l'échéance début de la vue en cours
	       isclubiste = 1, 					-- La personne est définit comme étant clubiste 
	       codes <> "" And codes <> "*",	-- (?)
	       situation = '', 					-- La situation est égale au parametre codes
	       refsituation = ':' 				-- La référence situation est égale au paramètre codes + le compteur de la vue séparé par un ':' (ex: A:84649)
	WHERE  codeclient = "" 					-- Le code client est égale au code p de la vue


Requete (#R16)
--------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~
:: info:

	Modifier_CmpAsso_Exper_Clubiste

On modifie la situation exper du clubiste

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE cmpasso 												-- Mise à jour de la table 'cmpasso'
	SET    datedemclubiste = '', 								-- La date de démission du clubiste est égale à l'échéance fin de la vue en cours
	       datenomclubiste = Isnull(datenomclubiste, ''), 		-- La date de nomination du clubiste est égale à null ou a la date de nomination du clubiste dans la vue en cours
	       isclubiste = 1, 										-- On définit la personne comme étant clubiste
	       situation_apr = '', 									-- La situation apr est égale au parametre codes_rusti
	       refsituation_apr = '' 								-- La référence situation apr est égale au parametre codes_rusti plus le compteur de la vue en cours séparé apr un ':' (ex: A:856555)
	WHERE  codeclient = "" 										-- Le code client est égale au code p de la vue en cours

Requete (#R17)
--------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~
:: info:

	Modifier_CmpAsso_abonnePur

(?)

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE cmpasso 				-- Mise à jour de la table cmpasso
	SET    datedemabo = '', 	-- La date de démission de l'abonner est égale à (?)
	       datenomabo = '', 	-- La date de nomination de l'abonner est égale à (?)
	       isabo = 1 			-- La personne est définit comme étant abonner

Requete (#R18)
--------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~
:: info:

	Modifier_CmpAsso_abonnePur_Demission

L'abonne pur met fin a son abonement

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE cmpasso 						-- Mise à jour de la table 'cmpasso'
	SET    datedemabo = Getdate(), 		-- La date de démission de l'abonner est égale à la date du jour
	       isabo = 0, 					-- L'abonner n'est plus déclaré comme étant abonnée
	       situation = 'X', 			-- La situation est égale à 'X'
	       refsituation = 'Magellan:' 	-- La référence situation ést égale à 'Magellan':
	WHERE  codeclient = "" 				-- Pour les lignes dont le code client est égale au code p de la vue en cours

Requete (#R19)
--------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~
:: info:

	Client_Changer_de_Club

Le client change de club et par conséquent de numéro de club


Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE clients 				-- Mise à jour de la table 'clients'
	SET    club = '' 			-- Le code club est égale au paramètre club
	WHERE  codeclient = "" 		-- Pour les ligne dont le code client est égale au code p de la vue en cours

Requete (#R20)
--------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~
:: info:

	Maj_TopageCa

On met a jour les information de derniere requete du topage ca

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE topageca 					-- Mise à jour de la table 'topageca'
	SET    dateretour = Getdate(), 		-- La date de retour est égale à la date du jour
	       idmagellan = "" 				-- L'identifiant magellan est égale au compteur de la vue en cours
	WHERE  codeclient = "" 				-- Pour les lignes dont le code client est égale au code p de la vue en cours
	       AND idmagellan IS NULL 		-- Et dont l'identifiant magellan est null

Requete (#R21)
--------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~
:: info:

	Is_Wait_TopageCa

On regarde si le client à un topageca ou pas pour si besoin le faire

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	SELECT Count(*) 				-- Selection du nombre de ligne
	FROM   topageca 				-- Dans la table 'topageca'
	WHERE  codeclient = "" 			-- Pour les ligne dont le code client est égale au code p de la vue en cours
	       AND idmagellan IS NULL 	-- Et dont l'identifiant magellan est égale à null

Requete (#R22)
--------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~
:: info:

	Synchro_Email_Tel

On syncrhonise les information de contact champ par champ en indant la date de modification

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE clients 										-- Mise à jour de la table 'clients'
	SET    datemodificationfiche = Getdate(), 			-- La date de modification de la fiche est égale à la date du jour
	       opmodif = 99 								-- (?) L'opération de modifcation est '99'
	       SET%%														
	WHERE  codeclient = "" 								-- LE code client est égale au code p de la vue actuelle
	
	UPDATE clients 
	SET    datemodificationfiche = Getdate(), 
	       opmodif = 99 
	       SET%%
	WHERE  codeclient = "" 

Requete (#R23)
--------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~
:: info:

	Synchro_MOADR

On met a jour la modification d'adresse en indiquant al date de modification

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE clients 								-- Mise à jour de la table clients
	SET    datemodificationfiche = Getdate(), 	-- La date de modification de la fiche est la date du jour
	       opmodif = 99 						-- L'id de l'opération de modificatione est '99'
	       SET%%
	WHERE  codeclient = "" 						-- Pour les ligne dont le code client est égale au code p de la vue en cours

Requete (#R24)
--------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~
:: info:

	Maj_TopageCa

On met a jour le ca du clinet

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE topageca 				-- Mise à jour de la table 'topageca'
	SET    dateretour = Getdate(), 	-- La date de retour est égale à la date du jour
	       idmagellan = "" 			-- L'identifiant magellan est égale à (?)
	WHERE  codeclient = "" 			-- Pour les lignes dont le code client est égale à (?)
	       AND idmagellan IS NULL 	-- Le l'identifiant magellan est égale a null

Requete (#R25)
--------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~
:: info:

	Is_Wait_TopageCa

Deja fait

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	SELECT Count(*) 				-- On compte le nombre de ligne
	FROM   topageca 				-- Dans la table 'topageca'
	WHERE  codeclient = "" 			-- Pour les lignes dont le code client est égale au code p de la vue en cours
	       AND idmagellan IS NULL 	-- Et dont l'id magellan est null

Requete (#R26)
--------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~
:: info:

	Client_ToperDateEditionCarte

On met a jour la date d'edition de la carte du client

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE cmpasso 					-- Mise à jour de la table 'cmpasso'
	SET    dateeditioncarte = '' 	-- La date d'edition de la carte est égale au parametre d
	WHERE  codeclient = "" 			-- Pour les lignes dont le code client est égale au code p de la vue en cours

Requete (#R27)
--------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~
:: info:

	Demander_CA

Quand on demande le ca d'un client on informe la table topageca de la date de la requete avec pour référence
Le code client et l'identifiant magelan

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	INSERT INTO topageca 			-- On insert un enregistrement dans la table topageca
	            (codeclient, 		-- Le code client
	             daterequete, 		-- La date de al requete
	             idmagellan) 		-- L'identifiant magelan
	VALUES      ("", 				-- Le code client est égale au parametre cc (cc = code client)
	             Getdate(), 		-- La date de la requete est égale à la date du jour
	             "") 				-- L'identifiant magellan est égale au caparemtre cpt (cpt est égale a l'id magelan)

Requete (#R28)
--------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~
:: info:

	MAJ_Adr_Fiche_Client

Mise à jour de l'adresse du client en changean la date de al modification de la fiche par la date du jour de l'action

On ne peut modifier qu'un champ a la fois

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

		-- On ne modifie qu'un champ a la fois avec cette requete
		
		UPDATE clients 									-- Mise à jour de la table 'clients'
		SET    datemodificationfiche = Getdate(), 		-- La date de modification de la fiche est égale à la date du jours
		       opmodif = 99, 							-- L'identifiant d'opération de la modif est égale à 99
		       champ = valeur 							-- Le champ précisé est égale à la valeur précisé
		WHERE  codeclient = "" 							-- Pour les lignes dont le code client est égale à (?) La référence situation est définir comme 'RAZ Mage.'
		
		UPDATE clients 
		SET    datemodificationfiche = Getdate(), 
		       opmodif = 99 
		       SET%%
		WHERE  codeclient = "" 

Requete (#R29)
--------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~
:: info:

	Toper_Fiche_Pour_Changement_club

Ici on insere les topage pour ls changement de club et ainsi conserver un historique

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	INSERT INTO topages 	-- On insert un enregistrement dans la table 'topages'
	VALUES     ("", 		-- LE code p est égale au code p de la vue en cours
	            'EdtFCA',   -- Le champ (?) est égale à 'EdtFCA'
	            Getdate(), 	-- Le champ (?) est égale à la date du jours
	            "", 		-- Le numéro de club est égale au numéro de club de la vue en cours
	            1) 			-- Le champ (?) est égale à 1

Requete (#R30)
--------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~
:: info:

	MenuItem7_Click

Suppression du code p d'une ligne dans magellan dont le compteur est égale au compteur de la vue en cours

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE magellan 		-- Mise à jour de la table 'magellan'
	SET    code_p = NULL 	-- Le code p est null
	WHERE  compteur = "" 	-- Pour les ligne dont le compteur est égale au compteur de la vue en cours

Requete (#R31)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~
:: info:

	Toper_ligne_Anomalie

Ici on insert les anomalie magellan quand il y en as en reseignent le code anomalie et le numéro de compteur

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	INSERT INTO magellan_anomalie 	-- On insere un enregistrement dans la table 'magellan_anomalie'
	VALUES      ("", 				-- Le compteur est égale au compteur de la vue en cours
	             "") 	Détails de la requete:Détails de la requete:			-- Le code anomalie est égale au paramètre Code_Anomalie

Requete (#R32)
--------------

Détails de la requete :
~~~~~~~~~~~~~~~~~~~~~~~

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

MAJ_DateSynchroCotiDétails de la requete:

Cette requete n'est pas vraiment une requete il faudras songé à la déplacée 
(?)

Schematique de la requeDétails de la requete:te : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	jdf_Magellan_Upd_dateSynchro


Requete (#R33)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

MenuItem4_Click
Détails de la requete:
Suppression d'une anomalie magellan

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::
	
	delete 							-- Supprime les enregistrement 
		from Magellan_anomalie 		-- de la table 'Magellan_anomalie'
		where     					-- Pour les ligne dont..
			compteur = ''			-- Le compteur est égale au compteur de la vue en cours


				