# Group Policy Objects (GPOs)

## Objective

Implement centralized security policies in the `abc.local` domain using Group Policy. A domain-level GPO sets the password and account lockout rules, and four GPOs linked to department and workstation OUs control user restrictions and computer settings.

## Prerequisites

- OU structure from [organizational-units.md](../02-active-directory-setup/organizational-units.md)
- Employee accounts from [user-creation.md](../03-user-management/user-creation.md), placed in their department OUs
- `CLIENT01` joined to the domain and moved to `OU=Workstations` (see [client-domain-join.md](../02-active-directory-setup/client-domain-join.md))
- Logged in to DC01 as a Domain Admin
- Tool: **Group Policy Management Console (GPMC)**, opened from **Server Manager > Tools > Group Policy Management**

## GPO Overview

| GPO | Linked to | Applies to | Purpose |
|-----|-----------|------------|---------|
| Domain-Account-Policy | `abc.local` (domain root) | All domain accounts | Password length and account lockout |
| IT-Security-Policy | `OU=IT` | IT users | Block Control Panel |
| HR-Workstation-Policy | `OU=HR` | HR users | Restrict Command Prompt and Control Panel, wallpaper, screen saver password |
| Finance-Security-Policy | `OU=Finance` | Finance users | Restrict Command Prompt and Control Panel, screen saver timeout and password |
| Company-Workstation-Policy | `OU=Workstations` | Computers in Workstations | Screen lock, firewall, legal notice |

> The GPO named `Domain-Account-Policy` is the one linked to the domain root. Rename it here if you used a different name (or the Default Domain Policy).

## Key Concepts

**User settings vs. computer settings.**
- **User Configuration** settings apply to the user objects in the OU the GPO is linked to, wherever those users log on.
- **Computer Configuration** settings apply to the computer objects in the OU the GPO is linked to, regardless of who logs in.

That is why the department GPOs use User Configuration settings, and the workstation GPO uses Computer Configuration settings.

**Why the password and lockout settings are not in the IT GPO.**
Domain account policies (password length, lockout) are only enforced for domain accounts when they are set in a GPO linked at the domain root. If they are set in a GPO linked to an OU, they do not apply to the users in that OU. For this reason, the password and lockout requirements in the IT section of the task were configured in a GPO linked to `abc.local`, as the task instructs.

## How to Create and Link a GPO (used for all GPOs)

1. In GPMC, expand **Forest > Domains > abc.local**.
2. Right-click the target OU (or the domain) > **Create a GPO in this domain, and Link it here...**.
3. Enter the GPO name and click **OK**.
4. Right-click the new GPO > **Edit** to open the Group Policy Management Editor.
5. Configure the settings listed below, then close the editor.

---

## GPO 0: Domain-Account-Policy (domain root)

**Linked to:** `abc.local`

Path: **Computer Configuration > Policies > Windows Settings > Security Settings > Account Policies**

| Policy | Location | Setting |
|--------|----------|---------|
| Minimum password length | Password Policy | 12 characters |
| Account lockout threshold | Account Lockout Policy | 5 invalid logon attempts |
| Account lockout duration | Account Lockout Policy | 15 minutes |
| Reset account lockout counter after | Account Lockout Policy | 15 minutes (must be less than or equal to the lockout duration) |

> Windows suggests default values for the lockout duration and reset counter when the threshold is set, so set both to 15 minutes to keep the policy consistent.

---

## GPO 1: IT-Security-Policy

**Linked to:** `OU=IT`

| Policy | Path | Setting |
|--------|------|---------|
| Prohibit access to Control Panel and PC settings | User Configuration > Policies > Administrative Templates > Control Panel | Enabled |

The 12-character password length and 5-attempt, 15-minute lockout requirements are enforced through the domain-level GPO above.

---

## GPO 2: HR-Workstation-Policy

**Linked to:** `OU=HR`

All settings are under **User Configuration > Policies > Administrative Templates**.

| Requirement | Policy | Path | Setting |
|-------------|--------|------|---------|
| Disable Command Prompt | Prevent access to the command prompt | System | Enabled |
| Block Control Panel | Prohibit access to Control Panel and PC settings | Control Panel | Enabled |
| Desktop wallpaper | Desktop Wallpaper | Desktop > Desktop | Enabled, with the wallpaper file path and style (for example, Fill) |
| Screen saver password | Password protect the screen saver | Control Panel > Personalization | Enabled |

Notes:
- The wallpaper path should be a location every user can read, such as a shared folder (UNC path like `\\DC01\<share>\<wallpaper-file>`). A path that exists only on the server's local disk will not work on the client.
- The screen saver password setting only takes effect when a screen saver is active, so also enable **Enable screen saver** and set **Screen saver timeout** in the same Personalization folder.
- For **Prevent access to the command prompt**, the extra option "Disable the command prompt script processing also?" controls whether batch files are blocked too. Choose it based on how strict the policy needs to be.

---

## GPO 3: Finance-Security-Policy

**Linked to:** `OU=Finance`

All settings are under **User Configuration > Policies > Administrative Templates**.

| Requirement | Policy | Path | Setting |
|-------------|--------|------|---------|
| Disable Command Prompt | Prevent access to the command prompt | System | Enabled |
| Block Control Panel | Prohibit access to Control Panel and PC settings | Control Panel | Enabled |
| Screen saver timeout | Screen saver timeout | Control Panel > Personalization | Enabled, **600** seconds |
| Screen saver password | Password protect the screen saver | Control Panel > Personalization | Enabled |

Also enable **Enable screen saver** in the same folder so the timeout and password setting take effect.

---

## GPO 4: Company-Workstation-Policy

**Linked to:** `OU=Workstations`

All settings are under **Computer Configuration > Policies > Windows Settings > Security Settings**.

| Requirement | Policy | Path | Setting |
|-------------|--------|------|---------|
| Automatic screen lock after 10 minutes | Interactive logon: Machine inactivity limit | Local Policies > Security Options | **600** seconds |
| Enable Windows Firewall | Firewall state | Windows Defender Firewall with Advanced Security > right-click the node > **Properties** | **On (recommended)** for the Domain, Private, and Public profiles |
| Legal notice before login | Interactive logon: Message title for users attempting to log on | Local Policies > Security Options | Notice title |
| Legal notice before login | Interactive logon: Message text for users attempting to log on | Local Policies > Security Options | Notice text |

Example legal notice (replace with your own wording):

- **Title:** `ABC Company: Authorized Use Only`
- **Text:** `This computer system is the property of ABC Company and is for authorized use only. Activity may be monitored. Unauthorized access or use is prohibited and may result in disciplinary action or legal action.`

---

## Applying and Verifying the Policies

### 1. Refresh policy on CLIENT01

Group Policy refreshes automatically (every 90 minutes by default, with a random offset), but you can force it for testing. On CLIENT01, open Command Prompt as an administrator and run:

```cmd
gpupdate /force
```

Computer policies are applied at startup, so restart `CLIENT01` if prompted. User policies apply at logon, so log off and log in again.

### 2. Check which GPOs were applied

```cmd
gpresult /r
```

Look under **Computer Settings** for `Company-Workstation-Policy` and under **User Settings** for the user's department GPO. Because Command Prompt is blocked for HR and Finance users, run this in an elevated Command Prompt from an admin account, or use `gpresult /h report.html` to create a report.

### 3. Check the domain password and lockout policy

On DC01, run:

```cmd
net accounts
```

Confirm the minimum password length is 12, the lockout threshold is 5, and the lockout duration is 15 minutes.

### 4. Test each policy

| Test | Expected result |
|------|-----------------|
| Log in to CLIENT01 as an IT user and open Control Panel | Blocked with a restriction message |
| Log in as an HR user and open Command Prompt | Blocked, "The command prompt has been disabled by your administrator" |
| Log in as an HR user and open Control Panel | Blocked |
| Log in as a Finance user and open Command Prompt and Control Panel | Both blocked |
| Check the HR user's desktop | The configured wallpaper is displayed |
| Wait for the screen saver timeout (600 seconds for Finance users) | Screen saver starts and asks for a password |
| Restart CLIENT01 and reach the login screen | The legal notice appears before login |
| Leave CLIENT01 idle for 10 minutes | The workstation locks |
| Open Windows Defender Firewall on CLIENT01 | Firewall is on for the Domain, Private, and Public profiles |
| Enter a wrong password 5 times for a test account | The account is locked out for 15 minutes |
| Set a new password shorter than 12 characters | The change is rejected |

Add the real results from your lab to this table or below it, for example by marking each row as passed.

---

## Troubleshooting Notes

- **A policy is not applying:** run `gpupdate /force`, then `gpresult /r` to check that the GPO is listed. A GPO listed under "denied" or "filtered out" points to a security-filtering or link problem.
- **The user policy has no effect:** check that the **user** account is in the OU the GPO is linked to. A GPO linked to the HR OU only applies to users stored in HR.
- **The computer policy has no effect:** check that `CLIENT01` is in the `Workstations` OU and not in the default `Computers` container, because policies cannot be linked to that container.
- **Password or lockout settings seem to be ignored:** make sure they are in a GPO linked to the **domain root**. They are not enforced for domain users when set in an OU-linked GPO.
- **Policy settings are in the wrong section:** User Configuration settings in a GPO that is linked to a computer OU, or Computer Configuration settings in a department GPO, will not apply to the intended objects.

## Optional: Different Password Policies per Department

Domain-level password policy applies to all users. To give different departments different password rules, use **Fine-Grained Password Policies (FGPP)**. They are created in **Active Directory Administrative Center** (under **System > Password Settings Container**) and applied to users or **security groups** such as `GG_IT`, not to OUs. This works well with the department groups built in [security-groups.md](../03-user-management/security-groups.md).

## Key Takeaways

- Domain account policies must be set in a GPO linked to the domain root.
- User settings follow the user's OU, and computer settings follow the computer's OU.
- Organizing users, computers, and groups in separate OUs makes it possible to target policies precisely.
- `gpupdate /force` and `gpresult` are the main tools for testing and troubleshooting Group Policy.

## Next Steps

Continue with the next lab task, and add this guide to the contents table in the folder README.
