Les requetes :
==============

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

La requête sélectionne le nom, prenom , adresse, code postal, ville de la table Client (Jdf), la table CmpAsso (Jdf) et la table magelan (magelan)

Elle alligne les lignes de ces tables avec les conditions suivantes :

m.Code_P = c.CodeClient , c.CodeClient = CmpAsso.codeclient

Elle ne prend que les lignes dont les champs suivant ne ressemble pas phonétiquement entre ceux de magelan et jdf

Il y a un champ dont elle ne prend pas toutes la valeur phonétique mais seulement le dernier mot grace a la fonction scalaire Dmot

Si la n'est pas déja synchronisé avec jdf (m.synchro) et n'est pas dans les anomalie on l'identifie si le compteur est dans les anomalies

Des ligne dont un des champs suivant :

	- nom
	- prenom
	- raison sociale
	- nom societe
	

de la table magelan sont en duplicata dans la table client 

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
					   	  		m.Date_adresse,													--		Sélection de la date de changement de l'adresse éffectué par magelan (magelan)
					   	  		m.Telephone,													--		Sélection du numéro de téléphone (magelan)
					   	  		m.Email,														--		Sélection de l'email (magelan)
					   	  		m.Motif_Ann,													--		Sélection du motif d'annulation (magelan)
					   	  		m.Motif_Stop_Rel,												--		Sélection du motif ... (magelan)
					   	  		RTRIM(m.Sous_type_tiers) AS	Sous_type_tiers,					--		Sélection du sous type tiers en supprimant les espace de droite avec pour alias Sous_type_tiers
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
							SELECT compteur FROM magellan_anomalie								--				les compteur considerer comme anomalie (magelan)
						)																		--			)
				) 																				--		)
				AND 																			--		Et
				(m.Ech_fin IS NOT NULL) 														--		L'écheance de fin de magelan est null (magelan)
				AND 																			--		Et
				(m.compteur BETWEEN @compteur_dep AND @compteur_fin)							--		Le compteur se trouve entre le compteur début et fin spécifier (magelan)
				
				