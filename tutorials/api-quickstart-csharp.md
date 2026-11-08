# API Quickstart: C#

A short C# program that uses the [REST API](../guide/api.md) to list your entity types, write two companies and read back what changed. The same script is available for [Python](api-quickstart-python.md), [Node.js](api-quickstart-nodejs.md), [PowerShell](api-quickstart-powershell.md) and [curl](api-quickstart-curl.md).

**You need:**

- The .NET 8 SDK or later. No extra packages are needed.
- An [API key](../guide/api-keys.md). Creating one needs the **admin** role, and your plan must include API access.
- The Company entity type and the CRM source from [Your First Golden Record](first-golden-record.md), or your own; adjust the names in the program.

## 1. Create the project and set your key

```bash
dotnet new console -n GreinQuickstart
cd GreinQuickstart
export GREIN_URL="https://<your Grein address>"
export GREIN_API_KEY="mdm_..."
```

On Windows, set the variables with `setx` and open a new terminal.

## 2. The program

Replace the contents of `Program.cs` with:

```csharp
using System.Net;
using System.Net.Http.Headers;
using System.Net.Http.Json;
using System.Text.Json.Nodes;

var baseUrl = Environment.GetEnvironmentVariable("GREIN_URL")!.TrimEnd('/') + "/api/v1/";
using var http = new HttpClient { BaseAddress = new Uri(baseUrl) };
http.DefaultRequestHeaders.Authorization =
    new AuthenticationHeaderValue("Bearer", Environment.GetEnvironmentVariable("GREIN_API_KEY"));

// Send a request, waiting and retrying if the rate limit is hit.
async Task<JsonNode> Call(HttpMethod method, string path, object? body = null)
{
    for (var attempt = 0; attempt < 5; attempt++)
    {
        using var req = new HttpRequestMessage(method, path);
        if (body is not null) req.Content = JsonContent.Create(body);
        using var res = await http.SendAsync(req);
        if (res.StatusCode == HttpStatusCode.TooManyRequests)
        {
            await Task.Delay(res.Headers.RetryAfter?.Delta ?? TimeSpan.FromSeconds(10));
            continue;
        }
        res.EnsureSuccessStatusCode();
        return JsonNode.Parse(await res.Content.ReadAsStringAsync())!;
    }
    throw new Exception("Still rate-limited after 5 attempts");
}

// 1. Which entity types and fields can this key see?
var entities = await Call(HttpMethod.Get, "entities");
foreach (var et in entities["data"]!.AsArray())
    Console.WriteLine($"{et!["slug"]}: {string.Join(", ", et["fields"]!.AsArray().Select(f => f!["slug"]))}");

// 2. Create or update two companies in the CRM source, matched on org_number.
var started = DateTimeOffset.UtcNow;
var results = await Call(HttpMethod.Put, "company?source=crm", new object[]
{
    new { org_number = "556100-0001", company_name = "Nordlys Analytics AB", employees = 44 },
    new { org_number = "556100-0011", company_name = "Granskog Data AB", country = "SE" },
});
Console.WriteLine(results.ToJsonString());

// 3. Read the golden records that changed since just before the write.
var since = Uri.EscapeDataString(started.AddMinutes(-5).ToString("o"));
for (var page = 1; ; page++)
{
    var res = await Call(HttpMethod.Get, $"company?source=merged&since={since}&page={page}&pageSize=200");
    foreach (var rec in res["data"]!.AsArray())
        Console.WriteLine($"{rec!["id"]} {rec["company_name"]} {rec["employees"]}");
    if (page >= (int)res["totalPages"]!) break;
}
```

Run it with `dotnet run`.

## 3. What happens

1. **`GET /entities`** lists the entity types and field names the key is cleared to see.
2. **`PUT /company`** with an array is a batch upsert. Nordlys Analytics already exists, so its CRM employee count is updated; Granskog Data is new and is created. Each row gets its own result, so one bad row doesn't stop the others.
3. **`GET /company?source=merged&since=…`** returns golden records changed since the given time, page by page.

The program uses `JsonNode` to keep it short. In a real integration, define a class for your entity type and use `ReadFromJsonAsync<T>()`. Fields without a value can be missing from a record, so make every property nullable.

> [!NOTE]
> Merging runs in the background, a few seconds after a write. If step 3 doesn't show the new values yet, run it again a moment later, or call `POST /company/merge` first to merge straight away.

## Next steps

- [Keeping Another System in Sync](keeping-another-system-in-sync.md) turns step 3 into a reliable sync.
- [The REST API](../guide/api.md) documents every endpoint, filter and error code.

---

[← Back to tutorials](README.md)
