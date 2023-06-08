# Projet de SQL/NoSQL/NewSQL 


Dans le cadre de ce projet, nous avons conçu une base de données en utilisant divers modèles NoSQL. Cette base de données intègre des informations générées ou provenant de la Base Carbone, nous permettant ainsi d'évaluer l'empreinte carbone de divers menus de restaurant.

## Table des matières

- [Mise en place de la base de données](#mise-en-place-de-la-base-de-données)
- [Requêtes](#requêtes)
- [Défis et solutions](#défis-et-solutions)
- [Conclusion](#conclusion)

## Mise en place de la base de données
Nos modèles sont conçus pour stocker et gérer des informations sur les ingrédients, les plats et les menus, avec une attention particulière portée à l'empreinte carbone de chaque élément. Chaque ingrédient est associé à une valeur d'empreinte carbone, chaque plat est composé d'un ensemble d'ingrédients, et chaque menu est une collection de plats. Cette structure nous permet d'évaluer l'empreinte carbone totale d'un plat ou d'un menu en sommant les empreintes carbone de leurs composants respectifs.

### Redis
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
**création des tables :**
```javascript
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
**Représentation visuelle**
![Logo de OpenAI](https://drive.google.com/uc?export=view&id=14o9ld-OF8WFpyWa1yli_qrhwV42Ne6MV)
![Logo de OpenAI](https://drive.google.com/uc?export=view&id=1bkBnq4DHdFxv3RsmIAchUnNTu1u3bVwf)
![Logo de OpenAI](https://drive.google.com/uc?export=view&id=1YVItAyS2RB8d0XbpUXQH7ISV0r77CsLH)

## Requêtes

### Redis
**Quelle est l’empreinte carbone d’un ingrédient donné ?**
```redis
GET ingredient:Tomate
```
> GET ingredient:Tomate
"1.2"

**Quels sont les ingrédients composant un plat ?**
```redis
LRANGE plat:Salade 0 -1
```
> LRANGE plat:Salade 0 -1
1) "Laitue"
2) "Tomate"
3) "Laitue"
4) "Tomate"

**Quels sont les plats composant un menu ?**
```redis
LRANGE menu:Menu_classique_avec_poulet 0 -1
```
> LRANGE menu:Menu_classique_avec_poulet 0 -1
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
mycompiler_mongodb>[
  {
    message: "L'ingrédient ayant la plus faible empreinte carbone est Laitue avec une empreinte carbone de 0.7"
  }
]

**Plats**
```javascript
db.plats.aggregate([
  {
    $unwind: "$ingrédients"
  },
  {
    $lookup: {
      from: "ingredients",
      localField: "ingrédients",
      foreignField: "_id",
      as: "ingrédient_info"
    }
  },
  {
    $group: {
      _id: "$_id",
      nom: { $first: "$nom" },
      total_empreinte_carbone: { $sum: { $arrayElemAt: ["$ingrédient_info.empreinte_carbone", 0] } }
    }
  },
  {
    $sort: { total_empreinte_carbone: 1 }
  },
  {
    $limit: 1
  },
  { $project: { _id: 0, message: { $concat: [ "Le plat ayant la plus faible empreinte carbone est ", "$nom", " avec une empreinte carbone de ", { $toString: "$total_empreinte_carbone" } ] } } }
])
```
mycompiler_mongodb> [
  {
    message: 'Le plat ayant la plus faible empreinte carbone est Salade avec une empreinte carbone de 1.9'
  }
]

**Menus**
```javascript
db.menus.aggregate([
  {
    $unwind: "$plats"
  },
  {
    $lookup: {
      from: "plats",
      localField: "plats",
      foreignField: "_id",
      as: "plat_info"
    }
  },
  {
    $unwind: "$plat_info"
  },
  {
    $unwind: "$plat_info.ingrédients"
  },
  {
    $lookup: {
      from: "ingredients",
      localField: "plat_info.ingrédients",
      foreignField: "_id",
      as: "ingrédient_info"
    }
  },
  {
    $group: {
      _id: "$_id",
      nom: { $first: "$nom" },
      total_empreinte_carbone: { $sum: { $arrayElemAt: ["$ingrédient_info.empreinte_carbone", 0] } }
    }
  },
  {
    $sort: { total_empreinte_carbone: 1 }
  },
  {
    $limit: 1
  },
  { $project: {_id: 0, message: { $concat: [ "Le menu ayant la plus faible empreinte carbone est ", "$nom", " avec une empreinte carbone de ", { $toString: "$total_empreinte_carbone" } ] } } }
])
```
mycompiler_mongodb>[
{
    message: 'Le menu ayant la plus faible empreinte carbone est Menu classique avec poulet avec une empreinte carbone de 35.8'
  }
]

**Quels sont les ingrédients, plats ou menus ayant une empreinte inférieure à un seuil donné ?**
**Ingrédients**
```javascript
let seuil = 10;
db.ingredients.find({ "empreinte_carbone": { $lt: seuil } }, { _id: 0 })
```
mycompiler_mongodb>[[
  { nom: 'Tomate', empreinte_carbone: 1.2 },
  { nom: 'Laitue', empreinte_carbone: 0.7 },
  { nom: 'Poulet', empreinte_carbone: 6.9 }
]

**Plats**
```javascript
let seuil = 25;
db.plats.aggregate([
  {
    $unwind: "$ingrédients"
  },
  {
    $lookup: {
      from: "ingredients",
      localField: "ingrédients",
      foreignField: "_id",
      as: "ingrédient_info"
    }
  },
  {
    $group: {
      id: "$_id",
      nom: { $first: "$nom" },
      total_empreinte_carbone: { $sum: { $arrayElemAt: ["$ingrédient_info.empreinte_carbone", 0] } }
    }
  },
  {
    $match: { total_empreinte_carbone: { $lt: seuil } }
  },
{
    $project: { _id: 0, nom: 1, total_empreinte_carbone: 1 }
  }
])
```
mycompiler_mongodb> [
{ nom: 'Sandwich au poulet', total_empreinte_carbone: 20.4 },
  { nom: 'Salade', total_empreinte_carbone: 1.9 },
  { nom: 'Assiette de fromages', total_empreinte_carbone: 13.5 }
]

**Menus**
```javascript
let seuil = 40;
db.menus.aggregate([
  {
    $unwind: "$plats"
  },
  {
    $lookup: {
      from: "plats",
      localField: "plats",
      foreignField: "_id",
      as: "plat_info"
    }
  },
  {
    $unwind: "$plat_info"
  },
  {
    $unwind: "$plat_info.ingrédients"
  },
  {
    $lookup: {
      from: "ingredients",
      localField: "plat_info.ingrédients",
      foreignField: "_id",
      as: "ingrédient_info"
    }
  },
  {
    $unwind: "$ingrédient_info"
  },
  {
    $group: {
      _id: "$_id",
      nom: { $first: "$nom" },
      total_empreinte_carbone: { $sum: "$ingrédient_info.empreinte_carbone" }
    }
  },
  {
    $match: { total_empreinte_carbone: { $lt: seuil } }
  },
  {
    $project: { _id: 0, nom: 1, total_empreinte_carbone: 1 }
  }
])
```
mycompiler_mongodb>[
{ nom: 'Menu classique avec poulet', total_empreinte_carbone: 35.8 }
]

### PostgreSQL avec clé-valeur
**Requête 1 :**
```javascript
SELECT *
FROM ingredient
WHERE ingredient = '2 oeufs’;
```
![Logo de OpenAI](https://drive.google.com/uc?export=view&id=1ctee5a6XJ3DVLleZAVGde0VPGJ8tycJo)

**Requête 2 :**
```javascript
SELECT p.plat AS plat, p.ingredients AS composition, SUM(i.gco) AS empreinte_carbone
FROM plat p
JOIN ingredient i ON i.ingredient = ANY (hstore_to_array(p.ingredients))
WHERE p.plat = 'poulet au riz'
GROUP BY p.plat, p.ingredients;
```
![Logo de OpenAI](https://drive.google.com/uc?export=view&id=1yTSPWZvAR9jNQgferap2zqi07yidBs-T)

**Requête 3 :**
```javascript
SELECT m.nom AS menu, p.plat AS plat, p.ingredients AS composition, SUM(i.gco) AS empreinte_carbone
FROM menuf m
JOIN plat p ON p.plat = ANY (hstore_to_array(m.plats))
JOIN ingredient i ON i.ingredient = ANY (hstore_to_array(p.ingredients))
WHERE m.nom = 'Repas classique 1 (avec poulet)'
GROUP BY m.nom, p.plat, p.ingredients;

```
![Logo de OpenAI](https://drive.google.com/uc?export=view&id=1x5augJe0sZVVKqcab88qMqY_qaBmA_Jm)

**Requête 4 :**
```javascript
SELECT p.plat AS plat, p.ingredients AS composition, SUM(i.gco) AS empreinte_carbone
FROM plat p
JOIN ingredient i ON i.ingredient = ANY (hstore_to_array(p.ingredients))
where 'pain' = ANY (hstore_to_array(p.ingredients))
GROUP BY p.plat, p.ingredients;
```
![Logo de OpenAI](https://drive.google.com/uc?export=view&id=1Ih1_-_zMd8N5nnTiq2nPA9agH8OG823B)

**Requête 5 :**
**Ingrédients**
```javascript
SELECT ingredient, gco AS empreinte_carbone
FROM ingredient
WHERE gco <= 127
ORDER BY gco
```
![Logo de OpenAI](https://drive.google.com/uc?export=view&id=1vp4hvFRdi5HIXMSbKayWMHWZlZ8Ho5Wl)
```javascript
SELECT ingredient, gco AS empreinte_carbone
FROM ingredient
ORDER BY gco
limit 1
```
![Logo de OpenAI](https://drive.google.com/uc?export=view&id=18JA9wxe-9DJAQWDNcvL_lRFJggOtOkpN)

**Plats**
```javascript
SELECT p.plat AS plat, p.ingredients AS composition, SUM(i.gco) AS empreinte_carbone
FROM plat p
JOIN ingredient i ON i.ingredient = ANY (hstore_to_array(p.ingredients))
GROUP BY p.plat, p.ingredients
HAVING SUM(i.gco) <= 200
ORDER BY SUM(i.gco)
```
![Logo de OpenAI](https://drive.google.com/uc?export=view&id=1hP3KnONz53smWeljTBZFRTuP5scP8pk9)

```javascript
SELECT p.plat AS plat, p.ingredients AS composition, SUM(i.gco) AS empreinte_carbone
FROM plat p
JOIN ingredient i ON i.ingredient = ANY (hstore_to_array(p.ingredients))
GROUP BY p.plat, p.ingredients
ORDER BY SUM(i.gco)
limit 1
```
![Logo de OpenAI](https://drive.google.com/uc?export=view&id=1QgJ2C3Lmjd4xHM7xyUhT2zCP6_8Z24KB)

**Menus**
```javascript

```
![Logo de OpenAI](https://drive.google.com/uc?export=view&id=1hP3KnONz53smWeljTBZFRTuP5scP8pk9)

```javascript

```
![Logo de OpenAI](https://drive.google.com/uc?export=view&id=1QgJ2C3Lmjd4xHM7xyUhT2zCP6_8Z24KB)

## Défis et solutions

(Description des défis rencontrés lors de la mise en œuvre du projet et des solutions utilisées pour les surmonter)

## Conclusion

(Conclusion sur le projet, les résultats obtenus et les perspectives futures)
