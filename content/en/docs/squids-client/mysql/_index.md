---
title: "Squids MySQL Command"
linkTitle: "MySQL"
weight: 1
description: >
   The lifecycle of managed mysql clusters.
---

### MySQL Cluster Lifecycle Management

**Usage**:

```shell script
squidsctl mysql [command]
```

**Command**：

| COMMAND | DESCRIPTION                                 |
| ------- | ------------------------------------------- |
| add     | Create a mysql cluster into the k8s cluster |
| delete  | Delete a mysql cluster from the k8s cluster |
| list    | List mysql cluster in the cluster           |
| update  | Update a mysql cluster in the k8s cluster   |
| show    | Show a mysql cluster in detail              |