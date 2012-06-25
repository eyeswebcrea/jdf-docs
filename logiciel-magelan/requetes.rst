Les des requetes Requete :
=========

Me.SqlSelectCommand1
--------------------

Resumée de la requête:
~~~~~~~~~~~~~~~~~~~~~~

Schematique de la requete : 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Détails de la requete:
~~~~~~~~~~~~~~~~~~~~~~

::

	-- m = Table magellan (Information de magelan)
	-- c = Table client (Information de jdf)
	
	SELECT -- Selectionne les champs de la table 'client' (JDF) et leur applique un alias
		c.Nom AS JDF_Nom,		-- Champ nom 
		c.Prenom AS JDF_Prenom, -- Champ prenom
		c.Adresse2 AS JDF_Adr2, -- Champ adresse 2
		c.Adresse3 AS JDF_adr3, -- Champ adresse 3
		c.CodePostal AS JDF_cp, -- Champ adresse 4
		c.Ville AS JDF_Ville,
			CASE WHEN raisoc IS NULL THEN                    	-- 			Si raisoc (Raison sociale) est nul alors ...
				CASE WHEN difference(m.nom, c.nom) > 2       	-- 					Si le nom est différent entre magelan et jdf  ...
					  AND 							 		 	-- 					ET
				      (									     	-- 					(   
				     	(m.prenom IS NULL OR m.prenom = '')  	-- 						(Si le champ nom de magelan est null ou vide) 
				     	AND 							 	 	-- 						ET
				      	(c.prenom IS NULL OR c.prenom = '')  	-- 						(Si Le champ prenom de magelan est nul ou vide)
				      ) 									 	-- 					)
				      OR 									 	-- 					OU
				      difference(m.prenom, c.prenom) > 2 THEN   -- 					Si le champ prenom est différent entre magelan et jdf
			    'O' ELSE 'N'									-- 						Alors O sinon N 
			    END												-- 					Fin 		
				ELSE CASE WHEN difference(m.raisoc, ltrim(c.adresse0)) > 2  --      Sinon si 
					AND
					(
						(charindex(m.nom, c.nomsociete) > 0 OR difference(m.nom, c.nom) > 2)
						OR 
						m.nom IS NULL
					) 
					THEN 'O' ELSE 'N' 
				END 
			END AS ctrl_nom,
			CASE WHEN c.codepostal = m.cp THEN 
			'O' ELSE 'N' 
			END AS ctrl_CP,
			CASE WHEN (
						 soundex(dbo.fn_Dmot(c.adresse2)) IN
						 (
						 	soundex(dbo.fn_Dmot(m.adr1)),
						  	soundex(dbo.fn_Dmot(m.adr2)),
						  	soundex(dbo.fn_Dmot(m.adr3)),
						  	soundex(dbo.fn_Dmot(m.adr4))
						 ) 
						 OR
						 (
						 	c.adresse2 IS NULL OR  ltrim(c.adresse2) = ''
					 	 )
					   ) 
					   AND 
					   (
					      soundex(dbo.fn_Dmot(c.adresse3)) IN 
					   (
					   	  soundex(dbo.fn_Dmot(m.adr1)),
					   	  soundex(dbo.fn_Dmot(m.adr2)),
					   	  soundex(dbo.fn_Dmot(m.adr3)),
					   	  soundex(dbo.fn_Dmot(m.adr4))
					   ) 
					   OR
					   (
					   	  c.adresse3 IS NULL OR ltrim(c.adresse3) = '')) THEN
					   	  'O' ELSE 'N' 
					   	  END AS 
					   	  		ctrl_adr,
					   	  		m.Code_R,
					   	  		m.Code_P,
					   	  		m.Code_Action,
					   	  		RTRIM(m.Titre) AS titre,
					   	  		m.Mnt_Offre,
					   	  		m.Duree,
					   	  		m.mnt_Reg,
					   	  		m.regle,
					   	  		m.Ech_deb,
					   	  		m.Ech_fin,
					   	  		m.Tirage_deb,
					   	  		m.Tirage_Fin,
					   	  		m.Date_evt,
					   	  		m.Raisoc,
					   	  		m.civ,
					   	  		m.Nom,
					   	  		m.Prenom,
					   	  		m.Adr1,
					   	  		m.Adr2,
					   	  		m.Adr3,
					   	  		m.Adr4,
					   	  		m.CP,
					   	  		m.Ville,
					   	  		m.pays,
					   	  		m.ZIP_Code,
					   	  		m.Date_adresse,
					   	  		m.Telephone,
					   	  		m.Email,
					   	  		m.Motif_Ann,
					   	  		m.Motif_Stop_Rel,
					   	  		RTRIM(m.Sous_type_tiers) AS	Sous_type_tiers,
					   	  		m.synchro,
					   	  		c.email AS JDF_email,
					   	  		CmpAsso.datedemADH,
					   	  		CmpAsso.datedemclubiste,
					   	  		c.club,
					   	  		ISNULL(CmpAsso.ISADH, 0) AS ISADH,
					   	  		CmpAsso.IsClubiste,
					   	  		CmpAsso.Situation,
					   	  		CmpAsso.RefSituation,
					   	  		CmpAsso.DateSituation,
					   	  		CmpAsso.DateEditionCarte,
					   	  		CmpAsso.IsCL,
					   	  		c.Adresse1 AS JDF_Adr1,
					   	  		c.nomsociete AS JDF_Cmpnom,
					   	  		c.telephone AS JDF_Tel,
					   	  		c.type,
					   	  		c.societe AS JDF_Societe,
					   	  		c.adresse0 AS JDF_Adr0,
					   	  		c.Titre AS JDF_titre, 
					   	  		c.CodeClient AS JDF_CC, 
					   	  		m.compteur, 
					   	  		c.DateModificationFiche, 
					   	  		c.Origine,
					   	  		c.CodeRustica AS JDF_CODER,
					   	  		CmpAsso.optDistrib,
					   	  		c.Pays AS Jdf_pays,
					   	  		CmpAsso.datenomADH,
					   	  		CmpAsso.Situation_APR,
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