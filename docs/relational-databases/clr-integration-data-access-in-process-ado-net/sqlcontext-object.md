---
title: "SqlContext Object"
description: When you invoke managed code in SQL Server in a user connection, access to the context of the caller is abstracted in a SqlContext object.
author: rwestMSFT
ms.author: randolphwest
ms.date: 12/27/2024
ms.service: sql
ms.subservice: clr
ms.topic: "reference"
helpviewer_keywords:
  - "Windows identity [CLR integration]"
  - "SqlContext object"
  - "context [CLR integration]"
---
# SqlContext object

[!INCLUDE [SQL Server](../../includes/applies-to-version/sqlserver.md)]

You invoke managed code in the server when you call a procedure or function, when you call a method on a common language runtime (CLR) user-defined type, or when your action fires a trigger defined in any of the .NET Framework languages. Because execution of this code is requested as part of a user connection, access to the context of the caller from the code running in the server is required. In addition, certain data access operations might only be valid if run under the context of the caller. For example, access to inserted and deleted pseudo-tables used in trigger operations is only valid under the context of the caller.

The context of the caller is abstracted in a `SqlContext` object. For more information, see [Microsoft.SqlServer.Server.SqlContext](/dotnet/api/microsoft.sqlserver.server.sqlcontext).

`SqlContext` provides access to the following components.

| Component | Description |
| --- | --- |
| `SqlPipe` | This object represents the *pipe* through which results flow to the client. For more information, see [SqlPipe object](sqlpipe-object.md). |
| `SqlTriggerContext` | This object can only be retrieved from within a CLR trigger. It provides information about the operation that caused the trigger to fire and a map of the columns that were updated. For more information, see [SqlTriggerContext object](sqltriggercontext-object.md). |
| `IsAvailable` | This property is used to determine context availability. |
| `WindowsIdentity` | This property is used to retrieve the Windows identity of the caller. |

## Determine context availability

Query the `SqlContext` class to see if the currently executing code is running in-process, by checking the `IsAvailable` property of the `SqlContext` object. The `IsAvailable` property is read-only, and returns `True` if the calling code is running inside [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] and if other `SqlContext` members can be accessed. If the `IsAvailable` property returns `False`, all the other `SqlContext` members throw an `InvalidOperationException`, if used. If `IsAvailable` returns `False`, any attempt to open a connection object that has "context connection=true" in the connection string fails.

## Retrieve Windows identity

CLR code executing inside [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] is always invoked in the context of the process account. If the code should perform certain actions using the identity of the calling user, instead of the [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] process identity, then an impersonation token should be obtained through the `WindowsIdentity` property of the `SqlContext` object. The `WindowsIdentity` property returns a `WindowsIdentity` instance representing the Windows identity of the caller, or null if the client was authenticated using [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] Authentication. Only assemblies marked with `EXTERNAL_ACCESS` or `UNSAFE` permissions can access this property.

After they obtain the `WindowsIdentity` object, callers can impersonate the client account and perform actions on their behalf.

The identity of the caller is only available through `SqlContext.WindowsIdentity` if the client that initiated execution of the stored-procedure or function connected to the server using Windows Authentication. If [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] Authentication was used instead, this property is null and the code is unable to impersonate the caller.

### Example

The following example shows how to get the Windows identity of the calling client and impersonate the client.

### [C#](#tab/csharp)

```csharp
[Microsoft.SqlServer.Server.SqlProcedure]
public static void WindowsIDTestProc()
{
    WindowsIdentity clientId = null;
    WindowsImpersonationContext impersonatedUser = null;

    // Get the client ID.
    clientId = SqlContext.WindowsIdentity;

    // This outer try block is used to thwart exception filter
    // attacks which would prevent the inner finally
    // block from executing and resetting the impersonation.
    try
    {
        try
        {
            impersonatedUser = clientId.Impersonate();
            if (impersonatedUser != null)
            {
                // Perform some action using impersonation.
            }
        }
        finally
        {
            // Undo impersonation.
            if (impersonatedUser != null)
                impersonatedUser.Undo();
        }
    }
    catch
    {
        throw;
    }
}
```

### [Visual Basic .NET](#tab/vb)

```vb
<Microsoft.SqlServer.Server.SqlProcedure()> _
Public Shared Sub  WindowsIDTestProcVB ()
    Dim clientId As WindowsIdentity
    Dim impersonatedUser As WindowsImpersonationContext

    ' Get the client ID.
    clientId = SqlContext.WindowsIdentity

    ' This outer try block is used to thwart exception filter
    ' attacks which would prevent the inner finally
    ' block from executing and resetting the impersonation.

    Try
        Try

            impersonatedUser = clientId.Impersonate()

            If impersonatedUser IsNot Nothing Then
                ' Perform some action using impersonation.
            End If

        Finally
            ' Undo impersonation.
            If impersonatedUser IsNot Nothing Then
                impersonatedUser.Undo()
            End If
        End Try

    Catch e As Exception

        Throw e

    End Try
End Sub
```

---

## Related content

- [SqlPipe object](sqlpipe-object.md)
- [SqlTriggerContext object](sqltriggercontext-object.md)
- [CLR triggers](/dotnet/framework/data/adonet/sql/clr-triggers)
- [SQL Server in-process specific extensions to ADO.NET](sql-server-in-process-specific-extensions-to-ado-net.md)
