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

Pour ce faire nous allons nous baser sur 2 critères. Le premier est le prix du carburant : **pour chaque carburant** nous allons effectuer un classement des régions où le carburant est en moyenne le moins cher. Ensuite, nous effecturons un classement final où le rang de chaque région est la somme des rangs obtenus pour chaque région. 
Le 2e critère est la disponiblité des carburants. Pour ce critère, nous allons également effectuer un classement où pour chaque région va être associé le nombre moyen de carburant mis à disposition.

Enfin, une fois avoir trouver la meilleur région nous allons réitérer les étapes au niveau départementale. 


### Quel est la meilleure région ?
#### 1. Carburant le moins cher 

Pour chaque carburant nous allons effectuer un classement des régions où le carburant est le moins cher, par exemple pour le Gazole : 

```
SELECT Région, AVG([Prix Gazole]) AS AvgGazole, ROW_NUMBER() OVER (ORDER BY AVG([Prix Gazole]) ASC) AS rn1
FROM prix_carburant
where Région is not null and Région <> 'Corse'
GROUP BY Région
```

|Région	|AvgGazole	|rn1|
|-------|---------------|---|
|Bretagne|	1,77772577696527|	1|
|Pays de la Loire|	1,78727372262774|	2|
|Normandie	|1,79283514492754	|3|
|Hauts-de-France	|1,79333079847909|	4|
|Nouvelle-Aquitaine	|1,79769216921692|	5|
|Occitanie	|1,80071178637201|	6|
|Centre-Val de Loire	|1,80432805429864	|7|
|Auvergne-Rhône-Alpes	|1,81098417483044	|8|
|Provence-Alpes-Côte d'Azur|	1,81108618504436	|9|
|Grand Est	|1,81436395759717|	10|
|Bourgogne-Franche-Comté	|1,82310169491525	|11|
|Île-de-France|	1,86778004807692|	12|

La Bretagne est la région où le Gazole est en moyenne le moins cher. Une fois avoir établi un classement pour chaque carburant, nous les regroupons dans un CTE qui lui même est contenu dans une tableau temporaire.
Enfin, un classement final est effectué où pour chaque région est associé la somme des rangs obtenus dans les classements auparavant (voir exemple après le code)


```
WITH CTE1 AS (
SELECT Région, AVG([Prix Gazole]) AS AvgGazole, ROW_NUMBER() OVER (ORDER BY AVG([Prix Gazole]) ASC) AS rn1
FROM prix_carburant
where Région is not null and Région <> 'Corse'
GROUP BY Région),

CTE2 AS (
SELECT Région, AVG([Prix SP95]) AS AvgGazole, ROW_NUMBER() OVER (ORDER BY AVG([Prix SP95]) ASC) AS rn2
FROM prix_carburant
where Région is not null and Région <> 'Corse'
GROUP BY Région),

CTE3 AS (
SELECT (Région), AVG([Prix SP98]) AS AvgGazole, ROW_NUMBER() OVER (ORDER BY AVG([Prix SP98]) ASC) AS rn3
FROM prix_carburant
where Région is not null and Région <> 'Corse'
GROUP BY Région),

CTE4 AS (
SELECT (Région), AVG([Prix E85]) AS AvgGazole, ROW_NUMBER() OVER (ORDER BY AVG([Prix E85]) ASC) AS rn4
FROM prix_carburant
where Région is not null and Région <> 'Corse'
GROUP BY Région),

CTE5 AS (
SELECT (Région), AVG([Prix GPLc]) AS AvgGazole, ROW_NUMBER() OVER (ORDER BY AVG([Prix GPLc]) ASC) AS rn5
FROM prix_carburant
where Région is not null and Région <> 'Corse'
GROUP BY Région),

CTE6 AS (
SELECT (Région), AVG([Prix E10]) AS AvgGazole, ROW_NUMBER() OVER (ORDER BY AVG([Prix E10]) ASC) AS rn6
FROM prix_carburant
where Région is not null and Région <> 'Corse'
GROUP BY Région)

SELECT Région AS RG, SUM(rn1 + rn2 + rn3 + rn4 + rn5 + rn6) AS Total_Rank, 
			   ROW_NUMBER() OVER (ORDER BY SUM(rn1 + rn2 + rn3 + rn4 + rn5 + rn6)) AS RN1
			   INTO #MoinsCher
FROM (
  SELECT CTE1.Région, rn1, rn2, rn3, rn4, rn5, rn6
	FROM CTE1
    JOIN CTE2 ON CTE1.Région = CTE2.Région
    JOIN CTE3 ON CTE1.Région = CTE3.Région
    JOIN CTE4 ON CTE1.Région = CTE4.Région
    JOIN CTE5 ON CTE1.Région = CTE5.Région
    JOIN CTE6 ON CTE1.Région = CTE6.Région
) AS AllRanks
GROUP BY Région


SELECT * FROM #MoinsCher
```

|RG (Région)|	Total_Rank|	RN1|
|-----------|-------------|--------|
|Bretagne|	8|	1|
|Pays de la Loire|	14|	2|
|Normandie|	23	|3|
|Nouvelle-Aquitaine|	26|	4|
|Hauts-de-France	|31	|5|
|Centre-Val de Loire	|37|	6|
|Grand Est	|41	|7|
|Occitanie	|49|	8|
|Provence-Alpes-Côte d'Azur|	53|	9|
|Bourgogne-Franche-Comté	|59	|10|
|Auvergne-Rhône-Alpes|	62	|11|
|Île-de-France	|65	|12|







