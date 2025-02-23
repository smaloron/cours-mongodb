# MongoDB

## Objectifs

- Installer MongoDB
- Identifier les spécificités de MongDB
- Lancer un shell Mongo

## Introduction

MongoDB est une base de données NoSQL orientée document, très adaptée aux applications modernes et aux systèmes de grande échelle. Contrairement aux bases de données relationnelles comme MySQL, elle stocke les données sous forme de documents JSON (ou BSON).

### Analogie : MongoDB comme un classeur de bureau

- Une base de données est un ensemble de classeurs.

- Une collection est un classeur contenant plusieurs documents.

- Un document est une fiche dans le classeur avec des informations structurées en JSON.

### Caractéristiques

- **Stockage orienté document** : Les données sont stockées sous forme de documents JSON/BSON, ce qui permet une structure flexible et modifiable.
- **Schéma dynamique** : Contrairement aux bases relationnelles, MongoDB ne nécessite pas de schéma fixe, facilitant les mises à jour et l'évolution des applications.
- **Scalabilité horizontale** : Grâce au sharding (partitionnement des données sur plusieurs serveurs), MongoDB peut gérer efficacement de gros volumes de données.
- **Indexation avancée** : MongoDB prend en charge divers types d'index pour améliorer la performance des requêtes.
- **Haute disponibilité** : La réplication des données via les replica sets assure une tolérance aux pannes et une continuité de service.
- **Performances optimisées** : MongoDB est conçu pour gérer des charges de travail élevées grâce à un accès rapide en lecture et en écriture.

### Comparaison avec les SGBDR

| **Critères**            | **MongoDB (NoSQL)** | **Base de données relationnelle (SQL)** |
|-------------------------|---------------------|-----------------------------------------|
| **Modèle de données**   | Orienté document (JSON/BSON) | Basé sur des tables et des relations |
| **Schéma**             | Flexible et dynamique | Fixe et structuré |
| **Scalabilité**        | Horizontale (sharding) | Verticale (ajout de ressources à un serveur unique) |
| **Requêtes**          | Langage spécifique (MongoDB Query Language) | SQL (Structured Query Language) |
| **Transactions**       | Partiellement supportées | Transactions ACID robustes |
| **Performance**       | Excellente pour les lectures/écritures rapides | Moins efficace pour des données volumineuses non structurées |
| **Adaptabilité**      | Idéal pour les big data et les applications modernes | Meilleur pour des applications nécessitant de fortes contraintes relationnelles |

## Installation
Le plus simple pour tester MongoDB est d'utiliser Docker.

```yaml
# docker-compose.yml
services:
  mongodb:
    image: mongo:latest
    container_name: mongo-container
    restart: always
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: 123
    volumes:
      - mongodb_data:/data/db
      - ./shared:/shared

volumes:
  mongodb_data:
```

```
docker compose up -d
```

- Le volume `mongodb_data` stockera les données de la base.
- Le volume `shared` est partagé avec l'hôte et facilitera l'import et l'export de données.

### Connection à la base de données

Pour lancer un shell MongoDB et commencer à faire des requêtes, il faut taper la ligne suivante dans un terminal

```
docker exec -it <service docker> mongosh -u <user> -p <password> --authenticationDatabase admin
```

Dans notre exemple, cela donne ceci : 

```
docker exec -it mongo-container mongosh -u admin -p 123 --authenticationDatabase admin
```


