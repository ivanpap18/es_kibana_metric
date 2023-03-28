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

## Créer un datastream avec ILM et un index-template

- Tout d'abord,connectez-vous à votre cluster ElasticSearch et ouvrez DevTools: 

## Créer a ILM

```
PUT _ilm/policy/30-days-policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_primary_shard_size": "50gb"
          }
        }
      },
      "warm": {
        "min_age": "30d",
        "actions": {
          "shrink": {
            "number_of_shards": 1
          },
          "forcemerge": {
            "max_num_segments": 1
          }
        }
      }
    }
  }
}
```

### Créer a component-template avec ILM

```
PUT _component_template/april-settings
{
  "template": {
    "settings": {
      "index.lifecycle.name": "30-days-policy"
    }
  }
}
```

### Créer un component-template avec mappings

```
PUT _component_template/april-mappings
{
  "template": {
    "mappings": {
      "properties": {
        "@timestamp": {
          "type": "date",
          "format": "date_optional_time||epoch_millis"
        },
        "message": {
          "type": "wildcard"
        }
      }
    }
  }
}
```

### Créer un index template pour data stream

```
PUT _index_template/april-index-template
{
  "index_patterns": ["vector-april-logs*"],
  "data_stream": { },
  "composed_of": [ "april-mappings", "april-settings" ],
  "priority": 500
}
```

### Créer data stream

```
PUT _data_stream/vector-april-logs
```