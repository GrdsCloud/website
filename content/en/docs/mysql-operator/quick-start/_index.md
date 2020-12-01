---
title: "Quick Start"
linkTitle: "Quick Start"
weight: 1
description: >
  Quick deploy MySQL Operator and start a MySQL cluster
---

## Prerequisites

- MySQL operator requires Kubernetes v1.14.x or later or k3s.
- For high availability MySQL,at least 3 nodes k3s/k8s cluster.

## Quick start step
1. deploy MySQL operator
2. install grds CLI tool
3. create a MySQL cluster
4. connect to MySQL cluster

> You can choose [helm](#deploy-mysql-operator-with-helm) or [kubernetes manifests](#deploy-mysql-operator-with-kubernetes-manifests) to depley MySQL operator

## Deploy MySQL operator with Helm

<p align="center"><img src="/website/images/helm2.svg" width="150"></p>

> Note: For the Helm-based installation you need [Helm](https://helm.sh/docs/intro/install/#helm) v3.2.4 or later.

1. Add operator chart repository.
    - Helm v3
    ```bash
    helm repo add grdscloud-stable https://grdscloud.github.io/charts/
    helm repo update
    ```

2. Install the MySQL Operator

    ```bash
    helm upgrade --install --wait --create-namespace --namespace grds mysql-operator grdscloud-stable/mysql-operator
    ```

> If you using k3s,sometimes helm will not access k3s cluster,please copy the k3s.yaml to .kube/config,refer to [k3s cluster access](https://rancher.com/docs/k3s/latest/en/cluster-access)

```
$ helm list -A
Error: Kubernetes cluster unreachable: Get "http://localhost:8080/version?timeout=32s": dial tcp [::1]:8080: connect: connection refused

cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
```

## Deploy MySQL operator with kubernetes manifests


1. Create a controlNamespace called "grds".

    ```bash
    kubectl create ns grds
    ```

2. Create a ServiceAccount and install cluster roles.

    ```bash
    kubectl -n grds create -f https://raw.githubusercontent.com/GrdsCloud/grds/{{< param operatorVersion >}}/installers/manifests/rbac.yaml
    ```

3. Apply the ClusterResources.

    ```bash
    kubectl -n grds create -f https://raw.githubusercontent.com/GrdsCloud/grds/{{< param operatorVersion >}}/installers/manifests/mysql.grds.cloud_mysqlclusters.yaml
    ```

4. Deploy the MySQL operator.

    ```bash
   kubectl -n grds create -f https://raw.githubusercontent.com/GrdsCloud/grds/{{< param operatorVersion >}}/installers/manifests/deployment.yaml
    ```

### Verify MySQL operator deployment

To verify that the installation, complete the following steps.

1. Check the status of the pods. You should see a new mysql-operator pod

    ```bash
    $ kubectl get pods -n grds
    NAME                                        READY   STATUS    RESTARTS   AGE
    mysql-operator-76c44cdc5c-lw4z6             1/1     Running   0          53s
    ```

2. Check the CRDs. You should see the following MySQL cluster CRDs.
mysql-cluster-crd.png

    ```bash
    $ kubectl get crd | grep grds
    NAME                                    CREATED AT
    mysqlclusters.mysql.grds.cloud          2020-10-28T09:53:01Z
    ```

### Install grds CLI tool

`grds` cli is a command tool for k8s database operator management,we recommend using `grds` to manage database operator in terminal console.
After database operator deployment,you can run following command to install `grds`:

```shell
curl https://raw.githubusercontent.com/GrdsCloud/grds/{{< param operatorVersion >}}/installers/kubectl/client-setup.sh > client-setup.sh
chmod +x client-setup.sh
./client-setup.sh
```
It will prompt for some environment variables,you should add these environment variables to bash profile:

```
cat <<EOF >> ~/.bash_profile
export GRDS_CA_CERT="${HOME?}/.grds/grds/client.crt"
export GRDS_CLIENT_CERT="${HOME?}/.grds/grds/client.crt"
export GRDS_CLIENT_KEY="${HOME?}/.grds/grds/client.key"
export GRDS_APISERVER_URL='https://127.0.0.1:8443'
export GRDS_NAMESPACE=grds
EOF

source ~/.bash_profile
```

### Create a MySQL Cluster using grds CLI tool

1. **Create database**

Example 1:
create a single instance MySQL 8.0 `my-single` in grds namespace,CPU/memory is unlimited.

```shell
$ grds mysql add my-single \
  --pvc-size=10Gi --replica-count=1 --slave-count=0 -n grds --version=8.0
MySQL cluster created, name: [my-single] namespace [grds].
```

Example 2:
create a high availability MySQL cluster `my-hacluster` with 2 primary candidates,1 read-only slave in grds namespace,
max 2 vCPU,2GB memory

```shell
$ grds mysql add my-hacluster \
  --cpu=0.5 --cpu-limit=2 --memory=2Gi --memory-limit=2Gi  \
  --pvc-size=10Gi --replica-count=2 --slave-count=1 -n grds
MySQL cluster created, name: [my-hacluster] namespace [grds].
```

2. **Add admin user**
```shell
$ grds mysql add user mysql-cluster-name \
  --username=mymanager1 --password=xnhhujmki09KU -n grds
MySQL user created, name: [mymanager1] password: [xnhhujmki09KU] namespace: [grds].
```


## Connect to MySQL Cluster

1. **Get connect information**

You can use `grds mysql show` command to get detail information of a MySQL cluster

```shell
$ grds  mysql show my-cluster-test  -w table

MYSQL CLUSTER INFO:

> Note: AccessIP is the access entrance of database cluster,firstly use a LoadBalanceIP,if LoadBalanceIP is null,use MasterNodeIP

+-----------------+-----------+---------+-------+-----------+-----------------+-------+
|      NAME       | NAMESPACE | REPLICA | SLAVE |  STATUS   |     ACCESSIP    | PORT  |
+-----------------+-----------+---------+-------+-----------+-----------------+-------+
| my-cluster-test |      grds |       2 |     1 | available |  10.10.120.174  | 30442 |
+-----------------+-----------+---------+-------+-----------+---------+-------+-------+
MYSQL DATABASE INFO:
+----------------------------+-----------+------------+----------+---------------+-------------+---------+---------------+---------+-----------+
|            NAME            | NAMESPACE | CPUREQUEST | CPULIMIT | MEMORYREQUEST | MEMORYLIMIT | PVCSIZE |   NODENAME    |  ROLE   |  STATUS   |
+----------------------------+-----------+------------+----------+---------------+-------------+---------+---------------+---------+-----------+
| my-cluster-test-replica0-0 |      grds |          0 |        2 |             0 |         2Gi |    10Gi | 10-10-120-174 |  master | available |
| my-cluster-test-replica1-0 |      grds |          0 |        2 |             0 |         2Gi |    10Gi | 10-10-120-133 | standby | available |
|   my-cluster-test-slave0-0 |      grds |          0 |        2 |             0 |         2Gi |    10Gi | 10-10-120-232 |   slave | available |
+----------------------------+-----------+------------+----------+---------------+-------------+---------+---------------+---------+-----------+
```

2. **connect MySQL database**

```
mysql -h <node-ip> -u root -p <root-password> -P <node-port>
```

You should then be greeted with the MySQL prompt:

```
$ mysql -h 10.10.120.174 -u adminuser -pxxxxx -P30442
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 5360
Server version: 5.7.26-log MySQL Community Server - (GPL)

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]>
```




