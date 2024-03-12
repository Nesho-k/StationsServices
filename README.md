# Meilleures Stations Services

## Table des Matières


### Source des données 

Les données utilsées pour cette analyse est le fichier "prix-des-carburants-en-france-flux-instantane.csv" contenant des informations sur : la ville où se situe les stations services, le code postal, le département, la région, le prix de chaque carburant, le nombre de carburant disponible, les services proposés et les horaires détaillées.   

### Aperçu du projet 

L'objectif de ce projet est de déterminer dans quel département se trouve les meilleurs stations services. Tout d'abord, il faut définir le terme "meilleurs stations services". Pour cela, nous allons nous baser sur 2 critères : le prix du carburant et la disponibilité des carburants dans chaque stations servoices en d'autres termes le nombre de carburants mis à disposition.   


### Logiciels 

 - SQL : Data Cleaning et Data Exploration


### Data Cleaning 

Dans cette phase, nous allons supprimer les colonnes inutiles qui ne serviront pas à notre analyse. 

```
alter table prix_carburant 
DROP COLUMN Adresse, horaires, services, prix, rupture

alter table prix_carburant
DROP COLUMN [Prix Gazole mis à jour le],
			[Prix SP95 mis à jour le],
			[Prix SP98 mis à jour le],
			[Prix E10 mis à jour le],
			[Prix E85 mis à jour le],
			[Prix GPLc mis à jour le],
			[latitude],
			[longitude],
			[pop],
			[geom]

alter table prix_carburant
DROP COLUMN [Début rupture e10 (si temporaire)],
			[Début rupture sp98 (si temporaire)],
			[Début rupture sp95 (si temporaire)],
			[Début rupture e85 (si temporaire)],
			[Début rupture GPLc (si temporaire)],
			[Début rupture Gazole (si temporaire)]


select * from prix_carburant
```

### Missing Data 

Pour déterminer quel département possède les meileurs stations services, il faut dans un premier temps déterminer quel région possède les meilleurs stations services avec les crotères que nosu avons établi auparavant. Cependant, au cours de notre analyse nous nous sommes rendus compte que la région Corse ne possède pas de SP98 et de E85. Pour faciliter l'analyse, nous n'allons pas prendre en compte la région Corse : il n'y aura donc que 12 régions. 


### Data Exploration 

Dans cette partie, nous allons répondre aux questions : 
 - Quel région possède les meilleurs stations services ?
 - Quel département possède les meilleurs stations services ?

Pour ce faire nous allons nous baser sur 2 critères. Le premier est le prix du carburant : pour chaque carburant nous allons effectuer un classement des régions où le carburant est en moyenne le moins cher. Enfin, nous effecturons un classement final où le rang de chaque région est la somme des rangs obtenus pour chaque région. 
Le 2e critère est la disponiblité des carburants. Pour ce critère, nous allons également effectuer un classement où pour chaque région va être associé le nombre moyen de carburant mis à disposition.







