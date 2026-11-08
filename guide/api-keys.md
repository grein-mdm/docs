# Configuring API Keys

API keys authenticate requests to the [REST API](api.md) and the read-only [OData feed](odata.md). Each key has its own [sensitivity level](sensitivity.md), which controls what data it can access.

API keys are managed by admins under **Settings**. API access is not included in every plan: if your plan doesn't include it, the **API Access** section says so and its **Configure** button is disabled, and existing keys are rejected with `403 Forbidden`.

## Creating an API key

1. Go to **Settings** in the sidebar and click **Configure** in the **API Access** section.
2. Under **New API Key**, enter a **Label** (e.g. "Data Pipeline" or "CRM Sync") and choose a **Sensitivity level** (Internal by default).
3. Click **Create API Key**.

The new key is shown at the top of the API Access dialog. Click **Copy** to copy it, then **I've copied it — dismiss**. The key is not shown again after you dismiss it.

> **Screenshot:** The API Access dialog showing a newly created key with the Copy and "I've copied it — dismiss" buttons.
> <!-- screenshot: images/api-keys-new-key.png -->

> [!WARNING]
> The API key is only displayed once. Grein cannot show it to you again; if you lose it, [rotate](#rotating-a-key) the key to get a new one.

API keys start with `mdm_`, followed by 32 hexadecimal characters (for example `mdm_0123456789abcdef0123456789abcdef`). The first 12 characters are shown as the key's prefix in the key list, so you can tell your keys apart.

## Using an API key

Include the key as a Bearer token in the `Authorization` header of your HTTP requests:

```http
Authorization: Bearer mdm_0123456789abcdef0123456789abcdef
```

Tools that can't send a Bearer token, such as Power BI and Microsoft Fabric, can connect to the [OData feed](odata.md) with **Basic** authentication instead: enter any user name and use the API key as the password.

See [The REST API](api.md) for full endpoint documentation and [The OData Feed](odata.md) for connecting reporting tools.

## API key sensitivity

Each API key has a sensitivity level (Public, Internal, Sensitive or Secret) that works the same way as a [user's access level](sensitivity.md). The key can only read and write fields within its clearance: other fields are left out of responses and ignored in request bodies. If none of an entity type's fields are visible to the key, the API returns `403 Forbidden` for that entity type, and the OData feed leaves it out.

This lets you create restricted keys for specific integrations — for example, a key with Internal sensitivity for a reporting tool that should not see sensitive or secret fields.

A key's sensitivity level is chosen when the key is created and cannot be changed afterwards. To change it, create a new key and delete the old one.

## Managing API keys

The key list in the API Access dialog shows each key's label, prefix, sensitivity, status (**Active** or **Disabled**) and when it was last used.

> [!NOTE]
> Changes to a key — rotating, disabling or deleting it — can take up to a minute to take effect for requests that used the key very recently. The **Last used** time is likewise accurate to within about a minute.

### Rotating a key

Click **Rotate** and confirm to replace the key with a new one. The old key stops working, and the new key is shown just like when you create a key. The label, sensitivity level and history of the key stay the same. Rotating a disabled key also enables it again.

### Disabling and enabling

Click **Disable** to suspend a key without deleting it. Requests with a disabled key are rejected with `401 Unauthorized`. Click **Enable** to reactivate it.

### Deleting a key

Click **Delete** and confirm to remove a key permanently. This cannot be undone.

Creating, rotating, disabling, enabling and deleting keys are recorded in the [privileged action log](audit.md#privileged-action-log).

## Multiple API keys

You can create multiple API keys with different labels and sensitivity levels. This is useful when different integrations need different levels of data access. For example:

- A "Data Warehouse" key with **Secret** sensitivity for full data replication.
- A "Public Website" key with **Public** sensitivity for displaying non-confidential data.
- A "CRM Sync" key with **Internal** sensitivity for a two-way CRM integration.

## Audit attribution

Each API key has its own user identity for audit purposes. When a request made with the key creates or changes a record, the change is attributed to that identity, which appears in the [record history](viewing-and-editing.md#record-history) as "API — *label*".

---

[← Back to overview](../README.md)
