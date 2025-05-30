---
title: Performance Guidelines
description: This article contains a list of performance guidelines for the warehouse item in Microsoft Fabric.
author: WilliamDAssafMSFT
ms.author: wiassaf
ms.reviewer: xiaoyul
ms.date: 04/09/2025
ms.topic: conceptual
ms.custom:
---
# Performance guidelines in Fabric Data Warehouse

**Applies to:** [!INCLUDE [fabric-dw](includes/applies-to-version/fabric-dw.md)]

These are guidelines to help you understand performance of your [!INCLUDE [fabric-dw](includes/fabric-dw.md)] in [!INCLUDE [product-name](../includes/product-name.md)]. [!INCLUDE [fabric-dw](includes/fabric-dw.md)] in [!INCLUDE [product-name](../includes/product-name.md)] is a SaaS platform where activities like workload management, concurrency, and storage management are managed internally by the platform. In addition to this internal performance management, you can still improve your performance by developing performant queries against well-designed warehouses.

## Cold run (cold cache) performance

[Caching with local SSD and memory](caching.md) is automatic. The first 1-3 executions of a query perform noticeably slower than subsequent executions. If you are experiencing cold run performance issues, here are a couple of things you can do that can improve your cold run performance:

- If the first run's performance is crucial, try manually creating statistics. Review the [statistics](statistics.md) article to better understand the role of statistics and for guidance on how to create manual statistics to improve your query performance. However, if the first run's performance is not critical, you can rely on automatic statistics that will be generated in the first query and will continue to be used in subsequent runs (so long as underlying data does not change significantly).

- If using Power BI, use [Direct Lake](../fundamentals/lakehouse-power-bi-reporting.md) mode where possible.
 
## Metrics for monitoring performance

Currently, the [Monitoring Hub](../admin/monitoring-hub.md) does not include [!INCLUDE [fabric-dw](includes/fabric-dw.md)]. If you choose **Data Warehouse**, you will not be able to access the **Monitoring Hub** from the navigation bar.

Fabric administrators will be able to access the **Capacity Utilization and Metrics** report for up-to-date information tracking the utilization of capacity that includes [!INCLUDE [fabric-dw](includes/fabric-dw.md)].

## Use dynamic management views (DMVs) to monitor query execution

You can use [dynamic management views (DMVs)](monitor-using-dmv.md) to monitor connection, session, and request status in the [!INCLUDE [fabric-dw](includes/fabric-dw.md)].

## Statistics

The [!INCLUDE [fabric-dw](includes/fabric-dw.md)] uses a query engine to create an execution plan for a given SQL query. When you submit a query, the query optimizer tries to enumerate all possible plans and choose the most efficient candidate. To determine which plan would require the least overhead, the engine needs to be able to evaluate the amount of work or rows that might be processed by each operator. Then, based on each plan's cost, it chooses the one with the least amount of estimated work. Statistics are objects that contain relevant information about your data, to allow the query optimizer to estimate these costs.

You can also [manually update statistics](statistics.md#manual-statistics-for-all-tables) after each data load or data update to assure that the best query plan can be built.

For more information statistics and how you can augment the automatically created statistics, see [Statistics in Fabric data warehousing](statistics.md).

## Data ingestion guidelines

There are four options for data ingestion into a [!INCLUDE [fabric-dw](includes/fabric-dw.md)]:

- COPY (Transact-SQL)
- Data pipelines
- Dataflows
- Cross-warehouse ingestion

To help determine which option is best for you and to review some data ingestion best practices, review [Ingest data](ingest-data.md#data-ingestion-options).

## Group INSERT statements into batches (avoid trickle inserts)

A one-time load to a small table with an `INSERT` statement, such as shown in the following example, might be the best approach depending on your needs. However, if you need to load thousands or millions of rows throughout the day, singleton inserts aren't optimal.

```sql
INSERT INTO MyLookup VALUES (1, 'Type 1') 
```

For guidance on how to handle these trickle-load scenarios, see [Best practices for ingesting data](ingest-data.md#best-practices).

## Minimize transaction sizes

`INSERT`, `UPDATE`, and `DELETE` statements run in a transaction. When they fail, they must be rolled back. To reduce the potential for a long rollback, minimize transaction sizes whenever possible. Minimizing transaction sizes can be done by dividing `INSERT`, `UPDATE`, and `DELETE` statements into parts. For example, if you have an insert that you expect to take 1 hour, you can break up the insert into four parts. Each run will then be shortened to 15 minutes.

Consider using [CREATE TABLE AS SELECT (CTAS)](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse?view=fabric&preserve-view=true) to write the data you want to keep in a table rather than using DELETE. If a CTAS takes the same amount of time, it's safer to run since it has minimal transaction logging and can be canceled quickly if needed.

## Collocate client applications and Microsoft Fabric

If you're using client applications, make sure you're using [!INCLUDE [product-name](../includes/product-name.md)] in a region that's close to your client computer. Client application examples include Power BI Desktop, [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms), or [the mssql extension with Visual Studio Code](/sql/tools/visual-studio-code/mssql-extensions?view=fabric&preserve-view=true).

## Utilize star schema data design

A [star schema](dimensional-modeling-overview.md#star-schema-design) organizes data into [fact tables](dimensional-modeling-fact-tables.md) and [dimension tables](dimensional-modeling-dimension-tables.md). It facilitates analytical processing by denormalizing the data from highly normalized OLTP systems, ingesting transactional data, and enterprise master data into a common, cleansed, and verified data structure that minimizes joins at query time, reduces the number of rows read and facilitates aggregations and grouping processing.

For more [!INCLUDE [fabric-dw](includes/fabric-dw.md)] design guidance, see [Tables in data warehousing](tables.md).

## Reduce query result set sizes

Reducing query result set sizes helps you avoid client-side issues caused by large query results. The [SQL Query editor](sql-query-editor.md) results sets are limited to the first 10,000 rows to avoid these issues in this browser-based user interface. If you need to return more than 10,000 rows, use [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms) or [the mssql extension with Visual Studio Code](/sql/tools/visual-studio-code/mssql-extensions?view=fabric&preserve-view=true).

## Choose the best data type for performance

When defining your tables, use the smallest data type that supports your data as doing so will improve query performance. This recommendation is important for **char** and **varchar** columns. If the longest value in a column is 25 characters, then define your column as **varchar(25)**. Avoid defining all character columns with a large default length.

Use integer-based data types if possible. Sort, join, and group by operations complete faster on integers than on character data.

For supported data types and more information, see [data types](data-types.md#autogenerated-data-types-in-the-sql-analytics-endpoint).

## SQL analytics endpoint performance

For information and recommendations on performance of the [!INCLUDE [fabric-se](includes/fabric-se.md)], see [SQL analytics endpoint performance considerations](sql-analytics-endpoint-performance.md).

## Data compaction

Data compaction optimizes read operations by consolidating smaller Parquet files into fewer, larger files. This process also helps in efficiently managing deleted rows by eliminating them from immutable Parquet files. The data compaction process involves rewriting tables or segments of tables into new Parquet files that are optimized for performance. For more information, see [Blog: Automatic Data Compaction for Fabric Warehouse](https://blog.fabric.microsoft.com/blog/announcing-automatic-data-compaction-for-fabric-warehouse/).

The data compaction process is seamlessly integrated into the warehouse. As queries are executed, the system identifies tables that could benefit from compaction and performs necessary evaluations. There is no manual way to trigger data compaction.

## Related content

- [Query the SQL analytics endpoint or Warehouse in Microsoft Fabric](query-warehouse.md)
- [Limitations](limitations.md)
- [Troubleshoot the Warehouse](troubleshoot-fabric-data-warehouse.md)
- [Data types](data-types.md)
- [T-SQL surface area](tsql-surface-area.md)
- [Tables in data warehouse](tables.md)
- [Caching in Fabric data warehousing](caching.md)
