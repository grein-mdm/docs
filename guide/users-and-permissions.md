# Managing Users and Permissions

Access to a workspace is controlled through two independent systems: **roles** determine what actions a user can perform, and **sensitivity levels** (shown as **Access Level** in the user list) determine what data they can see. This page covers roles and user management. For sensitivity levels, see [Working with Sensitivity Labels](sensitivity.md).

All user management happens under **Settings** in the sidebar, which only admins can see.

## Roles

Every user in a workspace has one of three roles. Your role is shown next to the workspace name at the top of the sidebar.

### Viewer

Read-only access. Viewers can:

- Browse records in the data grid and view [record history](viewing-and-editing.md#record-history).
- Export data as CSV.
- Look up entity types and fields in the [Data Dictionary](data-dictionary.md).
- Use [natural language query](ai-query.md), if it is enabled for the workspace.

Viewers cannot create, edit or delete records, and cannot access settings or configuration.

### Editor

Everything a viewer can do, plus:

- Create and edit records in the data grid.
- Delete records.
- [Import data](importing-data.md) by pasting it or uploading a file.
- Review and fix records in [data stewardship](stewardship.md) (**Fix data errors** in the sidebar): consolidate incomplete records and duplicates, accept or reject reference candidates, and correct deferred validation errors.
- Run [AI attribute extraction](ai-extraction.md), if it is enabled for the workspace.

### Admin

Everything an editor can do, plus:

- Create, edit and deactivate entity types and fields (**Schemas** in the sidebar).
- Manage [sources](sources.md) and their priority order.
- Invite, remove and manage other users.
- Create and manage [API keys](api-keys.md).
- Configure [API lookups](validation-and-lookups.md#api-lookups).
- Trigger [merges](merging.md) from the data grid.
- Change workspace settings: branding, the external AI model, billing, [audit exports](audit.md), purging history data, and deactivating the workspace.

## Inviting a user

1. Go to **Settings** in the sidebar.
2. In the **Users** section, click **Invite**.
3. Fill in:
   - **Email** — the user's work email address. It must match the email of the Microsoft account they sign in with.
   - **Display Name** — optional. Once the user has signed in, the name from their Microsoft account is used.
   - **Role** — Viewer, Editor or Admin. Defaults to Viewer.
   - **Access Level** — the user's [sensitivity level](sensitivity.md). Defaults to Internal.
4. Click **Send invite**.

No invitation email is sent. Let the user know they have been invited: they gain access automatically the next time they sign in with **Sign in with Microsoft Entra** using that email address. Until then, the invitation is listed under **Pending invites** in the Users dialog, where you can **Revoke** it.

If the user is already a member of another Grein workspace, the new workspace appears in their workspace list the next time they sign in.

Your plan may limit the number of users in a workspace. When the limit is reached, inviting another user shows an error asking you to upgrade.

> **Screenshot:** The Invite User dialog, with the Email, Display Name, Role and Access Level fields.
> <!-- screenshot: images/users-and-permissions-invite-user.png -->

## Changing a user's role

In the **Users** section of the settings page, click **Configure** to open the user list. Choose a new value in the **Role** dropdown next to the user. The change is saved immediately.

> **Screenshot:** The Users dialog, listing users with their Role and Access Level dropdowns and a Remove button.
> <!-- screenshot: images/users-and-permissions-user-list.png -->

## Changing a user's sensitivity level

Use the **Access Level** dropdown in the same user list. The change is saved immediately. See [Working with Sensitivity Labels](sensitivity.md) for how sensitivity levels control what data a user can see.

## Removing a user

Click **Remove** next to a user in the user list and confirm. This revokes their access to the workspace. Their changes are preserved and remain attributed to them in the record history and [audit trail](audit.md).

You cannot remove the last remaining user of a workspace.

> [!NOTE]
> A workspace always keeps at least one admin and at least one user. You can't demote or remove the only admin; make another user an admin first.

Invitations, role changes, access level changes and removals are recorded in the [privileged action log](audit.md#privileged-action-log).

## Multiple workspaces

A single user account can belong to multiple workspaces. Each workspace membership has its own role and sensitivity level — being an admin in one workspace does not grant any privileges in another.

---

[← Back to overview](../README.md)
