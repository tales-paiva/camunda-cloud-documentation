---
id: schema-and-migration
title: Schema and migration
---
Operate stores data in Elasticsearch. On first start, Operate creates all required indices and templates.

* [Schema](#schema)
* [Data migration](#data-migration)
* [How to migrate](#how-to-migrate)
* [Example for migrate in Kubernetes](#example-for-migrate-in-kubernetes)

## Schema

Operate uses several Elasticsearch indices that are mostly created using templates.

Each index has its own version of schema. This means the version reflected in the index name is *not* the version of Operate.

Index names follow the defined pattern below:

```
{operate-index-prefix}-{datatype}-{schemaversion}_[{date}]

```

Here, `operate-index-prefix` defines the prefix for index name (default `operate`), `datatype` defines which data is stored in the index (e.g. `user`, `variable` etc.,) `schemaversion` represents the index schema version, and `date` represents the finished date of the archived data. See [Data retention](data-retention.md).

Knowing the index name pattern, it's possible to customize index settings by creating Elasticsearch templates. See an [example of an index template](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/indices-templates.html).

To define the desired number of shards and replicas, define the following template:

```
PUT _template/template_operate
{
  "index_patterns": ["operate-*"],
  "settings": {
    "number_of_shards": 5,
    "number_of_replicas": 2
  }
}
```

:::note
For these settings to work, the template must be created before Operate runs.
:::

## Data migration

The version of Operate is reflected in Elasticsearch object names (e.g. `operate-user-1.0.0_` index contains the user data for Operate 1.0.0.) When upgrading from one version of Operate to another, migration of data must be performed. Operate distribution provides an application to perform data migration from older versions.

### Concept

The migration uses Elasticsearch [processors](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/ingest-processors.html) and [pipelines](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/pipeline.html) to reindex the data.

Each version of Operate delivers a set of migration steps which need to be applied for a corresponding version of Operate.

When upgrading from one version to another, necessary migration steps constitute the so-called migration plan.
All known migration steps (both applied and not) are persisted in the dedicated Elasticsearch index: `operate-migration-steps-repository`.

### How to migrate

#### Migrate by using standalone application

Ensure Elasticsearch contains the data Operate is running. The migration script will connect to a specified connection in Operate configuration (```<operate_home>/config/application.yml```).

Execute ```<operate_home>/bin/migrate``` (or ```<operate_home>/bin/migrate.bat``` for Windows).

What is expected to happen:

* New Elasticsearch indices are created if they don't exist.
* If an older version for some or all indices exists, the migration plan is built.
* For each index with an older version, the migration plan is executed.
* Older indices are deleted.

All known migration steps with metadata are stored in the `operate-migration-steps-repository` index.

:::note
The old indices are deleted *only* after successful migration. This might require more disk space during the migration process.
:::

:::note
Take care of data backup before performing migration.
:::

#### Migrate by using built-in automatic upgrade

When running a newer version of Operate against an older schema, it performs data migration on a startup.
The migration happens for every index, for which it detects exactly **one** older version. Migration fails if it detects more than one older version of some index. 

#### Further notes

* If migration fails, you can retry it. All applied steps are stored and only those steps are applied that haven't been executed yet.
* Operate should not be running while migration is happening.
* In the case version upgrade is performed in the cluster with several Operate nodes, only one node ([Webapp module](importer-and-archiver.md)) must execute data migration. The others must be stopped and started only after migration is fully finished.

#### Configure migration

Automatic migration is enabled by default. It can be disabled by setting the configuration key:

`camunda.operate.migration.migrationEnabled = false`

The following migration settings may affect the duration of the migration process:

1. You can set the batch size for reindex of the documents. This can reduce the time needed to reindex the data.
Small document size means big batch size, while big document size means small batch size.

`camunda.operate.migration.reindexBatchSize = 5000` (Between 1 and 10.000, Default: 5.000)

2. In how many slices should the reindex be divided. For each shard used by the index, you normally use a slice.
Elasticsearch decides how many slices are used if the value is set to 0 (automatic).

`camunda.operate.migration.slices = 0` - Must be positive. Default is 0 (automatic). 

#### Example for migration in Kubernetes

To ensure the migration is executed *before* Operate is started, use
the [initContainer](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) feature of Kubernetes. 

This ensures only the "main" container is started if the initContainer is successfully executed.

The following snippet of a pod description for Kubernetes shows the usage of `migrate` script as initContainers:

```
...
  labels:
    app: operate
spec:
   initContainers:
     - name: migration
       image: camunda/operate:1.0.0
       command: ['/bin/sh','/usr/local/operate/bin/migrate']
   containers:
     - name: operate
       image: camunda/operate:1.0.0
       env:
...
```

