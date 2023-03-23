# Elasticsearch, Kibana, Vector et Metricbeat

Ce dépôt contient les fichiers nécessaires pour installer et configurer Elasticsearch, Kibana, Vector et Metricbeat. Ces outils sont utilisés pour collecter, stocker et analyser des données.

## Prérequis

- Docker
- Docker Compose

## Installation

1. Clonez ce dépôt sur votre machine locale.
2. Ouvrez un terminal et rendez-vous dans le répertoire racine du dépôt.
3. Exécutez la commande suivante pour lancer Elasticsearch, Kibana, Vector et Metricbeat :

```
bash
docker-compose up -d
```
4. Patientez quelques instants le temps que les conteneurs se lancent.
5. Une fois que les conteneurs sont lancés, vous pouvez accéder à Kibana en ouvrant votre navigateur web et en allant à l'adresse suivante : http://localhost:5601/