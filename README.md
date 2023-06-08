# SQL-NoSQL-NewSQL

Dans le cadre de ce projet, nous avons conçu une base de données en utilisant divers modèles NoSQL. Cette base de données intègre des informations générées ou provenant de la Base Carbone, nous permettant ainsi d'évaluer l'empreinte carbone de divers plats et menus.
## Table des matières
- [Mise en place de la base de données](#mise-en-place-de-la-base-de-données)
- [Requêtes](#requêtes)
- [Tableau de performance](#performances)
- [Avantages et limites de nos choix de modélisation](#avantages-et-limites)
- [Conclusion](#conclusion)

## Mise en place de la base de données
Nos modèles sont conçus pour stocker et gérer des informations sur les ingrédients, les plats et les menus, avec une attention particulière portée à l'empreinte carbone de chaque élément. Chaque ingrédient est associé à une valeur d'empreinte carbone, chaque plat est composé d'un ensemble d'ingrédients, et chaque menu est une collection de plats. Cette structure nous permet d'évaluer l'empreinte carbone totale d'un plat ou d'un menu en sommant les empreintes carbone de leurs composants respectifs.

### Redis
Les instructions Redis permettent de mettre en place une base de données comprenant des ingrédients avec leur empreinte carbone, des plats composés en associant des ingrédients, et des menus constitués en associant des plats.
**Création des ingrédients avec leur empreinte carbone :**
```redis
SET ingredient:Tomate 1.2
SET ingredient:Laitue 0.7
SET ingredient:Pain 1.3
SET ingredient:Poulet 6.9
SET ingredient:Fromage 13.5
SET ingredient:Pomme 0.3
SET ingredient:Pomme_de_Terre 0.2
SET ingredient:Boeuf 27.0
```
**Création des plats avec leurs ingrédients :**
```redis
LPUSH plat:Salade Tomate Laitue
LPUSH plat:Sandwich_au_poulet Poulet Pain
LPUSH plat:Assiette_de_fromages Fromage
LPUSH plat:Tarte_aux_pommes Pomme Pain
LPUSH plat:Salade_de_pommes_de_terre Pomme_de_Terre Laitue
LPUSH plat:Steak_de_boeuf Boeuf
LPUSH plat:Salade_de_fruits Pomme Tomate
```
**Création des menus avec leurs plats :**
```redis
LPUSH menu:Menu_classique_avec_poulet Salade Sandwich_au_poulet Assiette_de_fromages Tarte_aux_pommes
LPUSH menu:Menu_classique_avec_boeuf Salade_de_pommes_de_terre Steak_de_boeuf Assiette_de_fromages Salade_de_fruits
```
### Document sous MongoDB 
Ces instructions MongoDB permettent de créer une base de données contenant des ingrédients avec leur empreinte carbone, des plats associant ces ingrédients, et des menus regroupant les plats.
**Création des ingrédients avec leur empreinte carbone :**
```javascript
db.ingredients.insertMany([
  {"_id": 1, "nom": "Tomate", "empreinte_carbone": 1.2},
  {"_id": 2, "nom": "Laitue", "empreinte_carbone": 0.7},
  {"_id": 3, "nom": "Poulet", "empreinte_carbone": 6.9},
  {"_id": 4, "nom": "Boeuf", "empreinte_carbone": 27.0},
  {"_id": 5, "nom": "Fromage", "empreinte_carbone": 13.5}
])
```
**Création des plats avec leurs ingrédients :**
```javascript
db.plats.insertMany([
  {"_id": 6, "nom": "Salade", "catégorie_de_plat": "Entrée", "ingrédients": [1, 2]},
  {"_id": 7, "nom": "Sandwich au poulet", "catégorie_de_plat": "Plat principal", "ingrédients": [3, 5]},
  {"_id": 8, "nom": "Assiette de fromages", "type_de_repas": "Plateau de fromages", "ingrédients": [5]},
  {"_id": 9, "nom": "Steak de boeuf", "catégorie_de_plat": "Plat principal", "ingrédients": [4]}
])
```
**Création des menus avec leurs plats :**
```javascript 
db.menus.insertMany([
  {"_id": 10, "nom": "Menu classique avec poulet", "plats": [6, 7, 8]},
  {"_id": 11, "nom": "Menu classique avec boeuf", "plats": [6, 9, 8]}
])
```
### PostgreSQL avec clé-valeur
Le modèle est composé de 3 tables : "menuf", "plat" et "ingredient", contenant respectivement des menus avec un hstore des plats associés, des plats avec un hstore des ingrédients, et des ingrédients avec leur valeur d'empreinte carbone. 
```postgresql
CREATE TABLE menuf (
nom varchar(100) primary key,
plats hstore
);
insert into menuf values ('Repas classique 1 (avec poulet)',  '"Entree"=>"legumes à la grecque", "Plat principal"=>"poulet au riz", "Sortie"=>"plateau de fromage"'), ('Repas classique 2 (avec boeuf)',  '"Entree"=>"tzatziki", "Plat principal"=>"bifteck-frites", "Dessert"=>"tarte aux poires"'), ('Repas vegegtarien 1',  '"Entree"=>"soupe de legumes", "Plat principal"=>"omelette aux pommes de terre", "Dessert"=>"salade de fruit »')
CREATE TABLE plat (
plat varchar(100) primary key,
ingredients hstore
);
insert into plat values ('legumes à la grecque',  '1=>"legumes de saison", 2=>"huile dolive (1/2 c.s)"'), ('poulet au riz',  '1=>"poulet", 2=>"riz", 3=>"beurre"'), ('plateau de fromage',  '1=>"fromage à pâte molle", 2=>"fromage à pâte dure", 3=>"pain"'), ('tzatziki',  '1=>"yaourt", 2=>"concombre", 3=>"huile dolive (1/2 c.s)"'), ('bifteck-frites',  '1=>"bifteck", 2=>"frites"'), ('tarte aux poires',  '1=>"farine", 2=>"poires", 3=>"huile (1 c.s)"'), ('soupe de legumes',  '1=>"legumes de saison", 2=>"huile dolive (1/2 c.s)"'), ('omelette aux pommes de terre',  '1=>"2 oeufs", 2=>"pommes de terre", 3=>"huile (1/2 c.s)"'), ('salade de fruits',  '1=>"fruits de saison", 2=> »pain"')

CREATE TABLE ingredient (
ingredient varchar(100) primary key,
gco float
);
insert into ingredient values ('legumes de saison', 53.4), ('huile dolive (1/2 c.s)', 18.2), ('poulet', 774), ('riz', 84.6), ('beurre', 94.7), ('fromage à pâte molle', 107), ('fromage à pâte dure', 114), ('pain', 76), ('yaourt', 360), ('concombre', 129), ('bifteck', 5370), ('frites', 260), ('farine', 46.8), ('poires', 71), ('huile (1 c.s)', 32.3), ('2 oeufs', 276.7), ('pommes de terre', 15.5), ('huile (1/2 c.s)', 17.1), ('fruits de saison', 53.4)
```
### PostgreSQL JSON
Le modèle est composé de deux tables : "menujson" et "platjson". La table "menujson" contient des menus avec leurs plats associés, stockés sous forme de données JSON. La table "platjson" contient des plats avec leur composition d'ingrédients, également stockée au format JSON.
```postgresql
CREATE TABLE menujson (
menu varchar(100) primary key,
plats json not null
);
INSERT INTO menujson VALUES('Repas classique 1 (avec poulet)', '{"entree": "Legumes à la grecque","plat": "Poulet au riz","sortie": "Plateau de fromages"}'), ('Repas classique 2 (avec boeuf)', '{"entree": "Tzatziki","plat": "Bifteck-frites","sortie": "Tarte aux poires"}'), ('Repas végétarien 1', '{"entree": "Soupe de legumes","plat": "Omelette aux pommes de terre","sortie": "Salade de fruits"}')
CREATE TABLE platjson (
plat varchar(100) primary key,
ingredients json not null
);
INSERT INTO platjson VALUES('Legumes à la grecque', '{"composition": ["legumes de saison", "huile dolive (1/2 c.s)"]}'), ('Poulet au riz', '{"composition": ["poulet", "riz", "beurre"]}'), ('Plateau de fromages', '{"composition": ["fromage à pâte molle", "fromage à pâte dure", "pain"]}'), ('Tzatziki', '{"composition": ["yaourt", "concombre", "huile dolive (1/2 c.s)"]}'), ('Bifteck-frites', '{"composition": ["bifteck", "frites"]}'), ('Tarte aux poires', '{"composition": ["farine", "poires", "huile (1 c.s)"]}'), ('Soupe de legumes', '{"composition": ["legumes de saison", "huile dolive (1/2 c.s)"]}'), ('Omelette aux pommes de terre', '{"composition": ["2 oeufs", "pommes de terre", "huile (1/2 c.s)"]}'), ('Salade de fruits', '{"composition": ["fruits de saison", "pain"]}')
```
### Graphe avec Neo4j
Ce script permet de créer des nœuds pour les ingrédients, les types de plats, les plats et les menus dans la base de données Neo4j, ainsi que des relations entre eux. Les ingrédients sont liés aux plats et les plats sont liés aux menus en fonction de leur composition.
**Création de la table :**
```neo4j
CREATE (legumes:Ingredient {name:'Légumes de saison', gco:53}), (olive:Ingredient {name:'huile dolive (1/2 c.s)', gco:18}), (poulet:Ingredient {name:'poulet' , gco:774}), (riz:Ingredient {name:'riz', gco:84.6}), (beurre:Ingredient {name:'beurre', gco:94}), (fromou:Ingredient {name:'fromage à pate molle', gco:107}), (frodure:Ingredient {name:'fromage à pate dure', gco:140}), (pain:Ingredient {name:'pain', gco:76}), (yaourt:Ingredient {name:'yaourt', gco:360}),(concombre:Ingredient {name:'concombre', gco:129}), (bifteck:Ingredient {name:'bifteck', gco:5370}), (frites:Ingredient {name:'frites', gco:260}), (farine:Ingredient {name:'farine', gco:46}), (poire:Ingredient {name:'poire', gco:71}), (huile1:Ingredient {name:'huile (1 c.s)', gco:32}), (huilemoit:Ingredient {name:'huile (1/2 c.s)', gco:15}), (oeufs:Ingredient {name:'2 oeufs', gco:276}), (pdt:Ingredient {name:'pommes de terre', gco:15}), (fruits:Ingredient {name:'fruitsde saison', gco:53}), (entree:Type { name:'Entrée' }), (plat:Type { name:'Plat principal' }), (dessert:Type { name:'Dessert' }), (fromage:Type { name:'Plateau de fromages' }), (legrecque:Plat { name:'Légumes à la grecque' }), (pouletriz:Plat { name:'Poulet au riz' }), (platfrom:Plat { name:'Plateau de fromage' }), (tzatziki:Plat { name:'Tzatziki' }), (bifrites:Plat { name:'Bifteck-frites' }), (tartepoire:Plat { name:'Tarte aux poires' }), (soupleg:Plat { name:'Soupe de légumes' }), (ompdt:Plat { name:'Omelette aux pommes de terre' }), (salfruit:Plat { name:'Salade de fruits' }), (menu1:Menu { name:'Repas classique 1 (avec poulet)' }), (menu2:Menu { name:'Repas classique 1 (avec boeuf)' }), (menu3:Menu { name:'Repas végétarien 1' }), (menu1) -[:est_compose_de]-> (legrecque), (menu1) -[:est_compose_de]-> (pouletriz), (menu1) -[:est_compose_de]-> (platfrom), (menu2) -[:est_compose_de]-> (tzatziki), (menu2) -[:est_compose_de]-> (bifrites), (menu2) -[:est_compose_de]-> (tartepoire), (menu3) -[:est_compose_de]-> (soupleg), (menu3) -[:est_compose_de]-> (ompdt), (menu3) -[:est_compose_de]-> (salfruit), (legrecque) -[:contient]-> (legumes), (legrecque) -[:contient]-> (olive), (pouletriz) -[:contient]-> (poulet), (pouletriz) -[:contient]-> (riz), (pouletriz) -[:contient]-> (beurre), (platfrom) -[:contient]-> (fromou), (platfrom) -[:contient]-> (frodure), (platfrom) -[:contient]-> (pain), (tzatziki) -[:contient]-> (yaourt), (tzatziki) -[:contient]-> (concombre), (tzatziki) -[:contient]-> (olive), (bifrites) -[:contient]-> (bifteck), (bifrites) -[:contient]-> (frites), (tartepoire) -[:contient]-> (farine), (tartepoire) -[:contient]-> (poire), (tartepoire) -[:contient]-> (huile1), (soupleg) -[:contient]-> (legumes), (soupleg) -[:contient]-> (huilemoit), (ompdt) -[:contient]-> (oeufs), (ompdt) -[:contient]-> (pdt), (ompdt) -[:contient]-> (huilemoit), (salfruit) -[:contient]-> (fruits), (salfruit) -[:contient]-> (pain), (legrecque) -[:est_un_type_de]-> (entree), (tzatziki) -[:est_un_type_de]-> (entree), (soupleg) -[:est_un_type_de]-> (entree), (pouletriz) -[:est_un_type_de]-> (plat), (bifrites) -[:est_un_type_de]-> (plat), (ompdt) -[:est_un_type_de]-> (plat), (tartepoire) -[:est_un_type_de]-> (dessert), (salfruit) -[:est_un_type_de]-> (dessert), (platfrom) -[:est_un_type_de]-> (fromage)
```
**Représentation graphique :**
![img](https://drive.google.com/uc?export=view&id=1ZH8LvrLsSOHDwKos8MoxIkAUKCdz6gtJ)
## Requêtes
### Redis
**Quelle est l’empreinte carbone d’un ingrédient donné ?**
```redis
GET ingredient:Tomate
```
>"1.2"
**Quels sont les ingrédients composant un plat ?**
```redis
LRANGE plat:Salade 0 -1
```
1) "Laitue"
2) "Tomate"
3) "Laitue"
4) "Tomate"
**Quels sont les plats composant un menu ?**
```redis
LRANGE menu:Menu_classique_avec_poulet 0 -1
```
1) "Tarte_aux_pommes"
2) "Assiette_de_fromages"
3) "Sandwich_au_poulet"
4) "Salade"
### Document sous MongoDB 
**Quelle est l’empreinte carbone d’un ingrédient donné ?**
```javascript 
db.ingredients.find({_id: 1}, {empreinte_carbone: 1, _id: 0})
```
**Quelle est l’empreinte carbone d’un plat donné et quels sont les ingrédients composant un plat (avec leur empreinte carbone) ?**
```javascript 
var plat = db.plats.findOne({nom: "Salade"}, {ingrédients: 1, _id: 0})
var ingredients = db.ingredients.find({_id: {$in: plat.ingrédients}}).toArray()
var empreinte_carbone_plat = 0;
for (var i = 0; i < ingredients.length; i++) {
  var ingredient = ingredients[i];
  print("L'ingrédient numéro " + (i+1) + ", qui est " + ingredient.nom + ", a une empreinte carbone de " + ingredient.empreinte_carbone + ".");
(function() {
    empreinte_carbone_plat += ingredient.empreinte_carbone;
  })();
}
print("L’empreinte carbone totale du plat est : " + empreinte_carbone_plat);
```
mycompiler_mongodb>
L'ingrédient numéro 1, qui est Tomate, a une empreinte carbone de 1.2.
L'ingrédient numéro 2, qui est Laitue, a une empreinte carbone de 0.7.
L’empreinte carbone totale du plat est : 1.9
**Quelles sont la composition et l’empreinte carbone de chacun des plats composant un menu (avec les détails sur la composition des plats comme sur les affichages précédents par exemple) ?**
```javascript
var menu = db.menus.findOne({_id: 10}, {plats: 1, _id: 0})
menu.plats.forEach(function(platId) {
  var plat = db.plats.findOne({_id: platId});
  print('\nPlat: ' + plat.nom);
  var empreinte_carbone_plat = 0;
  plat.ingrédients.forEach(function(ingredientId) {
    var ingredient = db.ingredients.findOne({_id: ingredientId});
    if (ingredient) {
      print('Ingrédient: ' + ingredient.nom + ', Empreinte carbone: ' + ingredient.empreinte_carbone);
      empreinte_carbone_plat += ingredient.empreinte_carbone;
    } else {
      print('Ingrédient avec ID ' + ingredientId + ' non trouvé');
    }
  });
  print('L’empreinte carbone totale du plat est : ' + empreinte_carbone_plat);
});
```
mycompiler_mongodb>
Plat: Salade
Ingrédient: Tomate, Empreinte carbone: 1.2
Ingrédient: Laitue, Empreinte carbone: 0.7
L’empreinte carbone totale du plat est : 1.9
**Quels sont les plats (avec leur empreinte carbone) contenant un ingrédient donné ?**
```javascript
var ingredientId = 1
var plats = db.plats.find({"ingrédients": ingredientId}).toArray();
for (var i = 0; i < plats.length; i++) {
  print('Plat: ' + plats[i].nom);
}
```
mycompiler_mongodb> Salade
**Quels sont les ingrédients, plats ou menus ayant la plus faible empreinte carbone ?**
**Ingrédients**
```javascript
db.ingredients.aggregate([
  { $sort: { "empreinte_carbone": 1 } },
  { $limit: 1 },
  { $project: {_id: 0, message: { $concat: [ "L'ingrédient ayant la plus faible empreinte carbone est ", "$nom", " avec une empreinte carbone de ", { $toString: "$empreinte_carbone" } ] } } }
])
```
mycompiler_mongodb>[{message: "L'ingrédient ayant la plus faible empreinte carbone est Laitue avec une empreinte carbone de 0.7"}]
**Plats**
```javascript
db.plats.aggregate([
  {$unwind: "$ingrédients"},
  {$lookup: {
      from: "ingredients",
      localField: "ingrédients",
      foreignField: "_id",
      as: "ingrédient_info"}},
  {$group: {
      _id: "$_id",
      nom: { $first: "$nom" },
      total_empreinte_carbone: { $sum: { $arrayElemAt: ["$ingrédient_info.empreinte_carbone", 0] } }
  }},
  {$sort: { total_empreinte_carbone: 1 }},
  {$limit: 1},
  { $project: { _id: 0, message: { $concat: [ "Le plat ayant la plus faible empreinte carbone est ", "$nom", " avec une empreinte carbone de ", { $toString: "$total_empreinte_carbone" } ] } } }])
```
mycompiler_mongodb> [
  { message: 'Le plat ayant la plus faible empreinte carbone est Salade avec une empreinte carbone de 1.9'
  }]
**Menus**
```javascript
db.menus.aggregate([
  {$unwind: "$plats"},
  { $lookup: {
      from: "plats",
      localField: "plats",
      foreignField: "_id",
      as: "plat_info"} },
  {$unwind: "$plat_info"},
  {$unwind: "$plat_info.ingrédients" },
  {$lookup: {
      from: "ingredients",
      localField: "plat_info.ingrédients",
      foreignField: "_id",
      as: "ingrédient_info"}},
  { $group: {
      _id: "$_id",
      nom: { $first: "$nom" },
      total_empreinte_carbone: { $sum: { $arrayElemAt: ["$ingrédient_info.empreinte_carbone", 0] } }}},
  {$sort: { total_empreinte_carbone: 1 }},
  {$limit: 1},
  { $project: {_id: 0, message: { $concat: [ "Le menu ayant la plus faible empreinte carbone est ", "$nom", " avec une empreinte carbone de ", { $toString: "$total_empreinte_carbone" } ] } } }])
```
mycompiler_mongodb>[
{ message: 'Le menu ayant la plus faible empreinte carbone est Menu classique avec poulet avec une empreinte carbone de 35.8'}]
**Quels sont les ingrédients, plats ou menus ayant une empreinte inférieure à un seuil donné ?**
**Ingrédients**
```javascript
let seuil = 10;
db.ingredients.find({ "empreinte_carbone": { $lt: seuil } }, { _id: 0 })
```
mycompiler_mongodb>[ 
  { nom: 'Tomate', empreinte_carbone: 1.2 },
  { nom: 'Laitue', empreinte_carbone: 0.7 },
  { nom: 'Poulet', empreinte_carbone: 6.9 }]
**Plats**
```javascript
let seuil = 25;
db.plats.aggregate([
  {$unwind: "$ingrédients"},
  {$lookup: {
      from: "ingredients",
      localField: "ingrédients",
      foreignField: "_id",
      as: "ingrédient_info"}},
  {$group: {
      id: "$_id",
      nom: { $first: "$nom" },
      total_empreinte_carbone: { $sum: { $arrayElemAt: ["$ingrédient_info.empreinte_carbone", 0] } }}},
  {$match: { total_empreinte_carbone: { $lt: seuil } }},
{$project: { _id: 0, nom: 1, total_empreinte_carbone: 1 }}])
```
mycompiler_mongodb> [
{ nom: 'Sandwich au poulet', total_empreinte_carbone: 20.4 },
  { nom: 'Salade', total_empreinte_carbone: 1.9 },
  { nom: 'Assiette de fromages', total_empreinte_carbone: 13.5 }]
**Menus**
```javascript
let seuil = 40;
db.menus.aggregate([
  { $unwind: "$plats"},
  {$lookup: {
      from: "plats",
      localField: "plats",
      foreignField: "_id",
      as: "plat_info"}},
  {$unwind: "$plat_info"},
  {$unwind: "$plat_info.ingrédients"},
  {$lookup: {
      from: "ingredients",
      localField: "plat_info.ingrédients",
      foreignField: "_id",
      as: "ingrédient_info"} },
  {$unwind: "$ingrédient_info"},
  {$group: {
      _id: "$_id",
      nom: { $first: "$nom" },
      total_empreinte_carbone: { $sum: "$ingrédient_info.empreinte_carbone" }}},
  {$match: { total_empreinte_carbone: { $lt: seuil } }},
  { $project: { _id: 0, nom: 1, total_empreinte_carbone: 1 }}])
```
mycompiler_mongodb>[
{ nom: 'Menu classique avec poulet', total_empreinte_carbone: 35.8 }]
### PostgreSQL avec clé-valeur
**Requête 1 :**
```javascript
SELECT *
FROM ingredient
WHERE ingredient = '2 oeufs’;
```
![img](https://drive.google.com/uc?export=view&id=1ctee5a6XJ3DVLleZAVGde0VPGJ8tycJo)
**Requête 2 :**
```javascript
SELECT p.plat AS plat, p.ingredients AS composition, SUM(i.gco) AS empreinte_carbone
FROM plat p
JOIN ingredient i ON i.ingredient = ANY (hstore_to_array(p.ingredients))
WHERE p.plat = 'poulet au riz'
GROUP BY p.plat, p.ingredients;
```
![img](https://drive.google.com/uc?export=view&id=1yTSPWZvAR9jNQgferap2zqi07yidBs-T)
**Requête 3 :**
```javascript
SELECT m.nom AS menu, p.plat AS plat, p.ingredients AS composition, SUM(i.gco) AS empreinte_carbone
FROM menuf m
JOIN plat p ON p.plat = ANY (hstore_to_array(m.plats))
JOIN ingredient i ON i.ingredient = ANY (hstore_to_array(p.ingredients))
WHERE m.nom = 'Repas classique 1 (avec poulet)'
GROUP BY m.nom, p.plat, p.ingredients;

```
![img](https://drive.google.com/uc?export=view&id=1x5augJe0sZVVKqcab88qMqY_qaBmA_Jm)
**Requête 4 :**
```javascript
SELECT p.plat AS plat, p.ingredients AS composition, SUM(i.gco) AS empreinte_carbone
FROM plat p
JOIN ingredient i ON i.ingredient = ANY (hstore_to_array(p.ingredients))
where 'pain' = ANY (hstore_to_array(p.ingredients))
GROUP BY p.plat, p.ingredients;
```
![img](https://drive.google.com/uc?export=view&id=1Ih1_-_zMd8N5nnTiq2nPA9agH8OG823B)
**Requête 5 :**
**Ingrédients**
```javascript
SELECT ingredient, gco AS empreinte_carbone
FROM ingredient
WHERE gco <= 127
ORDER BY gco
```
```javascript
SELECT ingredient, gco AS empreinte_carbone
FROM ingredient
ORDER BY gco
limit 1
```
![img](https://drive.google.com/uc?export=view&id=18JA9wxe-9DJAQWDNcvL_lRFJggOtOkpN)
**Plats**
```javascript
SELECT p.plat AS plat, p.ingredients AS composition, SUM(i.gco) AS empreinte_carbone
FROM plat p
JOIN ingredient i ON i.ingredient = ANY (hstore_to_array(p.ingredients))
GROUP BY p.plat, p.ingredients
HAVING SUM(i.gco) <= 200
ORDER BY SUM(i.gco)
```
![img](https://drive.google.com/uc?export=view&id=1hP3KnONz53smWeljTBZFRTuP5scP8pk9)
```javascript
SELECT p.plat AS plat, p.ingredients AS composition, SUM(i.gco) AS empreinte_carbone
FROM plat p
JOIN ingredient i ON i.ingredient = ANY (hstore_to_array(p.ingredients))
GROUP BY p.plat, p.ingredients
ORDER BY SUM(i.gco)
limit 1
```
![img](https://drive.google.com/uc?export=view&id=1QgJ2C3Lmjd4xHM7xyUhT2zCP6_8Z24KB)
**Menus**
```javascript
select * from (SELECT m.nom AS menu, p.plat AS plat, p.ingredients AS composition, SUM(i.gco) AS empreinte_carbone
FROM menuf m
JOIN plat p ON p.plat = ANY (hstore_to_array(m.plats))
JOIN ingredient i ON i.ingredient = ANY (hstore_to_array(p.ingredients))
GROUP BY m.nom, p.plat, p.ingredients) as tt
natural join (select menu, sum(empreinte_carbone) as empreinte_menu
from (SELECT m.nom AS menu, p.plat AS plat, p.ingredients AS composition, SUM(i.gco) AS empreinte_carbone
FROM menuf m
JOIN plat p ON p.plat = ANY (hstore_to_array(m.plats))
JOIN ingredient i ON i.ingredient = ANY (hstore_to_array(p.ingredients))
GROUP BY m.nom, p.plat, p.ingredients) as t
group by t.menu) as men
where empreinte_menu <= 1000
```
```javascript
select * from (SELECT m.nom AS menu, p.plat AS plat, p.ingredients AS composition, SUM(i.gco) AS empreinte_carbone
FROM menuf m
JOIN plat p ON p.plat = ANY (hstore_to_array(m.plats))
JOIN ingredient i ON i.ingredient = ANY (hstore_to_array(p.ingredients))
GROUP BY m.nom, p.plat, p.ingredients) as tt
natural join (select menu, sum(empreinte_carbone) as empreinte_menu
from (SELECT m.nom AS menu, p.plat AS plat, p.ingredients AS composition, SUM(i.gco) AS empreinte_carbone
FROM menuf m
JOIN plat p ON p.plat = ANY (hstore_to_array(m.plats))
JOIN ingredient i ON i.ingredient = ANY (hstore_to_array(p.ingredients))
GROUP BY m.nom, p.plat, p.ingredients) as t
group by t.menu) as men
order by empreinte_menu
limit 1
```
### PostgreSQL JSON
**Requête 1 :**
```postgresql
SELECT *
FROM ingredient
WHERE ingredient = '2 oeufs’;
```
**Requête 2 :**
```postgresql
select *
from (select plat, ingredient, gco
from ingredient i 
natural join (SELECT p.plat, json_array_elements_text(p.ingredients->'composition') as ingredient
FROM platjson AS p) as tmp) as tt
natural join (select plat, sum(gco) as empreinte_plat
from (select *
from ingredient i 
natural join (SELECT p.plat, json_array_elements_text(p.ingredients->'composition') as ingredient
FROM platjson AS p) as tmp) as t
group by plat) as tm
where plat = 'Poulet au riz'
```
**Requête 3 :**
```postgresql
select *
from (select menu, json_each_text(plats) as composition
from menujson) as compo
join (select menu, sum(sum) as gco_menu
from (select menu, plats->>'plat' as plat, ingredient as ingredient_plat, gco as gco_plat, sum
from menujson
left join (select *
from (select plat, ingredient, gco
from ingredient i 
natural join (SELECT p.plat, json_array_elements_text(p.ingredients->'composition') as ingredient
FROM platjson AS p) as tmp) as tt
natural join (select plat, sum(gco)
from (select *
from ingredient i 
natural join (SELECT p.plat, json_array_elements_text(p.ingredients->'composition') as ingredient
FROM platjson AS p) as tmp) as t
group by plat) as tm) as gco on plats->>'plat' = gco.plat) as gco_menu
group by menu) as empreinte_menu on compo.menu = empreinte_menu.menu
where compo.menu = 'Repas classique 1 (avec poulet)'
```
**Requête 4 :**
```postgresql
Sselect plat, sum
from (select plat, ingredient, gco
from ingredient i 
natural join (SELECT p.plat, json_array_elements_text(p.ingredients->'composition') as ingredient
FROM platjson AS p) as tmp) as tt
natural join (select plat, sum(gco)
from (select *
from ingredient i 
natural join (SELECT p.plat, json_array_elements_text(p.ingredients->'composition') as ingredient
FROM platjson AS p) as tmp) as t
group by plat) as tm
where ingredient = 'pain'
```
**Requête 5 :**
**Ingrédients**
```postgresql
SELECT ingredient, gco AS empreinte_carbone
FROM ingredient
WHERE gco <= 127
ORDER BY gco
```
```postgresql
SELECT ingredient, gco AS empreinte_carbone
FROM ingredient
WHERE gco <= 127
ORDER BY gco
limit 1
```
**Plats**
```postgresql
select distinct plat, sum 
from (select plat, ingredient, gco
from ingredient i 
natural join (SELECT p.plat, json_array_elements_text(p.ingredients->'composition') as ingredient
FROM platjson AS p) as tmp) as tt
natural join (select plat, sum(gco)
from (select *
from ingredient i 
natural join (SELECT p.plat, json_array_elements_text(p.ingredients->'composition') as ingredient
FROM platjson AS p) as tmp) as t
group by plat) as tm
where sum <=127
```
```postgresql
select distinct plat, empreinte_plat 
from (select plat, ingredient, gco
from ingredient i 
natural join (SELECT p.plat, json_array_elements_text(p.ingredients->'composition') as ingredient
FROM platjson AS p) as tmp) as tt
natural join (select plat, sum(gco) as empreinte_plat
from (select *
from ingredient i 
natural join (SELECT p.plat, json_array_elements_text(p.ingredients->'composition') as ingredient
FROM platjson AS p) as tmp) as t
group by plat) as tm
order by empreinte_plat
limit 1
```
**Menus**
```postgresql
select distinct compo.menu, gco_menu
from (select menu, json_each_text(plats) as composition
from menujson) as compo
join (select menu, sum(sum) as gco_menu
from (select menu, plats->>'plat' as plat, ingredient as ingredient_plat, gco as gco_plat, sum
from menujson
left join (select *
from (select plat, ingredient, gco
from ingredient i 
natural join (SELECT p.plat, json_array_elements_text(p.ingredients->'composition') as ingredient
FROM platjson AS p) as tmp) as tt
natural join (select plat, sum(gco)
from (select *
from ingredient i 
natural join (SELECT p.plat, json_array_elements_text(p.ingredients->'composition') as ingredient
FROM platjson AS p) as tmp) as t
group by plat) as tm) as gco on plats->>'plat' = gco.plat) as gco_menu
group by menu) as empreinte_menu on compo.menu = empreinte_menu.menu
where gco_menu <= 1000
```
```postgresql
select distinct compo.menu, gco_menu
from (select menu, json_each_text(plats) as composition
from menujson) as compo
join (select menu, sum(sum) as gco_menu
from (select menu, plats->>'plat' as plat, ingredient as ingredient_plat, gco as gco_plat, sum
from menujson
left join (select *
from (select plat, ingredient, gco
from ingredient i 
natural join (SELECT p.plat, json_array_elements_text(p.ingredients->'composition') as ingredient
FROM platjson AS p) as tmp) as tt
natural join (select plat, sum(gco)
from (select *
from ingredient i 
natural join (SELECT p.plat, json_array_elements_text(p.ingredients->'composition') as ingredient
FROM platjson AS p) as tmp) as t
group by plat) as tm) as gco on plats->>'plat' = gco.plat) as gco_menu
group by menu) as empreinte_menu on compo.menu = empreinte_menu.menu
order by gco_menu
limit 1
```
### Graphe avec Neo4j
**Requête 1 :**
```neo4j
MATCH (i:Ingredient {name: 'pain'}) RETURN i.name, i.gco
```
**Requête 2 :**
```neo4j
MATCH (p:Plat {name: 'Poulet au riz'})-[:contient]->(i:Ingredient) WITH p, i, i.gco AS empreinteCarboneIngredient RETURN p.name AS Plat, collect(i.name) AS Ingredients, collect(empreinteCarboneIngredient) AS Empreinte_ingrédient, sum(empreinteCarboneIngredient) AS Empreinte_plat
```
**Requête 3 :**
```neo4j
MATCH (m:Menu {name: 'Repas classique 1 (avec boeuf)'})-[:est_compose_de]->(p:Plat)-[:contient]->(i:Ingredient) WITH m, p, collect(i) AS ingredients, sum(i.gco) AS Empreinte_plat RETURN m.name AS Menu, p.name AS Plat, [ingredient IN ingredients | ingredient.name] AS Ingredients, Empreinte_plat
```
**Requête 4 :**
```neo4j
MATCH (p:Plat)-[:contient]->(i:Ingredient) WHERE i.name = 'pain' WITH p, sum(i.gco) AS empreinteCarbonePlat RETURN p.name AS Plat, empreinteCarbonePlat AS Empreinte_plat, [(p)-[:contient]->(i:Ingredient) | i.name] AS Ingredients, REDUCE(total = 0, n IN [(p)-[:contient]->(i:Ingredient) | i.gco] | total + n) AS Empreinte_ingredient
```
**Requête 5 :**
**Ingrédients**
```neo4
MATCH (p:Plat)-[:contient]->(i:Ingredient) WITH p, sum(i.gco) AS totalGCO WHERE totalGCO <= 300 RETURN p.name AS Plat, totalGCO AS Empreinte_carbone ORDER BY totalGCO ASC
```
```neo4
MATCH (i:Ingredient) RETURN i.name AS Ingredient, i.gco AS Empreinte_carbone ORDER BY i.gco ASC LIMIT 1
```
**Plats**
```neo4j
MATCH (p:Plat)-[:contient]->(i:Ingredient) WITH p, sum(i.gco) AS totalGCO WHERE totalGCO <= 300 RETURN p.name AS Plat, totalGCO AS Empreinte_carbone ORDER BY totalGCO ASC
```
```neo4j
MATCH (p:Plat)-[:contient]->(i:Ingredient) WITH p, sum(i.gco) AS totalGCO RETURN p.name AS Plat, totalGCO AS Empreinte_carbone ORDER BY totalGCO ASC LIMIT 1
```
**Menus**
```javascript
MATCH (m:Menu)-[:est_compose_de]->(p:Plat)-[:contient]->(i:Ingredient) WITH m, sum(i.gco) AS totalGCO WHERE totalGCO <= 800 RETURN m.name AS Menu, totalGCO AS Empreinte_carbone ORDER BY totalGCO ASC
```
```javascript
MATCH (m:Menu)-[:est_compose_de]->(p:Plat)-[:contient]->(i:Ingredient) WITH m, sum(i.gco) AS totalGCO RETURN m.name AS Menu, totalGCO AS Empreinte_carbone ORDER BY totalGCO ASC LIMIT 1
```


## Tableau de performance
Avec les requêtes a, b, c, d dans l’ordre demandé dans le sujet, et les requêtes e dans l’ordre suivant : d’abord les ingrédients, puis les plats, puis les menus, avec d’abord en dessous d’un seuil et dans un deuxième temps les minimums.
![img](https://drive.google.com/uc?export=view&id=1aPSeCyGCwhg6nFXhN9MdxZFktIAv0SnL)
Malheureusement, pour Redis, nous n’avons pas pu relever de temps d’exécution car nous n’avons pas compris le fonctionnement de la commande Time, et son application pour les requêtes que nous avions.
Nous pouvons constater qu’en termes de performance en temps d’exécution, les modèles sous PostgreSQL sont nettement plus performants, sans grande distinction entre les deux types d’attribut, à part peut-être un léger avantage pour Json au niveau de certaines requêtes. 
Le modèle le moins performant est de loin MongoDB; atteignant pour certaines requêtes plus d’une seconde.

## Avantages et limites de nos choix de modélisation
Dans le cadre de notre projet, nous avons exploré 5 options de modélisation de données. Chaque option présente des avantages et des limites spécifiques qui ont été évalués en fonction des besoins.
1. **Redis** : L'utilisation de Redis n'est apparue efficace qu'en mémoire, grâce à la vitesse de ses opérations Redis qui le rend idéale pour les situations où la rapidité est essentielle. Il a su supporter des structures de données plus complexes comme les listes, offrant ainsi une flexibilité accrue. Cependant, sa capacité est limitée par la taille de la mémoire du serveur et il n'est pas conçu pour gérer des requêtes complexes ou des relations entre les données, dans notre cas, ce qui a rendu impossible la récupération de l'empreinte carbone globale d'un plat ou d'un menu.
2. **MongoDB** : MongoDB  nous a offert une grande flexibilité et évolutivité grâce à son format de document JSON, qui a été particulièrement adapté au stockage de nos ingrédients, plats et menus. Cependant, MongoDB n'est pas le choix optimal pour les requêtes complexes, particulièrement avec sa fonction d'agrégation. La complexité des requêtes d'agrégation nécessite une expertise en modélisation de données et peut augmenter la consommation de ressources, affectant les performances du système
3. **PostgreSQL avec clé-valeur et PostgreSQL JSON** : Une autre option que nous avons explorée est le modèle de PostgreSQL avec le stockage de clé-valeur et de données JSON. En effet, ce modèle nous a offert une grande flexibilité pour modéliser des données tout en conservant les avantages d'une base de données relationnelle. Mais l'utilisation de types de données non relationnels pour le stockage de clé-valeur a affecté les performances, et les requêtes complexes ont été plus difficiles à écrire et à optimiser.
4. **Graphe avec Neo4j** : Enfin,  Neo4j nous a permis d'écrire des requêtes complexes et des analyses de graphes. Cependant, il peut être surdimensionné pour des cas d'utilisation simples qui ne nécessitent pas de relations complexes.



## Conclusion 
Les principales difficultés rencontrées lors de ce projet étaient notamment dues au fait que nous n'avons pas pu réaliser toutes les requêtes demandées sous Redis. En outre, avec MongoDB, les requêtes prennent la forme de fonctions qui peuvent être longues et complexes, ce qui s'éloigne de la conception traditionnelle des requêtes de bases de données. Une autre difficulté résidait dans la manipulation délicate des consoles en ligne : par exemple, en cas d'erreurs, il nous était ardu de localiser l'origine de l'erreur pour la rectifier. Récemment, nous avons également rencontré des problèmes lors de la rédaction du rapport. En effet, souhaitant un rapport avec un code lisible, nous avons décidé de le rédiger en Markdown, ce qui n'était pas aisé à maîtriser. De plus, nous cherchions à produire un rapport complet, ce qui rendait difficile le respect de la limite de 15 pages maximum. 
En ce qui concerne la répartition du travail, nous avons divisé les modèles à réaliser et les requêtes associées entre nous. Meryeme s'est chargée de Redis et de MongoDB, tandis qu'Emilie s'est occupée des modèles PostgreSQL et Neo4j.
