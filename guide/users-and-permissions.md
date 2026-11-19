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
- Change workspace settings: branding, the external AI model, billing, [audit exports](audit.md), purging history data, and [deleting the workspace](#deleting-a-workspace).

## Inviting a user

1. Go to **Settings** in the sidebar.
2. In the **Users** section, click **Invite**.
3. Fill in:
   - **Email** — the user's work email address. It must match the email of the Microsoft account they sign in with.
   - **Display Name** — optional. Once the user has signed in, the name from their Microsoft account is used.
   - **Role** — Viewer, Editor or Admin. Defaults to Viewer.
   - **Access Level** — the user's [sensitivity level](sensitivity.md). Defaults to Internal.
4. Click **Send invite**.

Grein emails the user an invitation, with your name, the workspace name, their role and a link that signs them in and opens the workspace. The email can take a minute or two to arrive. They gain access the next time they sign in with **Sign in with Microsoft Entra** using that email address, whether through the link or not. Until then, the invitation is listed under **Pending invites** in the Users dialog, where you can **Revoke** it. An invitation that hasn't been used within 30 days expires and disappears from the list; invite the user again if they still need access.

The invitation, and its revocation if you revoke it, are recorded in the [privileged action log](audit.md#privileged-action-log) with the invited email address. If the invitation ends without being used (it is revoked or expires), and the person neither signs in nor is invited to any Grein workspace again within 30 days, that email address is replaced in the log by a pseudonym, **invitee #&lt;number&gt;**. The entries themselves stay. An entry with the action `invitee_anonymised` records that this was done. If the person does sign in, their address is handled like any other user's (see [what happens to a removed user's personal data](#what-happens-to-a-removed-users-personal-data)).

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

### What happens to a removed user's personal data

When someone has been removed from every Grein workspace they belonged to, and hasn't signed in for 30 days, their account is anonymised. If they are added back to a workspace within those 30 days, nothing changes.

Anonymising an account:

- Removes their name, email address and Microsoft sign-in identity from Grein.
- Shows them as **Former user #&lt;number&gt;** in record history, in place of their name.
- Replaces their email address in the privileged action log with the same **Former user #&lt;number&gt;**, and removes the IP addresses of their own actions. The entries themselves stay, so the log still shows what happened and which actions were taken by the same person. An entry with the action `user_anonymised` records that this was done.

If the person signs in again later, they start with a new, empty account.

A person can also ask for their account to be anonymised straight away, without waiting 30 days. Remove them from your workspace first, then contact Grein support.

> [!TIP]
> If you need to keep a record of who a former user was, for example for an investigation, [export the privileged action log](audit.md) before the account is anonymised.

You cannot remove the last remaining user of a workspace.

> [!NOTE]
> A workspace always keeps at least one admin and at least one user. You can't demote or remove the only admin; make another user an admin first.

Invitations, role changes, access level changes and removals are recorded in the [privileged action log](audit.md#privileged-action-log).

## Multiple workspaces

A single user account can belong to multiple workspaces. Each workspace membership has its own role and sensitivity level — being an admin in one workspace does not grant any privileges in another.

## Creating a workspace

When you sign in for the first time and aren't a member of any workspace, Grein asks you to create one. If you already belong to one, you can create another with **+ Create new workspace** in the workspace list. Give it a name, and you become its admin, with the **Secret** access level, so you can see all of its data. You can lower your own access level later under [Settings](#changing-a-users-sensitivity-level). A workspace set up for you by Grein works the same way: its first admin gets the Secret access level too.

If you were given a promo code, enter it in **Promo code (optional)**. The code puts the new workspace on the plan it was made for. A code may be limited to a number of workspaces or have an expiry date; if it's no longer valid, you'll see a message, and you can try again or leave the field empty to start on the Free plan. After 10 wrong codes, you have to wait 15 minutes before you can try another.

A code only applies when a workspace is created. If the workspace later subscribes to a plan and cancels, or is deleted and restored, it moves to the Free plan like any other workspace.

## Deleting a workspace

An admin can delete the workspace under **Settings**, in the **Danger Zone** section, with **Delete workspace…**.

- Everyone loses access immediately, including API keys and the OData feed, and the workspace disappears from everyone's workspace list.
- The subscription, if there is one, is cancelled immediately, and the workspace moves to the Free plan.
- Every admin of the workspace gets an email saying who deleted it and when it will be permanently deleted.
- After **7 days**, the workspace and all its data are permanently deleted. This can't be undone.

If the workspace was deleted by mistake, reply to that email before the 7 days are up, and the workspace will be restored with all its data. It's restored on the Free plan; subscribe again under **Settings** to get your previous plan back.

## Inactive workspaces

A workspace that nobody uses for a long time is deleted. Any of these counts as using it:

- anyone in the workspace, with any role, opening it in Grein
- a request with one of its API keys, to the [REST API](api.md) or the [OData feed](odata.md)

Requests that are refused, for example with a disabled API key, don't count.

When a workspace hasn't been used for that long, every admin of the workspace gets an email saying when it was last used. To keep the workspace, open it (the email has a link) or make a request with one of its API keys within **7 days**. Using it in any of the ways above cancels the warning, and the clock starts over.

If nobody uses it within those 7 days, the workspace is deleted in the same way as [when an admin deletes it](#deleting-a-workspace): everyone loses access at once, the admins get another email, and the workspace and all its data are permanently deleted 7 days after that. To keep it at that point, reply to that email before then.

Workspaces with an active subscription are never deleted for inactivity.

## Workspace maintenance

The platform administrator can put a workspace under maintenance for a short time, for example to restore its data. While it is under maintenance, nobody can open it, including the workspace's own admins. Anyone who tries sees an **Under maintenance** page, sometimes with a note about when the workspace is expected back. In the workspace list it is marked **Under maintenance** and can't be selected. API keys and the OData feed are refused too (see [the REST API](api.md#maintenance)).

If you were working in the workspace when maintenance started, your next click shows the maintenance page. Anything you had already saved is kept. Once maintenance ends, select **Try again** or open the workspace as usual.

---

[← Back to overview](../README.md)
