
###### If you like it, please consider buy me a beer :beer:
###### [![paypal](https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=6NKR7XQH5E2P2&source=url)


# SnsMsSqlPsModule PowerShell Module

* This is a PowerShell module for working with [MS SQL Server](https://www.microsoft.com/en-us/sql-server/sql-server-downloads) DataBases, based on my previous project related with SQLite serverless DataBases, named ["SnsSqlitePsModule"](https://github.com/svesavov/SnsSqlitePsModule). Working with SQLite is nice and free, it is good for learning SQL, and decent for usage within production automations and scripts, to keep temporary or configuration data (If the machine that hosts the automation have fast Hard Disks). However where it comes to large volumes of data, accessed by multiple automations or scripts, arises the need of some more powerful Server / Service based DataBase, capable to better manage the DataBase locks.
* In general the PowerShell binary CmdLets are much faster comparing with the written-on PowerShell ones. Which was my main argument to make this PowerShell module a Binary Module.
* The Pipeline design is usually related with compromises. Within the CmdLets in this module, I choose to make the pipelines capable to process multiple SQL queries, or single SQL query with multiple SQL Parameters against a single DataBase. Rather than running a single SQL Query with or without SQL Parameters set against multiple DataBases. The previous experience shows that such a scenario has no usage at all.


## Features

* Progress Bar is visible only when a CmdLet is executed interactively. This improves the performance of the scripts and the automations running as a service on a schedule.
![ProgressBar](/Media/ProgressBar.jpg)

* Working with the pipeline. Using "begin", "process" and "end" methods improves the performance.
![ObjectInsertNoPipeline](/Media/ObjectInsertNoPipeline.jpg)
![ObjectInsertWithPipeline](/Media/ObjectInsertWithPipeline.jpg)

* Bulk insert of PowerShell Objects into specified Table within specified DataBase with Managing of the Primary Key uniqueness violation SQL errors, when "ConflictingClause" parameter is used and the inserted entries already exist.

* Usage of SQL Transactions. The CmdLets Can evaluate whether transaction is required and automatically manage the transactions. This feature can be disabled if the transactions are manually managed within the specified SQL Query.
 
* Built-in performance measurement accessible via Verbose stream.
![VerifyTableCreation](/Media/VerifyTableCreation.jpg)

* PowerShell Parameter Sets corresponds to the supported by the CmdLets SQL Authentication methods.

* Possibility to keep the SQL Queries outside of the scripts to simplify the changes related with future scripts versions releases. Allowing either to change the scripts PowerShell code without to modify the SQL code, or modify only the SQL code without need of new script version release, whenever something is changed on SQL level.

* The SQL Queries are evaluated about keywords and modified accordingly with information from the CmdLet Parameters. In this way switching from DEV DataBase to Production DataBase is transperant and does not require change of the scripts code or SQL files.
When the keyword <DataBaseName> is used in a query it will be replaced with the value specified to the "DataBase" parameter.
![QrySelect](/Media/QrySelect.jpg)

For additional information, please use the CmdLets built-in help.
```powershell
Get-Help Invoke-SnsMsSqlQuery -Full;
Get-Help Invoke-SnsMsSqlObjectInsert -Full;
```


## Requirements

* .NET Framework 4.8
* PowerShell 4


## Instructions

Simply run
```powershell
Install-Module "SnsMsSqlPsModule" -Scope "AllUsers";
```
OR
1. Download SnsMsSqlPsModule.zip.
2. Don't forget to check the .ZIP file for viruses and etc.
3. File MD5 hash: `D9518C5FF8148548F6F66E8AD8E79122`
4. Unzip in one of the following folders depending of your preference:
* `C:\Users\UserName\Documents\WindowsPowerShell\Modules` - Replace "UserName" with the actual username If you want the module to be available for specific user.
* `C:\Program Files\WindowsPowerShell\Modules` - If you want the module to be available for all users on the machine.
* Or any other location present in `$env:PSModulePath`
5. Run the following command replacing "PathWhereModuleIsInstalled" with the actual path where the module files were unzipped.
```powershell
Get-ChildItem -Path "PathWhereModuleIsInstalled" -Recurse | Unblock-File
```


## PowerShell Examples:

* Import the module, and create a table inside the specified DataBase.
```powershell


# Import the Module
Import-Module "SnsMsSqlPsModule";
$DBServer = "DB01.contoso.com\DEV";
$DataBase = "MyDataBase";


```


Create table "TestTable" in the specified DataBase.
![CreateTable](/Media/CreateTable.jpg)
```powershell


# Initialize the SQL Query string variable.
# The CmdLet will take care to replace the keyword <DataBaseName> with the actual DataBase name provided with CmdLet parameter.
$strQry = @"
BEGIN TRANSACTION
SET QUOTED_IDENTIFIER ON
SET ARITHABORT ON
SET NUMERIC_ROUNDABORT OFF
SET CONCAT_NULL_YIELDS_NULL ON
SET ANSI_NULLS ON
SET ANSI_PADDING ON
SET ANSI_WARNINGS ON
COMMIT;
BEGIN TRANSACTION
CREATE TABLE [<DataBaseName>].[dbo].[TestTable]
(
	[ID] BIGINT NOT NULL,
	[Message] VARCHAR(500) NOT NULL,
	[Severity] VARCHAR(10) NOT NULL,
	[Date] DATETIME NOT NULL,
	CONSTRAINT UC_TestTable_ID UNIQUE ([ID]),
	CONSTRAINT PK_TestTable_ID PRIMARY KEY ([ID]),
	CONSTRAINT CHK_TestTable_Severity CHECK ([Severity] IN ('Information','Warning','Error'))
)  ON [PRIMARY]
CREATE NONCLUSTERED INDEX [IND_TestTable_ID] ON [<DataBaseName>].[dbo].[TestTable] ([ID]) WITH( STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
CREATE NONCLUSTERED INDEX [IND_TestTable_Message] ON [<DataBaseName>].[dbo].[TestTable] ([Message]) WITH( STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
CREATE NONCLUSTERED INDEX [IND_TestTable_Severity] ON [<DataBaseName>].[dbo].[TestTable] ([Severity]) WITH( STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
CREATE NONCLUSTERED INDEX [IND_TestTable_Date] ON [<DataBaseName>].[dbo].[TestTable] ([Date]) WITH( STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
ALTER TABLE [<DataBaseName>].[dbo].[TestTable] SET (LOCK_ESCALATION = TABLE)
COMMIT;
"@


# Runs the SQL query against the specified DataBase under the security context of the currently logged on user
# Creates table "TestTable"
Invoke-SnsMsSqlQuery `
	-Computer "$($DBServer)" `
	-DatabaseName "$($DataBase)" `
	-Query "$($strQry)" `
	-UseCurrentLogOnSession `
	-Verbose;


```


Verify the table creation
![VerifyTableCreation](/Media/VerifyTableCreation.jpg)
```powershell


# Verifies the "TestTable" creation
# The Query is not specified because Query Parameter have default value: SELECT [TABLE_NAME] FROM [<DataBaseName>].[INFORMATION_SCHEMA].[TABLES] WHERE [TABLE_TYPE] = 'BASE TABLE' AND [TABLE_CATALOG] = '<DataBaseName>'
Invoke-SnsMsSqlQuery `
	-Computer "$($DBServer)" `
	-DatabaseName "$($DataBase)" `
	-UseCurrentLogOnSession `
	-As "PSCustomObject" `
	-Verbose | ?{$_.TABLE_NAME -eq "TestTable"};


```


* Generate large amount of data, provide it to "Invoke-SnsMsSqlQuery" via the Pipeline, and verify the insert.


Generate large amount of data with PowerShell
![GenerateHashtableData](/Media/GenerateHashtableData.jpg)
```powershell


# Generate large amount of data with PowerShell and measure the time.
$CmdStart = [System.DateTime]::now;
[System.Collections.Hashtable[]]$Params = @();
1..100000 | ForEach `
{
	$Params += `
	@{
		"ID" = "$($_)";
		"Message" = "Fake Event Message";
		"Severity" = "Error";
		"Date" = [System.DateTime]::UtcNow.AddMinutes(100000 - $_);
	};
}
[System.DateTime]::now - $CmdStart;


```


Insert the Data using the pipeline
![QryInsertPipeline](/Media/QryInsertPipeline.jpg)
```powershell


# Insert the large amount of data in the DataBase
$CmdStart = [System.DateTime]::now;
$Params | Invoke-SnsMsSqlQuery `
	-Computer "$($DBServer)" `
	-DatabaseName "$($DataBase)" `
	-UseCurrentLogOnSession `
	-Query "INSERT INTO [<DataBaseName>].[dbo].[TestTable] ([ID], [Message], [Severity], [Date]) VALUES (@ID, @Message, @Severity, @Date)";
[System.DateTime]::now - $CmdStart;


# Query the DataBase about the previously inserted data
# The output is limited in PowerShell, not in SELECT, to retrieve all data from the DataBase
$CmdStart = [System.DateTime]::now;
$Output = Invoke-SnsMsSqlQuery `
	-Computer "$($DBServer)" `
	-DatabaseName "$($DataBase)" `
	-UseCurrentLogOnSession `
	-Query "SELECT * FROM [<DataBaseName>].[dbo].[TestTable]";
[System.DateTime]::now - $CmdStart;
$Output | Select-Object -First 2 | ft;
$Output.Count;


# delete all previously inserted entries to prepare for next insert
Invoke-SnsMsSqlQuery -Computer "$($DBServer)" -DatabaseName "$($DataBase)" -Query "TRUNCATE TABLE [<DataBaseName>].[dbo].[TestTable]" -UseCurrentLogOnSession -Verbose;


```

* Insert data without using the Pipeline, and verify the insert.
![QryInsertNoPipeline](/Media/QryInsertNoPipeline.jpg)
```powershell


# Insert the large amount of data in the DataBase using the SqlParameters
$CmdStart = [System.DateTime]::now;
Invoke-SnsMsSqlQuery `
	-Computer "$($DBServer)" `
	-DatabaseName "$($DataBase)" `
	-UseCurrentLogOnSession `
	-SqlParameters $Params `
	-Query "INSERT INTO [<DataBaseName>].[dbo].[TestTable] ([ID], [Message], [Severity], [Date]) VALUES (@ID, @Message, @Severity, @Date)";
[System.DateTime]::now - $CmdStart;


# Query the DataBase about the previously inserted data
# The output is limited in PowerShell, not in SELECT, to retrieve all data from the DataBase
$CmdStart = [System.DateTime]::now;
$Output = Invoke-SnsMsSqlQuery `
	-Computer "$($DBServer)" `
	-DatabaseName "$($DataBase)" `
	-UseCurrentLogOnSession `
	-Query "SELECT * FROM [<DataBaseName>].[dbo].[TestTable]";
[System.DateTime]::now - $CmdStart;
$Output | Select-Object -First 2 | ft;
$Output.Count;


```

* Testing the ValueFromPipelineByPropertyName Pipeline input
```powershell


# Create an empty array to hold the InputObjects which will be send to "Invoke-SnsMsSqlQuery"
[System.Array]$arrInput = @();


# Generate InputObject for the first query to delete all the entries in the Table
# The object must have property names matching the CmdLet parameters or their aliases.
[System.Object]$objObject = New-Object -TypeName "System.Object";
$objObject | Add-Member -Force -MemberType "NoteProperty" -Name "Query" -Value "TRUNCATE TABLE [<DataBaseName>].[dbo].[TestTable]";
[System.Array]$arrInput += $objObject;


# Generate InputObject for the second query to insert some data in the Table
# The object must have property names matching the CmdLet parameters or their aliases.
[System.Object]$objObject = New-Object -TypeName "System.Object";
$objObject | Add-Member -Force -MemberType "NoteProperty" -Name "Query" -Value "INSERT INTO [<DataBaseName>].[dbo].[TestTable] ([ID], [Message], [Severity], [Date]) VALUES (@ID, @Message, @Severity, @Date)";
$objObject | Add-Member -Force -MemberType "NoteProperty" -Name "SqlParameters" -Value `
(
	@{ "ID" = 1; "Message" = "Fake Message 01"; "Severity" = "Error"; "Date" = [System.DateTime]::UtcNow.AddMinutes(-2); },
	@{ "ID" = 2; "Message" = "Fake Message 02"; "Severity" = "Warning"; "Date" = [System.DateTime]::UtcNow.AddMinutes(0); }
);
[System.Array]$arrInput += $objObject;


# Generate InputObject for the third query to verify the inserting the entry from the second query
[System.Object]$objObject = New-Object -TypeName "System.Object";
$objObject | Add-Member -Force -MemberType "NoteProperty" -Name "Query" -Value "SELECT * FROM [<DataBaseName>].[dbo].[TestTable]";
[System.Array]$arrInput += $objObject;


# Run All The 3 Queries
$arrInput | Invoke-SnsMsSqlQuery `
	-Computer "$($DBServer)" `
	-DatabaseName "$($DataBase)" `
	-UseCurrentLogOnSession `
	-Verbose | ft;


# delete all previously inserted entries to prepare for next insert
Invoke-SnsMsSqlQuery -Computer "$($DBServer)" -DatabaseName "$($DataBase)" -Query "TRUNCATE TABLE [<DataBaseName>].[dbo].[TestTable]" -UseCurrentLogOnSession -Verbose;


```

* Example how to use "Invoke-SnsMsSqlQuery" for bulk upload

Generate a collection of objects
![GenerateObjectData](/Media/GenerateObjectData.jpg)
```powershell


# Generate Some Test Data
$CmdStart = [System.DateTime]::now;
[System.Array]$arrInput = @();
ForEach ($a in 1..100000)
{
	[System.Object]$objObject = New-Object -TypeName "System.Object";
	$objObject | Add-Member -Force -MemberType "NoteProperty" -Name "ID" -Value "$($a)";
	$objObject | Add-Member -Force -MemberType "NoteProperty" -Name "Message" -Value "Fake Message";
	$objObject | Add-Member -Force -MemberType "NoteProperty" -Name "Severity" -Value "Warning";
	$objObject | Add-Member -Force -MemberType "NoteProperty" -Name "Date" -Value([System.DateTime]::UtcNow.AddMinutes(100000 - $a));
	[System.Array]$arrInput += $objObject;
}
[System.DateTime]::now - $CmdStart;


```


Convert the objects array to Hashtables collection
![ConvertPsObjToHashtbl](/Media/ConvertPsObjToHashtbl.jpg)
```powershell


# Example how to convert objects to Hashtables
# Keep in mind that the table where they must be inserted should be already created
# And the table should have columns like the objects properties
# ToHashTbl() custom method works with PSCustomObject only
# From other hand any .NET object can be converted to PSCustomObject using "Select-Object *" command
$CmdStart = [System.DateTime]::now;
$arr = [SnsMsSqlPsModule.PsObjToHashTbl]::ToHashTbl($($arrInput | Select-Object *));
[System.DateTime]::now - $CmdStart;


# Bulk insert of the converted objects
$CmdStart = [System.DateTime]::now;
$arr| Invoke-SnsMsSqlQuery `
	-Computer "$($DBServer)" `
	-DatabaseName "$($DataBase)" `
	-UseCurrentLogOnSession `
	-Query "INSERT INTO [<DataBaseName>].[dbo].[TestTable] ([ID], [Message], [Severity], [Date]) VALUES (@ID, @Message, @Severity, @Date)" `
	-Verbose;
[System.DateTime]::now - $CmdStart;


# Query the DataBase about the previously inserted data
# The output is limited in PowerShell, not in SELECT, to retrieve all data from the DataBase
$CmdStart = [System.DateTime]::now;
$Output = Invoke-SnsMsSqlQuery `
	-Computer "$($DBServer)" `
	-DatabaseName "$($DataBase)" `
	-UseCurrentLogOnSession `
	-Query "SELECT * FROM [<DataBaseName>].[dbo].[TestTable]";
[System.DateTime]::now - $CmdStart;
$Output | Select-Object -First 2 | ft;
$Output.Count;


# delete all previously inserted entries to prepare for next insert
Invoke-SnsMsSqlQuery -Computer "$($DBServer)" -DatabaseName "$($DataBase)" -Query "TRUNCATE TABLE [<DataBaseName>].[dbo].[TestTable]" -UseCurrentLogOnSession -Verbose;


```

* Examples of how to insert a collection of objects without to convert them to hash tables in advance.
The main reason for this CmdLet is to allow users without or with limited knowledge about SQL query language to work with DataBases. The CmdLet creates the SQL query and the SQL parameters on its own. The user has no need even to know what SQL injection is and how to avoid it. However this comes with limitations. The CmdLet is capable only to insert data. It can manage the Primary Key constraints violations if that is requested, but it cannot manage the UNIQUE constraints on not Primary Key columns. The values in the object properties must have type, either struct or string class. Values from any other classes might lead to unexpected results. The destination DataBase Table must exist. The CmdLet Can evaluate the destination table but cannot create tables.
During the insert the values in properties from the input objects that correspond to columns in the destination table will be inserted in the corresponding columns. The values in input object properties that do not correspond to a column in the destination table will be ignored. The columns in the destination table with no matching properties will have value NULL.

Without using the pipeline
![ObjectInsertNoPipeline](/Media/ObjectInsertNoPipeline.jpg)
```powershell


# Insert the generated test objects collection without using the pipeline
[System.DateTime]$cmdStart = [System.DateTime]::Now;
Invoke-SnsMsSqlObjectInsert `
	-Computer "$($DBServer)" `
	-DatabaseName "$($DataBase)" `
	-UseCurrentLogOnSession `
	-Table "TestTable" `
	-InputObject $arrInput `
	-ConflictClause "Fail";
[System.DateTime]::Now - $cmdStart;


# Query the DataBase about the previously inserted data
# The output is limited in PowerShell, not in SELECT, to retrieve all data from the DataBase
$CmdStart = [System.DateTime]::now;
$Output = Invoke-SnsMsSqlQuery `
	-Computer "$($DBServer)" `
	-DatabaseName "$($DataBase)" `
	-UseCurrentLogOnSession `
	-Query "SELECT * FROM [<DataBaseName>].[dbo].[TestTable]";
[System.DateTime]::now - $CmdStart;
$Output | Select-Object -First 2 | ft;
$Output.Count;


# delete all previously inserted entries to prepare for next insert
Invoke-SnsMsSqlQuery -Computer "$($DBServer)" -DatabaseName "$($DataBase)" -Query "TRUNCATE TABLE [<DataBaseName>].[dbo].[TestTable]" -UseCurrentLogOnSession -Verbose;


```


Using the pipeline
![ObjectInsertWithPipeline](/Media/ObjectInsertWithPipeline.jpg)
```powershell


# Insert the same objects collection using Pipeline
# The performance boost is significant
[System.DateTime]$cmdStart = [System.DateTime]::Now;
$arrInput | Invoke-SnsMsSqlObjectInsert `
	-Computer "$($DBServer)" `
	-DatabaseName "$($DataBase)" `
	-UseCurrentLogOnSession `
	-Table "TestTable" `
	-ConflictClause "Fail";
[System.DateTime]::Now - $cmdStart;


# Query the DataBase about the previously inserted data
# The output is limited in PowerShell, not in SELECT, to retrieve all data from the DataBase
$CmdStart = [System.DateTime]::now;
$Output = Invoke-SnsMsSqlQuery `
	-Computer "$($DBServer)" `
	-DatabaseName "$($DataBase)" `
	-UseCurrentLogOnSession `
	-Query "SELECT * FROM [<DataBaseName>].[dbo].[TestTable]";
[System.DateTime]::now - $CmdStart;
$Output | Select-Object -First 2 | ft;
$Output.Count;


# drop the "TestTable"
Invoke-SnsMsSqlQuery -Computer "$($DBServer)" -DatabaseName "$($DataBase)" -Query "DROP TABLE [<DataBaseName>].[dbo].[TestTable]" -UseCurrentLogOnSession -Verbose;


```


## External Links

- svesavov on GitHub: [https://github.com/svesavov](https://github.com/svesavov)
- svesavov on PowerShell Gallery: [https://www.powershellgallery.com/packages/SnsMsSqlPsModule/](https://www.powershellgallery.com/packages/SnsMsSqlPsModule/)
- Svetoslav Savov on LinkedIn: [https://www.linkedin.com/in/svetoslavsavov](https://www.linkedin.com/in/svetoslavsavov)
- MS SQL Server: [https://www.microsoft.com/en-us/sql-server/sql-server-downloads](https://www.microsoft.com/en-us/sql-server/sql-server-downloads)
- MS SQL Data Types: [https://docs.microsoft.com/en-us/sql/t-sql/data-types/data-types-transact-sql](https://docs.microsoft.com/en-us/sql/t-sql/data-types/data-types-transact-sql)
- MS SQL Supported SQL Syntax: [https://docs.microsoft.com/en-us/sql/t-sql/language-reference](https://docs.microsoft.com/en-us/sql/t-sql/language-reference)
- MS SQL Tutorials: [https://www.sqlservertutorial.net/](https://www.sqlservertutorial.net/)

