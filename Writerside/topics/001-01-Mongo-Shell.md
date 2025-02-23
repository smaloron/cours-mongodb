# Les opérations

## Définir la base de données active

La base de données active est celle sur laquelle s'exécuterons les actions. Si la base demandée n'existe pas, elle sera automatiquement créée.

```mongodb
use <nom de la base de données>
```

exemple

```mongodb
use formation
```

Nous pouvons lister les base de données avec la commande suivante :

```mongodb
show dbs
```


## Créer une collection

La création d'une collection est également automatique, il suffit de créer un nouveau document dans une collection inexistante.

Nous pouvons lister les collections avec la commande suivante :

```mongodb
show collections
```

### Insertion d'un document

```mongodb
db.<nom de la collection>.insertOne({<données>})
```

Par exemple

```mongodb
db.persons.insertOne({
    name: "Brahé", 
    firstName: "Tycho", 
    job: "Astronome"
})
```

Création d'une autre personne

```mongodb
db.persons.insertOne({
    name: "Malcolm", 
    firstName: "Reynolds", 
    job: "Capitaine", 
    movie: "Firefly"
 })
```

![CleanShot 2025-02-04 at 19.41.56@2x.png](CleanShot 2025-02-04 at 19.41.56@2x.png)

> Il est possible d'aller à la ligne, tant que la commande n'est pas terminée.

### Insertion de plusieurs documents

```mongodb
db.<nom de la collection>.insertMany(
    [
        {<données>},
        {<données>}
    ]
 )
```

Exemple

```mongodb
db.persons.insertMany(
    [
        {firstName: "Ansel", name: "Adams", job : "Photographe"},
        {firstName: "Dorothea", name: "Lange", job : "Photographe"},
        {firstName: "Denis", name: "Brihat", job : "Photographe"},
        {firstName: "Sabine", name: "Weiss", job : "Photographe"},
        {firstName: "Niels", name: "Bohr", job : "Physicien"},
    ]
 )
```

Une clef peut contenir un tableau ou un sous document

```mongodb
db.persons.insertMany(
    [
        {   
            firstName: "Sophie", 
            name: "Adams", 
            school: "Ada's kids",
            age: 32,
            courses : ["Java", "Spring"]   
        },
        {   
            firstName: "Paul", 
            name: "Adams", 
            school: "Ada's kids",
            age: 28,
            courses : ["CSS", "HTML"],
            address : {
                street: "232 rue des Pyrénées",
                city: "Paris",
                zipCode: "75020"
            }     
        },
        {   
            firstName: "Jean", 
            name: "Reynolds", 
            school: "Ada's kids",
            age: 28,
            courses : ["Android", "Java", "HTML"],
            address : {
                street: "5 rue Orfila",
                city: "Paris",
                zipCode: "75020"
            }   
        },   
    ]
 )
```

### A vous

- Insérer 3 nouvelles personnes dont une sans adresse

## Importation de documents

MongoDB propose un outil d'importation de fichiers JSON dans une collection. Ce programme est Indépendant du Shell Mongo.

```Bash
docker exec -i <container docker> \
       mongoimport \
            --db <base de données> \
            --collection <nom de la collection> \
            --file <chemin du fichier> \
            --jsonArray \
            -u <user> -p <mot de passe> \
            --authenticationDatabase admin
```

Si nous avons lancé un container MongoDB sans authentification, nous pouvons nous passer des deux dernières lignes.

### Un exemple

Téléchargeons 
[un fichier de personnes](https://github.com/smaloron/mongodb-playground/blob/master/data/persons.json) et plaçons-le dans le dossier data de notre projet.

![CleanShot 2025-02-09 at 15.25.57@2x.png](CleanShot 2025-02-09 at 15.25.57@2x.png)

Exécutons ensuite l'import dans une base de données `formation`, dans une collection `persons`.

```Bash
docker exec -i mongo-container \
       mongoimport \
            --db formation \
            --collection persons \
            --file /shared/persons.json \
            --jsonArray \
            -u admin -p 123 \
            --authenticationDatabase admin
```

## Sélection de documents

MongoDB dispose de deux méthodes, `find` qui retourne un tableau de documents et `findOne` qui n'en retourne qu'un.

Ces deux méthodes admettent en argument un objet qui correspond aux critères de recherche.

### Sélectionner selon un critère

Il suffit d'indiquer la clef et la valeur recherchée.

```mongodb
db.persons.find({firstName: 'Sophie'})
```

Pour rechercher sur la clef d'un sous document, il faut encadrer la clef et la sous clef par des guillemets.

```mongodb
db.persons.find({'address.city': 'Paris'})
```

### Sélectionner selon plusieurs critères

Il suffit de passer un objet contenant plusieurs clefs.

```mongodb
db.persons.find({
    firstName: 'Sophie',
    'address.city': 'Paris'
})
```

### Compter le nombre de documents

La fonction `countDocument` admet également un objet de critères, elle ne retourne que le nombre de documents qui correspondent à la recherche.

```mongodb
db.persons.countDocuments({
    'address.city': 'Paris'
})
```

### Obtenir des valeurs distinctes

La fonction `distinct` retourne toutes les valeurs uniques d'une clef passée en argument

```mongodb
db.persons.distinct('hobbies')
```

```mongodb
db.persons.distinct('jobHistory.company')
```

Il est possible d'ajouter un objet de critères en second argument. 

Ici les hobbies des habitants de Galway
```mongodb
db.persons.distinct(
    'hobbies', 
    {'address.city': 'Galway'}
)
```

### Sélectionner les clefs à retourner

En vocabulaire de base de données, cette sélection s'appelle une projection. Il faut passer un objet en deuxième argument de la fonction `find`.

```mongodb
db.<collection>.find|findOne(
    {<critères},
    {<projection>}
)
```

#### Exemples

Ne retourne que les clefs sélectionnées en plus de la clef `_id`.

```mongodb
db.persons.find(
    {},
    {firstName: 1, lastName: 1}
)
```


Retourne toutes les clefs sauf `computerSkills` et `jobHistory`.

```mongodb
db.persons.find(
    {},
    {computerSkills: 0, jobHistory: 0}
)
```

Ne retourne que les clefs sélectionnées sans la clef `_id`.

```mongodb
db.persons.find(
    {},
    {firstName: 1, lastName: 1, _id: 0}
)
```

**Observations**

- 1 ajoute la clef à la projection.
- 0 retire la clef de la projection.
- Nous ne pouvons combiner inclusions et exclusions dans une même projection sauf pour la clef `_id`.

### Trier les résulats

La méthode `sort()` permet d'ordonner les documents retournés par une requête en fonction d'un ou plusieurs champs.

Trie les salaires par ordre croissant
```mongodb
db.persons.find({}).sort({yearlyIncome: 1})
```

Trie les salaires par ordre décroissant
```mongodb
db.persons.find({}).sort({yearlyIncome: -1})
```

Trie par ordre croissant de code postal
```mongodb
db.persons.find({}).sort({'address.zipCode': 1})
```

Trie par ordre croissant de nom de famille et de prénom
```mongodb
db.persons.find({}).sort({lastName: 1, firstName: 1})
```


### Limiter les résultats

Il est également possible de limiter le nombre de documents retournés.


Limite la recherche aux trois premiers résultats
```mongodb
db.persons.find(
    {},
    {firstName: 1, lastName: 1, _id: 0}
).limit(3)
```

Limite la recherche à trois résultats en ignorant les cinq premiers
```mongodb
db.persons.find(
    {},
    {firstName: 1, lastName: 1, _id: 0}
).limit(3).skip(5)
```


### A vous {id="a-vous_1"}

- Obtenir le nombre de personnes qui habitent à Lyon.
- Obtenir les personnes qui travaillent ou ont travaillé pour IBM.
- Obtenir les personnes qui connaissent Java.
- Obtenir la liste des hobbies pratiqués par les femmes irlandaises.

## Suppression de documents

Comme pour l'insertion, nous disposons de deux méthodes selon que nous souhaitons supprimer un seul document ou bien une série de documents.

### Suppression d'un document

```mongodb
db.nom_de_la_collection.deleteOne({ <critères> });
```

**Exemple**

```mongodb
db.persons.deleteOne({
    firstName: "Ansel",
    lastName: "Adams"
})
```

![CleanShot 2025-02-09 at 08.02.31@2x.png](CleanShot 2025-02-09 at 08.02.31@2x.png)

**Aucune erreur, mais ça ne marche pas (deleteCount: 0)**

Avec `db.persons.find({})`, nous constatons que la clef `firstName` n'existe pas, il faut utiliser `name`.

```mongodb
db.persons.deleteOne({
    firstName: "Ansel",
    name: "Adams"
})
```

![CleanShot 2025-02-09 at 08.08.26@2x.png](CleanShot 2025-02-09 at 08.08.26@2x.png)

On voit ici les limites de la permissivité de MongoDB et l'importance de lire les messages à la console.

#### Les ID
MongoDB utilise un hash pour identifier les documents, l'équivalent de la clef primaire dans le monde relationnel.

Toutefois, pour utiliser cet identifiant il nous faut construire un `ObjectId` à partir du hash.

Nous constatons avec un `db.persons.find({})` que nos documents possèdent tous une clef `_id`contenant le fameux `ObjectId`.

![CleanShot 2025-02-09 at 08.11.10@2x.png](CleanShot 2025-02-09 at 08.11.10@2x.png)

Nous pouvons donc identifier un document de façon unique pour la suppression, la mise à jour et la sélection.

```mongodb
db.persons.deleteOne({
    _id: ObjectId('67a852b37664e44ffbfc0421'),
})
```

#### Conclusions

- deleteOne supprime un document.
- deleteOne admet en argument un objet json contenant les critères de suppression.
- ObjectId identifie de façon unique un document.
- Si les critères retournent plus d'un document seul le premier trouvé sera supprimé.

### Suppression de plusieurs documents

La commande `deleteMany` fonctionne comme `deleteOne` mais elle supprime tous les documents ciblés par les critères.

```mongodb
db.persons.deleteMany({
    job: 'Photographe',
})
```

Il est également possible d'accèder aux sous documents, mais dans ce cas, il faudra encadrer le champ par des guillemets.


```mongodb
db.persons.deleteMany({
    'address.city': 'Paris',
})
```

### Suppression de tous les documents

```mongodb
db.persons.deleteMany({})
```

## Mise à jour de documents

### Mettre à jour un seul document

```mongodb
db.persons.updateOne(
   { _id: ObjectId("652a7d3f8b9b9f1a9f4e77b2") }, 
   { $set: { age: 30 } }
);
```

- Le premier argument est un objet qui indique les critères de recherche.
- Le second argument est un objet qui contient un opérateur `$set` qui lui-même contient un objet avec les modifications souhaitées. 
- Si une clef n'existe pas, elle est ajoutée, sinon sa valeur est modifiée.

### Mettre à jour plusieurs documents

```mongodb
db.persons.updateMany(
   { school: "Ada's kids" }, 
   { $set: { city: 'Orléans' } }
);
```

### Remplacer un document

Remplace l'intégralité d'un document, les anciennes valeurs sont toutes supprimées.

```mongodb
db.persons.replaceOne(
   { _id: ObjectId("652a7d3f8b9b9f1a9f4e77b2") },
   { 
    firstName: "Albert"
    name: "Dupont", 
    age: 40, 
    address: { 
        city: "Lyon", 
        country: "France" 
    } 
})
```

### Supprimer une clef dans un document

```mongodb
db.<nom_de_la_collection>.updateOne(
   { <critère> }, 
   { $unset: { <clef>: "" } }
);
```

Par exemple ici, nous supprimons les hobbies

```mongodb
db.persons.updateOne(
   { _id: ObjectId('67a8efaf04d902140fb36e19') }, 
   { $unset: { hobbies: "" } }
);
```

### Incrémenter une valeur

```mongodb
db.<nom_de_la_collection>.updateOne(
   { <critère> }, 
   { $inc: { <clef>: valeur } }
);
```

Par exemple ici, nous augmentons le salaire annuel de 1000.

```mongodb
db.persons.updateOne(
   { _id: ObjectId('67a8efaf04d902140fb36e19') }, 
   { $inc: { yearlyIncome: 1000 } }
);
```

> Pour décrémenter, il suffit d'utiliser une valeur négative.

Augmente toutes les femmes

```mongodb
db.persons.updateMany(
   { gender: 'femme' }, 
   { $inc: { yearlyIncome: 1000 } }
);
```

### Mettre à jour une clef dans un sous document

Il suffit d'encadrer la clef par des guillemets

```mongodb
db.persons.updateOne(
   { _id: ObjectId('652a7d3f8b9b9f1a9f4e77b2') }, 
   { $set: 
        { 
            'address.city': 'Lyon',
            'address.zipCode': '69001'
        } 
    }
);
```

### Ajouter un élément à un tableau

Ajout d'une valeur scalaire 

```mongodb
db.persons.updateOne(
   { _id:  ObjectId('67a913ea9c516e055a2574ed') },
   { $push: { hobbies: 'dessin' } }
);
```

Ajout dans un tableau de documents

```mongodb
db.persons.updateOne(
   { _id:  ObjectId('67a913ea9c516e055a2574ed') },
   { $push: { jobHistory: 
        {
            joinedAt: 2015,
            jobTitle: 'Waitress',
            company: 'Starbuck',
            leftAt: 2017
        } 
     } 
   }
);
```

### Supprimer un élément dans un tableau

Supprime le hobby Lecture

```mongodb
db.persons.updateOne(
   { _id:  ObjectId('67a913ea9c516e055a2574ed') },
   { $pull: { hobbies: 'Lecture' } }
);
```

Supprime toutes les expériences professionnelles chez Starbuck
```mongodb
db.persons.updateOne(
   { _id:  ObjectId('67a913ea9c516e055a2574ed') },
   { $pull: 
     { 
        jobHistory: {company: 'Starbuck'} 
     } 
   }
);
```

### Supprimer le premier ou le dernier élément d'un tableau

Supprime la dernière expérience de Sophie Martin.

```mongodb
db.persons.updateOne(
  { "firstName": "Sophie", "lastName": "Martin" },
  { "$pop": { "jobHistory": 1 } }
)
```
- 1 supprime le dernier élement du tableau.
- -1 supprime le premier élément.

### Ajoute un élément en évitant les doublons

Ajoute la compétence Python uniquement si elle n'est pas déjà présente.

```mongodb
db.persons.updateOne(
  { "firstName": "Jean", "lastName": "Dupont" },
  { "$addToSet": { "computerSkills": "Python" } }
)
```

## Exercice

### Création

Créer une collection de films contenant les clefs suivantes :

- title
- releaseYear
- actors (un tableau des acteurs principaux)
- genre

Insérer 5 films et faire les requêtes suivantes :

### Requêtes

- Les films d'un genre donné
- Les films dans lesquels joue un acteur donné

### Modifications

- Ajouter un acteur à un film
- Supprimer un acteur dans un film
- Changer le genre d'un film





