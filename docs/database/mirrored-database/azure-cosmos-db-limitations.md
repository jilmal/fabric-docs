---
title: "Limits and Quotas in Microsoft Fabric Mirrored Databases From Azure Cosmos DB (Preview)"
description: This article includes a list of limitations and quotas for Microsoft Fabric mirrored databases from Azure Cosmos DB.
author: seesharprun
ms.author: sidandrews
ms.reviewer: anithaa, wiassaf
ms.date: 05/06/2025
ms.topic: limits-and-quotas
ms.custom:
  - references_regions
---

# Limitations in Microsoft Fabric mirrored databases from Azure Cosmos DB (Preview)

This article details the current limitations for Azure Cosmos DB accounts mirrored into Microsoft Fabric. The limitation and quota details on this page are subject to change in the future.

> [!IMPORTANT]
> Mirroring for Azure Cosmos DB is currently in [preview](../../fundamentals/preview.md). Production workloads aren't supported during preview. Currently, only Azure Cosmos DB for NoSQL accounts are supported.

## Availability

Mirroring is supported in a specific set of regions for Fabric and APIs for Azure Cosmos DB.

### Supported APIs

Mirroring is only available for the Azure Cosmos DB account types listed here.

| | Available |
| --- | --- |
| **API for NoSQL** | Yes |
| **API for MongoDB (RU-based)** | No |
| **API for MongoDB (vCore-based)** | No |
| **API for Apache Gremlin** | No |
| **API for Table** | No |
| **API for Apache Cassandra (RU-based)** | No |
| **Managed Instance for Apache Cassandra** | No |

### Supported regions

[!INCLUDE [fabric-mirroreddb-supported-regions](includes/fabric-mirroreddb-supported-regions.md)]

## Account and database limitations

- You can enable mirroring only if the Azure Cosmos DB account is configured with either 7-day or 30-day continuous backup.
- All current limitations of the continuous backup feature in Azure Cosmos DB also apply to Fabric mirroring.
  - These limitations include, but aren't limited to; the inability to disable continuous backup once enabled and lack of support for multi-region write accounts. For more information, see [Azure Cosmos DB continuous backup limitations](/azure/cosmos-db/continuous-backup-restore-introduction#current-limitations).
  - You can enable both the analytical store and continuous backup features on the same Azure Cosmos DB account.
- You can't disable the analytical store feature on Azure Cosmos DB accounts with continuous backup enabled.
- You can't enable continuous backup on an Azure Cosmos DB account that previously disabled the analytical store feature for a container.


## Security limitations

- Azure Cosmos DB read-write account keys and Microsoft Entra ID authentication with role-based access control are the only supported mechanisms to connect to the source account. Read-only account keys and managed identities aren't supported.
  - For Microsoft Entra ID authentication, the following RBAC permissions are required:
  `Microsoft.DocumentDB/databaseAccounts/readMetadata`
  `Microsoft.DocumentDB/databaseAccounts/readAnalytics`
  
  To learn more, please visit our [data plane role-based access control documentation](/azure/cosmos-db/nosql/how-to-grant-data-plane-access).
- You must update the connection credentials for Fabric mirroring if the account keys are rotated. If you don't update the keys, mirroring fails. To resolve this failure, stop replication, update the credentials with the newly rotated keys, and then restart replication.
- Fabric users with access to the workspace automatically inherit access to the mirror database. However, you can granularly control workspace and tenant level access to manage access for users in your organization.
- You can directly share the mirrored database in Fabric.

### Permissions

- If you only have viewer permissions in Fabric, you can't preview or query data in the SQL analytics endpoint.
- If you intend to use the data explorer, the Azure Cosmos DB data explorer doesn't use the same permissions as Fabric. Requests to view and query data using the data explorer are routed to Azure instead of Fabric.

### Network security

- The source Azure Cosmos DB account must enable **public network access** for **all networks**.
- Private endpoints aren't supported for Azure Cosmos DB accounts.
- Network isolation using techniques and features like IP addresses or service endpoints aren't supported for Azure Cosmos DB accounts.
- Data in OneLake doesn't support private endpoints, customer managed keys, or double encryption.

## Data explorer limitations

- Fabric Data Explorer queries are read-only. You can view existing containers, view items, and query items.
- You can't create or delete containers using the data explorer in Fabric.
- You can't insert, modify, or delete items using the data explorer in Fabric.
- You can avoid sharing the source database by only sharing the SQL analytics endpoint with other users for analytics.
- You can't turn off the data explorer in a mirrored database.

## Replication limitations

- Mirroring doesn't support containers that contain items with property names containing either whitespaces or wild-card characters. This limitation causes mirroring for the specific container to fail. Other containers within the same databases can still successfully mirror. If property names are updated to remove these invalid characters, you must configure a new mirror to the same database and container and you can't use the old mirror.
- Fabric OneLake mirrors from the geographically closest Azure region to Fabric's capacity region in scenarios where an Azure Cosmos DB account has multiple read regions. In disaster recovery scenarios, mirroring automatically scans and picks up new read regions as your read regions could potentially fail over and change.
- Delete operations in the source container are immediately reflected in Fabric OneLake using mirroring. Soft-delete operations using time-to-live (TTL) values isn't supported.
- Mirroring doesn't support custom partitioning.
- Fabric has existing limitations with T-SQL. For more information, see [T-SQL limitations](../../data-warehouse/tsql-surface-area.md#limitations).

### Schema and data changes

- Deleting and adding a similar container replaces the data in the warehouse tables with only the new container's data.
- Changing the type of data in a property across multiple items cause the replicator to upcast the data where applicable. This behavior is in parity with the native delta experience. Any data that doesn't fit into the supported criteria become a null type. For example, changing an array property to a string upcasts to a null type.
- Adding new properties to items cause mirroring to seamlessly detect the new properties and add corresponding columns to the warehouse table. If item properties are removed or missing, they have a null value for the corresponding record.
- Replicating data using mirroring doesn't have a full-fidelity or well-defined schema. Mirroring automatically and continuously tracks property changes and data type (when allowed).

### Nested data

- Nested JSON objects in Azure Cosmos DB items are represented as JSON strings in warehouse tables.
- Commands such as `OPENJSON`, `CROSS APPLY`, and `OUTER APPLY` are available to expand JSON string data selectively.
  - Auto schema inference through `OPENJSON` allows you to flatten and explore nested data with unknown or unpredictable nested schemas. For more information, see [how to query nested data](azure-cosmos-db-how-to-query-nested.md).
- PowerQuery includes `ToJson` to expand JSON string data selectively.
- Mirroring doesn't have schema constraints on the level of nesting. For more information, see [Azure Cosmos DB analytical store schema constraints](/azure/cosmos-db/analytical-store-introduction#schema-constraints).

## Data warehouse limitations

- Warehouse can't handle JSON string columns greater than 8 KB in size. The error message for this scenario is **"JSON text is not properly formatted. Unexpected character '"' is found at position"**.
  - A current workaround is to create a shortcut of your mirrored database in Fabric Lakehouse and utilize a Spark Notebook to query your data to avoid this limitation.
- Nested data represented as a JSON string in SQL analytics endpoint and warehouse tables can commonly cause the column to increase to more than 8 KB in size. Monitoring levels of nesting and the amount of data if you receive this error message.

## Mirrored item limitations

- Enabling mirroring for an Azure Cosmos DB account in a workspace requires either the **admin** or **member** role in your workspace.
- Stopping replication disables mirroring completely.
- Starting replication again reseeds all of the target warehouse tables. This operation effectively starts mirroring from scratch.

## Give feedback

If you would like to give feedback on current limitations, features, or issues; let us know at [fabriccosmosdbmirror@microsoft.com](mailto:fabriccosmosdbmirror@microsoft.com).

## Related content

- [Mirroring Azure Cosmos DB (Preview)](azure-cosmos-db.md)
- [FAQ: Microsoft Fabric mirrored databases from Azure Cosmos DB](azure-cosmos-db-faq.yml)
- [Troubleshooting: Microsoft Fabric mirrored databases from Azure Cosmos DB](azure-cosmos-db-troubleshooting.yml)
