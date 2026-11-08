# Scheduling a Nightly File Upload

Many systems can export a CSV file but can't call an API. In this tutorial you write a PowerShell script that uploads such a file to Grein, and schedule it to run every night with Windows Task Scheduler.

**You need:**

- Windows with PowerShell 7 or later (`pwsh`).
- An [API key](../guide/api-keys.md), and the Company entity type and the ERP source from [Your First Golden Record](first-golden-record.md).
- A file to upload. To try it out, use [companies-erp.csv](data/companies-erp.csv) and save it as `C:\Exports\companies.csv`.

## 1. The upload script

Save this as `C:\Scripts\Upload-Companies.ps1`:

```powershell
$ErrorActionPreference = 'Stop'
$file   = 'C:\Exports\companies.csv'
$url    = 'https://<your Grein address>/api/v1/company/upload'
$key    = [Environment]::GetEnvironmentVariable('GREIN_API_KEY', 'User')
$log    = 'C:\Scripts\upload-companies.log'

$result = Invoke-RestMethod -Method Post -Uri $url `
    -Authentication Bearer -Token (ConvertTo-SecureString $key -AsPlainText -Force) `
    -Form @{ file = Get-Item $file; source = 'erp' } `
    -MaximumRetryCount 3 -RetryIntervalSec 60

$line = '{0:u}  created {1}, updated {2}, skipped {3}' -f (Get-Date), $result.created, $result.updated, $result.skipped
Add-Content $log $line
foreach ($e in $result.errors) { Add-Content $log "  row $($e.row): $($e.messages -join '; ')" }
```

The upload endpoint matches rows to existing records by the unique key, so each run updates the companies that are already there and creates the new ones. Column headers must be the fields' API names (`org_number`, `company_name`, …). Columns that don't match a field are ignored. Rows that fail validation are skipped and listed in the log; the rest are loaded.

Files can be up to 50 MB, and Parquet files work too. See [Upload a file](../guide/api.md#upload-a-file).

## 2. Store the key

Run this once, as the Windows user that will run the task:

```powershell
[Environment]::SetEnvironmentVariable('GREIN_API_KEY', 'mdm_...', 'User')
```

The key is kept out of the script, so the script can be shared or checked into version control. Give the integration its own API key with a recognisable label, such as *ERP nightly upload*: changes it makes appear as "API — ERP nightly upload" in each record's history.

## 3. Try it

```powershell
pwsh -File C:\Scripts\Upload-Companies.ps1
Get-Content C:\Scripts\upload-companies.log
```

The first run against the sample data reports mostly updates, since the companies are already there from the first tutorial.

## 4. Schedule it

```powershell
$action  = New-ScheduledTaskAction -Execute 'pwsh.exe' -Argument '-NoProfile -File C:\Scripts\Upload-Companies.ps1'
$trigger = New-ScheduledTaskTrigger -Daily -At 2am
Register-ScheduledTask -TaskName 'Grein company upload' -Action $action -Trigger $trigger
```

The task runs as your user, so it can read the key you stored in step 2. Choose a time after the export from the source system has finished.

> [!NOTE]
> An upload only creates and updates. A company that disappears from the file keeps its ERP values in Grein. To remove it, [delete](../guide/api.md#delete-a-record) its ERP values through the API.

## Next steps

- [API Quickstart: PowerShell](api-quickstart-powershell.md) shows the JSON endpoints, for when you need more control than a file upload.
- [Fixing Incomplete Records](fixing-incomplete-records.md) shows how to handle rows that arrive without their unique key.

---

[← Back to tutorials](README.md)
