# Security Groups

## Objective

Implement department-based access control by creating security groups in the `Groups` OU, adding each department's employees to a matching Global group, and nesting those department groups inside a Domain Local group that will be used for resource permissions.

## Prerequisites

- OU structure from [organizational-units.md](../02-active-directory-setup/organizational-units.md) is in place
- Employee accounts from [user-creation.md](user-creation.md) exist in their department OUs
- Logged in to DC01 as a Domain Admin
- Tool: **Active Directory Users and Computers** (ADUC)

## Groups to Create

All groups are created in `OU=Groups,OU=ABC-Company,DC=abc,DC=local`.

| Group | Scope | Type | Purpose |
|-------|-------|------|---------|
| GG_IT | Global | Security | Contains IT department users |
| GG_HR | Global | Security | Contains HR department users |
| GG_Finance | Global | Security | Contains Finance department users |
| DL_CompanyFiles | Domain Local | Security | Used to grant access to the company files resource |

### Naming convention

- `GG_` = Global Group (holds people by department)
- `DL_` = Domain Local group (holds permissions to a resource)

## Group Membership

| Group | Members |
|-------|---------|
| GG_IT | jsantos, agarcia |
| GG_HR | mreyes, mvillanueva |
| GG_Finance | cdelacruz, lmendoza |
| DL_CompanyFiles | GG_IT, GG_HR, GG_Finance |

## Steps

### 1. Create the groups

1. In ADUC, expand `abc.local` > `ABC-Company` and select the **Groups** OU.
2. Right-click **Groups** > **New** > **Group**.
3. Enter the group name, then set the scope and type:
   - `GG_IT`: Group scope **Global**, Group type **Security**
   - `GG_HR`: Group scope **Global**, Group type **Security**
   - `GG_Finance`: Group scope **Global**, Group type **Security**
   - `DL_CompanyFiles`: Group scope **Domain local**, Group type **Security**
4. Click **OK** for each group.

### 2. Add users to the department groups

1. Double-click the group (for example, `GG_IT`) > **Members** tab > **Add**.
2. Type the usernames, click **Check Names**, then **OK**.
3. Add the members listed in the table above to each group.

### 3. Add the department groups to DL_CompanyFiles

1. Double-click `DL_CompanyFiles` > **Members** tab > **Add**.
2. Add `GG_IT`, `GG_HR`, and `GG_Finance`.
3. Click **OK**.

### 4. Verify

Check the Groups OU in ADUC: it should list `DL_CompanyFiles`, `GG_Finance`, `GG_HR`, and `GG_IT`, all as **Security Group**.


![Network Topology](img/group-management.png)
To verify from PowerShell:

```powershell
Get-ADGroup -Filter * -SearchBase "OU=Groups,OU=ABC-Company,DC=abc,DC=local" |
  Select-Object Name, GroupScope, GroupCategory
```

Expected scopes: the three `GG_` groups are `Global`, and `DL_CompanyFiles` is `DomainLocal`. All have the category `Security`.

Check each group's members:

```powershell
"GG_IT","GG_HR","GG_Finance","DL_CompanyFiles" | ForEach-Object {
  Write-Host "`n== $_ =="
  Get-ADGroupMember $_ | Select-Object Name, SamAccountName, objectClass
}
```

### Optional: create the groups and memberships with PowerShell

```powershell
$path = "OU=Groups,OU=ABC-Company,DC=abc,DC=local"

"GG_IT","GG_HR","GG_Finance" | ForEach-Object {
  New-ADGroup -Name $_ -GroupScope Global -GroupCategory Security -Path $path
}
New-ADGroup -Name "DL_CompanyFiles" -GroupScope DomainLocal -GroupCategory Security -Path $path

Add-ADGroupMember -Identity GG_IT      -Members jsantos, agarcia
Add-ADGroupMember -Identity GG_HR      -Members mreyes, mvillanueva
Add-ADGroupMember -Identity GG_Finance -Members cdelacruz, lmendoza
Add-ADGroupMember -Identity DL_CompanyFiles -Members GG_IT, GG_HR, GG_Finance
```

## Bonus: Why Global Groups for Departments and Domain Local Groups for Permissions?

### AGDLP

AGDLP is Microsoft's recommended way to nest groups for access control:

| Letter | Meaning | In this lab |
|--------|---------|-------------|
| **A** | Accounts | User accounts such as `jsantos` and `mreyes` |
| **G** | Global groups | `GG_IT`, `GG_HR`, `GG_Finance` (users grouped by department) |
| **DL** | Domain Local groups | `DL_CompanyFiles` (grouped by the resource being accessed) |
| **P** | Permissions | Access rights (for example, Read or Modify) given to the DL group on a shared folder |

The flow is:

```
Accounts  →  Global groups  →  Domain Local group  →  Permissions
jsantos   →  GG_IT          ┐
agarcia   →                 │
mreyes    →  GG_HR          ├→  DL_CompanyFiles  →  Access to company files
mvillanueva →               │
cdelacruz →  GG_Finance     │
lmendoza  →                 ┘
```

### Why this design works

- **Global groups answer "who are these people?"** A Global group holds user accounts from its own domain, so it is a natural fit for department membership. Because it can be used in other domains in the forest, the same department group can be reused wherever that department needs access.
- **Domain Local groups answer "what can they access?"** A Domain Local group is used to assign permissions to resources in its own domain. It can contain accounts and Global groups from any domain, which makes it ideal as the permission holder.
- **Permissions are set once.** Permissions go on `DL_CompanyFiles` only. When someone joins or leaves a department, the admin changes the Global group and no folder permissions need to be touched.
- **It scales and stays clean.** A new department needs only a new Global group added to the Domain Local group, rather than editing permissions on every folder.
- **It is easier to audit.** Looking at a folder's permissions shows resource-named groups instead of a long list of individual users.

### Example

If a new IT hire starts, the admin adds the account to `GG_IT`. Because `GG_IT` is already a member of `DL_CompanyFiles`, the new hire gets access to the company files with no change to the folder's permissions.

> Assigning permissions directly to users, or to department groups for every folder, creates a mess that is hard to maintain. AGDLP avoids this.

## Key Takeaways

- Groups, not individual users, should be used to grant access.
- Global groups organize people, Domain Local groups organize permissions.
- Following AGDLP makes access control easy to scale, manage, and audit.
- OU placement does not give group membership, so users must be added to groups explicitly.

## Next Steps

Create a shared folder, give `DL_CompanyFiles` the right permissions, and test access from CLIENT01 (see `permissions-and-access.md`).
