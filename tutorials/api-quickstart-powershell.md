# API Quickstart: PowerShell

A short PowerShell script that uses the [REST API](../guide/api.md) to list your entity types, write two companies and read back what changed. The same script is available for [Python](api-quickstart-python.md), [Node.js](api-quickstart-nodejs.md), [C#](api-quickstart-csharp.md) and [curl](api-quickstart-curl.md).

**You need:**

- PowerShell 7 or later (`pwsh`). Windows PowerShell 5.1 lacks some of the options used here.
- An [API key](../guide/api-keys.md). Creating one needs the **admin** role, and your plan must include API access.
- The Company entity type and the CRM source from [Your First Golden Record](first-golden-record.md), or your own; adjust the names in the script.

## 1. Set your address and key

```powershell
$env:GREIN_URL = "https://<your Grein address>"
$env:GREIN_API_KEY = "mdm_..."
```

## 2. The script

Save this as `grein-quickstart.ps1`:

```powershell
$ErrorActionPreference = 'Stop'
$base = $env:GREIN_URL.TrimEnd('/') + '/api/v1'
$common = @{
    Authentication     = 'Bearer'
    Token              = ConvertTo-SecureString $env:GREIN_API_KEY -AsPlainText -Force
    MaximumRetryCount  = 5    # retry failed requests, such as 429 Too Many Requests
    RetryIntervalSec   = 10
}

# 1. Which entity types and fields can this key see?
$entities = Invoke-RestMethod @common -Uri "$base/entities"
foreach ($et in $entities.data) {
    "{0}: {1}" -f $et.slug, ($et.fields.slug -join ', ')
}

# 2. Create or update two companies in the CRM source, matched on org_number.
$started = [DateTime]::UtcNow
$rows = @(
    @{ org_number = '556100-0001'; company_name = 'Nordlys Analytics AB'; employees = 44 }
    @{ org_number = '556100-0011'; company_name = 'Granskog Data AB'; country = 'SE' }
)
$results = Invoke-RestMethod @common -Method Put -Uri "$base/company?source=crm" `
    -ContentType 'application/json' -Body (ConvertTo-Json -InputObject $rows -Depth 5)
$results | Format-List

# 3. Read the golden records that changed since just before the write.
$since = $started.AddMinutes(-5).ToString('o')
$page = 1
do {
    $res = Invoke-RestMethod @common -Uri "$base/company" -Body @{
        source = 'merged'; since = $since; page = $page; pageSize = 200
    }
    $res.data | Select-Object id, company_name, employees
    $page++
} while ($page -le $res.totalPages)
```

Run it with `pwsh ./grein-quickstart.ps1`.

## 3. What happens

1. **`GET /entities`** lists the entity types and field names the key is cleared to see.
2. **`PUT /company`** with an array is a batch upsert. Nordlys Analytics already exists, so its CRM employee count is updated; Granskog Data is new and is created. Each row gets its own result, so one bad row doesn't stop the others.
3. **`GET /company?source=merged&since=…`** returns golden records changed since the given time, page by page. For a `GET`, `-Body` is sent as query parameters.

`MaximumRetryCount` makes `Invoke-RestMethod` wait and retry when a request fails, for example because the [rate limit](../guide/api.md#rate-limiting) was hit. It also retries errors that won't go away by themselves, such as `400 Bad Request`, so a mistake in a request takes a little longer to surface.

> [!NOTE]
> Merging runs in the background, a few seconds after a write. If step 3 doesn't show the new values yet, run it again a moment later, or call `POST /company/merge` first to merge straight away.

## Next steps

- [Scheduling a Nightly File Upload](scheduling-a-nightly-upload.md) uses PowerShell to load a file every night.
- [Keeping Another System in Sync](keeping-another-system-in-sync.md) turns step 3 into a reliable sync.
- [The REST API](../guide/api.md) documents every endpoint, filter and error code.

---

[← Back to tutorials](README.md)
