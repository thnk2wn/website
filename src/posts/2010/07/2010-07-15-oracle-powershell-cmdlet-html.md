---
title: "Oracle Powershell Cmdlet"
date: "2010-07-15"
---

For more on this topic, check out my book [Instant Oracle Database and PowerShell How-to](https://www.packtpub.com/oracle-database-and-powershell/book).

### The What

The past couple days in between tasks at work I decided to start a basic set of PowerShell cmdlets for Oracle. I wanted an easy way to work with our DB from the command line while leveraging the .net framework. This was my first attempt at writing PowerShell cmdlets and it seemed a good fit.  
  

Naturally there are [SQL Server](http://msdn.microsoft.com/en-us/library/cc281954.aspx) cmdlets but unfortunately we work with Oracle at my company. You could still build cmdlets like this and target SQL Server or another database platform besides Oracle, though obviously the implementation details may differ.

### The Motivation

So why might you want to do this? Perhaps...

1. Sometimes you don't want to open a bloated DBMS GUI tool like Toad for simple DB ops
2. Command line tools such as SQL \*Plus exist but do not provide the power of .NET
3. You may want to automate DB tasks without a job, service, batch file (hello 90Õs) etc.
4. Sometimes you want DB + .NET without creating a new app or shoehorning existing
5. Because mastering Powershell gives you [ninja](http://www.catspictures.net/2008/12/ninja-cats.html) skills and all the cool kids are doing it
6. Because directly coding raw, direct Oracle DB powershell script is usually not pleasant
7. Because we need to use the mouse less and be more keyboard cowboys. Wait ninjas
8. Because Microsoft + Oracle = Do It Yourself

### Example Session

Sometimes starting at the end result of usage is easier. Below is output of a sample session using the Oracle cmdlets. I've modified the inputs and outputs to protect the innocent and shorten the content for brevity. Most of this is pretty vanilla usage of the cmdlets; the real power is combining cmdlets and leveraging .net. I'm sure you can use your imagination to think of better usage scenarios.

C:\\scripts   
\> get-command "\*oracle\*" -commandtype:cmdlet | ft -auto   
   
CommandType Name              Definition   
\----------- ---- ---------- 
Cmdlet      Add-Oracle        Add-Oracle \[-SQL\] <String> \[\[-Key\] <String>\] \[-KeyOutputOnly\] \[-Silent\] \[-Verbose\] \[-Debug\] \[-ErrorAction <ActionPreference>\] \[-WarningAction <ActionPreference>\] \[-ErrorVariable <String>\] \[-WarningVariable <String>\] \[-OutVariable <String>\] \[-OutBuffer <Int32>\] \[-Wha...   
Cmdlet      Connect-Oracle    Connect-Oracle \[\[-TNS\] <String>\] \[\[-UserId\] <String>\] \[\[-Password\] <String>\] \[-ConfigKey <String>\] \[-Verbose\] \[-Debug\] \[-ErrorAction <ActionPreference>\] \[-WarningAction <ActionPreference>\] \[-ErrorVariable <String>\] \[-WarningVariable <String>\] \[-OutVariable <String>\]...   
Cmdlet      Disconnect-Oracle Disconnect-Oracle \[-Exit\] \[-Verbose\] \[-Debug\] \[-ErrorAction <ActionPreference>\] \[-WarningAction <ActionPreference>\] \[-ErrorVariable <String>\] \[-WarningVariable <String>\] \[-OutVariable <String>\] \[-OutBuffer <Int32>\] \[-WhatIf\] \[-Confirm\]...   
Cmdlet      Get-Oracle        Get-Oracle \[-Object\] <String> \[-Verbose\] \[-Debug\] \[-ErrorAction <ActionPreference>\] \[-WarningAction <ActionPreference>\] \[-ErrorVariable <String>\] \[-WarningVariable <String>\] \[-OutVariable <String>\] \[-OutBuffer <Int32>\] \[-WhatIf\] \[-Confirm\]...   
Cmdlet      Invoke-Oracle     Invoke-Oracle \[-SQL\] <String> \[-Verbose\] \[-Debug\] \[-ErrorAction <ActionPreference>\] \[-WarningAction <ActionPreference>\] \[-ErrorVariable <String>\] \[-WarningVariable <String>\] \[-OutVariable <String>\] \[-OutBuffer <Int32>\] \[-WhatIf\] \[-Confirm\]...   
Cmdlet      Remove-Oracle     Remove-Oracle \[-SQL\] <String> \[-Verbose\] \[-Debug\] \[-ErrorAction <ActionPreference>\] \[-WarningAction <ActionPreference>\] \[-ErrorVariable <String>\] \[-WarningVariable <String>\] \[-OutVariable <String>\] \[-OutBuffer <Int32>\] \[-WhatIf\] \[-Confirm\]...   
Cmdlet      Search-Oracle     Search-Oracle \[-Type\] <String> \[\[-Owner\] <String>\] \[\[-Name\] <String>\] \[-Verbose\] \[-Debug\] \[-ErrorAction <ActionPreference>\] \[-WarningAction <ActionPreference>\] \[-ErrorVariable <String>\] \[-WarningVariable <String>\] \[-OutVariable <String>\] \[-OutBuffer <Int32>\] \[-WhatI...   
Cmdlet      Select-Oracle     Select-Oracle \[-SQL\] <String> \[-Scalar\] \[\[-Out\] <String>\] \[-Verbose\] \[-Debug\] \[-ErrorAction <ActionPreference>\] \[-WarningAction <ActionPreference>\] \[-ErrorVariable <String>\] \[-WarningVariable <String>\] \[-OutVariable <String>\] \[-OutBuffer <Int32>\] \[-WhatIf\] \[-Confirm...   
Cmdlet      Select-OracleTNS  Select-OracleTNS \[\[-Out\] <String>\] \[-Verbose\] \[-Debug\] \[-ErrorAction <ActionPreference>\] \[-WarningAction <ActionPreference>\] \[-ErrorVariable <String>\] \[-WarningVariable <String>\] \[-OutVariable <String>\] \[-OutBuffer <Int32>\] \[-WhatIf\] \[-Confirm\]...   
Cmdlet      Update-Oracle     Update-Oracle \[-SQL\] <String> \[-Verbose\] \[-Debug\] \[-ErrorAction <ActionPreference>\] \[-WarningAction <ActionPreference>\] \[-ErrorVariable <String>\] \[-WarningVariable <String>\] \[-OutVariable <String>\] \[-OutBuffer <Int32>\] \[-WhatIf\] \[-Confirm\]...   
   
   
c:\\scripts   
\> connect-oracle -configKey:physicians.DB   
Connected with: Data Source=PHYDEV1;User Id=PHYAPP;Pooling=false   
c:\\scripts   
\> disconnect-oracle   
You are now disconnected from PHYDEV1   
c:\\scripts   
\> select-oracletns |ft -auto   
   
InstanceName    ServerName      ServiceName Protocol Port   
\------------ ---------- ----------- -------- ---- PHYDEV1             ORA-DEV         phydev1     TCP      1521   
PHYDEV2         ORA-DEV         phyDev2     TCP      1521   
PHYTEST1        ORA-TST1        phytest1    TCP      1521   
   
   
c:\\scripts   
\> connect-oracle phydev1 phyuser phypass   
Connected with: Data Source=phydev1;User Id=phyuser;Pooling=false   
c:\\scripts   
\> search-oracle package -owner:APP -name:sh\* | ft -auto   
   
Name                        Type    Created              LastDDL              Status Temporary   
\---- ---- ------- ------- ------ --------- 
SH\_APP                      PACKAGE 1/20/2010 3:18:09 PM 3/19/2010 2:11:14 PM VALID      False   
SH\_BILLING                  PACKAGE 1/20/2010 3:18:09 PM 1/20/2010 3:25:23 PM VALID      False   
SH\_COMMENTS                 PACKAGE 1/20/2010 3:18:09 PM 1/20/2010 3:25:24 PM VALID      False   
   
   
c:\\scripts   
\> search-oracle tables -name:\*bill\* | ft -auto   
   
Name                         Type  Created               LastDDL               Status Temporary   
\---- ---- ------- ------- ------ --------- 
BILLING\_GROUP                TABLE 1/20/2010 9:16:22 AM  1/20/2010 3:21:13 PM  VALID      False   
BILLING\_SITE                 TABLE 1/20/2010 9:16:22 AM  7/12/2010 11:19:56 AM VALID      False   
   
   
c:\\scripts   
\> get-oracle app.billing\_site | ft -auto   
This table will store billing site data for lockboxes .   
   
COLUMN\_ID COLUMN\_NAME     DATA\_TYPE DATA\_LENGTH NULLABLE COMMENTS   
\--------- ----------- --------- ----------- -------- -------- 
 1 BILLING\_SITE\_ID NUMBER             22 N        Unique ID for billing\_site table   
 2 NAME            VARCHAR2          100 Y        Name of the billing site   
 3 START\_DATE      DATE                7 Y   
 4 END\_DATE        DATE                7 Y   
 5 CREATED\_BY      VARCHAR2           30 Y   
 6 CREATED\_ON      DATE                7 Y   
 7 EDITED\_BY       VARCHAR2           30 Y   
 8 EDITED\_ON       DATE                7 Y   
   
   
c:\\scripts   
\> select-oracle "select billing\_site\_id, name, start\_date from app.billing\_site" | ft -auto   
   
BILLING\_SITE\_ID NAME                                        START\_DATE   
\--------------- ---- ---------- 
 1 ACME BILLING CENTER                         10/2/2004 9:52:34 AM   
 2 DENVER BILLING CENTER                       10/2/2004 9:52:34 AM   
 3 MEDICAL SYSTEMS INC.                        10/2/2004 9:52:34 AM   
   
c:\\scripts   
\> select-oracle "\* from billing\_group where division\_id = 22" -out:html | out-file c:billing\_groups.html   
c:\\scripts   
\> select-oracle "\* from billing\_group where division\_id = 22" -out:xml | out-file c:billing\_groups.xml   
c:\\scripts   
\> $xDoc = select-oracle "\* from phy\_status\_class" -out:xdoc   
c:\\scripts   
\> $xDoc | gm   
   
   
 TypeName: System.Xml.Linq.XDocument   
   
Name               MemberType Definition   
\---- ---------- ---------- 
Changed            Event      System.EventHandler\`1\[System.Xml.Linq.XObjectChangeEventArgs\] Changed(System.Object, System.Xml.Linq.XObjectChangeEventArgs)   
Changing           Event      System.EventHandler\`1\[System.Xml.Linq.XObjectChangeEventArgs\] Changing(System.Object, System.Xml.Linq.XObjectChangeEventArgs)   
Add                Method     System.Void Add(System.Object content), System.Void Add(Params System.Object\[\] content)   
AddAfterSelf       Method     System.Void AddAfterSelf(System.Object content), System.Void AddAfterSelf(Params System.Object\[\] content)   
AddAnnotation      Method     System.Void AddAnnotation(System.Object annotation)   
AddBeforeSelf      Method     System.Void AddBeforeSelf(System.Object content), System.Void AddBeforeSelf(Params System.Object\[\] content)   
AddFirst           Method     System.Void AddFirst(System.Object content), System.Void AddFirst(Params System.Object\[\] content)   
Ancestors          Method     System.Collections.Generic.IEnumerable\[System.Xml.Linq.XElement\] Ancestors(), System.Collections.Generic.IEnumerable\[System.Xml.Linq.XElement\] Ancestors(System.Xml.Linq.XName name)   
Annotation         Method     System.Object Annotation(type type), T Annotation\[T\]()   
Annotations        Method     System.Collections.Generic.IEnumerable\[System.Object\] Annotations(type type), System.Collections.Generic.IEnumerable\[T\] Annotations\[T\]()   
CreateReader       Method     System.Xml.XmlReader CreateReader()   
CreateWriter       Method     System.Xml.XmlWriter CreateWriter()   
DescendantNodes    Method     System.Collections.Generic.IEnumerable\[System.Xml.Linq.XNode\] DescendantNodes()   
Descendants        Method     System.Collections.Generic.IEnumerable\[System.Xml.Linq.XElement\] Descendants(), System.Collections.Generic.IEnumerable\[System.Xml.Linq.XElement\] Descendants(System.Xml.Linq.XName name)   
Element            Method     System.Xml.Linq.XElement Element(System.Xml.Linq.XName name)   
Elements           Method     System.Collections.Generic.IEnumerable\[System.Xml.Linq.XElement\] Elements(), System.Collections.Generic.IEnumerable\[System.Xml.Linq.XElement\] Elements(System.Xml.Linq.XName name)   
ElementsAfterSelf  Method     System.Collections.Generic.IEnumerable\[System.Xml.Linq.XElement\] ElementsAfterSelf(), System.Collections.Generic.IEnumerable\[System.Xml.Linq.XElement\] ElementsAfterSelf(System.Xml.Linq.XName name)   
ElementsBeforeSelf Method     System.Collections.Generic.IEnumerable\[System.Xml.Linq.XElement\] ElementsBeforeSelf(), System.Collections.Generic.IEnumerable\[System.Xml.Linq.XElement\] ElementsBeforeSelf(System.Xml.Linq.XName name)   
Equals             Method     bool Equals(System.Object obj)   
GetHashCode        Method     int GetHashCode()   
GetType            Method     type GetType()   
IsAfter            Method     bool IsAfter(System.Xml.Linq.XNode node)   
IsBefore           Method     bool IsBefore(System.Xml.Linq.XNode node)   
Nodes              Method     System.Collections.Generic.IEnumerable\[System.Xml.Linq.XNode\] Nodes()   
NodesAfterSelf     Method     System.Collections.Generic.IEnumerable\[System.Xml.Linq.XNode\] NodesAfterSelf()   
NodesBeforeSelf    Method     System.Collections.Generic.IEnumerable\[System.Xml.Linq.XNode\] NodesBeforeSelf()   
Remove             Method     System.Void Remove()   
RemoveAnnotations  Method     System.Void RemoveAnnotations(type type), System.Void RemoveAnnotations\[T\]()   
RemoveNodes        Method     System.Void RemoveNodes()   
ReplaceNodes       Method     System.Void ReplaceNodes(System.Object content), System.Void ReplaceNodes(Params System.Object\[\] content)   
ReplaceWith        Method     System.Void ReplaceWith(System.Object content), System.Void ReplaceWith(Params System.Object\[\] content)   
Save               Method     System.Void Save(string fileName), System.Void Save(string fileName, System.Xml.Linq.SaveOptions options), System.Void Save(System.IO.TextWriter textWriter), System.Void Save(System.IO.TextWriter textWriter, System.Xml.Linq.SaveOptions options), System.Void Save(Sys...   
ToString           Method     string ToString(), string ToString(System.Xml.Linq.SaveOptions options)   
WriteTo            Method     System.Void WriteTo(System.Xml.XmlWriter writer)   
BaseUri            Property   System.String BaseUri {get;}   
Declaration        Property   System.Xml.Linq.XDeclaration Declaration {get;set;}   
Document           Property   System.Xml.Linq.XDocument Document {get;}   
DocumentType       Property   System.Xml.Linq.XDocumentType DocumentType {get;}   
FirstNode          Property   System.Xml.Linq.XNode FirstNode {get;}   
LastNode           Property   System.Xml.Linq.XNode LastNode {get;}   
NextNode           Property   System.Xml.Linq.XNode NextNode {get;}   
NodeType           Property   System.Xml.XmlNodeType NodeType {get;}   
Parent             Property   System.Xml.Linq.XElement Parent {get;}   
PreviousNode       Property   System.Xml.Linq.XNode PreviousNode {get;}   
Root               Property   System.Xml.Linq.XElement Root {get;}   
   
   
c:\\scripts   
\> $xDoc.Descendants("STATUS\_TYPE") | where {$\_.Value -like "\*ACTIVE\*"} | sort-object Value | select-object Value   
   
Value   
\----- 
BILLING  - ACTIVE   
EMP ACTIVE - FULL TIME   
EMP ACTIVE - PART TIME   
EMP ACTIVE - PRN   
EMP ACTIVE - TEMPORARY   
IC ACTIVE - FULL TIME   
IC ACTIVE - PART TIME   
   
   
c:\\scripts   
\> add-oracle "insert into billing\_site(billing\_site\_id, name) values (0, 'CPR')" -key:billing\_site\_id   
Inserted new row with billing\_site\_id of 38   
c:\\scripts   
\> update-oracle "update billing\_site set name = 'CPR INC.' where name = 'CPR'"   
Updated 1 row(s)   
c:\\scripts   
\> remove-oracle "delete from billing\_site where name = 'CPR INC.'"   
Deleted 1 row(s)   
c:\\scripts   
\> $dt = select-oracle "\* from contract\_view where start\_date >= to\_date('06/01/10','mm/dd/yy')"   
c:\\scripts   
\> $dt.DataSet.WriteXml("C:ContractsJuneAndLater.xml")   
c:\\scripts   
\> $dt | export-csv "c:ContractsJuneAndLater.csv"   
c:\\scripts   
\> foreach ($i in "Tampa", "DC", "NYC") {add-oracle "into billing\_site (billing\_site\_id, name) values (0,'$i')"}   
Inserted 1 row(s)   
Inserted 1 row(s)   
Inserted 1 row(s)   
c:\\scripts   
\> $result = select-oracle "select holiday\_name, holiday\_date from holidays  where holiday\_type='H' AND holiday\_date between sysdate and add\_months(sysdate, 12)"   
c:\\scripts   
\> foreach ($r in $result.Rows) {Write-Host (\[DateTime\]::Parse($r\["HOLIDAY\_DATE"\]) - \[DateTime\]::Now).TotalDays.ToString("###") Days Until $r\["holiday\_name"\]}   
55 Days Until Labor Day   
135 Days Until Thanksgiving   
136 Days Until Day after Thanksgiving   
164 Days Until Christmas Eve   
165 Days Until Christmas Day   
172 Days Until New Year's Day   
321 Days Until Memorial Day   
356 Days Until Independence Day   
c:\\scripts   
\> invoke-oracle "create table health\_images (name varchar2(256) not null)"   
DDL / DML operation complete   
c:\\scripts   
\> $dirPath = "T:GraphicsMedical ImagesPNGRegular16x16"   
c:\\scripts   
\> foreach ($i in get-childitem $dirPath -name) {add-oracle "into health\_images (name) values ('$i')" -silent}   
c:\\scripts   
\> write-host ("Added", (select-oracle "count(\*) from health\_images" -scalar), "image records")   
Added 1259 image records   
c:\\scripts   
\> invoke-oracle "drop table health\_images"   
DDL / DML operation complete   
c:\\scripts   
\> disconnect-oracle   
You are now disconnected from phydev1   
c:\\scripts   
\>  .Get-ConsoleAsHtml | out-file "c:\\console.html" -encoding UTF8   

### Getting started

There are plenty of blog posts about creating PowerShell cmdlets so I won't go into that here. A couple cmdlet overview examples include [CodeProject](http://www.codeproject.com/KB/vista/PowerShell.aspx) and [MSDN Magazine](http://msdn.microsoft.com/en-us/magazine/cc163293.aspx). In short I created a new class library project and referenced System.Management.Automation for PowerShell, System.Configuration for ConfigurationManager, System.Configuration.Install to install the Cmdlets, and Oracle.DataAccess (11.1.0 / 2.111.6.20) from [ODP.NET](http://www.oracle.com/technology/tech/windows/odpnet/index.html). There were some PowerShell project templates out there but they were for prior versions of Visual Studio and doing it from scratch provided a better learning experience.

### OracleCmdletBase

This class serves as base for cmdlets needing some access to Oracle. It has some base helper methods for retrieving and updating data; I've omitted the code for brevity but it does nothing special other than dumping out the error and sql upon exceptions. The main item of interest here is the Connection property which uses session state to store our connection that all the cmdlets can share. This requires inheriting from [PSCmdlet](http://msdn.microsoft.com/en-us/library/system.management.automation.pscmdlet(VS.85).aspx) instead of [Cmdlet](http://msdn.microsoft.com/en-us/library/system.management.automation.cmdlet(v=VS.85).aspx); differences are described [here](http://weblogs.asp.net/shahar/archive/2008/01/24/cmdlet-vs-pscmslet-windows-powershell.aspx).

\[csharp highlight="8,14,17"\] using System; using System.Data; using System.Management.Automation; using Oracle.DataAccess.Client;

namespace OracleCmdlets.Cmdlets { public abstract class OracleCmdletBase : PSCmdlet { public OracleConnection Connection { get { var conn = this.SessionState.PSVariable.GetValue(Constants.VAR\_CONN, null) as OracleConnection; return conn; } set { this.SessionState.PSVariable.Set(Constants.VAR\_CONN, value); } }

protected bool EnsureConnected() { if (null == this.Connection) { WriteObject("You must be connected for this operation."); return false; }

return true; }

protected DataTable GetDataTable(string sql) { /\* ... \*/ }

protected virtual void WriteError(Exception ex) { WriteError(new ErrorRecord(ex, "", ErrorCategory.NotSpecified, this)); }

protected virtual object ExecuteScalar(string sql) { /\* ... \*/ }

protected virtual int ExecuteNonQuery(string sql) { /\* ... \*/ } } } \[/csharp\]

### OracleConnectCmdlet

Connects to Oracle via TNS + User + Password or machine.config connection string key.

Examples:  

connect-oracle -configKey:myApp.database  
connect-oracle sometnsname userid pass

  

By setting the Connection property of the base class it establishes a connection in Powershell's session so a connect/disconnect is not required with every operation. Adding an option to connect via a full connection string might be a good idea but I didn't have the need for it.  

\[csharp\] using System; using System.Data; using System.Management.Automation; using Oracle.DataAccess.Client; using System.Configuration;

namespace OracleCmdlets.Cmdlets { /// <summary> /// connect-oracle TNS UserId Password /// </summary> /// <remarks> /// You must use the x86 version of powershell when running this /// </remarks> \[Cmdlet(VerbsCommunications.Connect, "Oracle", SupportsShouldProcess = true)\] // VerbsCommon.Open public class OracleConnectCmdlet : OracleCmdletBase { \[Alias("Datasource")\] \[Parameter(Position = 0, Mandatory = false, ValueFromPipelineByPropertyName = true, HelpMessage = "Oracle TNS Name / Data Source name")\] public string TNS { get; set; }

\[Alias("User")\] \[Parameter(Position = 1, Mandatory = false, ValueFromPipelineByPropertyName = true, HelpMessage = "Oracle User Id / Owner")\] public string UserId { get; set; }

\[Alias("Pass", "Pwd")\] \[Parameter(Position = 2, Mandatory = false, ValueFromPipelineByPropertyName = true, HelpMessage = "Oracle password")\] public string Password { get; set; }

\[Alias("Key", "ConnectKey", "Config")\] \[Parameter(Mandatory = false, ValueFromPipelineByPropertyName = true, HelpMessage = "The key value of a connection string tag in machine.config")\] public string ConfigKey { get; set; }

protected override void ProcessRecord() { var connect = BuildConnectString(); this.Connection = new OracleConnection(connect);

try { this.Connection.Open(); } catch (Exception ex) { WriteObject(this.Connection.ConnectionString); WriteError(ex); }

if (Connection.State == ConnectionState.Open) WriteObject("Connected with: " + this.Connection.ConnectionString); }

private string BuildConnectString() { string connect; if (string.IsNullOrEmpty(this.ConfigKey)) { if (string.IsNullOrEmpty(this.TNS)) throw new NullReferenceException("TNS is required"); if (string.IsNullOrEmpty(this.UserId)) throw new NullReferenceException("UserId is required"); if (string.IsNullOrEmpty(this.Password)) throw new NullReferenceException("Password is required");

const string format = "Data Source={0};User Id={1};Password={2};Pooling=false"; connect = string.Format(format, this.TNS, this.UserId, this.Password); } else { var config = ConfigurationManager.OpenMachineConfiguration(); var item = config.ConnectionStrings.ConnectionStrings\[this.ConfigKey\]; if (null != item) connect = item.ConnectionString; else throw new ArgumentException("Connection string key " + this.ConfigKey + " not found in " + config.FilePath); }

return connect; } } } \[/csharp\]

### OracleSelectCmdlet

The select cmdlet takes a SELECT SQL string ("select" is optional) and outputs results in a DataTable, xml string, XDocument or html string according to the Out parameter. It defaults to a DataTable. You could just pipe the default output to PowerShell's built-in [CovertTo-XML](http://www.microsoft.com/technet/scriptcenter/topics/winpsh/converttoxml.mspx) or [ConvertTo-HTML](http://www.microsoft.com/technet/scriptcenter/resources/pstips/jan08/pstip0104.mspx) but it was easy enough to implement and allows a little less typing and perhaps more customization if desired.

A "-scalar" [switch parameter](http://msdn.microsoft.com/en-us/library/system.management.automation.switchparameter(VS.85).aspx) indicates you want a single result back such as with a select count operation.

Examples:  

select-oracle "select billing\_site\_id, name, start\_date from app.billing\_site" | ft -auto  
select-oracle "\* from billing\_group where division\_id = 22" -out:xml | out-file c:billing\_groups.xml  
$xDoc = select-oracle "\* from phy\_status\_class" -out:xdoc  
$dataTable = select-oracle "\* from phy\_status\_class"

  

\[csharp\] using System; using System.Collections.Generic; using System.Data; using System.IO; using System.Management.Automation; using System.Text; using System.Linq; using System.Xml.Linq;

namespace OracleCmdlets.Cmdlets { // select-oracle \[Cmdlet(VerbsCommon.Select, "Oracle", SupportsShouldProcess = true)\] public class OracleSelectCmdlet : OracleCmdletBase { \[Parameter(Position = 0, Mandatory = true, ValueFromPipelineByPropertyName = true, HelpMessage = "select sql. may ommit 'select'")\] public string SQL { get; set; }

\[Parameter(Position = 1, Mandatory = false, ValueFromPipelineByPropertyName = true, HelpMessage = "Switch parameter if -scalar is specified, only single result is output")\] public SwitchParameter Scalar { get; set; }

\[ValidateSet("dt", "xml", "xdoc", "html")\] \[Parameter(Position = 3, Mandatory = false, HelpMessage = "Output format defaulting to DataTable (dt). Can be dt, xml, xdoc, or html")\] public string Out { get; set; }

private readonly Dictionary<string, Func<DataTable, object>> \_outDict = new Dictionary<string, Func<DataTable, object>> { {"dt", OutDataTable}, {"xml", OutXml}, {"xdoc", OutXmlDoc }, {"html", OutHtml} };

public OracleSelectCmdlet() { this.Out = "dt"; }

private static object OutDataTable(DataTable dt) { return dt; }

private static object OutXml(DataTable dt) { var xDoc = OutXmlDoc(dt); return xDoc.ToString(SaveOptions.None); }

private static XDocument OutXmlDoc(DataTable dt) { using (var sw = new StringWriter()) { dt.WriteXml(sw); var xDoc = XDocument.Parse(sw.ToString()); return xDoc; } }

private static object OutHtml(DataTable dt) { var sb = new StringBuilder(); sb.AppendLine("<html><body><table border='1'><tr>"); dt.Columns.OfType<DataColumn>().ToList().ForEach(c => sb.AppendFormat("<th>{0}</th>", c.ColumnName)); sb.AppendLine("</tr>"); dt.Rows.OfType<DataRow>().ToList().ForEach(r=> { sb.AppendLine("<tr>"); dt.Columns.OfType<DataColumn>().ToList().ForEach(c=> sb.AppendFormat("<td>{0}</td>", r.IsNull(c) ? "" : r\[c\])); sb.AppendLine("</tr>"); }); sb.AppendLine("</table></body></html>"); return sb.ToString(); }

protected override void ProcessRecord() { if (!EnsureConnected()) return;

var sqlText = this.SQL;

if (!sqlText.ToLower().TrimStart().StartsWith("select ")) sqlText = "select " + this.SQL;

if (!this.Scalar) { var dt = GetDataTable(sqlText); var result = \_outDict\[this.Out\](dt); WriteObject(result); } else { var result = this.ExecuteScalar(sqlText); WriteObject(result); } } } } \[/csharp\]

### OracleInsertCmdlet

The insert cmdlet has extra parameters around whether you want any sequence type key value back and what sort of output to write.

If the Key parameter is set to the key column name, that value will be retrieved and output. I only tested this with our setup that includes insert triggers on each table that select the next value of the associated sequence when the key value coming in is zero. We also don't use multi-part keys. If you are interested in getting the new ids back, this cmdlet might need modifications depending on your scenarios.

Examples:  

add-oracle "insert into billing\_site(billing\_site\_id, name) values (0, 'CPR')" -key:billing\_site\_id  
foreach ($i in "Tampa", "DC", "NYC") {add-oracle "into billing\_site (billing\_site\_id, name) values (0,'$i')"}

  

\[csharp\] using System; using System.Data; using System.Management.Automation; using Oracle.DataAccess.Client;

namespace OracleCmdlets.Cmdlets { // add-oracle \[Cmdlet(VerbsCommon.Add, "Oracle", SupportsShouldProcess = true)\] public class OracleInsertCmdlet : OracleCmdletBase { \[Parameter(Position = 0, Mandatory = true, ValueFromPipelineByPropertyName = true, HelpMessage = "Insert SQL statement. insert may be omitted.")\] public string SQL { get; set; }

\[Alias("KeyColumn", "Id", "PK")\] \[Parameter(Position = 1, Mandatory = false, ValueFromPipelineByPropertyName = true, HelpMessage = "Key column name if you want the inserted id value back")\] public string Key { get; set; } \[Parameter(Position = 2, Mandatory = false, ValueFromPipelineByPropertyName = true, HelpMessage = "If Key is set, and this is true, only key values for inserted row is output. " + "Otherwise if key is set and this is false, informational message is output.")\] public SwitchParameter KeyOutputOnly { get; set; }

\[Parameter(Position = 3, Mandatory = false, ValueFromPipelineByPropertyName = true, HelpMessage = "If set no output is displayed in any circumstance exception for exceptions / errors")\] public SwitchParameter Silent { get; set; }

public OracleInsertCmdlet() { return; }

protected override void ProcessRecord() { if (!EnsureConnected()) return;

var sqlText = this.SQL;

// might want a setting for this if (!sqlText.ToLower().TrimStart().StartsWith("insert ")) sqlText = "insert " + this.SQL;

ExecuteNonQuery(sqlText); }

protected override int ExecuteNonQuery(string sql) { int result = -1;

using (var tx = this.Connection.BeginTransaction(IsolationLevel.ReadCommitted)) { var cmd = new OracleCommand(sql, this.Connection); OracleParameter idParam = null; const string keyParamName = "newKeyId";

if (!string.IsNullOrEmpty(this.Key)) { cmd.CommandText = string.Format("{0} RETURNING {1} INTO :{2} ", cmd.CommandText, Key, keyParamName); idParam = new OracleParameter {OracleDbType = OracleDbType.Decimal, Direction = ParameterDirection.Output, Value = DBNull.Value, SourceColumn = Key, ParameterName = keyParamName}; cmd.Parameters.Add(idParam); } try { result = cmd.ExecuteNonQuery(); tx.Commit();

if (null != idParam) { var value = idParam.Value; if (!this.Silent) WriteObject(!this.KeyOutputOnly ? string.Format("Inserted new row with {0} of {1}", this.Key, value) : value); idParam.Dispose(); } else { if (!this.KeyOutputOnly && !this.Silent) WriteObject(string.Format("Inserted {0} row(s)", result)); } } catch (Exception ex) { tx.Rollback(); WriteError(ex); WriteObject(cmd.CommandText); }

cmd.Dispose(); }

return result; } } } \[/csharp\]

### OracleSearchCmdlet

The search cmdlet builds a query against [all\_objects](http://download.oracle.com/docs/cd/B19306_01/server.102/b14237/statviews_2005.htm) or user\_objects depending on whether an owner is specified. I considered dba\_objects though that has security implications. This could be done manually instead by the OracleSelectCmdlet but I found this to be a bit friendlier.

One item of interest here is the use of [WildcardPattern](http://www.owenpellegrin.com/blog/powershell/expanding-wildcards-in-powershell/) to provide automatic support for searching of an object name with wildcards. The cmdlet returns an IEnumerable of a custom POCO since the columns are known and that is friendlier than a DataTable.

\[csharp highlight="69, 70, 73"\] using System; using System.Collections.Generic; using System.Data; using System.Linq; using System.Management.Automation; using System.Text; using OracleCmdlets.Model;

namespace OracleCmdlets.Cmdlets { \[Cmdlet(VerbsCommon.Search, "Oracle", SupportsShouldProcess = true)\] public class OracleSearchCmdlet : OracleCmdletBase { \[ValidateSet( "table", "tables", "tab", "t", "view", "views", "viw", "v", "package", "package spec", "package body", "pkg", "function", "functions", "func", "f", "procedure", "procedures", "proc" )\] \[Parameter(Position = 0, Mandatory = true, ValueFromPipelineByPropertyName = true, HelpMessage = "Type of object to search such as table, view, package, function, procedure etc.")\] public string Type { get; set; }

\[Parameter(Position = 1, Mandatory = false, ValueFromPipelineByPropertyName = true, HelpMessage = "Owner of the object / schema")\] public string Owner { get; set; }

\[Parameter(Position = 2, Mandatory = false, ValueFromPipelineByPropertyName = true, HelpMessage = "Name of the object. May include wildcards such as \*PARTIAL\_OBJ\_NAME\*")\] public string Name { get; set; }

protected override void ProcessRecord() { var sb = new StringBuilder(); sb.AppendLine( "SELECT object\_name, object\_type, created, last\_ddl\_time, timestamp, status, temporary ");

var hasOwner = (!string.IsNullOrEmpty(this.Owner)); var view = (hasOwner) ? "all\_objects" : "user\_objects";

sb.AppendFormat("FROM {0}{1} ", view, Environment.NewLine);

var type = this.Type.ToLower(); if (type.StartsWith("t")) type = "TABLE"; else if (type.StartsWith("v")) type = "VIEW"; else if (type == "package body" || type == "pkg") type = "PACKAGE BODY"; else if (type == "package" || type == "package spec") type = "PACKAGE"; else if (type.StartsWith("f")) type = "FUNCTION"; else if (type.StartsWith("pr")) type = "PROCEDURE";

sb.AppendFormat("WHERE OBJECT\_TYPE = '{0}'{1}", type, Environment.NewLine);

if (hasOwner) sb.AppendFormat("AND OWNER = '{0}'{1}", this.Owner, Environment.NewLine);

var dt = GetDataTable(sb.ToString());

IEnumerable<DataRow> rows = null;

if (!string.IsNullOrEmpty(this.Name)) { const WildcardOptions options = WildcardOptions.Compiled | WildcardOptions.IgnoreCase; var pattern = new WildcardPattern(this.Name, options);

rows = from item in dt.Rows.OfType<DataRow>() where pattern.IsMatch(item\["OBJECT\_NAME"\].ToString()) select item; } else rows = dt.Rows.OfType<DataRow>();

var objs = from r in rows select new OracleObject { Name = r\["OBJECT\_NAME"\].ToString(), Type = r\["OBJECT\_TYPE"\].ToString(), Created = DateTime.Parse(r\["CREATED"\].ToString()), LastDDL = DateTime.Parse(r\["LAST\_DDL\_TIME"\].ToString()), //Timestamp = DateTime.Parse(r\["TIMESTAMP"\].ToString()), Status = r\["STATUS"\].ToString(), Temporary = (r\["TEMPORARY"\].ToString() == "N") ? false : true }; WriteObject(objs);

} } } \[/csharp\]

### Other Cmdlets

Other cmdlets in the library include the below. Some cmdlets could be combined (update and delete for example) but for more specific output and likely divergent changes to come I kept them separated.

- OracleDdlCmdlet (invoke-oracle) - Used for DDL, or DCL operations such as create table or grant
- OracleDeleteCmdlet (remove-oracle) - For issuing delete statements
- OracleDescribeCmdlet (get-oracle) - Get's a table or view object description including comments and column definitions
- OracleDisconnnectCmdlet (disconnect-oracle) - Disconnects from currently connected datasource
- OracleTnsCmdlet (select-oracletns) - Outputs [TNS](http://www.orafaq.com/wiki/TNS) info such as server names, protocols and ports using [OracleDataSourceEnumerator](http://download.oracle.com/docs/html/B28089_01/OracleDataSourceEnumeratorClass.htm)
- OracleUpdateCmdlet (update-oracle) - Issues an update statement.

### Limitations & possible future enhancements

- Operations - This was a quick 2 day project so various operations are not specifically supported, only the basics. For example: executing procedures, functions, packages & some other operations
- Filtering with where-object stopped working for me on some operations. At one point something like the following worked but it started returning null and I never figured out why: select-oracletns | where {$\_.InstanceName -like '\*DEV\*'} | ft -auto
- Inserts, updates and deletes currently always do a commit on success and a rollback on failure. More transactional support / control could be useful
- [Oracle Developer Tools for .Net (ODT.NET)](http://www.oracle.com/technology/tech/dotnet/tools/index.html) includes nice managed dlls (namely Oracle.Management.Omo.dll) for working with Oracle objects in a managed fashion. At some point incorporating this might be worth the negative tradeoffs of its dependencies and the fact that it is undocumented, unsupported and your code tends to break with every upgrade they do.

### Source, Installation & Usage

- **Project Source:** [OracleCmdlets.zip](/wp-content/uploads/2017/06/OracleCmdlets.zip)

- **PowerShell Environment:** If you are running a 64 bit OS, you must use an x86 version of the PowerShell command line, at least with this version of Oracle.DataAccess which is only compatible on x86

- **Installation:** Run OracleCmdlet\_Install.ps1 piecemeal in Reference folder of the source

- **Help:** Help is available for all the cmdlets once installed via get-help commandName. for example: get-help get-oracle. On a side note, I found out the hard way that the help xml file has to be set to be copied to the output directory in order for it to get picked up.

- **Examples:** Some additional examples with comments are available in the OracleCmdlet\_Examples.ps1 file in the Reference folder of the source

### Updates

- 03/05/2013 - For more, check out my recent book [Instant Oracle Database and PowerShell How-to](https://www.packtpub.com/oracle-database-and-powershell/book)
