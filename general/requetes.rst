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

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	SELECT m.compteur 											-- Compteur
	FROM   magellan m 											-- Dans la table magelan avec pour alias m 
	       LEFT OUTER JOIN clients c 							-- Joint à la table client avec pour alias c
	                    ON m.code_p = c.codeclient 				-- On aligne le champ code client de la table ``client`` au code p de la table ``magelan``
	       LEFT OUTER JOIN cmpasso 								-- Joint à la table cmpasso 
	                    ON c.codeclient = cmpasso.codeclient 	-- On aligne le champ code client de la table ``client`` au code client de la table ``cmpasso``
	WHERE  ( m.synchro = 0 ) 									-- Quand la ligne n'est pas encore dispatché en base
	       AND ( NOT ( m.code_p IS NULL ) ) 					-- Et que le code p n'est pas nul
	       AND ( m.compteur NOT IN (SELECT compteur 			-- Ainsi que le compteur ne se trouve pas dans les anomalie
	                                FROM   magellan_anomalie) ) -- De la table ``magellan_anomalie``
	       AND ( m.ech_fin IS NOT NULL ) 						-- Et que le champ ech_fin n'est pas nulj'ai
       
        
Requete (#R5)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

:: info:
	Magellan_Affecter_Code_Client


Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~
*
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


Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~   

::

	UPDATE magellan 
	SET    code_p = NULL 
	WHERE  compteur = "" 


Requete (#R7)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~     

::

	update CmpAsso  
			set
	            datedemadh=''
			    datenomadh='', -- [IIF]
	            optdistrib=1 ou optdistrib=0 -- [IIF]
	            isadh=1,
	            situation = '',
	            refsituation=''
	        where 
	        codeclient=
        
Requete (#R8)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

   select 
   		code_r,		   -- Le code R   
   		ech_fin 	   -- L'écheance début
   		from magellan  -- sur la table magelan
   where 			   -- Quand...
   		compteur = 	   -- Le compteur est égale au parametre compteur

Requete (#R9)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

Modifier_CmpAsso_ADH_DEMISSION

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~
       
::

	UPDATE cmpasso 
	SET    datedemadh = Getdate(), 
	       isadh = 0, 
	       situation = 'X', 
	       refsituation = 'A:' 
	WHERE  codeclient = "" 
        
Requete (#R10)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

Modifier_CmpAsso_ADR_DEMISSION

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE cmpasso 
	SET    datedemclubiste = Getdate(), 
	       isclubiste = 0, 
	       situation_apr = 'X', 
	       refsituation_apr = '> Magellan' 
	WHERE  codeclient = "" 


Requete (#R11)
--------------

Resumé de la requete :
~~~~~~~~~~~~~~~~~~~~~~

Modifier_CompAsso_Clubiste

Schematique de la requete :
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE cmpasso 
	SET    datedemclubiste = '', 
	       datenomclubiste = '', 
	       isclubiste = 1, 
	       situation = '', 
	       refsituation = '' 
	WHERE  codeclient = "" 


Requete (#R12)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

Modifier_CmpAsso_Clubiste

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE cmpasso 
	SET    datedemclubiste = '', 
	       datenomclubiste = Isnull(datenomclubiste, ''), 
	       isclubiste = 1, 
	       situation_apr = '', 
	       refsituation_apr = '', 
	       datenomadh = NULL, 
	       datedemadh = NULL, 
	       situation = NULL, 
	       refsituation = 'RAZ Mage.' 
	WHERE  codeclient = "p" 

Requete (#R13)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

Modifier_CmpAsso_Exper_Clubiste

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE cmpasso 
	SET    datedemclubiste = '', 
	       datenomclubiste = Isnull(datenomclubiste, ''), 
	       isclubiste = 1, 
	       situation_apr = '', 
	       refsituation_apr = '', 
	       datenomadh = NULL, 
	       datedemadh = NULL, 
	       situation = NULL, 
	       refsituation = 'RAZ Mage.' 
	WHERE  codeclient = "" 

Requete (#R14)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

	Modifier_CmpAsso_abonnepur

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE cmpasso 
	SET    datedemabo = '', 
	       datenomabo = '', 
	       isabo = 1, 
	       situation = '', 
	       refsituation = '' 
	WHERE  codeclient = "" 

Requete (#R15)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

Modifier_CmpAsso_abonnePur_Demission

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE cmpasso 
	SET    datedemabo = Getdate(), 
	       isabo = 0, 
	       situation = 'X', 
	       refsituation = 'Magellan:000' 
	WHERE  codeclient = "" 

Requete (#R16)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

Suspens un compte utilisateur pas son code client
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
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

Modifier_CmpAsso_ADH

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	update CmpAsso set datedemadh='' -- On met a jour la table CmpAsso avec le champ datedemadh (équivalent de ech_fin)
				datenomadh='' 	-- [OPTIONELLE] Date dmh = ech_fin
	            optdistrib=1 optdistrib=0 -- (?)
	            isadh=1, 		-- Definir que le client est un adhérent
	            situation = '',
	            refsituation=':'
			where 				-- Pour les lignes ou ..
				codeclient="" 	-- Le code client est égale au parametre code client

Requete (#R12)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

	IsAnnuCorrespondSynchroEnCours

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

	

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	SELECT code_r, 
	       ech_fin 
	FROM   magellan 
	WHERE  compteur = "" 

Requete (#R13)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

Modifier_CmpAsso_ADH_DEMISSION

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE cmpasso 
	SET    datedemadh = Getdate(), 
	       isadh = 0, 
	       situation = 'X', 
	       refsituation = 'A:0001' 
	WHERE  codeclient = "" 


Requete (#R14)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

Modifier_CmpAsso_APR_DEMISSION

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE cmpasso 
	SET    datedemclubiste = Getdate(), 
	       isclubiste = 0, 
	       situation_apr = 'X', 
	       refsituation_apr = '> Magellan' 
	WHERE  codeclient = "" 

Requete (#R15)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

Modifier_CmpAsso_Clubiste

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE cmpasso 
	SET    datedemclubiste = '', 
	       datenomclubiste = '', 
	       isclubiste = 1, 
	       codes <> "" And codes <> "*",
	       situation = '', 
	       refsituation = ':' 
	WHERE  codeclient = "" 


Requete (#R16)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

Modifier_CmpAsso_Exper_Clubiste

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE cmpasso 
	SET    datedemclubiste = '', 
	       datenomclubiste = Isnull(datenomclubiste, ''), 
	       isclubiste = 1, 
	       situation_apr = '', 
	       refsituation_apr = '' 
	WHERE  codeclient = "" 

Requete (#R17)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

Modifier_CmpAsso_abonnePur

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE cmpasso 
	SET    datedemabo = '', 
	       datenomabo = '', 
	       isabo = 1 

Requete (#R18)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

Modifier_CmpAsso_abonnePur_Demission


Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE cmpasso 
	SET    datedemabo = Getdate(), 
	       isabo = 0, 
	       situation = 'X', 
	       refsituation = 'Magellan:' 
	WHERE  codeclient = "" 

Requete (#R19)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

Client_Changer_de_Club


Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE clients 
	SET    coderustica = '' 
	WHERE  codeclient = "" 

Requete (#R20)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

Maj_TopageCa

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE topageca 
	SET    dateretour = Getdate(), 
	       idmagellan = "" 
	WHERE  codeclient = "" 
	       AND idmagellan IS NULL 

Requete (#R21)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

Is_Wait_TopageCa

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	SELECT Count(*) 
	FROM   topageca 
	WHERE  codeclient = "" 
	       AND idmagellan IS NULL 

Requete (#R22)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

Synchro_Email_Tel

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE clients 
	SET    datemodificationfiche = Getdate(), 
	       opmodif = 99 
	       SET%%
	WHERE  codeclient = "" 
	
	UPDATE clients 
	SET    datemodificationfiche = Getdate(), 
	       opmodif = 99 
	       SET%%
	WHERE  codeclient = "" 

Requete (#R23)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

Synchro_MOADR

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE clients 
	SET    datemodificationfiche = Getdate(), 
	       opmodif = 99 
	       SET%%
	WHERE  codeclient = "" 

Requete (#R24)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

Maj_TopageCa

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE topageca 
	SET    dateretour = Getdate(), 
	       idmagellan = "" 
	WHERE  codeclient = "" 
	       AND idmagellan IS NULL 

Requete (#R25)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

Is_Wait_TopageCa

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	SELECT Count(*) 
	FROM   topageca 
	WHERE  codeclient = "" 
	       AND idmagellan IS NULL 

Requete (#R26)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

Client_ToperDateEditionCarte

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE cmpasso 
	SET    dateeditioncarte = '' 
	WHERE  codeclient = "" 

Requete (#R27)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

Demander_CA

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	INSERT INTO topageca 
	            (codeclient, 
	             daterequete, 
	             idmagellan) 
	VALUES      ("", 
	             Getdate(), 
	             "") 

Requete (#R28)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

MAJ_Adr_Fiche_Client

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE clients 
	SET    datemodificationfiche = Getdate(), 
	       opmodif = 99, 
	       champ = valeur 
	WHERE  codeclient = "" 
	
	UPDATE clients 
	SET    datemodificationfiche = Getdate(), 
	       opmodif = 99 
	       SET%%
	WHERE  codeclient = "" 

Requete (#R29)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

Toper_Fiche_Pour_Changement_club

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	INSERT INTO topages 
	VALUES     ("", 
	            'EdtFCA', 
	            Getdate(), 
	            "", 
	            1) 

Requete (#R30)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

MenuItem7_Click

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	UPDATE magellan 
	SET    code_p = NULL 
	WHERE  compteur = "" 

Requete (#R31)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

Toper_ligne_Anomalie

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	INSERT INTO magellan_anomalie 
	VALUES      ("", 
	             "") 

Requete (#R32)
-------------

Resumé de la requete : 
~~~~~~~~~~~~~~~~~~~~~~

MAJ_DateSynchroCoti

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

jdf_Magellan_Upd_dateSynchro

				