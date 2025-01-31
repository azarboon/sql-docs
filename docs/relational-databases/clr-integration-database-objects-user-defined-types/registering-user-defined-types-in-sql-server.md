---
title: "Registering User-Defined Types in SQL Server"
description: You must register a UDT before you install it in SQL Server. You must register the assembly and create the type in the database where you wish to use it.
author: rwestMSFT
ms.author: randolphwest
ms.date: 12/27/2024
ms.service: sql
ms.subservice: clr
ms.topic: "reference"
helpviewer_keywords:
  - "UDTs [CLR integration], maintaining"
  - "user-defined types [CLR integration], maintaining"
  - "dependencies [CLR integration]"
  - "deploying user-defined types [CLR integration]"
  - "CurrencyConversion function"
  - "user-defined types [CLR integration], deploying"
  - "Transact-SQL deploying UDTs"
  - "assemblies [CLR integration], user-defined types"
  - "cross-database UDT support"
  - "CREATE ASSEMBLY statement"
  - "DROP TYPE statement"
  - "Currency UDT"
  - "CREATE TYPE statement"
  - "registering user-defined types"
  - "UDTs [CLR integration], deploying"
  - "removing user-defined types"
  - "user-defined types [CLR integration], registering"
  - "ALTER ASSEMBLY statement"
  - "UDTs [CLR integration], registering"
  - "ADD FILE clause"
dev_langs:
  - "TSQL"
---
# Register user-defined types in SQL Server

[!INCLUDE [SQL Server](../../includes/applies-to-version/sqlserver.md)]

In order to use a user-defined type (UDT) in [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)], you must register it. Registering a UDT involves registering the assembly and creating the type in the database in which you wish to use it. UDTs are scoped to a single database, and can't be used in multiple databases unless the identical assembly and UDT are registered with each database. Once the UDT assembly is registered and the type created, you can use the UDT in [!INCLUDE [tsql](../../includes/tsql-md.md)] and in client code. For more information, see [CLR user-defined types](clr-user-defined-types.md).

## Use visual studio to deploy UDTs

The easiest way to deploy your UDT is by using Visual Studio. For more complex deployment scenarios and the greatest flexibility, however, use [!INCLUDE [tsql](../../includes/tsql-md.md)] as discussed later in this article.

Follow these steps to create and deploy a UDT using Visual Studio:

1. Create a new **Database** project in the **Visual Basic** or **Visual C#** language nodes.

1. Add a reference to the [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] database that will contain the UDT.

1. Add a **User-Defined Type** class.

1. Write code to implement the UDT.

1. From the **Build** menu, select **Deploy**. This registers the assembly and creates the type in the [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] database.

## Use Transact-SQL to deploy UDTs

The [!INCLUDE [tsql](../../includes/tsql-md.md)] `CREATE ASSEMBLY` syntax is used to register the assembly in the database in which you wish to use the UDT. It's stored internally in database system tables, not externally in the file system. If the UDT is dependent on external assemblies, they too must be loaded into the database. The `CREATE TYPE` statement is used to create the UDT in the database in which it's to be used. For more information, see [CREATE ASSEMBLY](../../t-sql/statements/create-assembly-transact-sql.md) and [CREATE TYPE](../../t-sql/statements/create-type-transact-sql.md).

### Use create assembly

The `CREATE ASSEMBLY` syntax registers the assembly in the database in which you wish to use the UDT. Once the assembly is registered, it has no dependencies.

Creating multiple versions of the same assembly in a given database isn't allowed. However, it's possible to create multiple versions of the same assembly based on culture in a given database. [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] distinguishes multiple culture versions of an assembly by different names as registered in the instance of [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)]. For more information, see [Create and use strong-named assemblies](/dotnet/standard/assembly/create-use-strong-named).

When `CREATE ASSEMBLY` is executed with the `SAFE` or `EXTERNAL_ACCESS` permission sets, the assembly is checked to make sure that it's verifiable and type safe. If you omit specifying a permission set, `SAFE` is assumed. Code with the `UNSAFE` permission set isn't checked. For more information about assembly permission sets, see [Design assemblies](../clr-integration/assemblies-designing.md).

#### Example

The following [!INCLUDE [tsql](../../includes/tsql-md.md)] statement registers the `Point` assembly in [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] in the [!INCLUDE [sssampledbobject-md](../../includes/sssampledbobject-md.md)] database, with the `SAFE` permission set. If the `WITH PERMISSION_SET` clause is omitted, the assembly is registered with the `SAFE` permission set.

```sql
USE AdventureWorks2022;

CREATE ASSEMBLY Point
    FROM '\\ShareName\Projects\Point\bin\Point.dll'
    WITH PERMISSION_SET = SAFE;
```

The following [!INCLUDE [tsql](../../includes/tsql-md.md)] statement registers the assembly using *<assembly_bits>* argument in the `FROM` clause. This **varbinary** value represents the file as a stream of bytes.

```sql
USE AdventureWorks2022;
CREATE ASSEMBLY Point
FROM 0xfeac4 ... 21ac78
```

### Use create type

Once the assembly is loaded into the database, you can then create the type using the [!INCLUDE [tsql](../../includes/tsql-md.md)] `CREATE TYPE` statement. This adds the type to the list of available types for that database. The type has database scope and the type can only be used in the database in which it was created. If the UDT already exists in the database, the `CREATE TYPE` statement fails with an error.

> [!NOTE]  
> The `CREATE TYPE` syntax is also used for creating native [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] alias data types, and is intended to replace `sp_addtype` as a means of creating alias data types. Some of the optional arguments in the `CREATE TYPE` syntax refer to creating UDTs, and aren't applicable to creating alias data types (such as base type).

For more information, see [CREATE TYPE](../../t-sql/statements/create-type-transact-sql.md).

#### Example

The following [!INCLUDE [tsql](../../includes/tsql-md.md)] statement creates the `Point` type. The `EXTERNAL NAME` is specified using the two-part naming syntax of `<assembly_name>.<udt_name>`.

```sql
CREATE TYPE dbo.Point
EXTERNAL NAME Point.[Point];
```

## Remove a UDT from the database

The `DROP TYPE` statement removes a UDT from the current database. Once a UDT is dropped, you can use the `DROP ASSEMBLY` statement to drop the assembly from the database.

The `DROP TYPE` statement doesn't execute in the following situations:

- Tables in the database that contain columns defined using the UDT.

- Functions, stored procedures, or triggers that use variables or parameters of the UDT, created in the database with the `WITH SCHEMABINDING` clause.

### Example

The following [!INCLUDE [tsql](../../includes/tsql-md.md)] must execute in the following order. First the table which references the `Point` UDT must be dropped, then the type, and finally the assembly.

```sql
DROP TABLE dbo.Points;
DROP TYPE dbo.Point;
DROP ASSEMBLY Point;
```

### Find UDT dependencies

If there are dependent objects, such as tables with UDT column definitions, the `DROP TYPE` statement fails. It also fails if there are functions, stored procedures, or triggers created in the database using the `WITH SCHEMABINDING` clause, if these routines use variables or parameters of the user-defined type. You must first drop all dependent objects, and then execute the `DROP TYPE` statement.

The following [!INCLUDE [tsql](../../includes/tsql-md.md)] query locates all of the columns and parameters that use a UDT in the [!INCLUDE [sssampledbobject-md](../../includes/sssampledbobject-md.md)] database.

```sql
USE AdventureWorks2022;

SELECT o.name AS major_name,
       o.type_desc AS major_type_desc,
       c.name AS minor_name,
       c.type_desc AS minor_type_desc,
       at.assembly_class
FROM (SELECT object_id,
             name,
             user_type_id,
             'SQL_COLUMN' AS type_desc
      FROM sys.columns
      UNION ALL
      SELECT object_id,
             name,
             user_type_id,
             'SQL_PROCEDURE_PARAMETER'
      FROM sys.parameters) AS c
     INNER JOIN sys.objects AS o
         ON o.object_id = c.object_id
     INNER JOIN sys.assembly_types AS at
         ON at.user_type_id = c.user_type_id;
```

## Maintain UDTs

You can't modify a UDT once it's created in a [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] database, although you can alter the assembly on which the type is based. In most cases, you must remove the UDT from the database with the [!INCLUDE [tsql](../../includes/tsql-md.md)] `DROP TYPE` statement, make changes to the underlying assembly, and reload it using the `ALTER ASSEMBLY` statement. You then need to re-create the UDT and any dependent objects.

### Example

The `ALTER ASSEMBLY` statement is used after you have made changes to the source code in your UDT assembly and recompiled it. It copies the .dll file to the server and rebinds to the new assembly. For the complete syntax, see [ALTER ASSEMBLY](../../t-sql/statements/alter-assembly-transact-sql.md).

The following [!INCLUDE [tsql](../../includes/tsql-md.md)] `ALTER ASSEMBLY` statement reloads the Point.dll assembly from the specified location on disk.

```sql
ALTER ASSEMBLY Point
    FROM '\\Projects\Point\bin\Point.dll';
```

### Use alter assembly to add source code

The `ADD FILE` clause in the `ALTER ASSEMBLY` syntax isn't present in `CREATE ASSEMBLY`. You can use it to add source code or any other files associated with an assembly. The files are copied from their original locations and stored in system tables in the database. This ensures that you always have source code or other files on hand should you ever need to re-create or document the current version of the UDT.

The following [!INCLUDE [tsql](../../includes/tsql-md.md)] `ALTER ASSEMBLY` statement adds the Point.cs class source code for the `Point` UDT. This copies the text contained in the Point.cs file and stores it in the database under the name `PointSource`.

```sql
ALTER ASSEMBLY Point
ADD FILE FROM '\\Projects\Point\Point.cs' AS PointSource;
```

Assembly information is stored in the `sys.assembly_files` table in the database where the assembly has been installed. The `sys.assembly_files` table contains the following columns.

| Column | Description |
| --- | --- |
| `assembly_id` | The identifier defined for the assembly. This number is assigned to all objects relating to the same assembly. |
| `name` | The name of the object. |
| `file_id` | A number identifying each object, with the first object associated with a given `assembly_id` being given the value of `1`. If there are multiple objects associated with the same `assembly_id`, then each subsequent `file_id` value increments by `1`. |
| `content` | The hexadecimal representation of the assembly or file. |

You can use the `CAST` or `CONVERT` function to convert the contents of the `content` column to readable text. The following query converts the contents of the `Point.cs` file to readable text, using the name in the `WHERE` clause to restrict the result set to a single row.

```sql
SELECT CAST (content AS VARCHAR (8000))
FROM sys.assembly_files
WHERE name = 'PointSource';
```

If you copy and paste the results into a text editor, you see that the line breaks and spaces that existed in the original are preserved.

## Manage UDTs and assemblies

When planning your implementation of UDTs, consider which methods are needed in the UDT assembly itself, and which methods should be created in separate assemblies and implemented as user-defined functions or stored procedures. Separating methods into separate assemblies allows you to update code without affecting data that might be stored in a UDT column in a table. You can modify UDT assemblies without dropping UDT columns and other dependent objects only when the new definition can read the former values and the signature of the type doesn't change.

Separating procedural code that might change from the code required to implement the UDT greatly simplifies maintenance. Including only code that is necessary for the UDT to function, and keeping your UDT definitions as simple as possible, reduces the risk that the UDT itself might need to be dropped from the database for code revisions or bug fixes.

### The currency UDT and currency conversion function

The `Currency` UDT in the [!INCLUDE [sssampledbobject-md](../../includes/sssampledbobject-md.md)] sample database provides a useful example of the recommended way to structure a UDT and its associated functions. The `Currency` UDT is used for handling money based on the monetary system of a particular culture, and allows for storage of different currency types, such as dollars, euros, and so forth. The UDT class exposes a culture name as a string, and an amount of money as a **decimal** data type. All of the necessary serialization methods are contained within the assembly defining the class. The function that implements currency conversion from one culture to another is implemented as an external function named `ConvertCurrency`, and this function is located in a separate assembly. The `ConvertCurrency` function does its work by retrieving the conversion rate from a table in the [!INCLUDE [sssampledbobject-md](../../includes/sssampledbobject-md.md)] database. If the source of the conversion rates should ever change, or if there should be any other changes to the existing code, the assembly can be easily modified without affecting the `Currency` UDT.

The code listing for the `Currency` UDT and `ConvertCurrency` functions can be found by installing the common language runtime (CLR) samples.

### Use UDTs across databases

UDTs are by definition scoped to a single database. Therefore, a UDT defined in one database can't be used in a column definition in another database. In order to use UDTs in multiple databases, you must execute the `CREATE ASSEMBLY` and `CREATE TYPE` statements in each database on identical assemblies. Assemblies are considered identical if they have the same name, strong name, culture, version, permission set, and binary contents.

Once the UDT is registered and accessible in both databases, you can convert a UDT value from one database for use in another. Identical UDTs can be used across databases in the following scenarios:

- Calling stored procedure defined in different databases.

- Querying tables defined in different databases.

- Selecting UDT data from one database table UDT column and inserting it into a second database with an identical UDT column.

In these situations, any conversion required by the server occurs automatically. You aren't able to perform the conversions explicitly using the [!INCLUDE [tsql](../../includes/tsql-md.md)] `CAST` or `CONVERT` functions.

You don't need to take any action for using UDTs when [!INCLUDE [ssDEnoversion](../../includes/ssdenoversion-md.md)] creates work tables in the `tempdb` system database. This includes the handling of cursors, table variables, and user-defined table-valued functions that include UDTs and that transparently make use of `tempdb`. However, if you explicitly create a temporary table in `tempdb` that defines a UDT column, then the UDT must be registered in `tempdb` the same way as for a user database.

## Related content

- [CLR user-defined types](clr-user-defined-types.md)
