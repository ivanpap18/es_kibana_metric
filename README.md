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

# Création d'un data stream dans Elasticsearch avec Vector

Une politique de cycle de vie (ILM) est utilisée pour définir les phases de vie d'un index dans Elasticsearch, telles que "hot" et "warm", et pour spécifier les actions à effectuer lorsqu'un index atteint une phase de vie spécifique. 

## Création d'une politique de cycle de vie

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

## Création de modèles de composants

Les modèles de composants sont utilisés pour définir les paramètres de configuration pour les différents aspects de l'indexation et de la recherche, tels que les paramètres d'indexation, les paramètres de mappage et les paramètres de recherche. Ces modèles peuvent ensuite être utilisés dans plusieurs index.

### Créer un nouveau modèle de composant avec les paramètres de configuration suivants pour les paramètres ILM :

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

### Créer un nouveau modèle de composant avec les paramètres de mappage pour l'index :

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

## Création d'un modèle d'index

Un modèle d'index est utilisé pour définir les paramètres de configuration pour un index, tels que les paramètres de mappage et les paramètres ILM.

### Créer un nouveau modèle d'index avec les paramètres de configuration suivants :

```
PUT _index_template/april-index-template
{
  "index_patterns": ["vector-april-logs*"],
  "data_stream": { },
  "composed_of": [ "april-mappings", "april-settings" ],
  "priority": 500
}
```

### Créer un nouveau data stream

Le nom de votre data stream "vector-april-logs" dépend de trois paramètres qui sont définis dans le fichier vector.toml, à savoir :

- data_stream.type = "vector" : ce paramètre définit le type de la source de données.
- data_stream.dataset = "april" : ce paramètre définit le nom du dataset de la source de données.
- data_stream.namespace = "logs" : ce paramètre définit le namespace de la source de données.

Le nom complet du data stream sera `vector-april-logs`. Vous pouvez modifier ces paramètres en fonction de vos besoins.
Assurez-vous que le nom du data stream défini dans votre fichier `vector.toml` correspond au nom du data stream que vous avez créé dans Elasticsearch.

```
PUT _data_stream/vector-april-logs
```

## Configuration de Vector

Pour envoyer des données vers Elasticsearch dans un data stream, vous devez configurer les paramètres suivants dans le fichier `vector.toml` :

```
[sinks]
mode = "data_stream"
data_stream.type = "vector" 
data_stream.dataset = "april" 
data_stream.namespace = "logs"
```

