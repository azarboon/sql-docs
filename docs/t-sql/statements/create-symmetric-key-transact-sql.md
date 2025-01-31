---
title: "CREATE SYMMETRIC KEY (Transact-SQL)"
description: Generates a symmetric key and specifies its properties in SQL Server and Azure SQL.
author: VanMSFT
ms.author: vanto
ms.reviewer: randolphwest
ms.date: 08/30/2023
ms.service: sql
ms.subservice: t-sql
ms.topic: reference
ms.custom:
  - ignite-2024
f1_keywords:
  - "CREATE SYMMETRIC KEY"
  - "SYMMETRIC KEP"
  - "CREATE_SYMMETRIC_KEY_TSQL"
  - "SYMMETRIC_KEY_TSQL"
helpviewer_keywords:
  - "CREATE SYMMETRIC KEY statement"
  - "temporary symmetric keys [SQL Server]"
  - "symmetric keys [SQL Server], creating"
  - "symmetric keys [SQL Server]"
dev_langs:
  - "TSQL"
monikerRange: "=azuresqldb-current || =azuresqldb-mi-current || >=sql-server-2016 || >=sql-server-linux-2017 || =azure-sqldw-latest || =fabric"
---
# CREATE SYMMETRIC KEY (Transact-SQL)

[!INCLUDE [SQL Server Azure SQL Database Azure SQL Managed Instance Synapse Analytics FabricSQLDB](../../includes/applies-to-version/sql-asdb-asdbmi-asa-fabricsqldb.md)]

Generates a symmetric key and specifies its properties in [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)].

This feature is incompatible with database export using Data Tier Application Framework (DACFx). You must drop all symmetric keys before exporting.

:::image type="icon" source="../../includes/media/topic-link-icon.svg" border="false"::: [Transact-SQL syntax conventions](../../t-sql/language-elements/transact-sql-syntax-conventions-transact-sql.md)

> [!NOTE]  
> [!INCLUDE [synapse-analytics-od-unsupported-syntax](../../includes/synapse-analytics-od-unsupported-syntax.md)]

## Syntax

```syntaxsql
CREATE SYMMETRIC KEY key_name
    [ AUTHORIZATION owner_name ]
    [ FROM PROVIDER provider_name ]
    WITH
        [
            <key_options> [ , ... n ]
            | ENCRYPTION BY <encrypting_mechanism> [ , ... n ]
        ]

<key_options> ::=
    KEY_SOURCE = 'pass_phrase'
    | ALGORITHM = <algorithm>
    | IDENTITY_VALUE = 'identity_phrase'
    | PROVIDER_KEY_NAME = 'key_name_in_provider'
    | CREATION_DISPOSITION = { CREATE_NEW | OPEN_EXISTING }

<algorithm> ::=
    DES | TRIPLE_DES | TRIPLE_DES_3KEY | RC2 | RC4 | RC4_128
    | DESX | AES_128 | AES_192 | AES_256

<encrypting_mechanism> ::=
    CERTIFICATE certificate_name
    | PASSWORD = 'password'
    | SYMMETRIC KEY symmetric_key_name
    | ASYMMETRIC KEY asym_key_name
```

## Arguments

#### *key_name*

Specifies the unique name by which the symmetric key is known in the database. This is a required parameter. Temporary keys are designated when the *key_name* begins with one number (`#`) sign. For example, `#temporaryKey900007`. You can't create a symmetric key that has a name that starts with more than one `#`. You can't create a temporary symmetric key using an EKM provider.

#### AUTHORIZATION *owner_name*

Specifies the name of the database user or application role that owns this key.

#### FROM PROVIDER *provider_name*

Specifies an Extensible Key Management (EKM) provider and name. The key isn't exported from the EKM device. The provider must be defined first using the `CREATE PROVIDER` statement. For more information about creating external key providers, see [Extensible Key Management (EKM)](../../relational-databases/security/encryption/extensible-key-management-ekm.md).

> [!NOTE]  
> This option isn't available in a contained database.

#### KEY_SOURCE = '*pass_phrase*'

Specifies a passphrase from which to derive the key.

#### IDENTITY_VALUE = '*_identity\_phrase_*'

Specifies an identity phrase from which to generate a GUID for tagging data that is encrypted with a temporary key.

#### PROVIDER_KEY_NAME = '*key_name_in_provider*'

Specifies the name referenced in the Extensible Key Management provider.

> [!NOTE]  
> This option isn't available in a contained database.

#### CREATION_DISPOSITION = { CREATE_NEW | OPEN_EXISTING }

Creates or maps a symmetric key to an Extensible Key Management device.

- `CREATE_NEW`

  Creates a new key on the Extensible Key Management device. If a key already exists on the device, the statement fails with an error.

- `OPEN_EXISTING`

  Maps a [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] symmetric key to an existing Extensible Key Management key. If `CREATION_DISPOSITION = OPEN_EXISTING` isn't provided, this defaults to `CREATE_NEW`.

#### *certificate_name*

Specifies the name of the certificate that is used to encrypt the symmetric key. The certificate must already exist in the database.

#### '*password*'

Specifies a password from which to derive a TRIPLE_DES key with which to secure the symmetric key. *password* must meet the Windows password policy requirements of the computer that is running the instance of [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)]. Always use strong passwords.

#### *symmetric_key_name*

Specifies a symmetric key, used to encrypt the key that is being created. The specified key must already exist in the database, and the key must be open.

#### *asym_key_name*

Specifies an asymmetric key, used to encrypt the key that is being created. This asymmetric key must already exist in the database.

#### \<algorithm>

Specify the encrypting algorithm.

> [!WARNING]  
> Beginning with [!INCLUDE [sssql16-md](../../includes/sssql16-md.md)], all algorithms other than AES_128, AES_192, and AES_256 are deprecated. To use older algorithms (not recommended), you must set the database to database compatibility level 120 or lower.

## Remarks

When a symmetric key is created, the symmetric key must be encrypted by using at least one of the following options:

- certificate
- password
- symmetric key
- asymmetric key
- PROVIDER

The key can have more than one encryption of each type. In other words, a single symmetric key can be encrypted by using multiple certificates, passwords, symmetric keys, and asymmetric keys at the same time.

> [!CAUTION]  
> When a symmetric key is encrypted with a password instead of a certificate (or another key), the TRIPLE DES encryption algorithm is used to encrypt the password. Because of this, keys that are created with a strong encryption algorithm, such as AES, are themselves secured by a weaker algorithm.

The optional password can be used to encrypt the symmetric key before distributing the key to multiple users.

Temporary keys are owned by the user that creates them. Temporary keys are only valid for the current session.

`IDENTITY_VALUE` generates a GUID with which to tag data that is encrypted with the new symmetric key. This tagging can be used to match keys to encrypted data. The GUID generated by a specific phrase is always the same. After a phrase has been used to generate a GUID, the phrase can't be reused as long as there's at least one symmetric key in this database that is actively using the phrase. `IDENTITY_VALUE` is an optional clause; however, we recommend using it when you're storing data encrypted with a temporary key.

There's no default encryption algorithm.

> [!IMPORTANT]  
> We don't recommend using the RC4 and RC4_128 stream ciphers to protect sensitive data. [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] doesn't further encode the encryption performed with such keys.

Information about symmetric keys is visible in the [sys.symmetric_keys](../../relational-databases/system-catalog-views/sys-symmetric-keys-transact-sql.md) catalog view.

Symmetric keys can't be encrypted by symmetric keys created from the encryption provider.

### Clarification regarding DES algorithms

- DESX was incorrectly named. Symmetric keys created with ALGORITHM = DESX actually use the TRIPLE DES cipher with a 192-bit key. The DESX algorithm isn't provided. [!INCLUDE [ssNoteDepFutureAvoid](../../includes/ssnotedepfutureavoid-md.md)]
- Symmetric keys created with ALGORITHM = TRIPLE_DES_3KEY use TRIPLE DES with a 192-bit key.
- Symmetric keys created with ALGORITHM = TRIPLE_DES use TRIPLE DES with a 128-bit key.

### Deprecation of the RC4 algorithm

Repeated use of the same RC4 or RC4_128 KEY_GUID on different blocks of data, results in the same RC4 key because [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] doesn't provide a salt automatically. Using the same RC4 key repeatedly is a well known error that results in very weak encryption. Therefore we have deprecated the RC4 and RC4_128 keywords. [!INCLUDE [ssNoteDepFutureAvoid](../../includes/ssnotedepfutureavoid-md.md)]

> [!WARNING]  
> The RC4 algorithm is only supported for backward compatibility. New material can only be encrypted using RC4 or RC4_128 when the database is in compatibility level 90 or 100. (Not recommended.) Use a newer algorithm such as one of the AES algorithms instead. In [!INCLUDE [ssnoversion](../../includes/ssnoversion-md.md)] material encrypted using RC4 or RC4_128 can be decrypted in any compatibility level.

## Permissions

Requires ALTER ANY SYMMETRIC KEY permission on the database. If `AUTHORIZATION` is specified, requires IMPERSONATE permission on the database user or ALTER permission on the application role. If encryption is by certificate or asymmetric key, requires VIEW DEFINITION permission on the certificate or asymmetric key. Only Windows logins, [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] logins, and application roles can own symmetric keys. Groups and roles can't own symmetric keys.

## Examples

### A. Create a symmetric key

The following example creates a symmetric key called `JanainaKey09` by using the `AES 256` algorithm, and then encrypts the new key with certificate `Shipping04`.

```sql
CREATE SYMMETRIC KEY JanainaKey09
WITH ALGORITHM = AES_256
ENCRYPTION BY CERTIFICATE Shipping04;
GO
```

### B. Create a temporary symmetric key

The following example creates a temporary symmetric key called `#MarketingXXV` from the passphrase: `The square of the hypotenuse is equal to the sum of the squares of the sides`. The key is provisioned with a GUID that is generated from the string `Pythagoras` and encrypted with certificate `Marketing25`.

```sql
CREATE SYMMETRIC KEY #MarketingXXV
WITH ALGORITHM = AES_128,
KEY_SOURCE
     = 'The square of the hypotenuse is equal to the sum of the squares of the sides',
IDENTITY_VALUE = 'Pythagoras'
ENCRYPTION BY CERTIFICATE Marketing25;
GO
```

### C. Create a symmetric key using an Extensible Key Management (EKM) device

The following example creates a symmetric key called `MySymKey` by using a provider called `MyEKMProvider` and a key name of `KeyForSensitiveData`. It assigns authorization to `User1` and assumes that the system administrator has already registered the provider called `MyEKMProvider` in [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)].

```sql
CREATE SYMMETRIC KEY MySymKey
AUTHORIZATION User1
FROM PROVIDER EKMProvider
WITH
PROVIDER_KEY_NAME='KeyForSensitiveData',
CREATION_DISPOSITION=OPEN_EXISTING;
GO
```

## See also

- [Choose an Encryption Algorithm](../../relational-databases/security/encryption/choose-an-encryption-algorithm.md)
- [ALTER SYMMETRIC KEY (Transact-SQL)](../../t-sql/statements/alter-symmetric-key-transact-sql.md)
- [DROP SYMMETRIC KEY (Transact-SQL)](../../t-sql/statements/drop-symmetric-key-transact-sql.md)
- [Encryption Hierarchy](../../relational-databases/security/encryption/encryption-hierarchy.md)
- [sys.symmetric_keys (Transact-SQL)](../../relational-databases/system-catalog-views/sys-symmetric-keys-transact-sql.md)
- [Extensible Key Management (EKM)](../../relational-databases/security/encryption/extensible-key-management-ekm.md)
- [Extensible Key Management Using Azure Key Vault (SQL Server)](../../relational-databases/security/encryption/extensible-key-management-using-azure-key-vault-sql-server.md)
