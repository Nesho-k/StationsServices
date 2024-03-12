# Meilleures Stations Services

## Table des Matières


### Source des données 

Les données utilsées pour cette analyse est le fichier "prix-des-carburants-en-france-flux-instantane.csv" contenant des informations sur : la ville où se situe les stations services, le code postal, le département, la région, le prix de chaque carburant (qui sont dans cette base de données le SP98, le SP95, le Gazole, le GPLc, le E10 et le E85), le nombre de carburant disponible, les services proposés et les horaires détaillées.   

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
SELECT Région, AVG([Prix Gazole]) AS AvgGazole,
       ROW_NUMBER() OVER (ORDER BY AVG([Prix Gazole]) ASC) AS rn1
FROM prix_carburant
WHERE Région is not null and Région <> 'Corse'
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

SELECT Région AS RG, SUM(rn1 + rn2 + rn3 + rn4 + rn5 + rn6) AS Sum_Rank, 
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

|RG (Région)|	Sum_Rank|	RN1|
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


Par exemple, la Bretagne est la 1er région la moins cher en ce qui concerne le SP98, le SP95, le Gazole, le E10 et le GPLc et la 3e région la moins cher en ce qui concerne le E85 donc 1+1+1+1+1+3 = 8 ce qui correspond au Sum_Rank
La colonne RN1 correspond au rang final. La Bretagne est donc la région où les carburants sont en moyenne les moins chers.



#### 2. Le nombre de carburants disponible

Dans cette partie, nous allons déterminer dans quel région y a-t-il en moyenne le plus de carburants disponilble. Pour ce faire nous allons utiliser la colonne "Carburants Disponibles". Le problème est que nous voulons savoir combien y-t-il de carburants disponibles et non pas quels carburants sont disponibles. Nous allons donc créer une nouvelle colonne indiquant le nombre de carburants disponible pour chaque stations services : 

```
SELECT [Carburants disponibles],  
	   LEN([Carburants Disponibles]) - LEN(REPLACE([Carburants Disponibles], ',', '')) + 1 AS NombreCarbuDispo
FROM prix_carburant
```

Les cinq première ligne du tableau sont : 

|id    | Carburants disponibles|	NombreCarbuDispo|
|------|---------------|------------------------|
|61400001		|Gazole, SP95, E85, GPLc, SP98|	5|
|31200020	|Gazole, E85, E10, SP98	|4|
|34070003	|Gazole, E10, SP98	|3|
|38300024	|Gazole, SP95	|2|
|82300005	|Gazole, E85, E10, SP98|	4|


Avant de continuer, nous allons vérifier que la somme entre les carburants disponibles et les carburants indsponibles est égale à 6 autrement dit qu'il y a bien que 6 carburants à chaque fois : 

```
SELECT [Carburants disponibles], [Carburants indisponibles]
FROM prix_carburant
WHERE LEN([Carburants Disponibles]) - LEN(REPLACE([Carburants Disponibles], ',', '')) + 1 + LEN([Carburants indisponibles]) - LEN(REPLACE([Carburants indisponibles], ',', '')) + 1 <> 6
```

Il y a aucune station service où le nombre de carburant (disponible + indisponible) est différent de 6

On crée ainsi la colonne "NombreCarbuDispo" : 

```
ALTER TABLE prix_carburant
ADD NombreCarbuDispo float

UPDATE prix_carburant
SET NombreCarbuDispo = LEN([Carburants Disponibles]) - LEN(REPLACE([Carburants Disponibles], ',', '')) + 1
```

On peut maintenant déterminer la région où il y en moyenne le plus de carburants disponibles : 

```
SELECT Région, AVG(NombreCarbuDispo) AS AvgDispo,
		ROW_NUMBER() OVER (ORDER BY AVG(NombreCarbuDispo) DESC) AS RN2
		INTO #BestDispo
FROM prix_carburant
WHERE Région is not null AND Région <> 'Corse'
GROUP BY Région 

SELECT * FROM #BestDispo
```

|Région	|AvgDispo|	RN2|
|-------|--------|---------|
|Île-de-France	|3,52251184834123|	1|
|Pays de la Loire	|3,52166064981949|	2|
|Provence-Alpes-Côte d'Azur|	3,52085967130215|	3
|Occitanie	|3,5160403299725	4|
|Centre-Val de Loire|	3,51569506726457	|5|
|Hauts-de-France|	3,50126262626263	|6|
|Bretagne	|3,48269581056466	|7|
|Nouvelle-Aquitaine|	3,47043010752688|	8|
|Grand Est	|3,44379391100703	|9|
|Normandie	|3,42086330935252	|10|
|Bourgogne-Franche-Comté|	3,3680981595092|	11|
|Auvergne-Rhône-Alpes	|3,36029962546816	|12|

C'est in Ile-de-France qu'il y a moyenne le plus de carburants disponible avec 3,522 carburants disponibles sur 6. La colonne RN2 correspond au rang final. 


**Pour rappel**, voici le classement des régions où le carburant est en moyenne le moins cher : 

|RG (Région)|	Sum_Rank|	RN1|
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


La dernière étape consiste à effectuer un classement final (le dernier), où pour chaque région est associé à la somme des rangs obtenus par les classement des carburants les moins cher et des disponibilité : 

```
Select RG, SUM(RN1 + RN2) AS MeilleureRégion
FROM #MoinsCher
JOIN #BestDispo ON #BestDispo.Région = #MoinsCher.RG
GROUP BY RG
ORDER BY 2
```

|RG (Région)|	MeilleureRégion (RN1 + RN2)|
|-----------|-----------------|
|Pays de la Loire	|4|
|Bretagne|	8|
|Centre-Val de Loire|	11|
|Hauts-de-France	|11|
|Nouvelle-Aquitaine|	12|
|Occitanie	|12|
|Provence-Alpes-Côte d'Azur|	12|
|Île-de-France	|13|
|Normandie	|13|
|Grand Est	|16|
|Bourgogne-Franche-Comté|	21|
|Auvergne-Rhône-Alpes|	23|

Ainsi, la région où se trouve les meilleurs stattions services est la région Pays de la Loire. Elle est la 2e région où l'on trouve les carburants les moins chers et la 2e région avec le plus de carburants disponibles : 2 + 2 = 4 ce qui correspond à la colonne MeilleureRégion. 



### Quel est le meilleur département ?

Maintenant que nous avons défini la région où se trouve les meilleures stations services, nous allons définir le département, au sein de cette région, où se trouve les meilleures stations services. En ce qui concerne le code, il suffit de rajouter la condition : "WHERE code_region = 52" car le code de la région Pays de la Loire est 52. 

```
SELECT DISTINCT(code_region)
FROM prix_carburant
WHERE Région = 'Pays de la Loire'
```

|code_region|
|--|
|52|

Ainsi pour la suite, nous n'allons pas détailler les explications. 

#### 1. Carburant le moins cher

```
WITH DCTE1 AS (
SELECT Département, AVG([Prix Gazole]) AS AvgGazole, ROW_NUMBER() OVER (ORDER BY AVG([Prix Gazole]) ASC) AS drn1
FROM prix_carburant
WHERE code_region = 52
GROUP BY Département),

DCTE2 AS (
SELECT Département, AVG([Prix Gazole]) AS AvgGazole, ROW_NUMBER() OVER (ORDER BY AVG([Prix Gazole]) ASC) AS drn2
FROM prix_carburant
WHERE code_region = 52
GROUP BY Département),

DCTE3 AS (
SELECT Département, AVG([Prix Gazole]) AS AvgGazole, ROW_NUMBER() OVER (ORDER BY AVG([Prix Gazole]) ASC) AS drn3
FROM prix_carburant
WHERE code_region = 52
GROUP BY Département),

DCTE4 AS (
SELECT Département, AVG([Prix Gazole]) AS AvgGazole, ROW_NUMBER() OVER (ORDER BY AVG([Prix Gazole]) ASC) AS drn4
FROM prix_carburant
WHERE code_region = 52
GROUP BY Département),

DCTE5 AS (
SELECT Département, AVG([Prix Gazole]) AS AvgGazole, ROW_NUMBER() OVER (ORDER BY AVG([Prix Gazole]) ASC) AS drn5
FROM prix_carburant
WHERE code_region = 52
GROUP BY Département),

DCTE6 AS (
SELECT Département, AVG([Prix Gazole]) AS AvgGazole, ROW_NUMBER() OVER (ORDER BY AVG([Prix Gazole]) ASC) AS drn6
FROM prix_carburant
WHERE code_region = 52
GROUP BY Département)

SELECT Département AS Déptmt, SUM(drn1 + drn2 + drn3 + drn4 + drn5 + drn6) AS Sum_Rank2, 
			   ROW_NUMBER() OVER (ORDER BY SUM(drn1 + drn2 + drn3 + drn4 + drn5 + drn6)) AS DRN1
			   INTO #MoinsCher2
FROM (
  SELECT DCTE1.Département, drn1, drn2, drn3, drn4, drn5, drn6
	FROM DCTE1
    JOIN DCTE2 ON DCTE1.Département = DCTE2.Département
    JOIN DCTE3 ON DCTE1.Département = DCTE3.Département
    JOIN DCTE4 ON DCTE1.Département = DCTE4.Département
    JOIN DCTE5 ON DCTE1.Département = DCTE5.Département
    JOIN DCTE6 ON DCTE1.Département = DCTE6.Département
) AS AllRanks2
GROUP BY Département

SELECT * FROM #MoinsCher2
```


|Déptmt (Département)	|Sum_Rank2	|RN1|
|--|--|--|
|Loire-Atlantique	|6	|1|
|Mayenne	|12|	2|
|Vendée	|18	|3|
|Maine-et-Loire	|24|	4|
|Sarthe |30|	5|

La Loire-Atlantique est le 1er département où en moyenne, le SP98, le SP95, le Gazole, le E10, le E85 et le GPLc est le moins cher donc : 1+1+1+1+1+1 = 6 ce qui correspond à la colonne Sum_Rank2. 


#### 2. Le nombre de carburants disponible



















