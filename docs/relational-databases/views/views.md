---
title: "Views"
description: "Learn about views, important database objects where the result set is defined by a query."
author: WilliamDAssafMSFT
ms.author: wiassaf
ms.date: 09/24/2024
ms.service: sql
ms.subservice: table-view-index
ms.topic: conceptual
ms.custom:
  - ignite-2024
helpviewer_keywords:
  - "views [SQL Server], about views"
monikerRange: ">=aps-pdw-2016 || =azuresqldb-current || =azure-sqldw-latest || >=sql-server-2016 || >=sql-server-linux-2017 || =azuresqldb-mi-current || =fabric"
---
# Views

[!INCLUDE [sql-asdb-asdbmi-asa-pdw-fabricse-fabricdw-fabricsqldb](../../includes/applies-to-version/sql-asdb-asdbmi-asa-pdw-fabricse-fabricdw-fabricsqldb.md)]

  A view is a virtual table whose contents are defined by a query. Like a table, a view consists of a set of named columns and rows of data. Unless indexed, a view does not exist as a stored set of data values in a database. The rows and columns of data come from tables referenced in the query defining the view and are produced dynamically when the view is referenced.  
  
 A view acts as a filter on the underlying tables referenced in the view. The query that defines the view can be from one or more tables or from other views in the current or other databases. Distributed queries can also be used to define views that use data from multiple heterogeneous sources. This is useful, for example, if you want to combine similarly structured data from different servers, each of which stores data for a different region of your organization.  
  
 Views are generally used to focus, simplify, and customize the perception each user has of the database. Views can be used as security mechanisms by letting users access data through the view, without granting users permissions to directly access the underlying tables of the query. Views can be used to provide a backward compatible interface to emulate a table that used to exist but whose schema has changed. Views can also be used when you copy data to and from [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] to improve performance and to partition data.  
  
## Types of views

Besides the standard role of basic user-defined views, [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] provides the following types of views that serve special purposes in a database.  
  
### Indexed views

 An indexed view is a materialized view. This means the view definition has been computed and the resulting data stored just like a table. You index a view by creating a unique clustered index on it. Indexed views can dramatically improve the performance of some types of queries. Indexed views work best for queries that aggregate many rows. They are not well-suited for underlying data sets that are frequently updated.  
  
### Partitioned views

A partitioned view joins horizontally partitioned data from a set of member tables across one or more servers. A partitioned view makes the data appear as if from one table. A view that joins member tables on the same instance of [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] is a local partitioned view.  
  
### System views

System views expose catalog metadata. You can use system views to return information about the instance of [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] or the objects defined in the instance. For example, you can query the `sys.databases` catalog view to return information about the user-defined databases available in the instance. For more information, see [System Views (Transact-SQL)](../../t-sql/language-reference.md).
  
## Common view tasks

 The following table provides links to common tasks associated with creating or modifying a view.  
  
|View Tasks|Article|  
|----------------|-----------|  
|Describes how to create a view.|[Create Views](../../relational-databases/views/create-views.md)|  
|Describes how to create an indexed view.|[Create Indexed Views](../../relational-databases/views/create-indexed-views.md)|  
|Describes how to modify the view definition.|[Modify Views](../../relational-databases/views/modify-views.md)|  
|Describes how to modify data through a view.|[Modify Data Through a View](../../relational-databases/views/modify-data-through-a-view.md)|  
|Describes how to delete a view.|[Delete Views](../../relational-databases/views/delete-views.md)|  
|Describes how to return information about a view such as the view definition.|[Get Information About a View](../../relational-databases/views/get-information-about-a-view.md)|  
|Describes how to rename a view.|[Rename Views](../../relational-databases/views/rename-views.md)|  
  
## Related content

- [Create Views over XML Columns](../../relational-databases/xml/create-views-over-xml-columns.md)
- [CREATE VIEW (Transact-SQL)](../../t-sql/statements/create-view-transact-sql.md)
- [GRANT Object Permissions (Transact-SQL)](../../t-sql/statements/grant-object-permissions-transact-sql.md)
- [Row-level security](../security/row-level-security.md)
