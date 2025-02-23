# Sélections avancées

## Les opérateurs de comparaison

### Égalité et différence

l'opérateur d'égalité n'est pas très utile car nous obtenons la même chose avec un syntaxe plus simple. Il nous permet cependant d'introduire l'utilisation des opérateurs.

- Un opérateur commence par un $.
- Un opérateur est toujours contenu dans son propre objet

```mongodb
db.persons.find(
    {gender: 'Femme'}
)
```

```mongodb
db.persons.find(
    {   
        gender: {$eq : 'Femme'}
    }
)
```

Exactement le même effet, mais un niveau d'accolade en moins avec la première syntaxe.

#### Opérateur de différence

Il s'agit de la négation de l'opérateur d'égalité (not equal).


Exemple, Les personnes étrangères vivant au Royaume-Uni :

```mongodb
db.persons.find({
    "address.country": 'Royaume-Uni',
    nationality: {$ne: 'Royaume-Uni'}
})
```

### Comparaison numérique

| Opérateur | Description              |
|-----------|--------------------------|
| $gt       | supérieur (greater than) |
| $gte      | supérieur ou égal        |
| $lt       | inférieur (lesser than)  |
| $lte      | inférieur ou égal        |


Exemple, les personnes qui gagnent plus de 50 000 par an.
```mongodb
db.persons.find({
    yearlyIncome: {$gt: 50000}
})
```

Ou bien les personnes qui gagnent entre 30 000 et 35 000.

Ici, nous groupons deux conditions dans un même objet ce qui a le même effet qu'un `AND` en `SQL`.

```mongodb
db.persons.find({
    yearlyIncome: {$gt: 30000, $lt: 35000}
})
```

### Comparaison de deux clefs

Pour comparer deux clefs d'un même document, il faut utiliser l'opérateur `$expr` et lui passer deux éléments :
- un opérateur de comparaison
- un tableau ordinal des clefs à comparer, les noms des champs doivent être préfixés avec un $.

Exemple, toutes les personnes dont le pays de résidence est différent de la nationalité.

```mongodb
db.persons.find({
    $expr: {
        $ne: ['$nationality', '$address.country']
    }
})
```

## Opérateurs combinatoires

En MongoDB, les opérateurs combinatoires permettent de combiner plusieurs conditions dans une requête pour affiner les résultats. 

Ces opérateurs admettent en argument un tableau de critères.

### L'opérateur $and

Permet de joindre une liste de conditions où toutes doivent être vraies.

```mongodb
{ "$and": [ { condition1 }, { condition2 } ] }
```

MongoDB applique implicitement `$and` lorsque l'objet spécifie plusieurs critères. Ainsi les deux requêtes suivantes sont équivalentes.

```mongodb
db.persons.find({
    firstName: 'Sophie',
    nationality: 'France'
})
```

```mongodb
db.persons.find({
    $and: [
        {firstName: 'Sophie'},
        {nationality: 'France'}
    ]
})
```

### L'opérateur $or

Permet de joindre une liste de conditions où au moins l'une d'entre elles doit être vraie.

```mongodb
{ "$or": [ { condition1 }, { condition2 } ] }
```

Les personnes vivant à Lyon ou à paris

```mongodb
db.persons.find({
    $or: [
        { "address.city": "Lyon" },
        { "address.city": "Paris" }
    ]
})
```

> Attention, pour combiner un opérateur `$or` avec d'autres critères, il faut impérativement encadrer les deux propositions dans un `$and`.

### Combinaison des deux opérateurs

Les personnes ayant un revenu entre 50 000 et 70 000 ET ayant travaillé chez Google ou Amazon.

```mongodb
db.persons.find({
    $and: [
        { yearlyIncome: { $gte: 50000, $lte: 70000 } },
        { $or: [ 
            { "jobHistory.company": "Google" }, 
            { "jobHistory.company": "Amazon" } 
        ]}
    ]
})
```

### L'opérateur $not

L'opérateur $not permet d'inverser une condition. Il est souvent utilisé avec des opérateurs de comparaison.

```mongodb
{ "clef": { "$not": { "opérateur": "valeur" } } }
```

Les personnes qui n'habitent pas en France.

```mongodb
db.persons.find({
    "address.country": { $not: { $eq: "France" } }
})
```

Pour l'inversion d'un égalité nous pouvons simplifier la syntaxe avec l'opérateur `$ne` (not equal).

```mongodb
db.persons.find({
    "address.country": { $ne: "France" }
})
```

### Combinaison des trois opérateurs

Les personnes vivant en France ou en Belgique ET qui n'ont jamais travaillé pour IBM ou Amazon.

```mongodb
db.persons.find({
    $and: [
        {
            $or: [
                {"address.country": "France"},
                {"address.country": "Belgique"}
            ]
        },
        {
            $or: [
                {"jobHistory.company": {$not: {$eq: "IBM"}}},
                {"jobHistory.company": {$not: {$eq: "Amazon"}}}
            ]
        }
    ]
})

```

## Recherche dans les tableaux

Cette requête retourne toutes les personnes qui possèdent "Bricolage" dans le tableau des hobbies.

```mongodb
db.persons.find({
    hobbies: "Bricolage"
})
```

Pour requêter sur plusieurs valeurs il nous faut utiliser un opérateur

### Obtenir tous les éléments

Toutes les personnes qui possèdent les deux hobbies

```mongodb
db.persons.find({
    hobbies: {
        $all: ["Bricolage", "Astronomie"]
    }
})
```

### Obtenir au moins un élément

```mongodb
db.persons.find({
    hobbies: {
        $in: ["Bricolage", "Astronomie"]
    }
})
```

Il existe également un opérateur `$nin` (not in)

### Vérifier la taille d'un tableau

Les personnes qui n'ont qu'un seul hobby

```mongodb
db.persons.find({
    hobbies: { $size: 1 }
})
```

#### Comparaison sur la taille d'un tableau

Cette requête est invalide, car l'opérateur de comparaison ne peut être au premier niveau (top level operator).

```mongodb
db.persons.find(
    { "$gte": [{ "$size": "$hobbies" }, 2] }
)
```

La solution est d'encadrer dans un opérateur `$expr`.

```mongodb
db.persons.find({
    "$expr":
        { "$gt": [{ "$size": "$hobbies" }, 3] }
})
```

##### Astuce

Il est également possible de tester la présence d'une clef dans un tableau ordinal. Ici, s'il existe une clef 2, c'est donc que le tableau contient au moins trois éléments (la première clef étant 0).

```
db.persons.find({
    "hobbies.2": { "$exists": true }
})
```

Les personnes qui ont au maximum deux hobbies (pas de clef 2 dans le tableau).
```mongodb
db.persons.find(
        {"hobbies.2": { "$exists": false }}
)
```

#### Le nombre d'éléments entre deux bornes

Pour utiliser deux critères dans l'opérateur `$expr`, il faudra encadrer ces critères dans un `$and`.

```mongodb
db.persons.find({
    "$expr": {
        "$and": [
            { "$gte": [{ "$size": "$hobbies" }, 2] },
            { "$lte": [{ "$size": "$hobbies" }, 3] }
        ]
    }
})
```

### Rechercher dans un tableau de documents

Filtre les documents où au moins un élément d'un tableau correspond à plusieurs critères.

Les personnes qui ont une expérience de consultant débutant après 2015.

```mongodb
db.persons.find({
  "jobHistory": {
    "$elemMatch": {
      "jobTitle": "Consultant",
      "joinedAt": { "$gt": 2015 }
    }
  }
})
```

## Exercices

#### Exercice 1

Sélectionner toutes les personnes qui ont un revenu annuel supérieur à 60 000 et qui travaillent actuellement.

##### Correction {collapsible="true"}
```mongodb
db.persons.find({
    yearlyIncome: { $gt: 60000 },
    'jobHistory.leftAt': null
})
```

#### Exercice 2

Sélectionner les personnes qui vivent dans un pays différent de leur nationalité.

##### Correction {collapsible="true" id="correction_1"}
```mongodb
db.persons.find({
    $expr: {
        $ne: ['$nationality', '$address.country']
    }
})
```

#### Exercice 3

Sélectionner les personnes qui ont travaillé dans une entreprise, mais qui sont actuellement indépendantes.

##### Correction {collapsible="true" id="correction_2"}
```mongodb
db.persons.find({
    $and: [
        {
            jobHistory: {
                $elemMatch: {
                    company: 'None (Self-Employed)',
                    leftAt: null
                }
            }
        },
        {
            'jobHistory.1': {$exists: true}
        }
    ]
})
```

#### Exercice 4

Sélectionner les personnes qui ont eu au moins un emploi dans une entreprise spécifique (ex: "Facebook") entre 2010 et 2020.

##### Correction {collapsible="true" id="correction_3"}
```mongodb
db.persons.find({
    jobHistory: { 
        $elemMatch: { 
            company: 'Facebook',
            joinedAt: { $gte: 2010 },
            leftAt: { $lte: 2020 }
        } 
    }
})
```





 
