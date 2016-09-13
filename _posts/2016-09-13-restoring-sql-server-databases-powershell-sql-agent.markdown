---
layout: post
title:  "Restoring SQL Server Databases with Powershell in SQL Server Agent"
date:   2016-09-13
categories: sqlserver
---

On occasion, we need to restore a copy of the Production SQL database into the Development SQL Server, so that our developers can make use of recent data during the development of new ETL and reports.  As we're using [Ola Hallengren's SQL Server Backup scripts](https://ola.hallengren.com/sql-server-backup.html) and storing the backup files on a network share (thanks Brent!), we decided to use the SQL Agent again to restore these databases onto the development server.

To restore our databases, we need to find the most-recent Full backup for the database, and issue the restore command on the Development SQL Server.  The script below is a modified version of [Steve Thompson's script](https://stevethompsonmvp.wordpress.com/2013/09/18/using-a-powershell-script-to-automate-sql-server-database-restore/), so that it will work when scheduled within a SQL Agent job.  The main addition to the version below is the use of the `filesystem::` prefix when executing the `Get-ChildItem` command, otherwise the SQLPS interpreter in the SQL Agent [will use the wrong filesystem provider](http://stackoverflow.com/a/27725079/1103962).

```
$RestoreServerName="DevSqlServer"
$BaseDir = "filesystem::\\BackupServer\SqlServerBackup\ProdSqlServer"
$Databases = @("databaseA","databaseB")
ForEach ($Database in $Databases) {
  $LastFullBackup = Get-ChildItem "$BaseDir\$Database\FULL\" | Sort-Object -Property Name -Descending | Select-Object -First 1
  $BackupFileName = $LastFullBackup.FullName
  Write-Output "Backup to Restore: $BackupFileName"
  $SQLConn = New-Object System.Data.SQLClient.SQLConnection
  $SQLConn.ConnectionString = "Server=$RestoreServerName;Trusted_Connection=True"
  $SQLConn.Open()
  $SQLCmd = New-Object System.Data.SQLClient.SQLCommand
  $SQLCmd = $SQLconn.CreateCommand()
  $SQLCmd.commandtimeout=0
  $SQLCmd.CommandText="ALTER DATABASE $Database SET SINGLE_USER WITH ROLLBACK IMMEDIATE;
    RESTORE DATABASE $Database FROM DISK = '$BackupFileName' WITH REPLACE"
  $SQLCmd.Executenonquery() | Out-Null
  $SQLCmd.CommandText="ALTER DATABASE $Database SET MULTI_USER;"
  $SQLCmd.Executenonquery() | Out-Null
  $SQLConn.Close()
  Write-Output "Restore Complete: $Database"
}
```
