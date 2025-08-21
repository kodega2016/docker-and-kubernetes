# Config Maps and Secrets

<!--toc:start-->

- [Config Maps and Secrets](#config-maps-and-secrets)
  - [Introduction](#introduction)
  - [ConfigMaps](#configmaps)
  <!--toc:end-->

## Introduction

We will explore how to use ConfigMaps and Secrets in Kubernetes to manage
configuration data and sensitive information.

We use config maps to store non-sensitive configuration data that can be
either in a file or in a key-value pair format. Secrets are used to store
the sensitive information such as passwords, tokens, and SSH keys.

## ConfigMaps

We will store the non sensitive configuration data in a ConfigMap.It is a
key-value store that can be used to store configuration data.

The maxium size of a ConfigMap is 1MB. The data can be stored in a file or in a
key-value pair format.

The environment variables can be used by the pods.

We can create config maps using both the imperative and declarative
approach.

```# Imperative approach
kubectl create configmap appconfig --from-literal=key1=value1 --from-literal=key2=value2
```

We can list the config maps using the following command:

```bash
kubectl get configmaps
kubectl get configmap appconfig -o yaml
```

For example.

```bash
kubectl create configmap mysqlconfig \
  --from-literal=MYSQL_ROOT_PASSWORD=supersecret \
  --from-literal=MYSQL_USER=kodega \
  --from-literal=MYSQL_PASSWORD=thekeythekey \
  --from-literal=MYSQL_DATABASE=lovethecode
```

We can also create configmap using the file.

```bash
kubectl create configmaps mysqlconfig \
  --from-file=dbconfig.properties.
```

We can also use configMap as volume in the pod. The configMap can be mounted
to the pod as a volume. The files in the configMap will be available in the
pod at the specified path.

For example,we can create a configMap with the following command:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: db-backup-script
data:
  db_backup.sh: |
    #!/bin/bash
    echo "backup was initialized"
    mysqldump -hmysql-svc.default -uroot -psupersecret --all-databases > /var/tmp/db_backup.sql
    echo "backup task was successful"
```

We can mount this configMap in the pod as a volume.

```yaml
volumes:
  - name: db-backup-script-volume
    configMap:
      name: db-backup-script
      items:
        - key: db_backup.sh

volumeMounts:
  - name: db-backup-script-volume
    mountPath: /usr/local/bin/db_backup.sh
    subPath: db_backup.sh
    readOnly: true
command: ["/bin/bash", "-c", "/usr/local/bin/db_backup.sh"]
```

We can also make the configMap immutable by setting the `immutable` field to `true`.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: db-backup-script
data:
  db_backup.sh: |
    #!/bin/bash
    echo "backup was initialized"
    mysqldump -hmysql-svc.default -uroot -psupersecret --all-databases > /var/tmp/db_backup.sql
    echo "backup task was successful"
immutable: true
```
