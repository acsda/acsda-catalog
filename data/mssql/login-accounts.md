# Data Architecture | Microsoft SQL Server | Login Accounts

Managing login accounts is one of the top essential configurations that must be done in any database systems.

## Group Managed Service Accounts (gMSA)

### Adding gMSA

* to add a gMSA for `GMSA_SAMPLE` account under `MY` AD domain

```sql
CREATE LOGIN [MY\GMSA_SAMPLE$] FROM WINDOWS
GO
```

### Removing gMSA

* to remove a gMSA for `GMSA_SAMPLE` account under `MY` AD domain

```sql
DROP LOGIN [MY\GMSA_SAMPLE$] FROM WINDOWS
GO
```
