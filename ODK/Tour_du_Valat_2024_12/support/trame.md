## CATI GEDEOP / Formation ODK
Montpellier les 20 et 21 avril 2022

Par Mathieu Bossaert / Cen Occitanie

### Gouvernance du projet

#### GetODK Inc.
_ODK has become a large and vibrant open source project, depended on by millions of users for critical data gathering needs. As such, the project requires substantial work to maintain. This work is best performed performed in a steady, reliable fashion by a concrete, well-resourced entity. In recognition of these facts, Get ODK Inc., a corporation in the U.S. State of California, was formed to serve as the primary, day-to-day manager and steward of the project.
Get ODK Inc. strives to preserve ODK as a healthy and bona fide open source project and sustains its operations through ODK-related business activites (or otherwise)._

#### Core team
![l'équipe d'ODK Inc.](/home/mathieu/notes_md/images_odk/team.png)

#### Technical Advisory Board
The technical Advisory Board (TAB) represents the broader ODK community. It reviews and gives feedback on major roadmap decisions, new designs, specfications, features, and protocol changes.

#### Forum
[https://forum.getodk.org](https://forum.getodk.org)

### Présentation des fonctionnalités d'ODK Central
#### Projets / utilisateurs web / utilisateurs mobiles
* Utilisateurs web / rôles de projet
	* Les "administrateurs" sur l'ensemble du site sont automatiquement considérés comme gestionnaires de chaque projet.
  Les autres utilisateurs peuvent avoir des rôles spécifiques à ce projet
	* Les "gestionnaires de projet" peuvent exécuter toutes les tâches administratives liées à ce projet 
  et remplir des formulaires depuis leurs navigateurs.
	* Les "lecteurs de projet" ont accès aux données de tous les formulaires du projet et peuvent les télécharger 
  mais ne peuvent apporter aucunes modifications aux données ou paramètres.
	* Les "collecteurs de données" peuvent remplir des formulaires depuis leur navigateur 
  mais n'ont aucun accès aux données ou paramètres du projet
* Utilisateurs mobiles
	* -> les "rôles" qui pourront pourront utiliser ODK Collect. 1 QRCode / par utilisateur web
	* une matrice Formulaires / Utilisateurs mobiles attribue les droits d'accès aux formulaires du projet 
#### Versionnement des formulaires : ébauche vs/ version publiée
* création d'un nouveau formulaire
	* chargement du xlsform (xls) et des médias éventuels
	* test via un QRCode qui crée un projet spécifique sur Collect
	* ou test via enketo (quelques limitations)
	* on peut envoyer des données (volatiles) et vérifier leur contenu
* publication du formulaire si OK
* affectation du nouveau formulaire aux utilisateurs mobiles

#### Accès public via Enketo

#### Gestion des soumissions

#### Mise à jour des référentiels (pousser des médias sur les téléphones)
* via l'interface de Central
* création d'une nouvelle ébauche avec nouveau numéro de version
* ajout des médias
* test si modification d ela structure et publication
* en chargeant directement le média sur le téléphone (dossier medias du formulaire)
* Via l'API
* https://forum.getodk.org/t/posting-form-attachments-via-odk-central-api/26457/5
* en combinaison avec PostgreSQL :
* [showcase ici](https://forum.getodk.org/t/updating-external-media-files-for-select-questions-from-another-form-using-centrals-api/37295)

#### Authentification CAS / LDAP ?
* pas pour le moment. ODK-X semble proposer cela.
* Central intégrera à long terme des fonctionnalités d'import/export d'utilisateurs et réglages :
"Bulk upload-download user email address and permissions"   
Au CEN N-A, un utilitaire python de génération de QRCode pour les nouveaux utilisateurs a été développé.
[Ce que peut contenir le QRCode](https://forum.getodk.org/t/qrealtime-a-qgis-plug-in-for-odk/13809/46?u=mathieubossaert)

### Conception de formulaires

#### XLSForm
* [la documentation d'ODK](https://docs.getodk.org/xlsform/)
* [le site du standart xlsform](https://xlsform.org/en/#basic-format)
* [la "cheatsheet" de cartong](https://docs.google.com/spreadsheets/d/1AB2BNb2dsAOMJuRDjC-ii2-AeljDZ8TeGqAK6tLmZvk/edit?usp=sharing)

##### les onglets
* survey*
* choices*
* settings
##### les colonnes
* survey
    * type
    * name
    * label
    * hint
    * constraint
    * constraint_message
    * calculation
    * required
    * appearance
    * default
    * relevant
    * read_only
    * choice_filter
    * repeat_count
    * parameters
    * media::image
    * bind::odk:length
    * body::accuracyThreshold 
* choices
    * list_name
    * name
    * label
    * _colonne pour filtre_
* settings


##### les sélections simples et multiples
* les sélections en cascade
      -> choice_filter dépendant d'un réponse précédente
* afficher des attributs de l’objet sélectionné
##### les boucles
* itérer un certain nombre de fois (repeat_count)
    * valeur fixe/statique
    * valeur variable dépendant d'une saisie préalable
* itérer jusqu’à ce qu’une condition soit remplie
    * https://docs.getodk.org/form-logic/#repeating-as-long-as-a-condition-is-met

##### les groupes
* Nommage des itérations (repeat)
   * pourquoi ?
* Affichages conditionnels (relevant)

### Récupération des données

#### Présentation de Central2PG
* sur le forum d'ODK
    * [premiers essais de SQL](https://forum.getodk.org/t/sql-first-try-to-get-central-data-into-internal-postgis-database/30102)
    * [central2pg](https://forum.getodk.org/t/postgresql-set-of-functions-to-get-data-from-central/33350)
* sur Github
    * [central2pg](https://github.com/mathieubossaert/central2pg/tree/master)

##### Exemple d'utilisation

```sql
SELECT odk_central.odk_central_to_pg(
	'sig@cen-occitanie.org','fdfdsfdcfdfc','central.sicen.fr',5,
	'Sicen_2022', -- form ID
	'odk_central', -- schema where to creta tables and store data
	'point_auto_5,point_auto_10,point_auto_15,point,ligne,polygone' 
    --> columns to ignore in json transformation to database attributes (geojson fields of GeoWidgets)
);

REFRESH MATERIALIZED VIEW odk_central.donnees_formulaire_sicen_2022;
REFRESH MATERIALIZED VIEW odk_central.obs_pressions_menaces_jalons_2022;
REFRESH MATERIALIZED VIEW odk_central.donnees_habitat_formulaire_sicen_2022;

SELECT odk_central.get_file_from_central(
	'sig@cen-occitanie.org',
	'fdfdsfdcfdfc',
	'central.sicen.fr',
	5,                                        
	'Sicen_2022',
	submission_id,
	prise_image,
	'/home/postgres/medias_odk',
	lower(concat(public.unaccent(replace(user_name,' ','_')),'_',prise_image))
) FROM odk_central.donnees_formulaire_sicen_2022
WHERE prise_image IS NOT NULL;

SELECT odk_central.formulaire_sicen_2022_alimente_saisie_observation_especes();
SELECT odk_central.sicen_2022_alimente_obs_pressions_menaces_jalons();
SELECT odk_central.sicen_2022_alimente_saisie_observation_habitats();

```

#### R et ODK
"There are many useful R packages that connect to ODK for cleaning and visualizing data. Our current favorites are :"
 
  * [ruODK](https://github.com/ropensci/ruODK) for data access by Florian May
  * [checkR-ODK](https://github.com/mtyszler/checkR-ODK) for data cleaning by Marcelo Tyszler
  * [repvisforODK](https://github.com/SwissTPH/repvisforODK) for data visualization by Lucas Silbernagel

#### QGIS et ODK
  * https://forum.getodk.org/t/qrealtime-a-qgis-plug-in-for-odk/13809/46?u=mathieubossaert

#### Questions diverses :
