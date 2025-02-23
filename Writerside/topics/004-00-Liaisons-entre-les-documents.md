# Liaisons entre les documents

En MongoDB, la liaison entre les documents se fait généralement de deux manières : les références (manual references ou DBRefs) et l’imbrication (embedded documents). Le choix entre ces deux dépend du modèle de données et des besoins de performance.

## L’imbrication de documents (Embedded Documents)
Cette méthode consiste à stocker directement un document dans un autre document. Elle est idéale pour les relations "un à plusieurs" où les données sont fréquemment consultées ensemble.

#### ✅ **Avantages :**
- Moins de requêtes, donc plus rapide.
- Pas besoin de jointures complexes.
- Données cohérentes et immédiatement accessibles.

#### ❌ **Inconvénients :**
- Taille de document limitée (16 Mo).
- Peut entraîner de la duplication et une mise à jour plus compliquée.

Exemple :

```mongodb
{
  "_id": 1,
  "nom": "Alice",
  "email": "alice@example.com",
  "adresses": [
    {
      "rue": "123 Rue Principale",
      "ville": "Paris",
      "code_postal": "75001"
    },
    {
      "rue": "456 Avenue Centrale",
      "ville": "Lyon",
      "code_postal": "69002"
    }
  ]
}
```

ou encore

```mongodb
{
  "_id": ObjectId("1234567890abcdef12345678"),
  "titre": "Les Misérables",
  "année": 1862,
  "auteur": {
    "nom": "Victor Hugo",
    "nationalité": "Française"
  }
}
```

## Références inter-collections

Si la redondance devient problématique ou que le nombre de sous documents dépasse un certain seuil, il est préférable de créer des collections séparées et de lier ses collections par le biais de références. Bien qu'il ne soit pas question ici de clé étrangère. Le principe est très similaire à ce que nous connaissons dans le monde relationnel. 

Dans l'exemple ci-dessous, nous avons une collection `authors` et une collection `books` qui référence la clef `_id` de `authors` dans une clef `author_id`.


```mongodb
{
  "_id": ObjectId("65123abc456def7890123456"),
  "name": "Victor Hugo",
  "nationality": "Française"
}
```

```mongodb
{
  "_id": ObjectId("1234567890abcdef12345678"),
  "title": "Les Misérables",
  "author_id": ObjectId("65123abc456def7890123456"),
  "publishedAt": 1862
}
```

### Requête de jointure

Voici l'équivalent d'une jointure avec MongoDB.

```mongodb
db.livres.aggregate([
  {
    "$lookup": {
      "from": "auteurs",
      "localField": "auteur_id",
      "foreignField": "_id",
      "as": "auteur"
    }
  },
  {
    "$unwind": "$auteur"
  }
])
```


## Quelle méthode choisir ?

| Critère  | Référencement (`$lookup`) | Imbrication (Embeddage) |
|----------|-----------------|----------------|
| 🔄 Fréquence des mises à jour | Élevée | Faible |
| 📚 Nombre de relations | "Un-à-plusieurs" ou "Plusieurs-à-plusieurs" | "Un-à-few" |
| Performances  | Plus lent (requête avec `$lookup`) | Plus rapide (lecture unique) |
| Complexité de la structure | Plus modulaire | Plus simple |

### Les questions à se poser

- **Volumétrie**, l'ordre de grandeur du nombre de documents liés.
- **Fréquence de mise à jour**.
- **Partage**, le document peut-il être potentiellement lié à combien d'autres documents ?
- **Indépendance**, est-il possible d'avoir à requêter sur le document lié indépendamment de son parent.

Comme souvent en conception de structure de données, il n'y a pas de bonne réponse absolue. La solution dépend du contexte, des fonctionnalités et des traitements que l'application doit fournir.

### Petit défi

Pour les situations qui suivent, arbitrer entre imbrication et référencement, motiver le choix.

- Un élève et ses devoirs (matière, note et date).
- Une commande et son client.
- Une commande et ses produits
- Une recette et ses ingrédients
- Un vol et ses escales
- Une personne et son adresse

