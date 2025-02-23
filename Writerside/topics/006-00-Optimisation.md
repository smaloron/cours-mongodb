# Optimisation

## Indexation

L'indexation est sans doute la méthode la plus efficace pour améliorer la performance des requêtes. Cependant l'utilisation des index ralentira les insertions et les mises à jour et augmentera la taille de la base de données.

Il ne faut donc définir des index que sur les champs qui seront effectivement utilisés dans les recherches, les tris, les liaisons ou les regroupements.

### Fonctionnement des index
La recherche par index se base sur une table d'indexation qui référence toutes les valeurs d'une clef et associe les documents correspondants. La table d'index est triée en ordre croissant ou décroissant.

Dans l'exemple ci-dessous, sans index pour rechercher tous les documents qui portent le nom Gramont, il faudra parcourir toute la table. En effet, la collection n'étant pas triée sur la clé, nom, nous ne pouvons savoir avec certitude que nous avons obtenu tous les documents voulus avant d'arriver à la fin de la table.

Avec un index, le parcours est bien plus rapide, car l'index étant trié, mongoDB, peut utiliser un algorithme de recherche dichotomique, bien plus performant que le parcours de l'ensemble de la collection. 

Cependant, lors des insertions des mises à jour et même des suppressions. Il faudra également modifier cette table d'indexation, ce qui aura un impact sur la performance. En outre, il faut bien sûr stocker ces informations sur le disque dur et en mémoire, vive ce qui augmente le poids de la base de données.

![CleanShot 2025-02-12 at 16.56.03@2x.png](CleanShot 2025-02-12 at 16.56.03@2x.png)

### Test de performance

Nous ajoutons une collection contenant 5000 documents.

l'ensemble des fichiers du projet se trouvent [ici](https://github.com/smaloron/mongodb-playground)

```shell
docker exec -i mongo-container mongoimport \
    -p 123 -u admin \
    --db formation \
    --collection randomusers \
    --file /shared/random_users.me_5000.json \
    --jsonArray \
    --authenticationDatabase admin
```

#### Requête sans index

Exécutons cette requête dans un terminal. La fonction explain nous donnera des détails sur la façon dont MongoDB exécute notre instruction.

```mongodb
db.randomusers.find({gender: 'male'})
              .explain('executionStats')
```

**Notons quelques points :**

- `WinningPlan.stage:'COLLSCAN'`: cela indique que MongoDB a réalisé un parcours complet de la collection.
- `totalDocsExamined: 5000`: voilà qui confirme la première information.


```shell
formation> db.randomusers.find({gender: 'male'}).explain('executionStats')
{
  explainVersion: '1',
  queryPlanner: {
    namespace: 'formation.randomusers',
    parsedQuery: { gender: { '$eq': 'male' } },
    indexFilterSet: false,
    queryHash: '2FC37AE0',
    planCacheKey: '2240FDC0',
    optimizationTimeMillis: 0,
    maxIndexedOrSolutionsReached: false,
    maxIndexedAndSolutionsReached: false,
    maxScansToExplodeReached: false,
    prunedSimilarIndexes: false,
    winningPlan: {
      isCached: false,
      stage: 'COLLSCAN',
      filter: { gender: { '$eq': 'male' } },
      direction: 'forward'
    },
    rejectedPlans: []
  },
  executionStats: {
    executionSuccess: true,
    nReturned: 2452,
    executionTimeMillis: 2,
    totalKeysExamined: 0,
    totalDocsExamined: 5000,
    executionStages: {
      isCached: false,
      stage: 'COLLSCAN',
      filter: { gender: { '$eq': 'male' } },
      nReturned: 2452,
      executionTimeMillisEstimate: 0,
      works: 5001,
      advanced: 2452,
      needTime: 2548,
      needYield: 0,
      saveState: 0,
      restoreState: 0,
      isEOF: 1,
      direction: 'forward',
      docsExamined: 5000
    }
  },
  command: {
    find: 'randomusers',
    filter: { gender: 'male' },
    '$db': 'formation'
  },
  serverInfo: {
    host: 'b42a0b2f8d25',
    port: 27017,
    version: '8.0.3',
    gitVersion: '89d97f2744a2b9851ddfb51bdf22f687562d9b06'
  },
  serverParameters: {
    internalQueryFacetBufferSizeBytes: 104857600,
    internalQueryFacetMaxOutputDocSizeBytes: 104857600,
    internalLookupStageIntermediateDocumentMaxSizeBytes: 104857600,
    internalDocumentSourceGroupMaxMemoryBytes: 104857600,
    internalQueryMaxBlockingSortMemoryUsageBytes: 104857600,
    internalQueryProhibitBlockingMergeOnMongoS: 0,
    internalQueryMaxAddToSetBytes: 104857600,
    internalDocumentSourceSetWindowFieldsMaxMemoryBytes: 104857600,
    internalQueryFrameworkControl: 'trySbeRestricted',
    internalQueryPlannerIgnoreIndexWithCollationForRegex: 1
  },
  ok: 1
}
```

#### Requête avec un index

Commençons par créer un index, la valeur 1 indique un ordre croissant, pour un ordre décroissant on utilise -1.

```mongodb
db.randomusers.createIndex({gender: 1})
```

Puis exécutons à nouveau la requête.


```mongodb
db.randomusers.find({gender: 'male'})
              .explain('executionStats')
```

**Observations**

- `stage: 'IXSCAN'`
- `indexName: 'gender_1'`
- `totalDocsExamined: 2452`

Nous avons divisé par deux le nombre de documents scannés. Nous sommes ici sur une information qui ne possède que peu de valeurs différentes, le gain serait bien supérieur avec un nom ou un prénom par exemple. 

Le cas le plus favorable est quand toutes les valeurs sont uniques. Plus la valeur recherchée est fréquente et moins l'index est efficace.

Il peut même arriver qu'une requête sur un index soit plus lente que sans index. Dans le cas où la requête retourne la majorité des documents de la collection, l'étape supplémentaire du parcours des index peut affecter négativement la performance.

```shell
formation> db.randomusers.find({gender: 'male'}).explain('executionStats')
{
  explainVersion: '1',
  queryPlanner: {
    namespace: 'formation.randomusers',
    parsedQuery: { gender: { '$eq': 'male' } },
    indexFilterSet: false,
    queryHash: '2FC37AE0',
    planCacheKey: '354E5737',
    optimizationTimeMillis: 0,
    maxIndexedOrSolutionsReached: false,
    maxIndexedAndSolutionsReached: false,
    maxScansToExplodeReached: false,
    prunedSimilarIndexes: false,
    winningPlan: {
      isCached: false,
      stage: 'FETCH',
      inputStage: {
        stage: 'IXSCAN',
        keyPattern: { gender: 1 },
        indexName: 'gender_1',
        isMultiKey: false,
        multiKeyPaths: { gender: [] },
        isUnique: false,
        isSparse: false,
        isPartial: false,
        indexVersion: 2,
        direction: 'forward',
        indexBounds: { gender: [ '["male", "male"]' ] }
      }
    },
    rejectedPlans: []
  },
  executionStats: {
    executionSuccess: true,
    nReturned: 2452,
    executionTimeMillis: 2,
    totalKeysExamined: 2452,
    totalDocsExamined: 2452,
    executionStages: {
      isCached: false,
      stage: 'FETCH',
      nReturned: 2452,
      executionTimeMillisEstimate: 0,
      works: 2453,
      advanced: 2452,
      needTime: 0,
      needYield: 0,
      saveState: 0,
      restoreState: 0,
      isEOF: 1,
      docsExamined: 2452,
      alreadyHasObj: 0,
      inputStage: {
        stage: 'IXSCAN',
        nReturned: 2452,
        executionTimeMillisEstimate: 0,
        works: 2453,
        advanced: 2452,
        needTime: 0,
        needYield: 0,
        saveState: 0,
        restoreState: 0,
        isEOF: 1,
        keyPattern: { gender: 1 },
        indexName: 'gender_1',
        isMultiKey: false,
        multiKeyPaths: { gender: [] },
        isUnique: false,
        isSparse: false,
        isPartial: false,
        indexVersion: 2,
        direction: 'forward',
        indexBounds: { gender: [ '["male", "male"]' ] },
        keysExamined: 2452,
        seeks: 1,
        dupsTested: 0,
        dupsDropped: 0
      }
    }
  },
  command: {
    find: 'randomusers',
    filter: { gender: 'male' },
    '$db': 'formation'
  },
  serverInfo: {
    host: 'b42a0b2f8d25',
    port: 27017,
    version: '8.0.3',
    gitVersion: '89d97f2744a2b9851ddfb51bdf22f687562d9b06'
  },
  serverParameters: {
    internalQueryFacetBufferSizeBytes: 104857600,
    internalQueryFacetMaxOutputDocSizeBytes: 104857600,
    internalLookupStageIntermediateDocumentMaxSizeBytes: 104857600,
    internalDocumentSourceGroupMaxMemoryBytes: 104857600,
    internalQueryMaxBlockingSortMemoryUsageBytes: 104857600,
    internalQueryProhibitBlockingMergeOnMongoS: 0,
    internalQueryMaxAddToSetBytes: 104857600,
    internalDocumentSourceSetWindowFieldsMaxMemoryBytes: 104857600,
    internalQueryFrameworkControl: 'trySbeRestricted',
    internalQueryPlannerIgnoreIndexWithCollationForRegex: 1
  },
  ok: 1
}
```

### A vous

- faire une requête sur `name.first` avec un explain.
- Créer un index sur `name.first`.
- Refaire la même requête et comparer le résultat.

Pour obtenir la liste des prénoms distincts

```mongodb
db.randomusers.distinct("name.first")
```

### Index composite

Il est possible de créer un index sur un groupe de chant. Dans ce cas, MongoDB utilisera l'ensemble des valeurs pour constituer l'index. L'ordre des champs est important et nous prendrons garde à commencer par les champs qui ont le moins de valeurs répétées dans notre collection.


Cet index accélérera les requêtes sur le champ âge et également les requêtes sur les champs âge et gender.

```mongodb
db.randomusers.createIndex({'dob.age': 1, gender: 1})
```

### Index et tri

Les index également grandement accélérer le tri. Dans certains cas, ils sont même indispensables, car si la collection résultant d'une requête fait plus de 32 Mo, MongoDB sera incapable de la trier, sauf si ce tri a déjà été effectué dans un index.

### Index unique

Les indexes peuvent également inclure une contrainte d'unicité.

```mongodb
db.randomusers.createIndex(
    {email: 1},
    {unique: true}
)
```

### Un cas particulier
Imaginons que nous souhaitions poser un index unique sur un champ facultatif. L'absence de valeur sera considérée par MongoDB comme un doublon et nous ne pourrons insérer plus d'un document omettant le champ unique.

Pour pallier ce problème, il nous faudra ajouter une contrainte sur l'index et ainsi indiquer une condition d'indexation.

Commençons par supprimer l'éventuel index déjà existant.
```mongodb
db.randomusers.dropIndex({{email: 1}})
```

Puis créons un index partiel qui ne s'appliquera qu'aux documents remplissant une condition.
```mongodb
db.randomusers.createIndex(
  {'email': 1},
  {
    unique: true,
    partialFilterExpression: {email; {exists: true}}
  }
)
```

### Visualiser les index

```mongodb
db.randomusers.getIndexes()
```



