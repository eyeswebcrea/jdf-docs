Data Mapping
============

::

'da_analyse_anomalie
        '
        Me.da_analyse_anomalie.SelectCommand = Me.SqlCommand2
        
        Me.da_analyse_anomalie.TableMappings.AddRange(New System.Data.Common.DataTableMapping() {New System.Data.Common.DataTableMapping("Table", "Magellan", New System.Data.Common.DataColumnMapping() {
	        New System.Data.Common.DataColumnMapping("JDF_Nom", "JDF_Nom"),
	        New System.Data.Common.DataColumnMapping("JDF_Prenom", "JDF_Prenom"),
	        New System.Data.Common.DataColumnMapping("JDF_Adr2", "JDF_Adr2"),
	        New System.Data.Common.DataColumnMapping("JDF_adr3", "JDF_adr3"),
	        New System.Data.Common.DataColumnMapping("JDF_cp", "JDF_cp"),
	        New System.Data.Common.DataColumnMapping("JDF_Ville", "JDF_Ville"),
	        New System.Data.Common.DataColumnMapping("ctrl_nom", "ctrl_nom"),
	        New System.Data.Common.DataColumnMapping("ctrl_CP", "ctrl_CP"),
	        New System.Data.Common.DataColumnMapping("ctrl_adr", "ctrl_adr"),
	        New System.Data.Common.DataColumnMapping("Code_R", "Code_R"),
	        New System.Data.Common.DataColumnMapping("Code_P", "Code_P"),
	        New System.Data.Common.DataColumnMapping("Code_Action", "Code_Action"),
	        New System.Data.Common.DataColumnMapping("Titre", "Titre"),
	        New System.Data.Common.DataColumnMapping("Mnt_Offre", "Mnt_Offre"),
	        New System.Data.Common.DataColumnMapping("Duree", "Duree"),
	        New System.Data.Common.DataColumnMapping("mnt_Reg", "mnt_Reg"),
	        New System.Data.Common.DataColumnMapping("regle", "regle"),
	        New System.Data.Common.DataColumnMapping("Ech_deb", "Ech_deb"),
	        New System.Data.Common.DataColumnMapping("Ech_fin", "Ech_fin"),
	        New System.Data.Common.DataColumnMapping("Tirage_deb", "Tirage_deb"),
	        New System.Data.Common.DataColumnMapping("Tirage_Fin", "Tirage_Fin"),
	        New System.Data.Common.DataColumnMapping("Date_evt", "Date_evt"),
	        New System.Data.Common.DataColumnMapping("Raisoc", "Raisoc"),
	        New System.Data.Common.DataColumnMapping("civ", "civ"),
	        New System.Data.Common.DataColumnMapping("Nom", "Nom"),
	        New System.Data.Common.DataColumnMapping("Prenom", "Prenom"),
	        New System.Data.Common.DataColumnMapping("Adr1", "Adr1"),
	        New System.Data.Common.DataColumnMapping("Adr2", "Adr2"),
	        New System.Data.Common.DataColumnMapping("Adr3", "Adr3"),
	        New System.Data.Common.DataColumnMapping("Adr4", "Adr4"),
	        New System.Data.Common.DataColumnMapping("CP", "CP"),
	        New System.Data.Common.DataColumnMapping("Ville", "Ville"),
	        New System.Data.Common.DataColumnMapping("pays", "pays"),
	        New System.Data.Common.DataColumnMapping("ZIP_Code", "ZIP_Code"),
	        New System.Data.Common.DataColumnMapping("Date_adresse", "Date_adresse"),
	        New System.Data.Common.DataColumnMapping("Telephone", "Telephone"),
	        New System.Data.Common.DataColumnMapping("Email", "Email"),
	        New System.Data.Common.DataColumnMapping("Motif_Ann", "Motif_Ann"),
	        New System.Data.Common.DataColumnMapping("Motif_Stop_Rel", "Motif_Stop_Rel"),
	        New System.Data.Common.DataColumnMapping("Sous_type_tiers", "Sous_type_tiers"),
	        New System.Data.Common.DataColumnMapping("synchro", "synchro")
        })})