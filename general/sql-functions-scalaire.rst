===================
Fonctions scalaires
===================

dbo.fn_Dmot

Cette fonction assez compliquée ne fait au final que récuperer le dernier mot

Alternative

SET @rue = RTRIM(@rue)
SET @rue = RIGHT(@rue, charindex(' ', REVERSE(@rue))) 

crée findOcuratedPositionInStringToRight => charindex(' ', REVERSE(@value))
crée getSubStringToRight => RIGHT(@value, @nombre_caractere)

Todo : Penser à créer un alias getLastWorldInString pour avoir une fonction plus explicite sans casser le système actuel et faire en sorte que dMot apelle l'alias

::

	USE [JDF2003] 																							-- On utilise la base de donée JDF2003
	GO																										-- C'est parti
	/****** Object:  UserDefinedFunction [dbo].[fn_Dmot]    Script Date: 06/26/2012 17:18:21 ******/
	SET ANSI_NULLS OFF																						-- Spécifie le comportement, compatible avec ISO, des opérateurs Égal à (=) et Différent de (<>), lorsqu'ils sont utilisés avec des valeurs Null.
	GO																										-- C'est parti
	SET QUOTED_IDENTIFIER ON																				-- Force SQL Server à suivre les règles ISO se rapportant aux guillemets qui délimitent les identificateurs et les chaînes littérales.
	GO
	ALTER FUNCTION [dbo].[fn_Dmot](@rue as varchar(38))														-- Modifie la table des fonctions sql pour ajouter la fonction dbo.fn_Dmot avec comme paramêtre rue qui est une varchar(3)
	
	
	RETURNS varchar(38) AS  -- On déclare que la fonction retourne une valeur varchar(38)
	
	BEGIN 																									-- Ici le contenu de la fonction commence
	declare @l varchar(38) 																					-- On déclare la variable l comme varchar(38)
	select @l=case when (charindex(' ',rtrim(@rue))=0 and len(@rue)>0) then									-- Si la variable rue n'est pas vide et qu'el ne contient plus d'espace blanc sur la droite après les avoir supprimés alors
			rtrim(@rue) 																					-- On retourne la variable avec les espaces blancs sur la droite tronqué
			else 																							-- Sinon
			ltrim(reverse(substring(reverse(rtrim(@rue)),0,charindex(' ',reverse(rtrim(@rue))))) )			-- On Recupère le dernier mot de la chaine
		end		
	return @l
	END																										-- Ici le contenu de la fonction se termine
	
	
	"      bonjour   "
	"             "
	"roujnob      "
	"roujnob"
	"bonjour"
	"bonjour"