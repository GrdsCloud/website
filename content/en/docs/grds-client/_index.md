---
title: "Using Grds CLI"
linkTitle: "Using Grds CLI"
weight: 3
description: >
  Here you can see the user manual for grds cli
---

## Global
**Command**：

| COMMAND       | DESCRIPTION                        |
| ------------- | ---------------------------------- |
| mysql | MySQL Cluster Lifecycle Management |
| backup | Backup Lifecycle Management |

**Flag**：

| CLI FLAG                   | DESCRIPTION                                                  |
| -------------------------- | ------------------------------------------------------------ |
| --endpoint                 | gRPC endpoints (default "127.0.0.1:8443")                    |
| --namespace                | The namespace to use for grds requests. (default "grds")     |
| --write-out                | set the output format ( simple, table) (default "simple")    |
| --cacert                   | verify certificates of TLS-enabled secure servers using this CA bundle |
| --cert                     | identify secure client using this TLS certificate file       |
| --key                      | identify secure client using this TLS key file               |
| --insecure-skip-tls-verify | skip server certificate verification (CAUTION: this option should be enabled only for testing purposes) |
| --debug                    | enable client-side debug logging                             |