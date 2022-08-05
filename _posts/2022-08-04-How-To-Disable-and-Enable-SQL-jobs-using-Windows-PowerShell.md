---
layout: post
title: How to disable and enable SQL jobs using Windows PowerShell.
---
**In this article we use Windows PowerShell to enable and disable SQL jobs running on Microsoft SQL Server.**

**Problem being solved:**
Usually to disable or enable SQL jobs, I would open the Microsoft SQL Server Management Studio console, navigate to SQL Server Agent>Job Activity Monitor[double click it] to get a display of the jobs, proceed to select the jobs I want to disable/enable and right click and select "Disable job". That can be time consuming, so here are 2 PowerShell scripts that I use to enable and disable SQL jobs without ever opening the Microsoft SQL Server Management Studio console.

The zipped folder conatining all the scripts used in this artcile is [here] (https://github.com/gracelugandakamya/this-worked-for-me/blob/main/enable-disable-sql-jobs.zip)
**Script to Disable SQL jobs using PowerShell:**
There are two parts to this script, the first part consists of the actual SQL script that will query for all the sql jobs that have the name "sandbox"

**The sql script that does this is below:**
```
USE MSDB;
GO

DECLARE @job_id uniqueidentifier

DECLARE job_cursor CURSOR READ_ONLY FOR  
SELECT job_id
FROM msdb.dbo.sysjobs
WHERE enabled = 1
  AND [name] like N'Sandbox%'

OPEN job_cursor   
FETCH NEXT FROM job_cursor INTO @job_id  

WHILE @@FETCH_STATUS = 0
BEGIN
   EXEC msdb.dbo.sp_update_job @job_id = @job_id, @enabled = 0
   FETCH NEXT FROM job_cursor INTO @job_id  
END

CLOSE job_cursor   
DEALLOCATE job_cursor


-- This portion of the script will return the list of the specified sql jobs so we can see if the jobs are indeed disabled.
SELECT 
 job_id
,name
,enabled
,date_created
,date_modified
FROM msdb.dbo.sysjobs
WHERE enabled = 0
  AND [name] like N'Sandbox%'
ORDER BY date_created
```

The second part of the script consists of the actual PowerShell script that calls the SQL script, the PowerShell script that calls the sql script is below:

```
$ConnectionString = 'Server=SQLSERVER01;Database=msdb;Trusted_Connection=true'

Invoke-Sqlcmd -ConnectionString $ConnectionString -InputFile "C:\Temp\scripts\disable-enable-sql-jobs\Disable-SQL-Jobs-By-Job-Name.sql" | Export-Csv -Delimiter "," -Path "C:\Users\johndoe\Downloads\results.csv"
Import-Csv "C:\Users\johndoe\Downloads\results.csv" | Format-table
```
![The results returned by the above script to disable the SQL jobs](assets/img/disable-sql-jobs.png "disable sql jobs image")

**Script to Enable SQL jobs using PowerShell:**

Similarly to the disable sql jobs script, there are also two parts to this script, the first part consists of the actual SQL script that will query for all the sql jobs that have the name "sandbox". Once we have the SQL jobs, we have them get enabled
The sql script that does this is below:

```
USE MSDB;
GO

DECLARE @job_id uniqueidentifier

DECLARE job_cursor CURSOR READ_ONLY FOR  
SELECT job_id
FROM msdb.dbo.sysjobs
WHERE enabled = 0
AND [name] like N'Sandbox%'

OPEN job_cursor   
FETCH NEXT FROM job_cursor INTO @job_id  

WHILE @@FETCH_STATUS = 0
BEGIN
   EXEC msdb.dbo.sp_update_job @job_id = @job_id, @enabled = 1
   FETCH NEXT FROM job_cursor INTO @job_id  
END

CLOSE job_cursor   
DEALLOCATE job_cursor


-- This portion of the script will return the list of the specified sql jobs so we can see if the jobs are indeed Enabled.
SELECT 
 job_id
,name
,enabled
,date_created
,date_modified
FROM msdb.dbo.sysjobs
WHERE enabled = 1
  AND [name] like N'Sandbox%'
ORDER BY date_created

```

The second part of the script consists of the actual PowerShell script that calls the SQL script, the PowerShell script that calls the sql script is below:

```
$ConnectionString = 'Server=SQLSERVER01;Database=msdb;Trusted_Connection=true'

Invoke-Sqlcmd -ConnectionString $ConnectionString -InputFile "C:\Temp\scripts\disable-enable-sql-jobs\Enable-SQL-Jobs-By-Job-Name.sql" | Export-Csv -Delimiter "," -Path "C:\Users\johndoe\Downloads\results.csv"
Import-Csv "C:\Users\johndoe\Downloads\results.csv" | Format-table
```
![The results returned by the above script to enable the SQL jobs](assets/img/enable-sql-jobs.png "enable sql jobs image")
A few notes:
- Make sure that all these scripts are in the same location[directory, folder]
- Make sure to run the PowerShell scripts as Administrator
